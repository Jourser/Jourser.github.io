---
title: FPGA学习笔记之VIVADO自定义IP核
date: 2024-06-24 09:36:39
img:
- /medias/featureimages/28.jpg
categories: 硬件
tags:
 - FPGA
 - vivado
 - 教程
 - 基础
---

# 自定义IP核

- 本文基于 Vivado 工程实践，系统介绍什么是自定义 IP 核、为什么要使用自定义 IP、以及如何创建、封装和修改自定义 IP 核。


## 自定义IP核介绍

### 什么是自定义IP核

- 在 Vivado 中，IP（Intellectual Property）核本质上是一个功能模块的封装单元，它可以是：
  - 一个标准接口控制器（如 AXI GPIO、AXI DMA）
  - 一个算法模块（FFT、FIR）
  - 也可以是**我们自己编写的 Verilog/VHDL 模块**

- Vivado 将这些模块以 IP 的形式进行统一管理，使其可以：
    - 被 Block Design（BD）直接调用
    - 具备参数化配置能力
    - 方便复用、版本管理和系统集成

### 为什么要使用自定义 IP 核？

- 在实际工程中，尤其是 SoC（如 Zynq）或大型 FPGA 项目中，直接在顶层写所有逻辑会带来很多问题：
    1. 工程结构混乱，难以维护
    2. 模块复用成本高
    3. 无法方便地与 AXI、PS 系统集成
    4. Block Design 无法直接调用 RTL 模块
- 而自定义 IP 核可以很好地解决这些问题：
    - 将功能模块独立成 IP，逻辑清晰
    - 支持 AXI、时钟、复位等标准接口
    - 方便在多个工程中复用
    - 非常适合 PS + PL 协同开发

### 自定义 IP 核的本质结构


- 一个 Vivado 自定义 IP，底层其实是一个标准的目录结构，核心包括：
    - RTL 源码（Verilog / VHDL）
    - IP 描述文件（component.xml）
    - 参数定义（Parameters）
    - 接口定义（AXI、Clock、Reset 等）

👉 注意：
- component.xml 是 Vivado 识别 IP 的关键文件，但通常不建议手动修改，而是通过 Vivado 的 IP 工具自动生成


## IP核的创建

1. 在 Vivado 中依次点击：`Tools → Create and Package New IP`

<div align="center">
<img src=./new-ip/1.png width=40%/>
</div>

2. 弹出“**创建和封装新 IP 向导**”
<div align="center">
<img src=./new-ip/2.png width=70%/>
</div>

3. 选择封装任务类型

    - 为 Vivado IP Catalog 封装一个新的 IP
      -  打包你当前的项目（本次采用）
      -  将当前项目中的模块设计打包
      -  打包指定目录
    - 创建一个新的 AXI4 外设

<div align="center">
<img src=./new-ip/3.png width=70%/>
</div>

4. 选择要存放的位置，这一步很重要，可以存放在以后专门自定义的IP文件夹内，方便以后添加和管理查看。

<div align="center">
<img src=./new-ip/4.png width=70%/>
</div>

5. Vivado 询问是否要将当前工程的源码复制到指定的 IP 库目录，并**打开一个全新的“临时工程”**来进行封装配置。

<div align="center">
<img src=./new-ip/5.png width=60%/>
</div>

6. Vivado 自动提取的信息清单
<div align="center">
<img src=./new-ip/6.png width=70%/>
</div>

7. 然后会弹出新项目，如下封装IP的界面，按如下执行`Package IP`
<div align="center">
<img src=./new-ip/7.png width=70%/>
</div>

<div align="center">
<img src=./new-ip/8.png width=60%/>
</div>



- `Customization Parameters`和`Addressing and Memory`没有打钩属于正常现象。
  - `Customization Parameters`：这个选项是用来识别你的 Verilog 代码中的 `parameter`（参数）的。比如，如果你想让外部可以修改采样计数器的最大值，你会写 `parameter MAX_CNT = 49;`。
    - 端口定义里没有任何 parameter 参数。既然没有参数可供配置，Vivado 就检测不到任何需要定制的内容，所以这一栏自然就是空的（或者显示未激活状态），这是完全正常的。
  - `Addressing and Memory`:这个选项是给总线接口（如 AXI4-Lite, AXI-Full）用的。如果你的 IP 核需要连接到 CPU（如 Zynq 的 PS 端或 MicroBlaze），并且需要 CPU 通过地址读写寄存器，就需要在这里配置地址映射。
    - 没有标准的 AXI 总线接口。因此，不需要分配地址空间，这一栏也是空的。
-  `Review and Package`这一项已经有了绿色的对勾。这意味着：
   -  Vivado 认为目前的配置已经足够生成一个合法的 IP 核了。
   -  可以直接点击左侧菜单底部的 `Review and Package`。
   -  然后点击界面下方的 `Re-Package IP` 按钮即可完成更新。
   -  **总结**：只要 `Review and Package` 是绿的，就可以放心打包！

1. 至此，IP就封装好了。

## IP核的使用

1. 新建一个vivado工程，IP核保持的路径加入到工程中
<div align="center">
<img src=./new-ip/9.png width=70%/>
</div>

2. 创建一个BD文件
<div align="center">
<img src=./new-ip/10.png width=30%/>
</div>

<div align="center">
<img src=./new-ip/11.png width=70%/>
</div>


3. 使用自定义的IP

<div align="center">
<img src=./new-ip/12.png width=70%/>
</div>

<div align="center">
<img src=./new-ip/13.png width=40%/>
</div>

- 添加完之后如下：

<div align="center">
<img src=./new-ip/14.png width=70%/>
</div>

4. 将引脚引出，与其他IP核相连。

<div align="center">
<img src=./new-ip/15.png width=70%/>
</div>

5. 然后右键选择`General Output Product`

<div align="center">
<img src=./new-ip/16.png width=60%/>
</div>

- Vivado 封装 IP 核或生成 Block Design 输出文件
  1. Global (全局综合)
  2. Out of context per IP (OOC，单个IP核，默认选项，最常用)
  3. Out of Context per Block Design(整个 Block Design 设计图)
<div align="center">
<img src=./new-ip/17.png width=50%/>
</div>

6. 然后右键选择 `create HDL wrapper`
<div align="center">
<img src=./new-ip/18.png width=50%/>
</div>

<div align="center">
<img src=./new-ip/19.png width=50%/>
</div>


7. 最后添加约束文件并进行综合编译产生bit流


## IP核的修改

- 添加到工程里的IP是不能直接修改的,文件是只读的形式
1.  右击IP核，选择`Edit in IP Packager`

<div align="center">
<img src=./new-ip/20.png width=50%/>
</div>
<div align="center">
<img src=./new-ip/21.png width=50%/>
</div>  

2. 打开一个IP核自身的工程，在这里IP是可以修改的。比如说加入ILA核。
3. 修改完以后需要进入，把Packaging Steps中的每一项都再编译一遍
<div align="center">
<img src=./new-ip/22.png width=70%/>
</div>

<div align="center">
<img src=./new-ip/23.png width=70%/>
</div>

4. 最后`Re-Package IP`

<div align="center">
<img src=./new-ip/24.png width=70%/>
</div>

5. 完成后关掉该工程。回到原来的工程后我们会发现IP处于锁定的状态
<div align="center">
<img src=./new-ip/25.png width=40%/>
</div>

6. 依次点击`Tools → Report → Report IP Status`

<div align="center">
<img src=./new-ip/26.png width=60%/>
</div>

- 或者

<div align="center">
<img src=./new-ip/27.png width=60%/>
</div>

- 点击`Upgrade Selected`

<div align="center">
<img src=./new-ip/28.png width=80%/>
</div>

