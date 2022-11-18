起因 pulpion 环境要求：
大问题在于 riscv 交叉编译工具链的安装
它要 riscv32-unknown-elf-gcc，应该重点在 32 位，newlib 版本

> [!attention] 注意
> 要切换到 root 路径下，打开下载好的 tool chain 文件夹
> 记得在 zsh/bash 中添加路径，[[在 zsh 中添加路径]]
> 安装错了可以先 `make clean` 再 make

## pulpion environment requirement
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181431287.png)


一些工具链：
[GitHub - acdefg/pulp-riscv-gnu-toolchain](https://github.com/acdefg/pulp-riscv-gnu-toolchain) pulp
[GitHub - acdefg/riscv-gnu-toolchain: GNU toolchain for RISC-V, including GCC](https://github.com/acdefg/riscv-gnu-toolchain) riscv
[GitHub - acdefg/ri5cy_gnu_toolchain](https://github.com/acdefg/ri5cy_gnu_toolchain) ri5cy
个人感觉 pulp tool chain 和 riscv 的应该是一样的，前几次安装 ri5cy 老是失败有可能是没有开 root 权限，直接换了 pulp tool chain
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181412510.png) ![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181419145.png)


## reference
[编译riscv32-unknown-elf-gcc_牧羊女说的博客-CSDN博客](https://blog.csdn.net/deliapu/article/details/120708442) 比较详细
[RISC-V GNU工具链的编译与安装 - 知乎](https://zhuanlan.zhihu.com/p/364638851) 有指令的解析
[riscv各种版本gcc工具链编译与安装_weiqi7777的博客-CSDN博客](https://blog.csdn.net/weiqi7777/article/details/88045720) riscv 的各种工具链
[PULPino 在 Modelsim 下的软件模拟 - 知乎](https://zhuanlan.zhihu.com/p/470281404) 看到的第一个模拟，modelsim，参考意义不大
[pulpino[1] nuttx：bring up · 大专栏](https://www.dazhuanlan.com/cnrootkit/topics/1415719) pulpion nuttx 内核移植
