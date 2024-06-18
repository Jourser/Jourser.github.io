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
实现：

#### 原理：

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

#### <font color="#dd0000">代码：</font>
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

###  实例代码实现：
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