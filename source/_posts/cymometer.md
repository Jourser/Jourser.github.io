---
title: FPGA学习笔记之频率计
date: 2024-07-01 13:36:39
img:
- /medias/featureimages/11.jpg
mathjax: true
categories: 硬件
tags:
 - FPGA
 - vivado
 - 教程
 - 基础
---


# 频率计




## 前言

- 频率测量在电子设计和测量领域中经常用到，因此对频率测量方法的研究在实际工程应用中具有重要意义。

## 何为频率计

### 周期测量法和频率测量法

- 常用的频率测量方法有两种：**周期测量法**和**频率测量法**。
- **周期测量法**是先测量出被测信号的周期 T，然后根据频率 f=1/T 求出被测信号的频率。 
- **频率测量法**是在时间 t 内对被测信号的脉冲数 N 进行计数，然后求出单位时间内的脉冲数，即为被测信号的频率。 
- **缺陷**：上述两种方法都会产生±1 个基准时钟误差或被测时钟的误差，在实际应用中有一定的局限性。根据测量原理，很容易发现<u>周期测量法适合于低频信号测量</u>，<u>频率测量法适合于高频信号测量</u>，但二者都不能兼顾高低频率同样精度的测量要求。


### 等精度测量法

- **等精度测量法**：是周期测量法和频率测量法二者的结合。
- 等精度测量的一个最大特点是测量的实际门控时间不是一个固定值，而是一个与被测信号有关的值，刚好是被测信号的整数倍。在计数允许时间内，同时对基准时钟和被测信号进行计数，再通过数学公式推导得到被测信号的频率。由于门控信号是被测信号的整数倍，就消除了对被测信号产生的±l 周期误差，但是会产生对基准时钟±1 周期的误差。等精度测量原理如下图所示：


<div align="center">
<img src=./cymometer/1.png width=90%/>
</div>

- 等精度测量法的**特点**：
  -  被测信号频率 `clk_fx` 的相对误差与被测信号的频率无关；
  -  增大测量时间段“软件闸门”或提高“基准频率”`clk_fs`，可以减小相对误差，提高测量精度；
  -  由于一般提供基准时钟 `clk_fs` 的石英晶振稳定性很高，所以基准时钟的相对误差很小，可忽略。

- 基于这种思想，设计中以被测信号的上升沿作为开启门闸和关闭门闸的驱动信号，只有在被测信号的上升沿才将上图中预置的“软件闸门”的状态锁存，因此在“实际闸门” $Tx$ 内被测信号的个数就能保证整数个周期，这样就避免普通测量方法中被测信号的 ±1 的误差，但会产生高频的基准时钟信号的 ±1 周期误差，由于基准时钟频率较高，因此它产生的  ±1 周期误差对测量精度的影响十分有限，特别是在中低频测量的时候，相较于传统的频率测量和周期测量方法，可以大大提高测量精度。 


- 若在一次实际闸门时间 **GATE_TIME** 中，计数器对被测信号的计数值为 $𝑓𝑥_{𝑐𝑛𝑡}$，对基准时钟的计数值为$𝑓𝑠_{𝑐𝑛𝑡}$，而基准时钟的频率值为 $𝑐𝑙𝑘_{𝑓𝑠}$ ，如下图所示。则被测信号的频率为 $𝑐𝑙𝑘_{𝑓𝑥}$，则由公式 :$$GATE_{TIME}=\frac{𝑓𝑠_{𝑐𝑛𝑡}}{𝑐𝑙𝑘_{𝑓𝑠}}=\frac{𝑓𝑥_{𝑐𝑛𝑡}}{𝑐𝑙𝑘_{𝑓𝑥}}$$

<div align="center">
<img src=./cymometer/2.png width=90%/>
</div>

- 误差分析：$$\delta=\frac{𝑐𝑙𝑘_{𝑓𝑥e}-𝑐𝑙𝑘_{𝑓𝑥}}{𝑐𝑙𝑘_{𝑓𝑥e}}×100 \% $$
- 其中$𝑐𝑙𝑘_{𝑓𝑥𝑒}$为被测信号的频率准确值。
- 在测量中，由于 `clk_fx` 计数的启停时间都是由该信号的上升沿触发的，在闸门时间 **GATE_TIME** 内对`clk_fx` 的计数$𝑓𝑥_{𝑐𝑛𝑡}$无误差（$GATE_TIME= 𝑓𝑥_{𝑐𝑛𝑡}/ 𝑐𝑙𝑘_{𝑓𝑥𝑒}$）；对 `clk_fs` 的计数$𝑓𝑠_{𝑐𝑛𝑡}$最多相差一个时钟的误差，即$|∆𝑓𝑠_{𝑐𝑛𝑡}| ≤ 1$，其测量频率如下所示： $$𝑐𝑙𝑘_{𝑓𝑥e}=\frac{𝑓𝑥_{𝑐𝑛𝑡}}{𝑓𝑠_{𝑐𝑛𝑡}+∆𝑓𝑠_{𝑐𝑛𝑡}}×CLK_{FS}$$
- 将上式统一整理后得到：$$\delta=|\frac{∆𝑓𝑠_{𝑐𝑛𝑡}}{𝑓𝑠_{𝑐𝑛𝑡}}|≤\frac{1}{𝑓𝑠_{𝑐𝑛𝑡}}=\frac{1}{GATE_{TIME}×CLK_{FS}}$$
- 由上式可以看出，测量频率的相对误差与被测信号频率的大小无关，仅与**闸门时间**和**基准时钟频率**有关，即实现了整个测试频段的等精度测量。<u>闸门时间越长，基准时钟频率越高，测频的相对误差就越小</u>。基准时钟频率可由稳定度好、精度高的高频晶体振荡器产生，在保证测量精度不变的前提下，提高基准时钟频率，可使闸门时间缩短，即提高测试速度。 



## 频率计设计


### 频率计 设计思路
#### 硬件设计
- 本次实验不同于前面的实验，硬件设计同样需要重视。下面是将大致讲解本次实验流程。
- 本次实验只需将 FPGA 开发板扩展口的三个 IO 分别使用跳帽或者杜邦线连接即可（频率较高时，使用跳帽）。测 5Hz 时将 K16、L16 号引脚相连，测 133.33333MHz 时将 K16、F20 号引脚相连。FPGA 的L16、F20 引脚做为分频产生的时钟的输出端，K16 引脚作为被测时钟的输入端，通过一根导线（杜邦线）或者跳帽进行连接。 

<div align="center">
<img src=./cymometer/3.png width=60%/>
</div>



#### 总体模块设计

- **实验控制流程**：
  1. 需要一个被测试时钟产生模块或者锁相环生成被测的时钟
  2. 用等精度频率计模块测量被测时钟的频率
  3. 将测得的时钟频率值送入 LCD 显示模块进行显示

- 根据实验任务和控制流程，可以划分六个模块：**顶层模块**`top_cymometer`、**等精度频率计模块**`cymometer`、**除法器模块**`div_fsm`、**被测时钟产生模块**`clk_test`、**基准时钟产生模块**`pll_100m`以及 **LCD 显示字符模块**`lcd_rgb_char`
- **LCD 显示字符模块**是复用正点原子开发板的“RGB LCD 字符和图片显示实验”，不是本次实验的重点，因此不再赘述。不过，由于需要显示数字，所以需要新增一个**二进制转 BCD 码模块**`binary2bcd`，该模块运用范围广泛，可以较详细介绍。
- 接下来详细定义一下各个模块的端口功能：
  - **顶层模块**`top_cymometer`是为完成了对其它四个模块的例化、实现各模块之间的数据交互、以及与外部接口的通信。 
  - **被测时钟产生模块**`clk_test`是产生 1hz-50Mhz 的被测的时钟`clk_out1`。该时钟是系统时钟经过分频后得到的。被测时钟是通过 FPGA 引脚输出给等精度频率计模块的。可通过杜邦线选择使用的被测时钟。 
  - **基准时钟产生模块**`pll_100m`是通过锁相环 IP 核产生 100M 的基准时钟（clk_fs），为了节省 PLL 的资源我们同时产生一个 133.33333M 的高频的被测时钟（clk_out2）用于测试。 
  - **等精度频率计模块**`cymometer`是测量输入的被测时钟的频率。并将测得的频率结果输出给 LCD 显示模块。通过 FPGA 引脚输入被测时钟，并将测得的值传递给 LCD 显示模块。 
  - **除法器模块**`div_fsm`是将等精度测量模块获得的的值进行运算，获得商和余数。
  - **二进制转 BCD 码模块**`binary2bcd`是将二进制数转成 BCD 码，方便 lcd 显示模块显示。该模块是在LCD 显示字符模块（lcd_rgb_char）中。 
  - **LCD 显示字符模块**`lcd_rgb_char`是将等精度频率计测得的时钟频率值在 LCD 上显示出来。 

<div align="center">
<img src=./cymometer/4.png width=90%/>
</div>

- 这里生成了两路测试时钟分别是 500kHz 和 133.33333MHz，可通过调整跳线帽的连接位置测量频率。
-  整体模块主要的端口定义与功能描述:

<div align="center">
<img src=./cymometer/5.png width=90%/>
</div>

- 本次实验LCD 显示模块仅是作为显示功能，其的端口定义与功能描述不是本章重点。

#### 顶层模块设计

- **顶层模块设计**：是为完成了对其它模块的例化、实现各模块之间的数据交互、以及与外部接口的通信。
  

- 顶层模块端口定义与功能描述如下表所示：    

<div align="center">
<img src=./cymometer/6.png width=90%/>
</div>

#### 等精度频率计模块

- **等精度频率计模块**：该模块测量输入的被测时钟的频率，并将测得的频率结果输出给 LCD 显示模块。该模块的输入信号有**系统时钟信号**`sys_clk`、**基准时钟信号**`clk_fs`、**复位信号**`rst_n`和**被测时钟信号**`clk_fx`、以及与除法器相关的接口信号**商值**`quotient`、**余数**`remainder`、**数据有效信号**`ready`、**商值有效信号**`vld_out`，输出信号为**被测时钟的频率值**`data_fx`以及与除法器相关的接口**被除数**`dividend`、**除数**`divisor`、**计算使能信号**`en`。

- 该模块的框图如下图所示：

<div align="center">
<img src=./cymometer/7.png width=60%/>
</div>

- 等精度频率计模块端口定义与功能描述，如下表所示： 

<div align="center">
<img src=./cymometer/8.png width=90%/>
</div>


#### 除法器模块

- **除法器模块**:该模块是将等精度频率计模块测出的值进行除法操作，该模块的输入信号有**系统时钟信号**`clk`、**复位信号**`rst_n`、**被除数**`dividend`、**除数**`divisor`和**计算使能信号**`en`，输出信号有：**数据有效信号**`ready`、**商**`quotient`、**余数**`remainder`和**商值有效信号**`vld_out`。 

- 该模块的框图如下图所示：

<div align="center">
<img src=./cymometer/9.png width=50%/>
</div>

- 除法器模块端口与功能描述如下表所示： 

<div align="center">
<img src=./cymometer/10.png width=90%/>
</div>


#### 二进制转 BCD 码模块

- **二进制转 BCD 码模块**:该模块是将二进制数转成 BCD 码，方便 lcd 显示模块显示。该模块的输入信号有**系统时钟信号**`clk`、**复位信号**`sys_rst_n`、**要显示的数值**`data`，输出信号为**BCD数据**`bcd_data `.

- 该模块的框图如下图所示：

<div align="center">
<img src=./cymometer/11.png width=50%/>
</div>

- 二进制转 BCD 码模块端口与功能描述如下表所示： 

<div align="center">
<img src=./cymometer/12.png width=90%/>
</div>


------


**PS**：关于除法器和二进制转 BCD 码的原理，详情请看[《Verilog 特殊技巧——除法器设计》](https://jourser.github.io/2024/06/14/verilog-special-skills/#2-%E9%99%A4%E6%B3%95%E5%99%A8%E5%AE%9E%E7%8E%B0)和[《Verilog 特殊技巧——二进制转 BCD 码》](https://jourser.github.io/2024/06/14/verilog-special-skills/#3-%E4%BA%8C%E8%BF%9B%E5%88%B6%E8%BD%ACBCD%E7%A0%81)

### 频率计 重点程序设计


- 本章对于实验项目步骤不再从创建项目开始，则会重点会讲述关键模块代码


#### 创建顶层模块

- 顶层模块完成了对其它模块的例化。 
  
```verilog

module top_cymometer#(
    //parameter define
    parameter       DIV_N        = 26'd100   ,   // 分频系数
    parameter       CHAR_POS_X   = 11'd1            ,   // 字符区域起始点横坐标
    parameter       CHAR_POS_Y   = 11'd1            ,   // 字符区域起始点纵坐标
    parameter       CHAR_WIDTH   = 11'd88           ,   // 字符区域宽度
    parameter       CHAR_HEIGHT  = 11'd16           ,   // 字符区域高度
    parameter       WHITE        = 24'hFFFFFF       ,   // 背景色，白色
    parameter       BLACK        = 24'h0            ,   // 字符颜色，黑色
    parameter       CNT_GATE_MAX = 28'd75_000_000   ,   // 测频周期时间为1.5s  
    parameter       CNT_GATE_LOW = 28'd12_500_000   ,   // 闸门为低的时间0.25s
    parameter       CNT_TIME_MAX = 28'd80_000_000   ,   // 测频周期时间为1.6s
    parameter       CLK_FS_FREQ  = 28'd100_000_000  ,
    parameter       DATAWIDTH    = 8'd57
)(
    input              sys_clk    ,             // 时钟信号
    input              sys_rst_n  ,             // 复位信号
    input              clk_fx     ,             // 被测时钟

    output             clk_out1   ,             // 输出时钟500khz
    output             clk_out2   ,             // 输出时钟133.3333Mhz
	output             lcd_hs     ,             // LCD 行同步信号
	output             lcd_vs     ,             // LCD 场同步信号
	output             lcd_de     ,             // LCD 数据输入使能
	inout      [23:0]  lcd_rgb    ,             // LCD RGB颜色数据
	output             lcd_bl     ,             // LCD 背光控制信号
	output             lcd_clk    ,             // LCD 采样时钟
    output             lcd_rst
);
//wire define
wire    [29:0]       data_fx;        // 被测信号测量值       
wire                 clk_fs;

wire        en;             

wire [56:0] dividend;       
wire [56:0] divisor;        

wire        ready;          
wire [56:0] quotient;       
wire [56:0] remainder;     
wire        vld_out;        

//*****************************************************
//**                    main code
//*****************************************************

// 产生基准时钟100M
clk_wiz_0 u_pll_100m(
    .clk_out1       (clk_fs     ),      // 基准时钟，100M
    .clk_out2       (clk_out2  ),      // 50M以上被测信号
    .resetn          (sys_rst_n ),      // 复位信号
    .clk_in1        (sys_clk    )
);

//例化等精度频率计模块 
cymometer#(
    .CNT_GATE_MAX   (CNT_GATE_MAX),      // 测频周期时间为1.5s  
    .CNT_GATE_LOW   (CNT_GATE_LOW),      // 闸门为低的时间0.25s
    .CNT_TIME_MAX   (CNT_TIME_MAX),
    .CLK_FS_FREQ    (CLK_FS_FREQ )
)
u_cymometer(
    .sys_clk        (sys_clk    ),       // 系统时钟，50M
    .clk_fs         (clk_fs     ),       // 基准时钟，100M
    .sys_rst_n      (sys_rst_n  ),       // 系统复位，低电平有效
    .clk_fx         (clk_fx     ),       // 被测时钟信号
    .data_fx        (data_fx    ),       // 被测时钟频率值
    .dividend       (dividend   ),       
    .divisor        (divisor    ),       
    .en             (en         ),    
    .ready          (ready      ),       
    .quotient       (quotient   ),       
    .remainder      (remainder  ),       
    .vld_out        (vld_out    )        
    
);

//除法器模块
div_fsm
#(
    .DATAWIDTH      (DATAWIDTH  )
)
u_div_fsm(
    .clk            (sys_clk    ),      
    .rst_n          (sys_rst_n  ),      
    .en             (en         ),      

    .dividend       (dividend   ),      
    .divisor        (divisor    ),      

    .ready          (ready      ),      
    .quotient       (quotient   ),      
    .remainder      (remainder  ),      
    .vld_out        (vld_out    )       
);

//例化测试时钟模块，产生测试时钟
clk_test 
#(
    .DIV_N          (DIV_N      )
)
u_clk_test(
    .clk_in         (sys_clk    ),
    .rst_n          (sys_rst_n  ),
    .clk_out        (clk_out1    )
);

//例化LCD显示模块
lcd_rgb_char 
#(
    .CHAR_POS_X     (CHAR_POS_X ),
    .CHAR_POS_Y     (CHAR_POS_Y ),
    .CHAR_WIDTH     (CHAR_WIDTH ),
    .CHAR_HEIGHT    (CHAR_HEIGHT),
    .WHITE          (WHITE      ),
    .BLACK          (BLACK      )
)
u_lcd_rgb_char(
    .sys_clk        (sys_clk    ),
    .sys_rst_n      (sys_rst_n  ),
    .data           (data_fx    ),     
    .lcd_hs         (lcd_hs     ),          // LCD 行同步信号
    .lcd_vs         (lcd_vs     ),          // LCD 场同步信号
    .lcd_de         (lcd_de     ),          // LCD 数据输入使能
    .lcd_rgb        (lcd_rgb    ),          // LCD RGB888颜色数据
    .lcd_bl         (lcd_bl     ),          // LCD 背光控制信号
    .lcd_clk        (lcd_clk    ),          // LCD 采样时钟
    .lcd_rst        (lcd_rst    )
);

endmodule

```

#### 创建等精度频率计模块（重点）


##### 等精度频率计实现具体流程

- 通过前文对于等精度频率计实现原理的讲解可知，等精度测量需要一个闸门信号，该闸门可由被测时钟直接产生，也可由系统时钟产生软件闸门然后同步到被测时钟下获得。本次实验，我们用系统时钟产生，然后同步到被测时钟下。 

- 在产生软件闸门时，首先要确定软件闸门的宽度也即时间。这里我们用参数表示，方便后期修改。参数`CNT_GATE_MAX` 和 `CNT_GATE_LOW` 分别表示闸门的总时间和低电平时间，此处总时间设为**75_000_000**，低电平时间设置为 **12_500_000**，也即在系统时钟（50M）下，软件闸门总时间为 **1.5s**。其中产生 1s 的高电平信号用作软件闸门信号，前后各 0.25s 用于对数据进行传递和清零。如下图所示：


<div align="center">
<img src=./cymometer/13.png width=80%/>
</div>

- 由等精度测量的原理可知，需要把软件闸门同步到被测时钟下获得实际闸门`gate_fx`。本文在被测时钟下对软件闸门进行打四拍处理，获得稳定的闸门信号（`gate_fx_d0`、`gate_fx_d1`、`gate_fx_d2`、`gate_fx_d3`），同时也可以获得下降沿和上升沿（`gate_fx_nege`、`gate_fx_pose`）。
- 该**边沿标志信号**后面会用于计数值的缓存。
- **打四拍**是为了拓宽上升沿标志信号`gate_fx_pose`的宽度。这是由于最后是在系统时钟下进行赋值计算和清零操作的，为了能够在系统时钟下准确采集到**实际闸门**的上升沿`gate_fx_pose`，这里选择在被测时钟下对实际闸门打多拍处理，拓宽上升沿标志信号的宽度。（当然也可以将实际闸门同步到系统时钟下进行取沿）
- 在基准时钟下对实际闸门信号`gate_fx`，进行打拍处理以获取其在**基准时钟**下的下降沿`gate_fs_nege`。该下降沿是为了方便对数据进行缓存和上面打拍目的的一致。因为该模块的时钟有 50M、100M 以及未知频率的被测时钟，如果不进行跨时钟域的处理，数据会存在不稳定的状态。在获得标志信号后，接下来就是计数操作。 
- 在实际闸门为高时，分别在基准时钟和被测时钟下对脉冲个数进行计数得到 `cnt_fs` 和 `cnt_fx`，并分别在下降沿标志信号 `gate_fs_nege` 和 `gate_fx_nege` 为高时，进行缓存获得稳定值 `cnt_fx_reg` 和 `cnt_fs_reg`。
- 在得到脉冲个数后就可以计算得到被测时钟的频率。**PS**：但由前面的等精度的计算公式可知我们需要进行**大位宽的乘除法**。如果直接进行除法会耗费大量资源，所以综合来看，这里本文采取<u>先进行乘法运算再利用**除法器**进行除法运算</u>。
- 因为前面计算值已经被稳定缓存了，这里在 `cnt_gate_fs` 等于（`CNT_GATE_MAX - CNT_GATE_LOW + TIME`）时进行乘法运算，得到被除数（`numer`）。为了保证被除数值的稳定，我们将被除数（`numer`）在 `cnt_gate_fs == CNT_GATE_MAX - (CNT_GATE_LOW / 2'd2) - TIME)`时进行缓存获得 `numer_reg`。同时将除数 `cnt_fs_reg` 也进行缓存获得 `cnt_fs_reg_reg`。 
- 在获得到稳定的除数与被除数后，就要将值赋给除法器的被除数（`dividend`）和除数信号（`divisor`）获得商值（`quotient`）。为了保证被除数 `numer` 和除数 `cnt_fs_reg_reg` 能稳定的传递给除法器，这里产生一个周期的高电平的计算标志信号 `calc_flag`，该信号在计算使能信号（`en`）拉高后拉高一个时钟周期，在该信号（`calc_flag`）为高时，将除数和被除数输出给除法器。 
- 如何确定除法器使能信号（`en`）：
  - **拉高**：该信号输出给除法器，在除法器使能信号（`en`）为高电平时除法器进入计算过程。在 `cnt_gate_fs == (CNT_GATE_MAX - CNT_GATE_LOW / 2'd2)`时**拉高该信号**
  - **拉低**：在商值有效信号 `vld_out` 为高时，表示本次的计算过程已经结束，此时可以将得到的商（`quotient`）赋值给被测时钟频率值（`data_fx`），同时**拉低**计算使能信号（`en`）信号。
- 完成了频率值的计算，之后传递给 LCD 显示字符模块即可



- 最后是两个所需要注意的**问题**： 
  - 频率计存在上电后初始值不为零
  - 测量结束后或断开后，计数值不归零

- **解决上电初始值不为零**：上电初始值不为零，是由于刚开始没有进行赋值，但除法器还在运行此时显示的值是混乱值。可以设置一个标志信号（`fx_flag`），在（`clk_fx` 和 `gate_fx`）为高时将标志信号 `fx_flag` 拉高，在 `fx_flag` 标志信号为低时，将频率值（`data_fx`）赋为 0。
- **解决测量断开后计数值不清零**：这里采用到前面产生的上升沿标志信号（`gate_fx_pose`），同时定义一个计数器（`cnt_dely`）当检测到该标志信号（`gate_fx_pose`）为高时清零，其他时刻进行加一计数，当计数到 `CNT_TIME_MAX` 时拉高脉冲标志信号（`flag_dely`），当标志信号为高时，将频率值（`data_fx`）赋为 0。计数 2s 是因为我们的软件闸门周期为 1.5s，2s 内没有实际闸门产生，则测量结束后或断开，将频率值归零（测试低频信号时可适当延长，以避免数值闪烁）。
- 上电初始值不为零，是由于刚开始没有进行赋值，但除法器还在运行此时显示的值是混乱值。我们可以设置一个标志信号（`fx_flag`），在（`clk_fx` 和 `gate_fx`）为高时将标志信号 `fx_flag` 拉高，在 `fx_flag` 标志信号为低时，将频率值（`data_fx`）赋为 0。解决上电初始值不为零。 

<div align="center">
<img src=./cymometer/14.png width=100%/>
</div>

##### 等精度频率计编写代码 


- 等精度频率计模块（`cymometer`）是测量输入的被测时钟的频率。 

```verilog
module cymometer#(
    parameter   CNT_GATE_MAX = 28'd75_000_000,      // 测频周期时间为1.5s 
    parameter   CNT_TIME_MAX = 28'd100_000_000,     // 清零周期时间为2s     
    parameter   CNT_GATE_LOW = 28'd12_500_000,      // 闸门为低的时间0.25s
    parameter   CLK_FS_FREQ  = 28'd100_000_000
)(
    input               sys_clk,            // 系统时钟，50M
    input               clk_fs,             // 基准时钟，100M
    input               sys_rst_n,          // 系统复位，低电平有效
    input               clk_fx,             // 被测时钟信号

    output reg [29:0]   data_fx,            // 被测时钟频率值

    input  wire         ready,              // 值可被接受
    input wire [56:0]   quotient,           // 商
    input wire [56:0]   remainder,          // 余数
    input wire          vld_out,            // 值有效

    output reg [56:0]   dividend,           // 被除数
    output reg [56:0]   divisor,            // 除数
    output reg          en                  // 开始信号
 
);
//
localparam TIME = 10'd150;                   // 数据稳定时间
//reg define
reg             gate_sclk;
reg     [27:0]  cnt_gate_fs;
reg             gate_fx;
reg             gate_fx_d0;
reg             gate_fx_d1;
reg             gate_fx_d2;
reg             gate_fx_d3;
reg             gate_fs;
reg             gate_fs_d0;
reg             gate_fs_d1;
reg     [29:0]  cnt_fx;
reg     [29:0]  cnt_fx_reg;
reg     [29:0]  cnt_fs;
reg     [29:0]  cnt_fs_reg;
reg     [29:0]  cnt_fs_reg_reg;
reg             calc_flag;
reg     [56:0]  numer;
reg             fx_flag;
reg     [56:0]  numer_reg;
reg     [27:0]  cnt_dely;
reg             flag_dely;
//wire define
wire            gate_fx_pose;       // 上升沿
wire            gate_fx_nege;       // 下降沿
wire            gate_fs_nege;       // 下降沿

//*****************************************************
//**                    main code
//*****************************************************

// 计算公式 CLK_FX_FREQ = cnt_fx*CLK_FS_FREQ/cnt_fs
assign gate_fx_pose = ((gate_fx) && (!gate_fx_d3))? 1'b1:1'b0;//上升沿
assign gate_fx_nege = ((!gate_fx_d2) && gate_fx_d3)? 1'b1:1'b0;//下降沿
assign gate_fs_nege = ((!gate_fs_d0) && gate_fs_d1)? 1'b1:1'b0;//下降沿

// 产生软件闸门时间计数器 cnt_gate_fs 1.5s
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_gate_fs <= 28'd0;
    else if(cnt_gate_fs == CNT_GATE_MAX - 1'b1 )
        cnt_gate_fs <= 28'd0;
    else
        cnt_gate_fs <= cnt_gate_fs + 1'b1;
end

// 产生软件闸门 GATE_SCLK 
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        gate_sclk <= 1'b0;
    else if(cnt_gate_fs == CNT_GATE_LOW - 1'b1)
        gate_sclk <= 1'b1;
    else if(cnt_gate_fs == CNT_GATE_MAX - CNT_GATE_LOW - 1'b1)
        gate_sclk <= 1'b0;
    else
        gate_sclk <= gate_sclk;
end

// 将软件闸门同步到被测时钟下得到实际闸门,并进行打拍获取上升沿和下降沿
always@(posedge clk_fx or negedge sys_rst_n)begin
    if(!sys_rst_n)begin
        gate_fx <= 1'b0;
        gate_fx_d0 <= 1'b0;
        gate_fx_d1 <= 1'b0;
        gate_fx_d2 <= 1'b0;
        gate_fx_d3 <= 1'b0;
        end
    else begin
        gate_fx <= gate_sclk;
        gate_fx_d0 <= gate_fx;
        gate_fx_d1 <= gate_fx_d0;
        gate_fx_d2 <= gate_fx_d1;
        gate_fx_d3 <= gate_fx_d2;
        end
end

// 获取实际闸门的下降沿 在基准时钟下获取下降沿
always@(posedge clk_fs or negedge sys_rst_n)begin
    if(!sys_rst_n)begin
        gate_fs <= 1'b0;
        gate_fs_d0 <= 1'b0;
        gate_fs_d1 <= 1'b0;
        end
    else begin
        gate_fs <= gate_fx_d2;
        gate_fs_d0 <= gate_fs;
        gate_fs_d1 <= gate_fs_d0;
        end
end

// 在实际闸门下分别计算时钟周期数 cnt_fx(被测时钟) cnt_fs(基准时钟)
// 被测时钟下的周期个数 cnt_fx
always@(posedge clk_fx or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_fx <= 30'd0;
    else if(gate_fx_d2)
        cnt_fx <= cnt_fx + 1'b1;
    else if(gate_fx_nege )
        cnt_fx <= 30'd0;
    else
        cnt_fx <= cnt_fx;  
end

// 在下降沿将被测时钟的时钟周期数进行缓存
always@(posedge clk_fx or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_fx_reg <= 30'd0;
    else if(gate_fx_nege)
        cnt_fx_reg <= cnt_fx;
    else
        cnt_fx_reg <= cnt_fx_reg;
end

// 基准时钟下的周期个数 cnt_fs
always@(posedge clk_fs or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_fs <= 30'd0;
    else if(gate_fx)
        cnt_fs <= cnt_fs + 1'b1;
    else if(gate_fs_nege)
        cnt_fs <= 30'd0;
    else
        cnt_fs <= cnt_fs;  
end

// 在下降沿将基准时钟的时钟周期数进行缓存
always@(posedge clk_fs or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_fs_reg <= 30'd0;
    else if(gate_fs_nege)
        cnt_fs_reg <= cnt_fs;
    else
        cnt_fs_reg <= cnt_fs_reg;
end

// CLK_FX_FREQ = cnt_fx*CLK_FS_FREQ/cnt_fs
// 先计算得到分子 cnt_fx*CLK_FS_FREQ
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        numer <= 57'd0;
    else if(cnt_gate_fs == CNT_GATE_MAX - CNT_GATE_LOW + TIME)
        numer <= cnt_fx_reg * CLK_FS_FREQ;
    else
        numer <= numer;
end

// 打一拍对计算得到的值 numer_reg(分子) 进行同步并寄存
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        numer_reg <=57'd0;
    else if(cnt_gate_fs == (CNT_GATE_MAX - (CNT_GATE_LOW / 2'd2) - TIME))
        numer_reg <= numer;
    else
        numer_reg <= numer_reg;
end

// 打一拍对计算得到的值 cnt_fs_reg_reg(分母) 进行同步并寄存
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_fs_reg_reg <=30'd0;
    else if(cnt_gate_fs == (CNT_GATE_MAX - (CNT_GATE_LOW / 2'd2)- TIME))
        cnt_fs_reg_reg <= cnt_fs_reg;
    else
        cnt_fs_reg_reg <= cnt_fs_reg_reg;
end        
        
// 产生计算标志信号calc_flag
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        calc_flag   <=  1'b0;
    else if(cnt_gate_fs == (CNT_GATE_MAX - CNT_GATE_LOW / 2'd2 - 2'd2))
        calc_flag   <=  1'b1;
    else if(cnt_gate_fs == (CNT_GATE_MAX - CNT_GATE_LOW / 2'd2 - 2'd1))
        calc_flag   <=  1'b0;
    else
        calc_flag <= calc_flag;
end

// 被测时钟启动是否为零(被测信号插入前)
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        fx_flag <= 1'b0;
    else if(clk_fx && gate_fx)
        fx_flag <= 1'b1;
    else 
        fx_flag <= fx_flag;
end

// 1.闸门时间计数(被测信号插入后)
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        cnt_dely <= 28'b0;
    else if(gate_fx_pose)
        cnt_dely <= 28'b0;
    else if(cnt_dely == CNT_TIME_MAX)
        cnt_dely <= CNT_TIME_MAX;
    else 
        cnt_dely <= cnt_dely +1'b1;
end

// 2.上升沿到来后2s拉高
always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        flag_dely <= 1'b0;
    else if(cnt_dely >= CNT_TIME_MAX)               
        flag_dely <= 1'b1;
    else if(cnt_dely < CNT_TIME_MAX)               
        flag_dely <= 1'b0;
    else 
        flag_dely <= flag_dely;
end

// 3.获得被测信号的频率值
always@(posedge sys_clk  or negedge sys_rst_n)begin
    if(!sys_rst_n)
        data_fx <= 30'd0;
    else if(!fx_flag)                   //启动清零
        data_fx <= 30'd0;
    else if(flag_dely)                  //被测时钟被拔掉
        data_fx <= 30'd0;
    else if(vld_out)
        data_fx <= quotient;
    else
        data_fx <= data_fx;
end

always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)
        en <= 1'b0;
    else if(cnt_gate_fs == (CNT_GATE_MAX - CNT_GATE_LOW / 2'd2))
        en <= 1'b1;
    else if(vld_out)
        en <= 1'b0;
    else
        en <= en;        
end

always@(posedge sys_clk or negedge sys_rst_n)begin
    if(!sys_rst_n)begin
        dividend <= 57'd0;
        divisor <= 57'd1;
        end
    else if(calc_flag)begin
        dividend <= numer_reg;
        divisor <= cnt_fs_reg_reg;
        end
    else begin
        dividend <= dividend;
        divisor <= divisor;
        end
end


endmodule
```



#### 创建除法器模块（重点）

- 除法器模块的实现流程和原理将在[《Verilog 特殊技巧————除法器设计》](https://jourser.github.io/2024/06/14/verilog-special-skills/#2-%E9%99%A4%E6%B3%95%E5%99%A8%E5%AE%9E%E7%8E%B0)详细介绍


- **这一部分主要讲解代码**

- 除法器模块波形图：
  
<div align="center">
<img src=./cymometer/15.png width=100%/>
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






#### 创建二进制转 BCD 码模块（重点）

- 二进制转 BCD 码模块模块的实现流程和原理将在[《Verilog 特殊技巧————二进制转 BCD 码》](https://jourser.github.io/2024/06/14/verilog-special-skills/#3-%E4%BA%8C%E8%BF%9B%E5%88%B6%E8%BD%ACBCD%E7%A0%81)详细介绍


- **这一部分主要讲解代码**

- 二进制转 BCD 码模块模块波形图：
  
<div align="center">
<img src=./cymometer/16.png width=100%/>
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





#### 创建基准时钟产生模块

- 基准时钟产生模块通过锁相环产生基准时钟，该模块的输入信号有系统时钟与复位信号，输出端口为
倍频后的基准时钟信号`clk_fs`。以及 133.33333MHz 的高频测试时钟`clk_out2`。







#### 创建被测时钟产生模块


- 被测时钟产生模块是将系统时钟进行分频产生测试时钟，这里用偶数分频方法产生，通过修改 `DIV_N` 分频参数，可得到不同频率的时钟信号，时钟频率为`clk_in/DIV_N` 。由于该模块在顶层例化时 `clk_in` 为系统时钟 50MHz，分频参数为 100，产生的时钟频率为 50_000_000/10_000_000 = 500_000Hz。由此可以绘制出波形图如下图所示： 

<div align="center">
<img src=./cymometer/17.png width=80%/>
</div>

- 被测时钟产生模块代码如下：

```verilog
module clk_test(
     input        clk_in     ,                 // 输入时钟
     input        rst_n      ,                 // 复位信号

     output  reg  clk_out                      // 输出时钟
);
//paramater define
parameter       DIV_N = 26'd100;
//reg define
reg [25:0] cnt;                                 // 时钟分频计数

//*****************************************************
//**                    main code
//*****************************************************

//时钟分频，生成500KHz的测试时钟
always @(posedge clk_in or negedge rst_n) begin
    if(rst_n == 1'b0) begin
        cnt     <= 0;
        clk_out <= 0;
    end
    else begin
        if(cnt == DIV_N/2 - 1'b1) begin
            cnt     <= 26'd0;
            clk_out <= ~clk_out;
        end
        else
            cnt <= cnt + 1'b1;
    end
end

endmodule
```  






#### 创建 LCD 显示字符模块

- 该模块不是本文的重点，该模块下子模块有 **LCD 读 ID 模块**（`rd_id`）、**LCD 时钟分频模块**（`clk_div`）、**二进制转 BCD 码模块**（`binary2bcd`）、**LCD 显示模块**（`lcd_display`）和 **LCD 驱动模块**（`lcd_driver`）。

- 本文只提供 LCD 显示字符顶层模块的代码：

```verilog
module  lcd_rgb_char(
    input              sys_clk      ,
    input              sys_rst_n    ,
	input      [29:0]  data         , 

    output             lcd_hs       ,       // LCD 行同步信号
    output             lcd_vs       ,       // LCD 场同步信号
    output             lcd_de       ,       // LCD 数据输入使能
    inout      [23:0]  lcd_rgb      ,       // LCD RGB565颜色数据
    output             lcd_bl       ,       // LCD 背光控制信号
    output             lcd_clk      ,       // LCD 采样时钟
    output             lcd_rst              // LCD 复位
);
//parameter define
parameter  CHAR_POS_X  = 11'd1      ;       // 字符区域起始点横坐标
parameter  CHAR_POS_Y  = 11'd1      ;       // 字符区域起始点纵坐标
parameter  CHAR_WIDTH  = 11'd88     ;       // 字符区域宽度
parameter  CHAR_HEIGHT = 11'd16     ;       // 字符区域高度
parameter  WHITE       = 24'hFFFFFF ;       // 背景色，白色
parameter  BLACK       = 24'h0      ;       // 字符颜色，黑色

//wire define
wire  [10:0]  pixel_xpos            ;
wire  [10:0]  pixel_ypos            ;
wire  [23:0]  pixel_data            ;
wire  [15:0]  lcd_id                ;
wire  [23:0]  lcd_rgb_o             ;
wire          lcd_pclk              ;
wire  [35:0]  bcd_data;       //9位

//*****************************************************
//**                    main code
//*****************************************************

//RGB565数据输出
assign lcd_rgb = lcd_de ? lcd_rgb_o : {24{1'bz}};

//读rgb lcd ID 模块
rd_id u_rd_id(
    .clk            (sys_clk    ),
    .rst_n          (sys_rst_n  ),
    .lcd_rgb        (lcd_rgb    ), 
    
    .lcd_id         (lcd_id     )
);

//分频模块，根据不同的LCD ID输出相应的频率的驱动时钟
clk_div  u_clk_div(
    .clk            (sys_clk    ),
    .rst_n          (sys_rst_n  ),
    
    .lcd_id         (lcd_id     ),
    .lcd_pclk       (lcd_pclk   )
);

//二进制转BCD码
binary2bcd u_binary2bcd(
    .sys_clk        (sys_clk),
    .sys_rst_n      (sys_rst_n),
    .data           (data    ),
    
    .bcd_data       (bcd_data)
);

//lcd显示模块
lcd_display 
#(
    .CHAR_POS_X     (CHAR_POS_X  ),
    .CHAR_POS_Y     (CHAR_POS_Y  ),
    .CHAR_WIDTH     (CHAR_WIDTH  ),
    .CHAR_HEIGHT    (CHAR_HEIGHT ),
    .WHITE          (WHITE       ),
    .BLACK          (BLACK       )
)
u_lcd_display(
    .lcd_pclk       (lcd_pclk   ),
    .sys_rst_n      (sys_rst_n  ),
    .data           (bcd_data   ),
    
    .pixel_xpos     (pixel_xpos ),
    .pixel_ypos     (pixel_ypos ),
    .pixel_data     (pixel_data )
);

//lcd驱动模块    
lcd_driver u_lcd_driver(
    .lcd_pclk       (lcd_pclk   ),
    .rst_n          (sys_rst_n  ),
    .lcd_id         (lcd_id     ),
    .pixel_data     (pixel_data ),

    .pixel_xpos     (pixel_xpos ),
    .pixel_ypos     (pixel_ypos ),
    .lcd_de         (lcd_de     ),
    .lcd_hs         (lcd_hs     ),
    .lcd_vs         (lcd_vs     ),
    .lcd_bl         (lcd_bl     ),
    .lcd_clk        (lcd_clk    ),
    .lcd_rst        (lcd_rst    ),
    .lcd_rgb        (lcd_rgb_o  )
);

endmodule

```


#### 引脚约束


- 本实验中，各端口信号的管脚分配如下表所示：

<div align="center">
<img src=./cymometer/18.png width=80%/>
</div>


- 对应的 XDC 约束语句如下所示： 

```verilog

create_clock -period 20.000 -name sys_clk -waveform {0.000 10.000} [get_ports sys_clk]

set_property -dict {PACKAGE_PIN R4 IOSTANDARD LVCMOS33} [get_ports sys_clk]
set_property -dict {PACKAGE_PIN U2 IOSTANDARD LVCMOS33} [get_ports sys_rst_n]

set_property -dict {PACKAGE_PIN R16 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[0]}]
set_property -dict {PACKAGE_PIN P15 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[1]}]
set_property -dict {PACKAGE_PIN R14 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[2]}]
set_property -dict {PACKAGE_PIN P14 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[3]}]
set_property -dict {PACKAGE_PIN N14 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[4]}]
set_property -dict {PACKAGE_PIN N13 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[5]}]
set_property -dict {PACKAGE_PIN V9 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[6]}]
set_property -dict {PACKAGE_PIN W9 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[7]}]
set_property -dict {PACKAGE_PIN U18 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[8]}]
set_property -dict {PACKAGE_PIN U17 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[9]}]
set_property -dict {PACKAGE_PIN V19 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[10]}]
set_property -dict {PACKAGE_PIN T18 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[11]}]
set_property -dict {PACKAGE_PIN V20 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[12]}]
set_property -dict {PACKAGE_PIN R18 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[13]}]
set_property -dict {PACKAGE_PIN N17 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[14]}]
set_property -dict {PACKAGE_PIN P17 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[15]}]
set_property -dict {PACKAGE_PIN AB18 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[16]}]
set_property -dict {PACKAGE_PIN AA18 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[17]}]
set_property -dict {PACKAGE_PIN Y19 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[18]}]
set_property -dict {PACKAGE_PIN Y18 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[19]}]
set_property -dict {PACKAGE_PIN W20 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[20]}]
set_property -dict {PACKAGE_PIN W17 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[21]}]
set_property -dict {PACKAGE_PIN V18 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[22]}]
set_property -dict {PACKAGE_PIN V17 IOSTANDARD LVCMOS33} [get_ports {lcd_rgb[23]}]

set_property -dict {PACKAGE_PIN V8 IOSTANDARD LVCMOS33} [get_ports lcd_hs]
set_property -dict {PACKAGE_PIN U7 IOSTANDARD LVCMOS33} [get_ports lcd_vs]
set_property -dict {PACKAGE_PIN AB7 IOSTANDARD LVCMOS33} [get_ports lcd_de]
set_property -dict {PACKAGE_PIN V7 IOSTANDARD LVCMOS33} [get_ports lcd_bl]
set_property -dict {PACKAGE_PIN Y9 IOSTANDARD LVCMOS33} [get_ports lcd_clk]
set_property -dict {PACKAGE_PIN W7 IOSTANDARD LVCMOS33} [get_ports lcd_rst]


# clk_fx 未使用时钟专用引脚
set_property CLOCK_DEDICATED_ROUTE FALSE [get_nets clk_fx_IBUF]
set_property -dict {PACKAGE_PIN K16 IOSTANDARD LVCMOS33} [get_ports clk_fx]
set_property -dict {PACKAGE_PIN L16 IOSTANDARD LVCMOS33} [get_ports clk_out1]
set_property -dict {PACKAGE_PIN F20 IOSTANDARD LVCMOS33} [get_ports clk_out2]
```



### 上板验证

- 编译工程并生成比特流.bit 文件。将扩展口的 K16、L16 引脚通过跳帽短接到一起，并连接好 LCD 排线的接口。将下载器一端连电脑，另一端与开发板上的 JTAG 端口连接，连接电源线并打开电源开关。
- 下载完成后 LCD 上面显示“000500000Hz”，如下图所示，与期望的时钟频率一致。这里需要说明，lcd 屏幕底色显示为白色，字体为黑色。经拍照后颜色可能偏蓝。实际屏幕显示为白色，字体为黑色。

<div align="center">
<img src=./cymometer/20.png width=80%/>
</div>

- 将扩展口的 K16、F20 引脚通过跳帽短接到一起，LCD 上面显示“133333333Hz”，如下图所示，等精度频率计实验下载验证成功。 

<div align="center">
<img src=./cymometer/19.png width=80%/>
</div>