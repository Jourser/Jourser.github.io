---
title: FPGA学习笔记之工程压缩
date: 2025-08-05 13:10:39
img:
- /medias/featureimages/27.jpg
categories: 硬件
tags:
 - FPGA
 - vivado
 - 教程
 - 工具
---
# Vivado工程压缩

## 前言

- Vivado 是 Xilinx 提供的一款强大的 FPGA 开发工具，它在项目构建过程中会自动生成大量的临时文件、缓存数据、综合和实现结果。这些文件虽然对调试和开发过程很有帮助，但也使得整个项目文件夹变得庞大，动辄几百 MB，甚至上 GB。

- 这种“膨胀”的工程结构在以下场景中会带来不少麻烦：
    - 项目迁移：从一台电脑转移到另一台时，大体积文件夹拖慢了复制速度；
    - 协作开发：团队成员间通过 Git 或网盘共享工程时，不必要的文件不仅浪费空间，还可能引入版本混乱；
    - 项目备份：频繁备份会占用大量磁盘资源，尤其是多个版本同时保留时；
    - 成果提交：在论文或竞赛中需要提交项目源码时，一个“干净”的、压缩过的工程能大大提升可读性和专业感。

- 因此，学会如何清理工程中的冗余内容，并对工程进行有效压缩打包，是一项非常实用的技能。不仅可以节省存储空间，还能提升项目的可移植性和分享效率。

- 本文将详细介绍 Vivado 工程目录中哪些文件是必须保留的，哪些可以安全删除，并提供清理方法、压缩技巧以及相关自动化脚本，帮助你打造一个更轻便、更干净的 Vivado 项目结构。

----

## Vivado 工程结构简介

- 在清理 Vivado 工程之前，了解 Vivado 的默认工程结构非常重要。Vivado 会在项目构建过程中自动生成大量文件和目录，其中只有一部分是工程必需的，其余大多是缓存、中间结果或仿真文件，可以在不影响功能的前提下删除。

- 以下是一个典型的 Vivado 工程目录结构示意：

  ```
  <project_name>/
    ├── .Xil/                        # 缓存文件，可删除
    ├── project.cache/              # 缓存文件，可删除
    ├── project.hw/                 # 硬件相关文件，可删除
    ├── project.ip_user_files/      # IP核生成的中间文件，可删除
    ├── project.runs/               # 合成 & 实现的结果，可删除
    ├── project.sim/                # 仿真结果，可删除
    ├── project.srcs/               # 你的源码目录，必须保留
    ├── project.xpr                 # 工程主文件，必须保留
    ├── project.xdc / *.xdc         # 时序约束文件，必须保留
    ├── *.xci                       # IP核配置文件，必须保留
    ├── *.bd                        # Block Design 文件，必须保留（如有）
    └── *.tcl                       # 脚本文件（如存在），建议保留
  ```
### ✅ **必须保留的内容**

| 文件/目录           | 说明                       |
| --------------- | ------------------------ |
| `*.xpr`         | Vivado 工程主文件（入口）         |
| `*.xci`         | IP核配置文件（非中间结果）           |
| `*.xdc`         | 约束文件，用于设置引脚/时序           |
| `project.srcs/` | 源码文件夹，包括 Verilog/VHDL 文件 |
| `*.bd`          | Block Design 图形化设计（如使用）  |
| `.tcl` 脚本       | 项目配置脚本，建议保留备份            |

### ❌ **可清理的内容**

| 文件/目录            | 说明                    |
| ---------------- | --------------------- |
| `.Xil/`          | Vivado 的缓存文件，删除不影响工程  |
| `project.cache/` | 工程缓存目录，可删除            |
| `project.runs/`  | 合成、实现和 bit 文件生成结果，可重建 |
| `project.hw/`    | 硬件生成文件目录，可重建          |
| `project.sim/`   | 仿真生成文件目录，可重建          |
| `ip_user_files/` | IP核生成的中间文件，可删除        |

清理以上目录和文件不会破坏 Vivado 工程的完整性，只是在下次打开工程或重新综合时需要重新生成。建议在打包、迁移或上传工程时清理这些内容，以减小体积。

-----

## 清理 Vivado 工程的方法

- 在实际使用Vivado过程中，该工具会自动生成一系列文件，有些文件是不必时刻保存的中间文件，有些是为了提高编译效率而生成的（比如编译IP核后产生的文件）。然而在上传SVN或备份时，我们希望占用尽量少的存储空间。由于Vivado无法自动清理这些文件，因此需要手动清理。


### 清理vivado工程

- 使用 Vivado 会伴随产生多种辅助文件，包括临时性的中间文件和用于优化编译速度的文件（如编译 IP 核生成的文件）。这些文件在上传至 SVN 或执行备份时会占用额外空间。由于 Vivado 未提供自动清理功能，手动移除这些文件是实现高效存储管理的关键步骤。

#### reset_project 命令说明

- 该命令会清除工程中以下内容：
    - 合成结果（synthesis runs）
    - 实现结果（implementation runs）
    - 仿真结果
    - 临时中间文件
    - 生成的 bitstream 文件

- 但不会删除工程配置、源代码、约束文件、IP 配置等关键内容，所以使用起来是安全的。
- 如果想保留IP目录下的文件，也可通过命令`reset_project -exclude ip`实现


#### 📌 reset_project 命令执行

1. 打开你的 Vivado 工程；

2. 点击菜单栏：**Window → Tcl Console**；

3. 在 Tcl Console 中输入：

   ```tcl
   reset_project
   ```

4. 按回车执行，工程将被清理。



<div align="center">
<img src=./vivado-project-compression/2.png width=90%/>
</div>


- **效果如下：**

<div align="center">
<img src=./vivado-project-compression/1.png width=40%/>
<img src=./vivado-project-compression/3.png width=40%/>
</div>


### 清理仿真工程结果

- 执行 Vivado 仿真会积累大量缓存文件，这些数据将占据可观的磁盘容量。项目仿真目录下的文件可在仿真结束后通过 `reset_sim` TCL 命令进行清理。

- 同时，用户需留意系统临时文件夹（典型路径为 C 盘下的 Temp 目录）也可能存在仿真缓存。为应对 C 盘空间持续被占用的问题，定期清理系统临时目录是必要的维护措施。


#### reset_sim 命令说明

- 该命令只会清除工程中仿真相关的生成文件，包括：
    - 仿真运行目录（如 `xsim/`）
    - 仿真波形输出文件（如 `.wdb`, `.vcd` 等）
    - Vivado 仿真脚本和临时文件

- 它不会影响综合（synthesis）、实现（implementation）或原始仿真源文件（如 `.sv`, `.vhd`）等内容，因此适用于只想清空仿真结果、但保留其余设计流程状态的场景。


#### 📌 reset_sim 命令执行

1. 打开你的 Vivado 工程；

2. 点击菜单栏：**Window → Tcl Console**；

3. 在 Tcl Console 中输入：

   ```tcl
   reset_sim
   ```

4. 仿真输出将被清除，工程保持其他部分不变。


<div align="center">
<img src=./vivado-project-compression/5.png width=80%/>
</div>


- **效果如下：**

<div align="center">
<img src=./vivado-project-compression/4.png width=40%/>
<img src=./vivado-project-compression/6.png width=40%/>
</div>