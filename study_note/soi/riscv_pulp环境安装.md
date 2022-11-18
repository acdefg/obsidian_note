起因 pulpion 环境要求：
大问题在于 riscv 交叉编译工具链的安装，xs 后来发现 modelsim 才难搞
它要 riscv32-unknown-elf-gcc，应该重点在 32 位，newlib 版本

> [!attention] 注意
> 要切换到 root 路径下，打开下载好的 tool chain 文件夹
> 记得在 zsh/bash 中添加路径，[[在 zsh 中添加路径]]
> 安装错了可以先 `make clean` 再 make

## pulpion environment requirement
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181431287.png)
[[ubuntu安装modelsim]]

#### 一些工具链：
[GitHub - acdefg/pulp-riscv-gnu-toolchain](https://github.com/acdefg/pulp-riscv-gnu-toolchain) pulp
[GitHub - acdefg/riscv-gnu-toolchain: GNU toolchain for RISC-V, including GCC](https://github.com/acdefg/riscv-gnu-toolchain) riscv
[GitHub - acdefg/ri5cy_gnu_toolchain](https://github.com/acdefg/ri5cy_gnu_toolchain) ri5cy
个人感觉 pulp tool chain 和 riscv 的应该是一样的，前几次安装 ri5cy 老是失败有可能是没有开 root 权限，直接换了 pulp tool chain
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181412510.png) ![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181419145.png)


## reference link
[编译riscv32-unknown-elf-gcc_牧羊女说的博客-CSDN博客](https://blog.csdn.net/deliapu/article/details/120708442) 比较详细
[RISC-V GNU工具链的编译与安装 - 知乎](https://zhuanlan.zhihu.com/p/364638851) 有指令的解析
[riscv各种版本gcc工具链编译与安装_weiqi7777的博客-CSDN博客](https://blog.csdn.net/weiqi7777/article/details/88045720) riscv 的各种工具链
[PULPino 在 Modelsim 下的软件模拟 - 知乎](https://zhuanlan.zhihu.com/p/470281404) 看到的第一个模拟，modelsim，参考意义不大
[pulpino[1] nuttx：bring up · 大专栏](https://www.dazhuanlan.com/cnrootkit/topics/1415719) pulpion nuttx 内核移植
[安装一个可用的Linux版本Modelsim](https://junningwu.haawking.com/tech/2019/12/11/%E5%AE%89%E8%A3%85%E4%B8%80%E4%B8%AA%E5%8F%AF%E7%94%A8%E7%9A%84Linux%E7%89%88%E6%9C%ACModelsim/) modelsim 仿真 pulpion
[Running Modelsim on a 64-bit Ubuntu - Github personal blog](https://pcotret.github.io/modelsim-ubuntu/) 用到 freetype 不懂但是留着

[Issues · pulp-platform/pulpino · GitHub](https://github.com/pulp-platform/pulpino/issues) 😒鉴于我老点它，所以存一下
## log
### tool chain

下载 tool chain respository
```shell
git clone --recursive https://github.com/pulp-platform/pulp-riscv-gnu-toolchain
```
在使用官方库的时候用 `--recursive` 选项失败，这里可以成功

安装环境 ubuntu：
```shell
$ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev
```

安装，这一步要先切 root 用户
```shell
./configure --prefix=/opt/riscv --with-arch=rv32imc --with-cmodel=medlow --enable-multilib
make newlib -j4
```
这一步大部分选项算是一种尝试，直接使用下面 newlib 的指令安装的是 64 位，make 的选项可以参考 [RISC-V GNU工具链的编译与安装 - 知乎](https://zhuanlan.zhihu.com/p/364638851) 这里

### pulp 编译篇
#### part 1
在 pulpion 源码文件夹下面，去 SW 文件夹下面 `mkdir build`，然后在把 SW 下的这个复制到 build 里面，虽然会报错，但是还是运行一下 `cmake_configure.riscv.gcc.sh` ，其它几个对应不同的 riscv

```shell
./cmake_configure.riscv.gcc.sh
```
然后新建两个文件，并且 `chmod u+x *.sh` ，以防忘记请把*号替换成文件名
fix_m32.sh

```shell
#!/bin/zsh

#Find and replace all occurrances of '' and fix rest of line.

for file in $(find); do
    if [[ -f $file ]]; then
        [[ $(cat $file | grep m32) ]]
        if [[ $? == 0 ]]; then
            echo writing...
            echo $file
           sed 's/\//g' $file > tmp && mv tmp $file
       fi
    fi
done
```

fix_linker.sh

```shell
#!/bin/zsh

riscv32-unknown-elf-ld --verbose | head -n -1 | tail -n +7 | sed '168 a \ \ _fbss = .;' | sed '169 a \ \ . = .;' > /home/cici/code/qulp_test/pulpino/sw/build/CMakeFiles/CMakeTmp/riscv.ld
```
最后那个很长的路径要改掉

再修改 cmake***.sh 文件中的 `GCC_MARCH="RV32IMXpulpv2"`，把原来的这个标志换掉好嘛

先跑 fix_m32.sh，然后跑 fix_linker.sh，跑 cmake_configure.riscv.gcc.sh，和 fix_linker.sh 交替跑，直到成功

#### part 2


### 一些错误
1. /usr/bin/env: ‘python’: No such file or directory
没定位 python
2. Missing parentheses in call to 'print'. Did you mean print(...)?
python2，不能用 python3
这两个一起解决用：

```shell
sudo ln -s /usr/bin/python2 /usr/bin/python
```

当然先看看 python2 在哪，虽然一般就在/usr/bin 下面，没有 python2 就装一个，原来用 python3 定位的默认 python，直接 `sudo rm /usr/bin/python`，再定位就好了，反正就是很暴力，已经烦了
3. ImportError: No module named yaml
只能说不要再报错了，受够了，能不能一次把话说清楚
`sudo pip2 install pyyaml #python2` python2 这么装，别问我 python3 怎么装，自己去搜吧
4. 安装 modelsim 19.0 报错
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211181745047.png)
5. [zsh: permission denied问题的解决办法_sido的博客-CSDN博客](https://blog.csdn.net/chnyifan/article/details/104705437)
6. /bin/sh :1 tcsh not found 之类的，手敲的，具体的忘记了
[Linux中tcsh/tcsh和bash的环境变量的配置以及切换shell版本_SOC罗三炮的博客-CSDN博客_linux tcsh](https://blog.csdn.net/luolaihua2018/article/details/124382529)
```shell
sudo apt install tcsh
```
还安错了一次，安成了 tclsh，这些应该和 zsh 一样是一种 shell

## ideas
用 qemu 仿真试试，参考普通 riscv 的办法，难点在于没有程序可以测试