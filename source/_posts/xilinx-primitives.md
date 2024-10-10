---
title: FPGA学习笔记之Xilinx 原语
date: 2024-09-29 20:06:39
img:
- /medias/featureimages/24.jpg
categories: 硬件
tags:
 - FPGA
 - vivado
 - 教程
 - 工具
---

# Xilinx 原语


##  何为原语
- 原语，是FPGA厂商针对其器件特征开发的一系列常用模块的名称。原语是FPGA芯片中基本元件，代表FPGA中实际拥有的硬件逻辑单元，如LUT，D触发器，RAM等。原语在设计中可以直接例化使用，是最直接的代码输入方式，原语和HDL原语的关系，类似于汇编语言和C语言的关系。


### Xilinx 原语的分类

- Xilinx公司的原语按功能分为10类，包括计算组件、I/O端口组件、寄存器、时钟组件、处理器组件、移位寄存器、配置和检测组件、RAM/ROM组件、Slice/CLB组件以及吉比特收发器组件。常用的原语包括**时钟缓冲**、**差分和单端信号相互转换**以及**I/O处理（IDDR、ODDR）原语**等。



##  时钟组件

- 时钟组件包括各种全局时钟缓冲器、全局时钟复用器、普通I/O本地的时钟缓冲器以及高级时钟管理模块。与其相关的原语包括： `BUFG`、 `BUFR`、`BUFH`、 `BUFIO`、 `BUFGCE`、 `BUFGDLL`和`DCM`等。

### BUFG

- 全局缓冲，`BUFG` 的输出到达 FPGA 内部的 IOB、CLB、块 RAM 的时钟延迟和抖动最小。
- `BUFG` 原语模板如下： 
```verilog
BUFG BUFG_inst ( 
    .O(O), // 1-bit output: Clock output 
    .I(I)  // 1-bit input: Clock input 
);
```
- `BUFG`的输入和输出

<div align="center">
<img src=./xilinx-primitives/1.png width=80%/>
</div>


### BUFIO
- `BUFIO` 是 IO 时钟网络，其独立于全局时钟资源，适合采集源同步数据。它只能驱动 IO Block 里面的逻辑，不能驱动 CLB 里面的 LUT，REG 等逻辑。换句话说，就是其输出时钟只能作用在一个时钟区域的IO寄存器处，无法在FPGA内部逻辑使用
- BUFIO 原语模板如下： 
```verilog
BUFIO BUFIO_inst ( 
    .O(O), // 1-bit output: Clock output (connect to I/O clock loads). 
    .I(I)  // 1-bit input: Clock input (connect to an IBUF or BUFMR). 
); 
```
- `BUFIO` 在采集源同步 IO 数据时，提供非常小的延时，因此非常适合采集比如 RGMII 接收侧的数据，但是由于其不能驱动 FPGA 的内部逻辑，因此需要 `BUFIO` 和 `BUFG` 配合使用，以达到最佳性能。如 `ETH_RXC` 的时钟经过 BUFIO，用来采集端口数据；`ETH_RXC` 经过 `BUFG`，用来作为除端口采集外的其他模块的操作时钟。 

- `BUFIO`的输入和输出

<div align="center">
<img src=./xilinx-primitives/2.png width=80%/>
</div>

### 时钟区域视图

<div align="center">
<img src=./xilinx-primitives/3.png width=50%/>
</div>

### 缓冲器使用场景

<div align="center">
<img src=./xilinx-primitives/4.png width=70%/>
</div>




## IO端口组件

- I/O组件提供了**标准单端I/O缓存**(`IBUF/OBUF`)、**DDR专用I/O信号缓存**(`IDDR/ODDR`)、**可变抽头延迟链**(`IDELAY/ODELAY`)、**上拉**(`PULLUP`)、**下拉**(`PULLDOWN`)以及**单端信号和差分信号之间的相互转换**(`IBUFDS/ OBUFDS`) 等。

- `HP BANK` 与 `HR BANK` IO资源的区别：
  
<div align="center">
<img src=./xilinx-primitives/5.png width=70%/>
</div>

- 各系列 `BANK` 分布

<div align="center">
<img src=./xilinx-primitives/6.png width=80%/>
<img src=./xilinx-primitives/7.png width=80%/>
<img src=./xilinx-primitives/8.png width=80%/>
</div>

- IO资源结构
<div align="center">
<img src=./xilinx-primitives/9.png width=70%/>
</div>

### IDDR
- 在 7 系列设备的 `ILOGIC block` 中有专属的 `registers` 来实现 `input double-data-rate(IDDR) registers`，将输入的上下边沿 DDR 信号，转换成两位单边沿 SDR 信号。即为实现输入数据双沿采样。
- `IDDR` 的原语结构图如下图所示：
    - C：输入的同步时钟； 
    - D：输入的 1 位 DDR 数据； 
    - Q1 和 Q2：分别是“C”时钟上升沿和下降沿同步输出的 SDR 数据。 
    - CE：时钟使能信号； 
    - S/R：置位/复位信号，这两个信号不能同时拉高。
<div align="center">
<img src=./xilinx-primitives/10.png width=30%/>
</div>

- `IDDR` 原语模板如下：
```verilog
IDDR #( 
    .DDR_CLK_EDGE("OPPOSITE_EDGE"), // "OPPOSITE_EDGE", "SAME_EDGE"  指定时钟边沿
                                    //    or "SAME_EDGE_PIPELINED"  
    .INIT_Q1(1'b0), // Initial value of Q1: 1'b0 or 1'b1 指定输出Q1和Q2的初始值
    .INIT_Q2(1'b0), // Initial value of Q2: 1'b0 or 1'b1 
    .SRTYPE("SYNC") // Set/Reset type: "SYNC" or "ASYNC" 指定复位/置位信号的有效时机
) IDDR_inst ( 
    .Q1(Q1), // 1-bit output for positive edge of clock 输出数据，在时钟上升沿有效
    .Q2(Q2), // 1-bit output for negative edge of clock 输出数据，在时钟下降沿有效
    .C(C),   // 1-bit clock input 时钟输入
    .CE(CE), // 1-bit clock enable input 时钟使能输入
    .D(D),   // 1-bit DDR data input  数据输入
    .R(R),   // 1-bit reset 复位输入
    .S(S)    // 1-bit set 置位输入
);
```

- `DDR_CLK_EDGE` 参数为 `IDDR` 的三种采集模式，分别为`OPPOSITE_EDGE`、`SAME_EDGE`和`SAME_EDGE_PIPELINED`模式。 
- `OPPOSITE_EDGE` 模式的时序图如下图所示： 
  - `OPPOSITE_EDGE` 模式下，在时钟的上升沿输出的 Q1，时钟的下降沿输出 Q2。
<div align="center">
<img src=./xilinx-primitives/11.png width=70%/>
</div>

  

- `SAME_EDGE` 模式的时序图如下图所示：
    - `SAME_EDGE` 模式下，在时钟的上升沿输出 Q1 和 Q2，但 Q1 和 Q2 不在同一个 cycle 输出。 
<div align="center">
<img src=./xilinx-primitives/12.png width=70%/>
</div>


- `SAME_EDGE_PIPELINED` 模式的时序图如下图所示： 
    - `SAME_EDGE_PIPELINED` 模式下，在时钟的上升沿输出 Q1 和 Q2，Q1 和 Q2 虽然在同一个 cycle 输出，但整体延时了一个时钟周期。在使用 `IDDR` 时，**一般采用此种模式**。 
<div align="center">
<img src=./xilinx-primitives/13.png width=70%/>
</div>





### ODDR
- 通过 `ODDR` 把两路单端的数据合并到一路上输出，上下沿同时输出数据，上升沿输出 a 路，下降沿输出 b 路；如果两路输入信号一路固定为 1，另外一路固定为 0，那么输出的信号实际上是时钟信号。即为实现输出数据双沿采样。
- `0DDR` 的原语结构图如下图所示：
    - C：输入的同步时钟； 
    - Q：输出的 1 位 DDR 数据； 
    - D1 和 D2：分别是“C”时钟上升沿和下降沿同步输入的 SDR 数据。 
    - CE：时钟使能信号； 
    - S/R：置位/复位信号，这两个信号不能同时拉高。
<div align="center">
<img src=./xilinx-primitives/14.png width=30%/>
</div>

- `ODDR` 原语模板如下： 
```verilog
ODDR #( 
    .DDR_CLK_EDGE("OPPOSITE_EDGE"), // "OPPOSITE_EDGE" or "SAME_EDGE"  指定时钟边沿
    .INIT(1'b0),    // Initial   value of Q: 1'b0 or 1'b1 指定输出Q的初始值，可以是1'b0或1'b1
    .SRTYPE("SYNC") // Set/Reset type: "SYNC" or "ASYNC"  指定复位/置位类型
) ODDR_inst ( 
    .Q(Q),   // 1-bit DDR output 输出数据
    .C(C),   // 1-bit clock input 时钟输入
    .CE(CE), // 1-bit clock enable input 时钟使能输入
    .D1(D1), // 1-bit data input (positive edge) 数据输入1，在时钟上升沿有效
    .D2(D2), // 1-bit data input (negative edge)  数据输入2，在时钟下降沿有效
    .R(R),   // 1-bit reset 复位输入
    .S(S)    // 1-bit set 置位输入
);
```

- `DDR_CLK_EDGE` 参数为 `ODDR` 的两种输出模式，分别为`OPPOSITE_EDGE`和`SAME_EDGE`模式。 
- `OPPOSITE_EDGE` 模式的时序图如下图所示： 
  -  此种模式下，在 FPGA 内部需要两个反相时钟来同步 D1 和 D2，此种模式使用较少。 
<div align="center">
<img src=./xilinx-primitives/15.png width=70%/>
</div>

- `SAME_EDGE` 模式的时序图如下图所示： 
  - 此种模式下，数据可以在相同的时钟边沿输出到 Q，一般采用此种模式。 
<div align="center">
<img src=./xilinx-primitives/16.png width=70%/>
</div>


### IBUFDS
- 即**专用差分输入时钟缓冲器**，在实验工程中如果需要将**差分时钟转换成单端时钟**作为全局时钟。`IBUFDS`是一个输入缓冲器，支持低压差分信号（如`LVCMOS`、`LVDS`等）。在`IBUFDS`中，一个电平接口用两个独特的电平接口（`I`和`IB`）表示。一个可以认为是主信号，另外一个可以认为是从信号。主信号和从信号是同一个逻辑信号，但是相位相反。
- `IBUFDS` 原语示意图如下所示：
<div align="center">
<img src=./xilinx-primitives/28.png width=40%/>
</div>

<div align="center">
<img src=./xilinx-primitives/29.png width=70%/>
</div>

- `IBUFDS` 原语模板如下： 
```verilog
IBUFDS #(
      .DIFF_TERM("FALSE"),       // Differential Termination 是否使用差分终端,用于改善信号质量。
      .IBUF_LOW_PWR("TRUE"),     // Low power="TRUE", Highest performance="FALSE" 设置缓冲器的功耗模式
      .IOSTANDARD("DEFAULT")     // Specify the input I/O standard 输入输出的I/O标准
   ) IBUFDS_inst (
      .O(O),  // Buffer output  缓冲器的输出
      .I(I),  // Diff_p buffer input (connect directly to top-level port) 差分输入信号的正极
      .IB(IB) // Diff_n buffer input (connect directly to top-level port) 差分输入信号的负极
   );
```

### OBUFDS
- 即差分输出时钟缓冲器，将**单端信号转换成差分信号**。`OBUFDS`是一个输出缓冲器，支持低压差分信号。`OBUFDS`隔离出了内电路并向芯片上的信号提供驱动电流。它的输出用`O`和`OB`两个独立接口表示。一个可以认为是主信号，另外一个可以认为是从信号。主信号和从信号是同一个逻辑信号，但是相位相反。
- `OBUFDS` 原语示意图如下所示：
<div align="center">
<img src=./xilinx-primitives/30.png width=50%/>
</div>

- `OBUFDS` 原语模板如下： 
```verilog
OBUFDS #(
      .IOSTANDARD("DEFAULT"), // Specify the output I/O standard 指定输出的 I/O 标准
      .SLEW("SLOW")           // Specify the output slew rate 指定输出信号的转换速率
   ) OBUFDS_inst (
      .O(O),     // Diff_p output (connect directly to top-level port) 差分输出信号的正极
      .OB(OB),   // Diff_n output (connect directly to top-level port) 差分输出信号的负极
      .I(I)      // Buffer input 缓冲器的输入
   );
```






### IDELAYE2
- `IDELAY` ：每个I/O模块都包含了一个可编程的延迟原语，称作 `IDELAYE2` 。 `IDELAY2`是一个可编程的31阶延迟原语，它既可以应用于组合逻辑也可以应用于时序逻辑或者同时用于两者。
- 用于在信号通过引脚进入芯片内部之前，进行延时调节，一般高速端口信号由于走线延时等原因，需要通过 `IDELAYE2` 原语对数据做微调。
- 下表为`IDELAY`参数信息：
<div align="center">
<img src=./xilinx-primitives/20.png width=80%/>
</div>


- 下图为 `IDELAYE2` 的例化框图：

<div align="center">
<img src=./xilinx-primitives/17.png width=50%/>
</div>

- 由 IDELAYE2 的模块框图我们可以看一下相关端口的信号信息，如下表所示： 
<div align="center">
<img src=./xilinx-primitives/18.png width=80%/>
</div>

- `FIXED` 模式
  - 在固定延迟模式中，延迟值在配置时预设为属性 `IDELAY_VALUE` 确定的延迟值。 配置后，此值无法更改。
  - `FIXED` 模式时序

<div align="center">
<img src=./xilinx-primitives/19.png width=70%/>
</div>

- `VARIABLE` 模式
    - 在该模式下，延迟值可以在配置后通过CE和INC端口进行动态配置。
    
    <div align="center">
    <img src=./xilinx-primitives/21.png width=50%/>
    </div>
   
    - `VARIABLE` 模式时序
    <div align="center">
    <img src=./xilinx-primitives/22.png width=80%/>
    </div>

- `VAR_LOAD` 模式
    - 该模式下功能与`VARIABLE`模式下类似，只不过可以通过`CNTVALUEIN`加载延迟节拍数，多了一种延迟加载方法。当LD端口有效时可以加载新的延迟`CNTVALUE`值到控制模块。
    
    <div align="center">
    <img src=./xilinx-primitives/23.png width=70%/>
    </div>
   
    - `VAR_LOAD` 模式时序
    <div align="center">
    <img src=./xilinx-primitives/24.png width=70%/>
    </div>
    
- `VAR_LOAD_PIPE` 模式
  - `VAR_LOAD_PIPE` 类似于 `VAR_LOAD` 模式，能够存储 `CNTVALUEIN` 值以备将来更新。
  - `VAR_LOAD_PIPE` 模式时序

<div align="center">
<img src=./xilinx-primitives/25.png width=70%/>
</div>

- `IDELAYE2` 原语模板如下： 
```verilog
IDELAYE2 #( 
    .CINVCTRL_SEL("FALSE"),          // Enable dynamic clock inversion (FALSE, TRUE) 
    .DELAY_SRC("IDATAIN"),           // Delay input (IDATAIN, DATAIN) 
    .HIGH_PERFORMANCE_MODE("FALSE"), // Reduced jitter ("TRUE"), Reduced power ("FALSE") 
    .IDELAY_TYPE("FIXED"),           // FIXED, VARIABLE, VAR_LOAD, VAR_LOAD_PIPE 
    .IDELAY_VALUE(0),                // Input delay tap setting (0-31) 
    .PIPE_SEL("FALSE"),              // Select pipelined mode, FALSE, TRUE 
    .REFCLK_FREQUENCY(200.0),        // IDELAYCTRL clock input frequency in MHz  
    .SIGNAL_PATTERN("DATA")          // DATA, CLOCK input signal 
) 
IDELAYE2_inst ( 
    .CNTVALUEOUT(CNTVALUEOUT),// 5-bit output: Counter value output 
    .DATAOUT(DATAOUT),        // 1-bit output: Delayed data output 
    .C(C),                    // 1-bit input: Clock input 
    .CE(CE),                  // 1-bit input: Active high enable increment/decrement input 
    .CINVCTRL(CINVCTRL),      // 1-bit input: Dynamic clock inversion input 
    .CNTVALUEIN(CNTVALUEIN),  // 5-bit input: Counter value input 
    .DATAIN(DATAIN),          // 1-bit input: Internal delay data input 
    .IDATAIN(IDATAIN),        // 1-bit input: Data input from the I/O 
    .INC(INC),                // 1-bit input: Increment / Decrement tap delay input 
    .LD(LD),                  // 1-bit input: Load IDELAY_VALUE input 
    .LDPIPEEN(LDPIPEEN),      // 1-bit input: Enable PIPELINE register to load data input 
    .REGRST(REGRST)           // 1-bit input: Active-high reset tap-delay input 
);   
```

- `IDATAIN` 为延时前的输入信号，`DATAOUT` 为延时后的输出信号。 
- `REFCLK_FREQUENCY` 参数为 `IDELAYCTRL` 原语的参考时钟频率，一般为 200Mhz； 
- `IDELAY_VALUE` 参数用来设置延时的 `tap` 数，范围为 1~31，每个 `tap` 数的延时时间和参考时钟频率有关。



### IDELAYCTRL 

- `IDELAYCTRL` 模块连续校准其区域内的各个 `IDELAY/ODELAY`，以减少工艺、电压和温度变化的影响。 `IDELAYCTRL` 模块使用用户提供的 `REFCLK` 校准 `IDELAY` 和 `ODELAY` 。
- 也就是说，`IDELAYCTRL` 和 `IDELAYE2` 一般同时使用，`IDELAYCTRL` 对 `IDELAYE2` 延时进行校准。
- `IDELAYCTRL` 原语如下：
```verilog
(* IODELAY_GROUP = <iodelay_group_name> *)  

IDELAYCTRL IDELAYCTRL_inst ( 
    .RDY(RDY),       // 1-bit output: Ready output 
    .REFCLK(REFCLK), // 1-bit input: Reference clock input 
    .RST(RST)        // 1-bit input: Active high reset input 
);  

```

- `IODELAY_GROUP` 为延时 IO 分组，一般数据接口位于多个 `BANK` 时，才需要分组。 
- `IDELAYCTRL` 通过参考时钟 `REFCLK` 来校准 `IDELAY2` 每个 `tap` 的延时值，可用的 `REFCLK` 频率为190Mhz~210Mhz 或者 290Mhz~310Mhz。
- 时钟频率越高对应的 `tap` 延时平均值越小，即延时调节精度越高。当参考时钟为 200Mhz 时，一个 `tap` 为 78ps。 

- `IDELAYCTRL` 时序 
<div align="center">
<img src=./xilinx-primitives/26.png width=60%/>
</div>

- `IDELAYCTRL` 位置分布
  -  `IDELAYCTRL` 模块存在于每个时钟区域的每个 I/O 列中。 
  -  `IDELAYCTRL` 模块校准其时钟区域内的所有 `IDELAYE2` 和 `ODELAYE2` 模块
  
<div align="center">
<img src=./xilinx-primitives/27.png width=60%/>
</div>