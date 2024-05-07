down:: [[pulp-riscv]]
down:: [[开源项目和教程]]

有几个需要vpn
[串行接口芯片16550_xqhrs232的博客-CSDN博客](https://blog.csdn.net/xqhrs232/article/details/51218578)

risc-v参阅平台
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221025230341.png)
[riscv-platform-specs/riscv-platform-spec.adoc at main · riscv/riscv-platform-specs · GitHub](https://github.com/riscv/riscv-platform-specs/blob/main/riscv-platform-spec.adoc)

构建工具链仿真：
[GitHub - pulp-platform/pulpino: An open-source microcontroller system based on RISC-V](https://github.com/pulp-platform/pulpino)riscv-chain
[Site Unreachable](https://github.com/riscv-collab/riscv-gnu-toolchain)
配置参考
[PULPino 在 Modelsim 下的软件模拟 - 知乎](https://zhuanlan.zhihu.com/p/470281404)

好东西，啥都有risc，pulp介绍
[Page Not Found · GitBook (Legacy)](https://cnrv.gitbooks.io/riscv-soc-book/content/ch8/sec2-PULPino_overview.html)

cmake教程
[CMake教程（一） - 知乎](https://zhuanlan.zhihu.com/p/119426899)
cmake官方教程
[CMake Tutorial — CMake 3.17.5 Documentation](https://cmake.org/cmake/help/v3.17/guide/tutorial/index.html#adding-system-introspection-step-5)

较详细的介绍
[8.3 RI5CY介绍（已完成） · 关于RISC-V你所需要知道的一切](https://cnrv.gitbooks.io/riscv-soc-book/content/ch8/sec3-RI5CY_overview.html)

工具链搭建
⚠️upload failed, check dev console

指令集的简单介绍，初学可以看
[计算机系统基础（五）之RISC-V指令集_深度学习的学习僧的博客-CSDN博客_risc-v指令集](https://blog.csdn.net/qq_38915354/article/details/115696721)

### RISC-V 程序计数器 （PC）
在 ARM 的设计中，PC 是作为一个通用寄存器而存在的，这意味着任何能改变寄存器值的指令都有可能导致程序执行分支跳转。
这样带来了一个坏处，那就是对分支跳转的预测变复杂了。

RISC-V 将 PC 单独拿出来作为一个特殊的寄存器来对待，这使得能够改变 PC 寄存器的指令变少，分支跳转的预测准确性便于提高。

### riscv 指令集
[计基（二）RISC-V指令集介绍与汇编 - 知乎](https://zhuanlan.zhihu.com/p/558799873)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20231115120701.png)
[RISC-V架构学习-CSDN博客](https://blog.csdn.net/tristan_tian/article/details/106315232)

[【精选】RISC-V流水线CPU 计算机组成与设计第四章第二部分\_ld指令的数据通路-CSDN博客](https://blog.csdn.net/Photice/article/details/125240998)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20231115122052.png)
