---
title: FPGA学习笔记之VIO的使用
date: 2025-04-25 14:06:39
img:
- /medias/featureimages/26.jpg
categories: 硬件
tags:
 - FPGA
 - vivado
 - 教程
 - 工具
---


# VIO的使用

## 何为VIO

VIO（Virtual Input/Output）核是 Vivado 提供的一种硬件调试 IP，用于在 FPGA 上实时监视和驱动内部信号。它通过 JTAG 通道与 Vivado 硬件管理器（Hardware Manager）交互，允许用户在设计运行时查看信号状态并注入控制数据，而无需额外的物理引脚或外部测试设备。VIO 核的输入/输出端口宽度和数量均可自定义，典型用于低速控制信号（如复位、状态指示灯）和交互式调试场景。


### **VIO 核的主要作用**
1. **实时信号监控与修改**  
   - **输入（Virtual Input）**：通过 JTAG 接口（如 USB 下载器）向 FPGA 内部信号动态注入值（例如控制寄存器、使能信号等）。  
   - **输出（Virtual Output）**：实时读取 FPGA 内部信号的状态（例如状态机状态、计数器值等）。  
   - 支持组合逻辑和时序逻辑（需时钟驱动）。

2. **节省调试时间**  
   - 无需修改代码或重新综合设计即可调整参数，显著提高调试效率。

3. **资源占用低**  
   - 相比 ILA（逻辑分析仪）等调试工具，VIO 消耗的 FPGA 资源（LUT、FF）更少。

4. **与硬件管理器集成**  
   - 通过 Vivado 的 Hardware Manager 直接操作，无需额外工具。


### **VIO 的典型应用场景**
1. **动态参数调整**  
   - 例如：实时修改滤波器的系数、调整 PWM 占空比、改变通信协议的参数（如波特率）。

2. **控制信号调试**  
   - 手动触发复位（reset）、使能（enable）信号，或切换状态机的状态。

3. **交互式验证**  
   - 在验证算法或协议时，通过手动输入激励信号并观察响应。

4. **资源受限场景**  
   - 当 FPGA 板上物理输入/输出（如拨码开关、LED）不足时，用 VIO 替代。


### **VIO 与 ILA 的区别**
| **特性**               | **VIO**                          | **ILA**                          |
|------------------------|----------------------------------|----------------------------------|
| **功能**               | 实时输入/输出信号值              | 捕获信号波形（类似逻辑分析仪）    |
| **数据触发**           | 不支持触发条件                   | 支持复杂的触发条件设置            |
| **资源占用**           | 低                               | 较高（依赖采样深度和信号数量）    |
| **用途**               | 交互调试、参数调整               | 信号时序分析、故障诊断            |




----


## 配置VIO

### 使用IP核创建VIO调试环境

1. 点击左侧`PROJECT MANAGER`栏 → `IP Catalog`或者菜单栏下`Window` → `IP Catalog`，然后在右侧出现的`IP Catalog`窗口下搜索`VIO`，双击选择`Debug`下的`VIO`进行IP配置，操作步骤如下图所示：

<div align="center">
<img src=./VIO-use/1.png width=100%/>
</div>

2. VIO IP参数设置

    - `Input Probe Count`：VIO输入探头个数，即输入到VIO，需要查看实时数据值的信号个数；
    - `Output Probe Count`：VIO输出探头个数，即输出给其他模块的信号个数；
    - `Enable Input Probe Activity Detectors`：输入探头变化检测。若勾选，则在后续调试过程中，某个输入信号发生变化时，则会出现数据变化的提示；若不勾选，则无输入数据变化提示（注意，这里输入数据是指输入到VIO模块中的数据）；

<div align="center">
<img src=./VIO-use/2.png width=100%/>
</div>


<div align="center">
<img src=./VIO-use/3.png width=100%/>
</div>



<div align="center">
<img src=./VIO-use/4.png width=100%/>
</div>

3. 对VIO IP核配置完成后，生成IP核

-----


##  VIO的调试

- 本次以UART发送为例，通过VIO输出串口需要发送的8比特数据data和1比特的发送使能信号，同时将串口发送结束信号tx_done接入到VIO的输入端，观察其值和变换。
- 首先，在IP Sources窗口下找到`vio_0.veo`，该文件为生成的VIO IP核的例化模板，将该例化模板复制到UART发送模块的顶层文件中，对例化名称以及VIO的输入输出进行连接。


<div align="center">
<img src=./VIO-use/6.png width=60%/>
</div>



<div align="center">
<img src=./VIO-use/5.png width=60%/>
</div>



### VIO具体调试步骤

1.  对顶层文件综合实现后，生成比特流文件，将生成的.bit文件和.ltx文件下载到FPGA开发板。

<div align="center">
<img src=./VIO-use/7.png width=50%/>
</div>



2. 打开VIO调试窗口，在该窗口下可以看到接入VIO输入/输出探头的各个信号的信号名称Name、当前值Value、是否发生变化Activity等信息。

<div align="center">
<img src=./VIO-use/8.png width=60%/>
</div>