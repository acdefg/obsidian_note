### concepts
[逻辑综合——工艺库\_report\_lib\_沧海一升的博客-CSDN博客](https://blog.csdn.net/qq_21842097/article/details/111566443)
目标库包含各个门级单元的行为、引脚、面积、时序信息、功耗方面的参数等信息。DC 在综合时就是根据目标库给出的单元电路的时序信息来计算延迟，并根据各个单元的延时、面积和驱动能力的不同选择合适的单元来优化电路。

[深入理解dc的read\_verilog和analyze&elaborate区别-zjli1984](http://zjli1984.lofter.com/post/1cc905c9_10269fc0)

[DC综合的一些基本概念](https://blog.csdn.net/sinat_29862967/article/details/108286059)

[DC逻辑综合流程 - 简书](https://www.jianshu.com/p/7e36a639c11e)

[DC lab4和lab6略解 - 墨魂](https://mohun-8052.github.io/2022/05/04/DC-lab4%E5%92%8Clab6%E7%95%A5%E8%A7%A3/#3-%E7%8E%AF%E5%A2%83%E5%B1%9E%E6%80%A7%E7%BA%A6%E6%9D%9F) 好东西

使用Design Compiler进行综合得到的面积单位通常是以[绝对面积与相对面积：Design Compiler中的report\_area\_icsoc的博客-CSDN博客](https://blog.csdn.net/icsoc/article/details/50700892)

### Read_design
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230512091858.png)

#### filelist

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230525224033.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230525224402.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230525222340.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230525222400.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230525222458.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230525222538.png)

### compile
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527101621.png)

### optimization
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527101236.png)

### guide_path
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230512091954.png)

```
list_libs
```

```
help -verbose *clock ：列出与*clock有关的选项

create_clock -help ：查看create_clock这个命令的简单用法

man  create_clock ：查看create_clock这个命令的详细信息

**printvar  Mibrary ：查看 Mibrary这个变量的内容**

man target_library ：查看target_library这个命令的详细信息
```


### install
[安装design compiler的教程\_21岁的彭于晏的博客-CSDN博客](https://blog.csdn.net/DO_NOT_LOVE_ME/article/details/105899207)
[分享一个本人搭建的EDA虚拟机平台 - EDA资源使用讨论 - EETOP 创芯网论坛 (原名：电子顶级开发网) -](https://bbs.eetop.cn/thread-906855-1-1.html)
[EDA软件安装教程，包括RHEL7.8、cadence软件、calibre软件 - EDA资源使用讨论 - EETOP 创芯网论坛 (原名：电子顶级开发网) -](https://bbs.eetop.cn/thread-897274-1-1.html)

### save_files
[DC综合后处理（查看生成的网表和报告）\_dc时序报告怎么看\_北方爷们的博客-CSDN博客](https://blog.csdn.net/sinat_29862967/article/details/115113829)

### saif 文件
cmd 输入

```bash
vcd2saif input zoomout.vcd -output zoomout.saif
```

### save and load state

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230528223524.png)

###  POWER
只优化功耗
```
set compile_power_opto_only true
```

### constrians
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230531100445.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230531100459.png)


