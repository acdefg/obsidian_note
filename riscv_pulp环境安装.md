起因 pulpion 环境要求：
大问题在于 riscv 交叉编译工具链的安装
它要 riscv32-unknown-elf-gcc，应该重点在 32 位，newlib 版本
安装错了可以先 `make clean` 再 make

> [!attention] 注意
> 要切换到 root 路径下，打开下载好的 tool chain 文件夹
> 记得在 zsh/bash 中添加路径，[[在 zsh 中添加路径]]


一些工具链：
[Site Unreachable](https://github.com/acdefg/pulp-riscv-gnu-toolchain) pulp
[Fetching Title#26vv](https://github.com/acdefg/pulp-riscv-gnu-toolchain)
[Site Unreachable](https://github.com/acdefg/riscv-gnu-toolchain) riscv
[Site Unreachable](https://github.com/acdefg/ri5cy_gnu_toolchain) ri5cy
个人感觉 pulp tool chain 和 riscv 的应该是一样的，前几次安装 ri5cy 老是失败有可能是没有开 root 权限，直接换了 pulp tool chain
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181412510.png) ![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181419145.png)


## reference
[编译riscv32-unknown-elf-gcc_牧羊女说的博客-CSDN博客](https://blog.csdn.net/deliapu/article/details/120708442) 比较详细
[RISC-V GNU工具链的编译与安装 - 知乎](https://zhuanlan.zhihu.com/p/364638851) 有指令的解析