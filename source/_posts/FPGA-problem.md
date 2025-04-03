---
title: FPAG学习笔记之问题集合（更新中）
date: 2025-03-31 10:04:51
img:
- /medias/featureimages/25.jpg
categories: 硬件
tags: 
 - FPGA
 - vivado
 - 问题解决
---

# FPAG问题集合

## 1. Vivado生成存储器配置文件时没有所需型号的Flash芯片

- 具体问题描述：在程序固化时，尤其是在生成`.svf`文件时(具体参考[Vivado程序固化之生成.svf文件](https://jourser.github.io/2024/06/05/program-the-fpga/#%E7%94%9F%E6%88%90-svf%E6%96%87%E4%BB%B6))。vivado2017版本，没有找到w25q128型号flash。

### 解决办法

- 尝试上网搜索“vivado w25q128”，基本上可以找到添加方法。下面以w25q128型号为例，介绍如何添加。
1. 找到[安装目录]\2017.4\data\xicom下的`xicom_cfgmem_part_table.csv`文件。

    <div align="center">
    <img src=./FPGA-problem/1.png width=70%/>
    </div>

2. 打开该文件，在最下方添加一行数据。用vs code为例。

    <div align="center">
    <img src=./FPGA-problem/2.png width=70%/>
    </div>

3. 在文件最后，复制下面的配置信息，保存即可。

```
475,0,w25q128bv-spi-x1_x2_x4,- xa7a100t xa7a15t xa7a35t xa7a50t xa7a75t xc7a100t xc7a100ti xc7a100tl xc7a12t xc7a12ti xc7a12tl xc7a15t xc7a15ti xc7a15tl xc7a200t xc7a200ti xc7a200tl xc7a25t xc7a25ti xc7a25tl xc7a35t xc7a35ti xc7a35tl xc7a50t xc7a50ti xc7a50tl xc7a75t xc7a75ti xc7a75tl xq7a100t xq7a200t xq7a50t xc7k160t xc7k160ti xc7k160tl xc7k325t xc7k325ti xc7k325tl xc7k355t xc7k355ti xc7k355tl xc7k410t xc7k410ti xc7k410tl xc7k420t xc7k420ti xc7k420tl xc7k480t xc7k480ti xc7k480tl xc7k70t xc7k70tl xq7k325t xq7k325tl xq7k410t xq7k410tl xcku025 xcku035 xcku040 xcku060 xcku085 xcku095 xcku115 xqku040 xqku060 xqku095 xqku115 xc7s100 xc7s15 xc7s25 xc7s50 xc7s6 xc7s75 xcvu065 xcvu080 xcvu095 xcvu125 xcvu160 xcvu190 xcvu440,w25q128bv,spi,128,x1_x2_x4,,Winbond,,1,w25q128bv,w25q
```

<div align="center">
<img src=./FPGA-problem/3.png width=70%/>
</div>

- 注意：配置信息的最前方的数字要根据自己的`xicom_cfgmem_part_table.csv`文件情况而定，比如本文所提供的例子中，最前方的数字是475，而你的文件中可能是其他的。


4. 重新打开vivado，在生成配置文件时就可以看到w25q128型号了。

    <div align="center">
    <img src=./FPGA-problem/4.png width=70%/>
    </div>

-----

## 2. .svf文件无法写入Flash

- 具体问题描述： flash 型号是 W25Q128JVSIQ，FPGA 型号是 xc7a100t。当使用 Vivado 生成 `.svf` 文件，但是当我使用 execute_hw_svf 将文件写入flash时，出现一条错误消息：`Output value 00 does not match the tdo option value`。但是，当使用 vivado 将 `.mcs` 文件直接写入 flash 时，没有给出错误消息，程序运行正常。

    <div align="center">
    <img src=./FPGA-problem/5.png width=70%/>
    <img src=./FPGA-problem/6.png width=70%/>
    </div>

### 解决办法
- 原因可能是：时延不足导致 TDO 校验错误。需要增加时延。下面介绍需要修改哪些时延。
- Vivado/iMPACT 根据以下条件自动选择擦除模式：
    - 如果 Flash 支持扇区擦除且设计文件仅更新部分扇区，优先生成 `sector erase` 指令（减少烧录时间）。
    - 如果 Flash 不支持扇区擦除（如某些老款 SPI Flash），则强制使用 `fullchip erase`。



- 关键影响：时延要求不同
   | **擦除类型**       | 典型时延要求       | 风险                                  |
   |--------------------|--------------------|---------------------------------------|
   | `fullchip erase`   | 长（秒级）         | 时延不足会导致 TDO 校验失败           |
   | `sector erase`     | 短（毫秒级）       | 时延过长会降低效率，过短则可能失败    |




- **Full-Chip Erase**：  
  - 擦除整个 Flash 芯片（所有扇区一次性清除），通常需要 **更长时延**（例如 100ms~3s），适合首次烧录或彻底重写。  
  - **典型指令**：  
   ```svf
   // Modify the below delay for fullchip erase operation (0.000000 sec typical, 0.000000 sec maximum)
   RUNTEST 30.000000 SEC;
   ```
- **Sector Erase**：  
  - 仅擦除指定扇区（按地址分块），时延较短（如 50~500ms），适合局部更新。  
  - **典型指令**：  
   ```svf
   // Modify the below delay for sector erase operation (0.000000 sec typical, 0.000000 sec maximum)
   RUNTEST 0.100000 SEC;
   ```

- **program operation**
  - 除此之外，还有`program operation` 时延也需要调整。
  - 在 `.svf` 文件中，`program operation` 的时延（即 `RUNTEST` 指令后的时间值）用于 **控制 Flash 编程（写入）操作后的等待时间**。
  - **典型指令**：  
   ```svf
    // Modify the below delay for program operation (0.000000 sec typical, 0.000000 sec maximum)
    RUNTEST 0.005000 SEC;
   ```
  - 不同型号的 Flash 对编程操作的最小耗时（`tPROG`）有严格规定。若 `.svf` 中时延设为 `0.000000`，可能低于 Flash 实际需求，导致写入不完整。
  - 将 `RUNTEST 0.000000 SEC;` 改为 `RUNTEST 0.005000 SEC;`（即 **5ms**）可：
    1. **覆盖大多数 Flash 的 `tPROG` 要求**：  
        即使最慢的 Flash 页编程（如 3ms）也能完成。
    2. **平衡效率与可靠性**：  
        远低于全片擦除的秒级等待，但足够避免写入失败。


- **如何确定最佳时延值？**
  - **方法 1：查阅 Flash 数据手册**  
    找到 **`tPROG`（Page Program Time）** 参数，例如：
    - 若 `tPROG(max) = 2ms`，则设置 `RUNTEST 0.003000 SEC;`（留 50% 余量）。
  - **方法 2：实验调整**  
    从较小值（如 `0.001000 SEC`）逐步增加，直到写入成功。

---

