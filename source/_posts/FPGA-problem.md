---
title: FPAG学习笔记之问题集合（更新中）
date: 2025-03-31 10:04:51
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