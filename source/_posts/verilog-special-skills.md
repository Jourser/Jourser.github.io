---
title: FPGA学习笔记之Verilog 特殊技巧
date: 2024-06-14 13:36:39
img:
- /medias/featureimages/4.jpg
categories: 硬件
mathjax: true
tags:
 - FPGA
 - vivado
 - 教程
 - 语法
---


# Verilog 特殊技巧


## 1. 抓取上升沿和下降沿

### 实例 —— 触摸按键控制LED亮灭

#### 实验简介

- **实验目的**：本次实验通过电容式触摸按键控制LED的亮灭
- 电容触摸按键的**工作原理**：
  - 电容触摸按键主要由按键 IC 部分和电容部分构成。按键 IC 部分主要由元器件供应商提供，用于将电容的变化转换为电信号。电容部分指的是由电容极板、地、隔离区等组成触摸按键的电容环境。
  - 任何两个导电的物体之间都存在着感应电容，在周围环境不变的情况下，该感应电容值是固定不变的。如下图所示，手指接触到触摸按键时，按键和手指之间产生寄生电容，使按键的总容值增加。 
    <div align="center">
    <img src=./verilog-special-skills/1.png width=40%/>
    </div>
  - 触摸按键按下前后，电容的变化如下图所示。电容式触摸按键 IC 在检测到按键的感应电容值改变，并超过一定的阈值后，将输出有效信号表示按键被按下。
    <div align="center">
    <img src=./verilog-special-skills/2.png width=70%/>
    </div>
- **实验原理**：通过捕捉按键的上升沿或者下降沿，实现LED的亮灭控制。

### 技巧功能

#### 原理

- 设计两个寄存器，$d_0$ 和 $d_1$
  - 假设 $T$ 为电容触摸按键`touch_key`的电平，则 $d_0$ 和 $d_1$ 的值分别为：
    - $d_0$：保存 $T$ 的值
    - $d_1$：保存 $d_0$ 的值
  - PS：描述时序逻辑，使用**非阻塞赋值**`<=`， $d_0$ 和 $d_1$ 同时赋值
  - 目的：实现两个寄存器的值始终相差一个时钟周期
- 若是抓取**上升沿**，正常情况，`touch_key`的电平始终为0，所以 $d_0$ 和 $d_1$ 的值始终相同为0，当出现一个脉冲时，$d_0$正好在脉冲的上升沿，值为1，而 $d_1$ 却在脉冲前一个时钟，还是0。
- 所以抓取上升沿，可以采取操作判断：
> ~d1&d0 = 1
- 同理，抓取下降沿，操作判断为：
> d1&~d0 = 1

#### <font color="#dd0000">代码</font>
此代码可作为抓取上升或下降沿的参考代码架构：
```verilog
//reg define
reg    d0;    //端口的数据延迟一个时钟周期
reg    d1;    //端口的数据延迟两个时钟周期

//wire define
wire   en;       //有效脉信号

//捕获端口的上升沿，得到一个时钟周期的脉冲信号
assign  en = (~d1) & d0;


//对端口的数据延迟两个时钟周期
always @ (posedge sys_clk or negedge sys_rst_n) begin
    if(!sys_rst_n) begin
        d0 <= 1'b0;
        d1 <= 1'b0;
    end
    else begin
        d0 <= T;
        d1 <= d0;
    end 
end
```

###  实例代码实现
#### 实验代码
```verilog
module touch_led(
    //input
    input        sys_clk,      //时钟信号50Mhz
    input        sys_rst_n,    //复位信号
    input        touch_key,    //触摸按键 
 
    //output
    output  reg  [3:0] led           //LED灯
);


    
//reg define
reg    touch_key_d0;    //触摸按键端口的数据延迟一个时钟周期
reg    touch_key_d1;    //触摸按键端口的数据延迟两个时钟周期

//wire define
wire   touch_en;       //触摸有效脉信号

//捕获触摸按键端口的上升沿，得到一个时钟周期的脉冲信号
assign  touch_en = (~touch_key_d1) & touch_key_d0;


//对触摸按键端口的数据延迟两个时钟周期
always @ (posedge sys_clk or negedge sys_rst_n) begin
    if(!sys_rst_n) begin
        touch_key_d0 <= 1'b0;
        touch_key_d1 <= 1'b0;
    end
    else begin
        touch_key_d0 <= touch_key;
        touch_key_d1 <= touch_key_d0;
    end 
end

//根据触摸按键上升沿的脉冲信号切换led状态
always @ (posedge sys_clk or negedge sys_rst_n) begin
    if (!sys_rst_n)
        led <= 4'b0000;       //默认状态下,LED灯灭
    else begin 
        if (touch_en)      //检测到触摸按键信号
            led <= ~led;
    end
end


endmodule
```
#### Testbench 模块代码：
```verilog
`timescale 1ns / 1ps 
   
module tb_touch_led(); 
   
//reg define 
reg     sys_clk; 
reg     sys_rst_n;      
reg     touch_key; 
   
//wire define 
wire          led ; 
          
always #10 sys_clk = ~sys_clk;   
initial begin 
    sys_clk = 1'b0; 
    sys_rst_n = 1'b0; 
    touch_key = 0; 
    #200 
    sys_rst_n = 1'b1; 
//touch_key 信号变化 
    #40  touch_key = 1'b1 ;  //40ns 后触摸按键按下 
    #200 touch_key = 1'b0 ;  //200ns 触摸按键抬起 
    #40  touch_key = 1'b1 ;  //40ns 后触摸按键按下 
    #200 touch_key = 1'b0 ;  //200ns 触摸按键抬起 
    #40  touch_key = 1'b1 ;  //40ns 后触摸按键按下 
    #200 touch_key = 1'b0 ;  //200ns 触摸按键抬起 
    #40  touch_key = 1'b1 ;  //40ns 后触摸按键按下 
    #200 touch_key = 1'b0 ;  //200ns 触摸按键抬起              
end   
touch_led  u_touch_led( 
    .sys_clk   (sys_clk),  
    .sys_rst_n (sys_rst_n),  
    .touch_key (touch_key), 
    .led       (led) 
);   
endmodule
```
- 仿真得到的波形图如图所示。从图中可以看出，当 `touch_key` 信号由低电平变为高电平时，`touch_key_d0` 和 `touch_key_d1` 信号分别延迟 `touch_key` 一个时钟周期和两个时钟周期，将 `touch_key_d0` 信号的和取反后的 `touch_key_d1` 信号相与，就得到一个时钟周期的脉冲信号（`touch_en`）。当检测到 `touch_en` 信号为高电平时，对 led 信号进行取反，从而实现触摸按键控制 led 灯的功能。 
     <div align="center">
    <img src=./verilog-special-skills/3.png width=90%/>
    </div>



## 2. 除法器实现


### 除法器实现原理

- **为何要设计除法器？**：
  - 在 Verilog HDL 语言中虽然有除的运算指令，但是除运算符中的除数必须是 2 的幂，因此无法实现除数为任意整数的除法，很大程度上限制了它的使用领域。并且多数综合工具对于除运算指令不能综合出令人满意的结果，有些甚至不能给予综合。即使可以综合，也需要比较多的资源。对于这种情况，一般使用相应的算法来实现除法，分为两类，**基于减法操作**和**基于乘法操作**的算法。本节介绍基于减法操作的除法器。

- **基于减法实现的除法器**：
  - **基本流程**：以十进制中的 $a/b$ 为例，可先比较 $a$ 与 $b$ 的大小，如果 $a>b$，则商加 1，$a<=a-b$，再进行比较大小，直到 $a<b$，商不变，余数为 $a$。
  - **缺陷**：这种方式通常速度比较慢，效率低下。
  - **改进**：实际上更快速的方法是模拟手算除法的过程，因此可采用该方法改进基于减法实现的除法器

- **实例讲解**：
    - 以以 7 除以 2 为例
    - 首先，是 4 位除以 4 位，被除数 $a$ 高位补 4 个零，除数低位补 4 个零。
    - 然后比较 $a$ 和 $b$ 值，若 $a<b$，将 $a$ 的值左移一位赋给a，然后继续比较 $a$ 和 $b$ 的大小。如下图的前两次移位。
    - 当第三次移位后 $a>b$，此时用 $a-b+1$，并将得到的值赋给 $a$，然后移位，此时 $a$ 的值大于 $b$，再用 $a-b+1$，并将得到的值赋给 $a$。
    - 。此时已经移位了四次，此时 $a$ 的值，高四位是余数，低四位数是商。 

<div align="center">
<img src=./verilog-special-skills/4.png width=60%/>
</div>

- **状态机表示**，如下图所示： 

<div align="center">
<img src=./verilog-special-skills/5.png width=80%/>
</div>



### 除法器模块设计

- **除法器模块**:该模块是将等精度频率计模块测出的值进行除法操作，该模块的输入信号有**系统时钟信号**`clk`、**复位信号**`rst_n`、**被除数**`dividend`、**除数**`divisor`和**计算使能信号**`en`，输出信号有：**数据有效信号**`ready`、**商**`quotient`、**余数**`remainder`和**商值有效信号**`vld_out`。 

- 该模块的框图如下图所示：






<div align="center">
<img src=./verilog-special-skills/7.png width=50%/>
</div>



- 除法器模块端口与功能描述如下表所示： 



<div align="center">
<img src=./verilog-special-skills/8.png width=90%/>
</div>


### 除法器模块代码


- 除法器模块波形图：
  

<div align="center">
<img src=./verilog-special-skills/9.png width=100%/>
</div>


- 由上方波形图可知代码实现流程如下：
  1. 首先产生一个计算使能信号`en`，在该信号为高时进入计算过程，并通过状态机来实现移位和比较大小。
  2. 在检测到 `en` 信号拉高后由初始状态`IDLE`进入值的比较状态 `SUB`，比较 **a** 和 **b** 的值，并进行运算与移位
  3. 然后进入移位计数器判断状态`SHIFT`，判断移位计数器 `count` 是否计数到位宽数，如果未达到跳到状态`SUB`，直到以为计数器计数到位宽数，跳入状态 `DONE`，将商和余数输出。在计算过程中将数据有效信号`ready`拉低，此时不再从外部进行取值。
  4. 在进入状态`DONE`时，同时将商值有效信号`vld_out`拉高一个周期的高电平。 

- 除法器模块代码：
```verilog
module div_fsm#(
    parameter   DATAWIDTH = 10'd8
)
(
  input                         clk,
  input                         rst_n,
  input                         en,
  input     [DATAWIDTH-1:0]     dividend,//被除数
  input     [DATAWIDTH-1:0]     divisor,//除数
  
  output                        ready,
  output    [DATAWIDTH-1:0]     quotient,//商
  output    [DATAWIDTH-1:0]     remainder,//余数
  output                        vld_out  //商值有效信号 
);
//localparam define
localparam IDLE  = 2'b00;
localparam SUB   = 2'b01;
localparam SHIFT = 2'b10;
localparam DONE  = 2'b11;
//reg define
reg [DATAWIDTH * 2'd2 - 1'b1:0] dividend_e;
reg [DATAWIDTH * 2'd2 - 1'b1:0] divisor_e;
reg [DATAWIDTH - 1'b1:0]        quotient_e;
reg [DATAWIDTH - 1'b1:0]        remainder_e;
reg [1:0]                       current_state;
reg [1:0]                       next_state;
reg [DATAWIDTH-1'b1:0]          count;

//*****************************************************
//**                    main code
//*****************************************************

// 赋值 
assign quotient  = quotient_e;
assign remainder = remainder_e;

// 产生使能信号
assign ready=(current_state==IDLE)? 1'b1:1'b0;
assign vld_out=(current_state==DONE)? 1'b1:1'b0;

// 状态跳转
always@(posedge clk or negedge rst_n)begin
    if(!rst_n)
        current_state <= IDLE;
    else 
        current_state <= next_state;
end

always@(*)begin
    next_state <= 2'bx;
    case(current_state)
        IDLE: begin
            if(en)
                next_state <=  SUB;
            else
                next_state <=  IDLE;
        end
        SUB:  next_state <= SHIFT;
        SHIFT: begin
            if(count<DATAWIDTH) 
                next_state <= SUB;
            else 
                next_state <= DONE;
        end
        DONE: next_state <= IDLE;
        default:next_state <= IDLE;
    endcase
end

always@(posedge clk or negedge rst_n) begin
    if(!rst_n)begin
        dividend_e  <= 'b0;
        divisor_e   <= 'b0;
        quotient_e  <= 'b0;
        remainder_e <= 'b0;
        count       <= 'b0;
        end
    else begin
        case(current_state)
            IDLE:begin
                dividend_e <= {{DATAWIDTH{1'b0}},dividend};//被除数高位补0
                divisor_e  <= {divisor,{DATAWIDTH{1'b0}}};//除数低位补0
                end
            SUB:begin
                if(dividend_e>=divisor_e)begin               //被除数大于除数
                    quotient_e <= {quotient_e[DATAWIDTH-2'd2:0],1'b1};//第6位到第0位复制到新值的第7位到第1位，低位补1
                    dividend_e <= dividend_e-divisor_e;
                    end
                else begin				//被除数小于除数
                    quotient_e <= {quotient_e[DATAWIDTH-2'd2:0],1'b0};//第6位到第0位复制到新值的第7位到第1位，低位补0
                    dividend_e <= dividend_e;
                end
                end
            SHIFT:begin
                if(count<DATAWIDTH)begin
                    dividend_e <= (dividend_e << 1'b1);//被除数向左移位
                    count      <= count + 1'b1;      
                    end
                else begin
                    remainder_e <= dividend_e[DATAWIDTH*2-1:DATAWIDTH];
                end
                end
            DONE:begin
                count <= 1'b0;
                end    
        endcase
    end
end

endmodule
```




## 3. 二进制转BCD码

### 二进制转BCD码实现原理

- **为何要将二进制转BCD码？**：在进行 lcd 或数码管显示数字时，要显示数字的个位、十位、百位等，常用的办法是取余。但这种方法需要浪费大量的资源，且时序不容易优化。所以这里介绍一种通过 BCD 码来方便获取各个位上的数字。

- **BCD码介绍**：
  - **BCD 码**（Binary-Coded Decimal），又称二-十进制码，使用 4 位二进制数来表示 1 位十进制数中的 0~9 这 10 个数码，是一种二进制的数字编码形式，用二进制编码的十进制数码。
  - BCD 码根据权值的有无可分为“**有权码**”和“**无权码**”。其中的权字表示的是权值，有权码的四位二进制数中的每一位都有一个固定的权值，而无权码是没有权值的。 
  - 常见的有权码有 **8421 码**、**5421 码**和**2421 码**，8421 码它的权值从左到右是 8421，5421 码的权值从左到右是 5421，2421 码的权值是 2421，其中 8421 码是最为常用的 BCD 编码，本实验中使用的也是这种编码。常用的无权码有余 3 码、余 3 循环码，还有格雷码。 

<div align="center">
<img src=./verilog-special-skills/6.png width=80%/>
</div>

- BCD码进行十进制显示：
  - 以十进制数据 325为例， 325 的十进制数的二进制表示为：1_0100_0101，325 的 8421 BCD 码为：0011_0010_0101。每 4 位 8421 BCD 码代表一个十进制数，可以进行完美显示。


- **<font color="#dd0000">二进制转BCD码具体流程</font>**:依旧以十进制数据 325为例
  - 首先第一步，在十进制数 325 其对应的二进制数为 1_0100_0101前方补零。补0的数量由其位数决定。该十进制数是 3 位，而一位十进制数的 BCD 码是四位，所以这里我们就需要 **12 位** BCD 码，故我们就在前面补 12 个 0。其余位数的十进制补 0 数量也是这样进行计算。
  - 接下来进行判断运算移位操作。
    - 首先判断每一个 BCD 码其对应的十进制数是否大于 4：
      - 如果大于 4 就对 BCD 码做加 3 操作，
      - 若小于等于 4 就让其值保持不变。
    - 当对每一个 BCD 码进行判断运算后，都需要将运算后的数据像左移 1 位。移完位后我们仍按前面所述进行判断运算，判断运算后需再次移位，以此循环
  - 当我们进行 9 次判断移位后的 BCD 码部分数据就是我们转换的数据。

- **注意**：<u>输入转换的二进制码有多少位我们就需要进行多少次判断移位操作，这里输入的是 9 位二进制数（325 的二进制数：1_0100_0101），我们就进行 9 次判断移位操作。</u>



### 二进制转BCD码模块设计

- **二进制转 BCD 码模块**:该模块是将二进制数转成 BCD 码，方便 lcd 显示模块显示。该模块的输入信号有**系统时钟信号**`clk`、**复位信号**`sys_rst_n`、**要显示的数值**`data`，输出信号为**BCD数据**`bcd_data `.

- 该模块的框图如下图所示：



<div align="center">
<img src=./verilog-special-skills/10.png width=50%/>
</div>




- 二进制转 BCD 码模块端口与功能描述如下表所示： 




<div align="center">
<img src=./verilog-special-skills/11.png width=90%/>
</div>




### 二进制转BCD码模块代码

- 二进制转 BCD 码模块模块波形图：




<div align="center">
<img src=./verilog-special-skills/12.png width=100%/>
</div>




- 如上图所示，当复位键拉高后，移位计数器开始计时，当计时到最大值时移位数据寄存器完成一次寄存，然后取移位数据寄存器的高 36 位作为 BCD 码输出。 

- 由上方波形图可知代码设计如下：
  - 输入转换的二进制码有多少位我们就需要进行多少次判断移位操作，data 数据的位宽为 30 位，所以这里声明**移位判断计数器**`cnt_shift`对移位 30 次进行判断控制
  - 待转换成的 BCD 码位宽为 36 位（9 个数字，一个数字占 4 位，一共 36 位），所以这里声明**移位判断数据寄存器**`data_shift`的位宽为输入的二进制位宽和待转换完成的 BCD 码位宽之和，即 66 位
  - 设计当移位计数器等于 0 时寄存器的**低 30 位**即为待转换数据，而由于还没开始进行转换，**高 36 位**的 BCD 码补 0 即可。
  - 同时声明一个**移位判断操作标志信号**`shift_flag`，用于控制判断和移位的先后顺序，当 `shift_flag` 为低时对数据进行判断，当 `shift_flag` 为高时对数据进行移位。
  - 需要注意的是无论是移位操作和判断操作都是在单个系统时钟下完成的，故判断 30 次和移位 30 次一共在 60 个系统时钟内就能完成。





- 二进制转 BCD 码模块代码：

```verilog
module binary2bcd(
    input   wire             sys_clk,
    input   wire             sys_rst_n,
    input   wire    [29:0]   data,
    
    output  reg     [35:0]   bcd_data       //9位十进制数的值
);
//parameter define
parameter   CNT_SHIFT_NUM = 7'd30;  //由data的位宽决定这里是20
//reg define
reg [6:0]       cnt_shift;         //移位判断计数器该值由data的位宽决定这里是6
reg [65:0]      data_shift;        //移位判断数据寄存器，由data和bcd_data的位宽之和决定。
reg             shift_flag;        //移位判断标志信号

//*****************************************************
//**                    main code                      
//*****************************************************

//cnt_shift计数
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_shift <= 7'd0;
    else if((cnt_shift == CNT_SHIFT_NUM + 1) && (shift_flag))
        cnt_shift <= 7'd0;
    else if(shift_flag)
        cnt_shift <= cnt_shift + 1'b1;
    else
        cnt_shift <= cnt_shift;
end

//data_shift 计数器为0时赋初值，计数器为1~CNT_SHIFT_NUM时进行移位操作
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        data_shift <= 66'd0;
    else if(cnt_shift == 7'd0)
        data_shift <= {36'b0,data};
    else if((cnt_shift <= CNT_SHIFT_NUM)&&(!shift_flag))begin  
        data_shift[33:30] <= (data_shift[33:30] > 4) ? (data_shift[33:30] + 2'd3):(data_shift[33:30]);
        data_shift[37:34] <= (data_shift[37:34] > 4) ? (data_shift[37:34] + 2'd3):(data_shift[37:34]);
        data_shift[41:38] <= (data_shift[41:38] > 4) ? (data_shift[41:38] + 2'd3):(data_shift[41:38]);
        data_shift[45:42] <= (data_shift[45:42] > 4) ? (data_shift[45:42] + 2'd3):(data_shift[45:42]);
        data_shift[49:46] <= (data_shift[49:46] > 4) ? (data_shift[49:46] + 2'd3):(data_shift[49:46]);
        data_shift[53:50] <= (data_shift[53:50] > 4) ? (data_shift[53:50] + 2'd3):(data_shift[53:50]);
        data_shift[57:54] <= (data_shift[57:54] > 4) ? (data_shift[57:54] + 2'd3):(data_shift[57:54]);
        data_shift[61:58] <= (data_shift[61:58] > 4) ? (data_shift[61:58] + 2'd3):(data_shift[61:58]);
        data_shift[65:62] <= (data_shift[65:62] > 4) ? (data_shift[65:62] + 2'd3):(data_shift[65:62]);
        end
    else if((cnt_shift <= CNT_SHIFT_NUM)&&(shift_flag))
        data_shift <= data_shift << 1;
    else
        data_shift <= data_shift;
end

//shift_flag 移位判断标志信号，用于控制移位判断的先后顺序
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        shift_flag <= 1'b0;
    else
        shift_flag <= ~shift_flag;
end

//当计数器等于CNT_SHIFT_NUM时，移位判断操作完成，整体输出
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        bcd_data <= 36'd0;
    else if(cnt_shift == CNT_SHIFT_NUM + 1)
        bcd_data <= data_shift[65:30];
    else
        bcd_data <= bcd_data;
end

endmodule
```

## 4. 乒乓(pingpong)操作

### 理论学习


#### 乒乓操作简介

<div align="center">
<img src=./verilog-special-skills/13.png width=100%/>
</div>

- 上图为“乒乓操作”的结构图，其结构一般是由数据选择器和数据缓冲器构成的，数据缓冲模块可以为任何存储模块，比较常用的存储单元为双口 RAM(DPRAM) 、单口 RAM(SPRAM) 、FIFO等。<font color="#dd0000">乒乓ram结构：这种结构是将输入数据流通过输入数据选择单元等时地将数据流分配到两个数据缓冲区。通过两个数据缓冲区的读和写的切换，来实现数据的流水式传输。</font>
- 具体操作流程：
    1. 在**第一个缓冲周期**，数据流通过“输入数据流选择单元”将数据写入“数据缓冲模块1”。
    2. 进入**第二个缓冲周期**，在第二个缓冲周期数据流通过“输入数据流选择单元”将数据写入到“数据缓冲模块2”的同时“输出数据流选择单元”将“数据缓冲模块1”的数据流读出
    3. 进入**第三个缓冲周期**。在第三个缓冲周期数据流通过“输入数据流选择单元”将数据写入到“数据缓存模块1”的同时将“数据缓冲模块2”的数据读出。
    4. 重复第2、第3步操作，即为乒乓操作，直到数据流结束。
- 乒乓操作原理：就是打乒乓球一样，一个球（数据流），两个拍子（缓存），两个拍子相互击球（轮流读写数据，写1读2，写2读1）这样就可以做到球不停一直移动（数据流不会停，数据无丢失）。 
- **PS：<font color="#dd0000">注意这里的数据流始终是要从输入端移动到输出端</font>**，只是不同时间选取的路径不同，而不要理解成“乒乓操作”是数据在两端来回传输。换句话说，我们这里的“拍子”是两个缓存器，而不是两端的数据选择器。

#### 乒乓操作特点

- 乒乓操作的最大特点是**通过“输入数据流选择单元”和“输出数据选择单元”按节拍、相互配合的切换，将经过缓冲的数据流没有停顿地送到“数据流运算处理模块”进行运算与处理**。把乒乓操作模块当做一个整体，站在这个模块的两端看数据，输入数据流和输出数据流都是连续不断的，没有任何停顿的，因此非常适合对数据流进行流水线 式处理。所以乒乓操作常常应用于流水线式算法，完成数据的**无缝缓冲与处理**。
- 乒乓操作的第二个特点是**可以节约缓存空间**，使用双存储单元比单存储单元更节省存储空间，这是很明显的。同时在某些数据处理时，必须要数据达到一定个数才能进行运算，故还可以达到数据缓存的目的。
- 通过乒乓操作实现**低速模块处理高速数据**的实质是：通过缓存单元实现了数据流的串并转换，并行用 “ 数据缓冲器ram A” 和 “ 数据缓冲器ram B” 处理分流的数据，是**面积与速度互换原则的体现**。

#### 使用案例

- 在低速处理高速数据流时，可以使用乒乓操作。
- 例如：10M的数据流，用乒乓操作，分流成两个FIFO，一个FIFO的吞吐速度只有原来的一半5M，就可以满足低速的处理方法，处理高速的数据，处理后在用合并成一个10M的数据流，数据就被不丢失且高速的处理过了，top层看起就是10M的处理速度了。

### 乒乓操作模块设计

<div align="center">
<img src=./verilog-special-skills/14.png width=100%/>
</div>

- 上图为乒乓操作模块整体框图（提供的代码与该图参数上有差别，但形式一致）
    - **ram为数据缓冲模块**，这里我们使用两个双口RAM来缓存数据。
    - **clk_gen为时钟生成模块**，使用PLL核来生成不同的读写时钟。
    - **data_gen为数据生成模块**，产生输入数据（由于我们是举例为大家讲解乒乓操作，数据就由我们自己产生）。
    - **ram_ctrl为输入输出数据流选择模块**。

### 乒乓操作模块代码

#### ramA,ramB模块设计
- 该实验我们调用RAM IP核作为存储单元来完成乒乓操作，使用的是简单双口RAM，读写位宽设置为8，深度设置为32

#### clk_gen模块设计
- 调用PLL核来生成RAM的读写时钟，设计系统输入时钟是100MHZ，输出时钟有2个分别为clk_5M和clk_10M

#### data_gen模块设计
- 数据生成模块的设计，这里我们可以设计数据从0开始在10MHZ时钟的驱动下，逐渐加1。

```verilog
`timescale 1ns / 1ps

module data_gen(
    input  clk,
    input  rst,
    output data_en,             //使能信号
    output reg [7:0] data_gen   //输出数据

    );

assign data_en = (rst == 1'b1) ? 1'b0 : 1'b1;


always@(posedge clk or posedge rst)begin
    if(rst)
        data_gen <= 8'b0;
    else if (data_gen == 8'd31)
        data_gen <= 8'b0;
    else 
        data_gen <= data_gen + 1'b1;
end

endmodule
```

#### ram_ctrl模块设计
- 该模块我们需要产生控制两个双口RAM的读写相关信号，同时将两个RAM中读出的数据作为输入，通过简单的处理后对读出的数据进行输出。

<div align="center">
<img src=./verilog-special-skills/15.png width=80%/>
</div>

- 该模块要实现的功能，我们通过一个简单的状态机去进行了解
  - **IDLE**：初始状态，在不工作或复位时就让状态机置为初始状态。
  - **WRAM1**：写RAM1状态
  - **WRAM2_RRAM1**：写RAM2读RAM1状态
  - **WRAM1_RRAM2**：写RAM1读RAM2状态

```verilog
`timescale 1ns / 1ps

module ram_ctrl(
    input clk_50M,
    input clk_100M,
    input rst,
    input data_en,
    input [7:0] data_gen,
    input [7:0] ramA_data,
    input [7:0] ramB_data,

    output ramA_ena,
    output ramA_wea,
    output reg [4 : 0] ramA_addra,
    output [7 : 0] ramA_dina,
    output ramA_enb,
    output reg [4 : 0] ramA_addrb,

    output ramB_ena,
    output ramB_wea,
    output reg [4 : 0] ramB_addra,
    output [7 : 0] ramB_dina,
    output ramB_enb,
    output reg [4 : 0] ramB_addrb,

    output reg [7:0] dout
    );

assign ramA_ena = data_en;//ramA写端口使能
assign ramA_enb = data_en;//ramA读端口使能
assign ramB_ena = data_en;//ramB写端口使能
assign ramB_enb = data_en;//ramB读端口使能
assign ramA_wea = (state == WA || state == WA_RB) ? 1'b1 : 1'b0 ;//ramA写使能
assign ramB_wea = (state == RA_WB) ? 1'b1 : 1'b0 ;////ramB写使能


//parameter define  
parameter IDLE  = 4'b0001;//独热码定义方式
parameter WA    = 4'b0010;//写ramA      
parameter RA_WB = 4'b0100;//写ramB，读ramA
parameter WA_RB = 4'b1000;//写ramA，读ramB

reg  [4:0]   state     ;     //当前状态 
reg  [4:0]   next_state     ;     //下一个状态 
 
//状态机的第一段采用同步时序描述状态转移
always@(posedge clk_100M or posedge rst)begin
    if(rst)
        state <= IDLE;
    else 
        state <= next_state;
end

//状态机的第二段采用组合逻辑判断状态转移条件  
always@(*)begin
    case(state)
        IDLE  : next_state = data_en ? WA : IDLE ;
        WA    : next_state = (ramA_addra == 5'd31) ? RA_WB : WA ;
        RA_WB : next_state = (ramB_addra == 5'd31) ? WA_RB : RA_WB ;
        WA_RB : next_state = (ramA_addra == 5'd31) ? RA_WB : WA_RB ;
        default : next_state = IDLE;
    endcase
end

//状态机的第三段描述状态输出（这里采用时序电路输出）
always @(posedge clk_100M or negedge rst) begin
    if(rst) begin        
        ramA_addra <= 4'd0;
        ramB_addra <= 4'd0;
    end
    else if(ramA_addra < 5'd31 && ramB_addra < 5'd31 ) begin
        case (state)
            WA   :ramA_addra <= ramA_addra + 1'b1; 
            RA_WB:ramB_addra <= ramB_addra + 1'b1; 
            WA_RB:ramA_addra <= ramA_addra + 1'b1; 
            default:begin
                ramA_addra <= ramA_addra;
                ramB_addra <= ramB_addra;
            end
        endcase
    end
    else begin
        ramA_addra <= 0;
        ramB_addra <= 0;
    end
end

always @(posedge clk_100M or negedge rst) begin
    if(rst) begin        
        ramA_addrb <= 4'd0;
        ramB_addrb <= 4'd0;
    end
    else if(ramA_addrb < 5'd31 && ramB_addrb < 5'd31) begin
        case (state)
            RA_WB:ramA_addrb <= ramA_addrb + 1'b1; 
            WA_RB:ramB_addrb <= ramB_addrb + 1'b1; 
            default:begin
                ramA_addrb <= ramA_addrb;
                ramB_addrb <= ramB_addrb;
            end
        endcase
    end
    else begin
        ramA_addrb <= 0;
        ramB_addrb <= 0;
    end
end


assign ramA_dina = (state == WA || state == WA_RB) ? data_gen : 8'b0;
assign ramB_dina = (state == RA_WB) ? data_gen : 8'b0;


reg [4:0] state_r;
always@(posedge clk_100M)begin
    state_r <= state;
end

always@(*)begin 
    case(state_r)
        RA_WB : dout = ramA_data;
        WA_RB : dout = ramB_data;
        default : dout = 8'b0;
    endcase
end

endmodule
```

#### 顶层模块设计
```verilog
`timescale 1ns / 1ps

module pingpong(
    input sys_clk,
    input rst,
    output [7:0] dout
    );

wire clk_50M,clk_100M;
wire data_en;
wire [7:0] data_gen;
wire ramA_ena,ramA_enb,ramB_ena,ramB_enb;
wire ramA_wea,ramB_wea;
wire [4:0] ramA_addra,ramB_addra;
wire [4:0] ramA_addrb,ramB_addrb;
wire [7:0] ramA_data,ramB_data;
wire [7:0] ramA_dina,ramB_dina;


clk_div clk_div_u1(
    // Clock out ports
    .clk_50M(clk_50M),     // output clk_50M
    .clk_100M(clk_100M),     // output clk_100M
    // Status and control signals
    .reset(rst), // input reset
   // Clock in ports
    .clk_in1(sys_clk)
);      // input clk_in1

data_gen data_gen_u1(
    .clk                (clk_100M),
    .rst                (rst),
    .data_en            (data_en),
    .data_gen           (data_gen)
);


blk_mem_gen_0 ramA (
  .clka(clk_100M),    // input wire clka
  .ena(ramA_ena),      // input wire ena
  .wea(ramA_wea),      // input wire [0 : 0] wea
  .addra(ramA_addra),  // input wire [4 : 0] addra
  .dina(ramA_dina),    // input wire [7 : 0] dina
  .clkb(clk_100M),    // input wire clkb
  .enb(ramA_enb),      // input wire enb
  .addrb(ramA_addrb),  // input wire [4 : 0] addrb
  .doutb(ramA_data)  // output wire [7 : 0] doutb
);


blk_mem_gen_0 ramB (
  .clka(clk_100M),    // input wire clka
  .ena(ramB_ena),      // input wire ena
  .wea(ramB_wea),      // input wire [0 : 0] wea
  .addra(ramB_addra),  // input wire [4 : 0] addra
  .dina(ramB_dina),    // input wire [7 : 0] dina
  .clkb(clk_100M),    // input wire clkb
  .enb(ramB_enb),      // input wire enb
  .addrb(ramB_addrb),  // input wire [4 : 0] addrb
  .doutb(ramB_data)  // output wire [7 : 0] doutb
);

ram_ctrl ram_ctrl_u1(

    .clk_50M   (clk_50M   ),
    .clk_100M  (clk_100M  ),
    .rst       (rst       ),
    .data_en   (data_en   ),
    .data_gen  (data_gen  ),
    .ramA_data (ramA_data ),
    .ramB_data (ramB_data ),

    .ramA_ena  (ramA_ena  ),
    .ramA_wea  (ramA_wea  ),
    .ramA_addra(ramA_addra),
    .ramA_dina (ramA_dina ),
    .ramA_enb  (ramA_enb  ),
    .ramA_addrb(ramA_addrb),

    .ramB_ena  (ramB_ena  ),
    .ramB_wea  (ramB_wea  ),
    .ramB_addra(ramB_addra),
    .ramB_dina (ramB_dina ),
    .ramB_enb  (ramB_enb  ),
    .ramB_addrb(ramB_addrb),
    .dout      (dout      )


);

endmodule
```

#### 测试文件
```verilog
`timescale 1ns / 1ps

module tb_pingpang();

reg sys_clk;
reg rst;
wire [7:0] dout;
 
always #5 sys_clk = ~sys_clk;

initial begin
    sys_clk <= 0;
    rst     <= 0;
#15 
    rst     <= 1;
#10 
    rst     <= 0;
end
pingpong pingpong_u1(
    .sys_clk    (   sys_clk   ),
    .rst        (   rst       ),
    .dout       (   dout      )
    );

endmodule
```

#### 仿真波形

<div align="center">
<img src=./verilog-special-skills/16.png width=100%/>
</div>