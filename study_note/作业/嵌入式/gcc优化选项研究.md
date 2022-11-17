[gcc编译优化-O0 -O1 -O2 -O3 -OS解析_奔跑的码农的博客-CSDN博客_gcc o2](https://blog.csdn.net/wuxing26jiayou/article/details/96132721) 选项内容具体描述
[汇编视角:不同优化级别下的GCC行为分析 | 海森的博客](https://hisenz.com/post/%E6%B1%87%E7%BC%96%E8%A7%86%E8%A7%92-%E4%B8%8D%E5%90%8C%E4%BC%98%E5%8C%96%E7%BA%A7%E5%88%AB%E4%B8%8B%E7%9A%84GCC%E8%A1%8C%E4%B8%BA%E5%88%86%E6%9E%90/) 👍

想使用反汇编的方式查看不同选项对于程序的优化，于是想到了另一门课学到的交叉编译，可以生成 arm64 位环境下的.o 文件，再用.o 反汇编生成汇编码，对比查看不同选项在汇编角度的优化。
选择 memcpy 作为例子是因为它的实现代码简单, 但是涉及了传参, 条件判断和循环, 是逻辑密集型的代码, 能很好的体现 gcc 在逻辑上的优化。
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211171818644.png)

使用交叉编译，编译为 arm64 位环境的程序进行测试, 因为程序中没有 main 函数所以在交叉编译时加上 -c 选项：
```shell
aarch64-linux-gnu-gcc -c -o memcy0.o memcy.c -O0
```
使用 file 指令打印文件类型，可以看到该程序属于在 arm64 环境：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211171841155.png)
使用 `objdump ` 对程序进行反汇编：

