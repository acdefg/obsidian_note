### concepts
[逻辑综合——工艺库\_report\_lib\_沧海一升的博客-CSDN博客](https://blog.csdn.net/qq_21842097/article/details/111566443)
目标库包含各个门级单元的行为、引脚、面积、时序信息、功耗方面的参数等信息。DC 在综合时就是根据目标库给出的单元电路的时序信息来计算延迟，并根据各个单元的延时、面积和驱动能力的不同选择合适的单元来优化电路。

[深入理解dc的read\_verilog和analyze&elaborate区别-zjli1984](http://zjli1984.lofter.com/post/1cc905c9_10269fc0)

### Read_design
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230512091858.png)

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
