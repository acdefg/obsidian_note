![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230317204925.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230317211543.png)


### 综合
[Tcl与Design Compiler （三）——DC综合的流程 - IC_learner - 博客园](https://www.cnblogs.com/IClearner/p/6618992.html)
[Site Unreachable](https://blog.csdn.net/qq_40223983/article/details/96426938)

### 后仿

```verilog
vcs-sdf min:top.i_test.:test.sdf
```
-sdf   min|typ|max:instance_name:file.sdf
The min|typ|max notation is used to represent path delays in SDF files. The min delay represents the minimum delay that can occur on a path, while the max delay represents the maximum delay that can occur on a path.

### sdf 文件反标
#### 方法一
在 makefile 中调用, 使用如下命令：
```
vcs +neg_tchk -negdelay -sdf min|typ|max:instance_name:file.sdf
```
启用 SDF 反标。在 file.sdf 中指定的最小值、类型或最大值中的一种，在实例 instance_name 上进行反标。

#### 方法二 $sdf_annotate
使用$sdf_annotate 将 SDF 文件反标到网表中：

```
$sdf_annotate ("sdf_file"[, module_instance] [,"sdf_configfile"][,"sdf_logfile"][,"mtm_spec"] 
[,"scale_factors"][,"scale_type"]);
```
sdf_file: 指定指向 SDF 文件的路径;
module_instance: 调用$sdf_annotate 模块实例的范围。
sdf_configfile: 指定 SDF 配置文件。
sdf_logfile: 指定 SDF log 文件，可以使用+sdfverbose 显示所有的 sdf 反标错误。
mtm_spec: 指定哪一种延迟类型，通常有三种 min：typ：max，它的可能值是"MINIMUM", "TYPICAL", "MAXIMUM", or "TOOL_CONTROL"（默认值）。在仿真器读入 SDF 的时候，要指定使用哪一组。避免出现指定的组的时序信息不存在的情况。
scale_factors: 指定 min：typ：max 的缩放因子，默认为三个正实数“1.0：1.0：1.0”。
scale_type: 指定 SDF 文件中在缩放前使用的延迟值。它可能的值是“FROM_TYPICAL”、“FROM_MIMINUM”、“FROM_MAXIMUM”和“FROM_MTM”（默认）;

##### 在 tb 中加载 sdf 文件
```
`ifdef SDF
initial
begin
  $sdf_annotate("../../rtl/post_sim/file.sdf",tb,,"sdf.log",);
end
`endif
```
[芯片后仿及SDF反标 - 知乎](https://zhuanlan.zhihu.com/p/439180974)
### EDA
[数字IC设计全流程介绍 - 知乎](https://zhuanlan.zhihu.com/p/85063131)

#### vcs+verdi

```verilog
vcs -timescale=1ns/1ns  \ #设置仿真精度
    -sverilog           \ #Systemverilog的支持
    +v2k                \ #兼容verilog 2001 以前的标准
    -Mupdate            \ #只编译有改动的.v文件
    -f ***.f            \ #添加.f文件里的源码
##  -o simv             \ #默认编译后产生可执行文件为simv，可修改文件名，一般不使用
    -R                  \ #编译后立即运行./simv文件
    -l ***.log          \ #编译信息存放在.log中，也就是出现在终端上的所有信息
    -P  ***/verdi/share/PLI/VCS/LINUX/novas.tab \   #调用verdi的库，也就是在tb文件中添加几行代码所需要的文件路径
        ***/verdi/share/PLI/VCS/LINUX/pli.a
```
-   -fsdb : 仿真过程同时生成 fsdb 格式的波形
-   -full64 : 匹配64位服务器系统
-   -f : 读取仿真文件
-   -R : 编译后自动运行

```verilog
verdi   -sv                 \ #Systemverilog 的支持
        +v2k                \ #兼容 verilog 2001 以前的标准
        -f ***.f            \ #添加.f 文件里的源码
	-ssf tb_top.fsdb    \ #加载 fsdb 波形，tb.sv 中产生的波形名字有关
        -nologo               #打开界面时不出现 logo
```

##### sdf
```shell
vcs +neg_tchk -negdelay -sdf min|typ|max:instance_name:file.sdf
```

[Site Unreachable](https://blog.csdn.net/JasonFuyz/article/details/107508893)  --这个查看和添加波形教程不错