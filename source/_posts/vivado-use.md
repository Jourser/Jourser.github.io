---
title: FPGA学习笔记之vivado使用
date: 2024-05-29 20:06:39
categories: 硬件
tags:
 - FPGA
 - vivado
 - 教程
 - 工具
---

# Vivado的使用

Vivado设计套件，是赛灵思(Xilinx)公司最新的为其产品定制的集成开发环境，支持Block Design、Verilog、VHDL等多种设计输入方式，内嵌综合器以及仿真器,可以完成从设计输入、综合适配、仿真到下载的完整FPGA设计流程。

**注意:本文使用的是Vivado的开发环境2018.3版本**

## Vivado 开发流程
<div align="center">
<img src=./vivado-use/1.png width=80%/>
</div>

## Vivado 新建工程

1. **首次进入Vivado的界面：**
<div align="center">
<img src=./vivado-use/2.png width=100%/>
</div>

2. **创建新的一个项目：**
<div align="center">
<img src=./vivado-use/5.png width=100%/>
</div>

3. **输入项目名称：**
<div align="center">
<img src=./vivado-use/3.png width=100%/>
</div>

4. **选择项目的性质：**
<div align="center">
<img src=./vivado-use/4.png width=100%/>
</div>

5. **本文采用的是正点原子的达芬奇FPGA开发板，选择型号芯片**`xc7a35tfgg484-2`**型号芯片**

<div align="center">
<img src=./vivado-use/6.png width=100%/>
</div>

6. **点击**`Finish`**，完成项目创建。**

<div align="center">
<img src=./vivado-use/7.png width=100%/>
</div>

## Vivado 设计输入
### 项目目录介绍
<div align="center">
<img src=./vivado-use/8.png width=100%/>
</div>

  1. **Design Sources（设计源文件）**：
这个文件夹包含你的设计文件，包括VHDL、Verilog或SystemVerilog的源代码。
这些文件定义了你的硬件设计的逻辑部分，是你设计的主要部分。
  2. **Constraints（约束文件）**：
这个文件夹包含约束文件，通常是XDC（Xilinx Design Constraints）文件。
约束文件用于定义设计的物理和时序约束，如引脚分配、时钟约束、时序要求等。
  3. **Simulation Sources（仿真源文件）**：
这个文件夹包含用于仿真的文件，如测试平台（testbenches）和其他仿真模型。
这些文件用于验证你的设计在仿真环境中的行为，确保设计逻辑按照预期工作。
  4. **Utility Sources（实用工具源文件）**：
这个文件夹通常包含一些辅助文件或脚本，可能包括TCL脚本或其他辅助设计和实现过程的文件。
这些文件有助于自动化某些任务或提供额外的功能和工具支持。

### 输入文件步骤
1. **添加源文件：**

<div align="center">
<img src=./vivado-use/9.png width=100%/>
</div>

2. **创建新的文件：**

<div align="center">
<img src=./vivado-use/10.png width=100%/>
</div>

3. **输入**`led_twinkle`**为创建文件的名称：**

<div align="center">
<img src=./vivado-use/11.png width=100%/>
</div>

4. **创建完文件，点击finish：**

<div align="center">
<img src=./vivado-use/12.png width=100%/>
</div>

5. **输入**`led_twinkle.v`**文件内容：**
在源代码部分输入以下源代码，这是一段led灯交替闪烁的代码：

```verilog
module led_twinkle(
    input   sys_clk,    //系统时钟
    input   sys_rst_n,    //N系统复位，低电平有效

    output [1:0] led    //LEDT
);

////////////////////reg define
    reg [25:0] cnt;
/////////////////
//      main code
////////////////

//对计数器的值进行判断，以输出LED的状态
assign led = (cnt < 26'd2500_0000) ? 2'b01:2'b10;

//计数器在0“5000 000之间进行计数
always @(posedge sys_clk or negedge sys_rst_n) begin
    if(!sys_rst_n)
        cnt <= 26'd0;
//    else if(cnt < (25'd2500_0000 - 25'd1))
    else if(cnt < 26'd5000_0000)  
        cnt <= cnt + 1'b1;
    else
        cnt <= 26'd0;
end


endmodule
```

## Vivado 分析与综合
### 何为代码综合
在Vivado开发流程中，代码综合（Synthesis）是一个关键步骤。综合的主要目的是将高层次的硬件描述语言（HDL）代码转换为门级网表（gate-level netlist），即由逻辑门和触发器等基本单元构成的电路描述。具体来说，代码综合要完成以下任务：

1. **代码分析和优化：**
   - Vivado会首先分析你的VHDL、Verilog或SystemVerilog代码，检查语法和语义，确保代码是正确的。
   - 在分析过程中，Vivado可能会进行一些初步的代码优化，如常量传播（constant propagation）、死代码消除（dead code elimination）等。

2. **逻辑综合：**
   - 将HDL代码中的高级逻辑结构（如if-else语句、case语句、循环等）转换为基本的逻辑门和触发器。
   - Vivado会根据目标FPGA的架构，选择适当的基本单元来实现你的设计。

3. **优化与映射：**
   - 在生成初步的门级网表后，Vivado会对该网表进行优化，以提高性能或减少资源使用。
   - 优化包括时序优化（提高时钟频率）、面积优化（减少逻辑单元的使用）等。
最后，优化后的门级网表会被映射到目标FPGA的具体资源上，如查找表（LUT）、触发器、块RAM等。

4. **生成综合报告：**
   - 综合过程结束后，Vivado会生成一份综合报告，包含关于逻辑单元使用、时序约束满足情况、功耗估计等信息。
   - 这份报告可以帮助设计者评估综合结果，了解设计在目标FPGA上的资源使用情况和性能指标。

### 综合步骤
1. **点击**`Run Synthesis`：
<div align="center">
<img src=./vivado-use/13.png width=100%/>
</div>

2. **开始综合**
<div align="center">
<img src=./vivado-use/15.png width=100%/>
</div>

3. **综合进行中：**
<div align="center">
<img src=./vivado-use/14.png width=100%/>
</div>

4. **综合完成：**
<div align="center">
<img src=./vivado-use/16.png width=100%/>
</div>


## Vivado 约束输入
### 何为约束
在Vivado开发流程中，约束输入（constraints input）是指对设计施加的各种限制条件和要求，以确保设计在硬件上能够正确运行并满足特定性能指标。约束输入通常包括以下几种类型：

1. **时序约束（Timing Constraints）**：
    - 定义设计中各种时序要求，如时钟周期、建立时间（setup time）、保持时间（hold time）等。
    - 常用的时序约束文件格式是XDC（Xilinx Design Constraints），其中包括定义时钟信号（create_clock）、设置输入和输出延迟（set_input_delay、set_output_delay）、多周期路径（multicycle paths）等。
    - 这些约束用于指导综合和布局布线工具进行优化，以确保设计能够在指定的时钟频率下可靠运行。
2. **物理约束（Physical Constraints）**：
    - 指定设计中各种信号的物理位置要求，如引脚分配（pin assignment）、区域约束（area constraints）等。
    - 物理约束可以确保设计中的关键信号和模块放置在适当的位置，以满足性能和信号完整性的要求。
    - 例如，通过约束将某些信号分配到特定的FPGA引脚，可以满足板级布局和连接的需求。
3. **I/O 标准约束（I/O Standard Constraints）**：
    - 定义输入输出信号的电气特性，如电压标准（LVTTL、LVCMOS等）、驱动强度（drive strength）、上拉/下拉配置等。
    - 这些约束确保FPGA的I/O端口与其他外部设备正确兼容，并满足电气规格。
4. **功耗约束（Power Constraints）**：
    - 用于优化设计的功耗，通过指定功耗目标和策略，指导综合和布局布线工具在满足性能要求的同时尽量降低功耗。
    - 功耗约束可以帮助设计者在能耗敏感的应用中优化设计，例如便携设备或电池供电系统。
5. **其他约束（Miscellaneous Constraints）**：
    - 可能还包括一些特定应用的特殊约束，如时钟域交叉（clock domain crossing）、异步信号处理等。
    - 这些约束有助于处理设计中的复杂情况和特殊需求。

### 约束文件步骤(以创建物理约束，即管脚约束为例)

#### 方法一：
1. **点击**`Schematic`
<div align="center">
<img src=./vivado-use/17.png width=100%/>
</div>

2. **等待后，选择右上角的**`I/O Planning`
<div align="center">
<img src=./vivado-use/18.png width=100%/>
</div>

3. **设计管脚约束**
可搜索开发板的引脚图
<div align="center">
<img src=./vivado-use/19.png width=100%/>
</div>

4. **保存后输入文件名，可在Constraints（约束文件）文件夹中，找到相关的XDC文件**
<div align="center">
<img src=./vivado-use/20.png width=100%/>
</div>

#### 方法二：
1. **添加约束文件：**
<div align="center">
<img src=./vivado-use/21.png width=100%/>
</div>

2. **之后的步骤与添加源文件相类似**





## Vivado 生成和下载比特流


1. **点击**`Generate Bitstream`

<div align="center">
<img src=./vivado-use/32.png width=80%/>
</div>

2. **点击**`OK`

<div align="center">
<img src=./vivado-use/31.png width=100%/>
</div>

3. **等待生成，点击**`Project Summary`**，查看生成情况**

<div align="center">
<img src=./vivado-use/33.png width=100%/>
</div>

4. **生成成功**

<div align="center">
<img src=./vivado-use/34.png width=100%/>
</div>

- 该窗口介绍：
   1. **Open Implemented Design**
    选择这个选项会打开实现后的设计布局，包括布线和放置信息。你可以查看设计在FPGA中的具体实现情况。这对于检查布线和放置是否符合预期，以及调试设计问题非常有用。

   2. **View Reports**
    选择这个选项可以查看各种报告，如利用率报告、时序报告、功耗报告等。这些报告提供了关于设计实现的详细信息，可以帮助你检查设计是否满足所有约束和性能要求。

   3. **Open Hardware Manager**
    选择这个选项会打开硬件管理器。硬件管理器允许你将生成的比特流文件下载到FPGA设备中，并进行在线调试和验证。这一步通常在你准备好对实际硬件进行编程和调试时使用。

- 本次实验点击`Cancel`即可





## 连接开发板验证

1. **点击**`Open Hardware Manager`
<div align="center">
<img src=./vivado-use/22.png width=100%/>
</div>

2. **点击自动连接** `Auto Connect`
<div align="center">
<img src=./vivado-use/27.png width=100%/>
</div>

3. **等待连接成功**
<div align="center">
<img src=./vivado-use/28.png width=100%/>
</div>


4. **连接成功后，点击**`Program Device`
<div align="center">
<img src=./vivado-use/29.png width=100%/>
</div>

5. **点击**`Program`
<div align="center">
<img src=./vivado-use/30.png width=100%/>
</div>

6. **等待下载完成，可在板子上看到LED闪烁**
<div align="center">
<img src=./vivado-use/31.jpg width=100%/>
</div>


## 额外内容

### 资源利用率报告
- FPGA 资源利用率报告（Utilization Report）有助于从层级、用户定义的 Pblock 或 SLR 层面来分析含不同资源的设计的使用率。

- 这个报告非常重要，它显示了设计在目标 FPGA 芯片上使用了多少硬件资源，以及剩余多少资源可用。如下图所示：

<div align="center">
<img src=./vivado-use/35.png width=70%/>
</div>

- 由上图所示，对硬件资源的占用情况如下：

| 资源类型 | 利用率 (Utilization) | 分析 |
| :--- | :--- | :--- |
| **LUT** | **82%** | **高利用率。** 逻辑资源非常接近满载。如果设计需要进一步复杂化，您可能需要考虑逻辑优化、使用更大容量的 FPGA，或进行重构。 |
| **DSP** | **78%** | **高利用率。** 专用数字信号处理块接近满载。这表明您的设计包含了大量的乘法、累加或滤波操作。 |
| **BRAM** | **61%** | **中高利用率。** 片上存储资源占用较高，表明您的设计使用了大量缓存或大容量查找表。 |
| **FF** | **53%** | **中等利用率。** 触发器（用于存储和状态机）占用适中，说明设计时序逻辑复杂性适中。 |
| **LUTRAM** | **11%** | **低利用率。** 极少量的查找表被配置为内存使用。 |
| **IO** | **11%** | **低利用率。** 芯片引脚（Input/Output ports）只使用了很小一部分。 |
| **BUFG** | **13%** | **低利用率。** 专用的全局时钟缓冲资源使用很少。 |


#### FPGA 核心资源解释

- FPGA（Field-Programmable Gate Array，现场可编程门阵列）的核心优势在于它由多种可配置的硬件资源组成。以下是报告中提到的主要资源及其功能：

**逻辑资源**

| 资源名称 | 英文全称 | 功能解释 |
| :--- | :--- | :--- |
| **LUT** | Look-Up Table (查找表) | **FPGA最基本的逻辑单元。** 它是一个小型的真值表，可以实现任何输入数量有限的布尔逻辑函数（例如 AND, OR, XOR）。您的所有组合逻辑（如加法器、译码器、状态机跳转条件）最终都映射到 LUT 上。 |
| **FF** | Flip-Flop (触发器) | **FPGA最基本的存储单元。** 用于存储一位数据，实现时序逻辑。所有状态机、寄存器、数据暂存器都由 FF 实现。 |

**存储资源**

| 资源名称 | 英文全称 | 功能解释 |
| :--- | :--- | :--- |
| **BRAM** | Block Random Access Memory (块RAM) | **FPGA内嵌的大容量存储块。** 它们是专用的硬核内存，用于实现 FIFO、大容量缓存、双口 RAM 等。速度快，效率高。 |
| **LUTRAM** | LUT as RAM (用作RAM的查找表) | **将小型 LUT 配置成极小容量的存储器。** 当您需要的存储器很小，不足以用 BRAM 实现，或者需要分布式存储时，会使用 LUTRAM。 |

**专用功能资源**

| 资源名称 | 英文全称 | 功能解释 |
| :--- | :--- | :--- |
| **DSP** | Digital Signal Processing Block | **专用的数字信号处理硬件乘法器。** 这些是硬核电路，针对乘法和累加运算进行了优化（例如 $A \times B + C$）。使用 DSP 块比用 LUT 实现乘法器快得多，且资源占用少得多。 |
| **IO** | Input/Output (输入/输出) | **芯片的物理引脚。** 用于将 FPGA 连接到外部世界（如传感器、总线、LED等）。 |
| **BUFG** | Buffer Global (全局时钟缓冲) | **专用的低抖动时钟分配网络。** 用于将时钟信号从输入引脚分配到整个 FPGA 的所有触发器，确保时序同步。 |

