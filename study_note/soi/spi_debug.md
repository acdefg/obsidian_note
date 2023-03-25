![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230317204925.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230317211543.png)


### 综合
[Tcl与Design Compiler （三）——DC综合的流程 - IC_learner - 博客园](https://www.cnblogs.com/IClearner/p/6618992.html)
[Site Unreachable](https://blog.csdn.net/qq_40223983/article/details/96426938)
[DC中常用到的命令（示例）总结 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1665000)

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
[VCS与Verdi的联合仿真 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1669477)  比较详细
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

##### 四、VCS+Verdi 如何 dump 波形

在 dump 波形时会用到那些命令，解决的是生成 fsdb 波形的问题，为了生成.fsbd 格式的文件，可以使用 verilog 波形函数，也可以使用 ucli/tcl 接口： 

###### （一）使用 Verilog 系统函数 

作为小白，我觉得这种方式很友好，通过 Verilog 的 PLI 接口实现，在 tb 中添加两个函数：

```verilog
initial begin
$fsdbDumfile(“uart.fsdb”);     //指定生成的 fsdb 文件的文件名
$fsdbDumpars(0,uart_byte_tx_tb); //指定 dump 的层次，0 表示存储所有的 wave，tb 为起始层
end 
```

###### （二）、使用 ucli/tcl 接口 

使用 ucli/tcl 接口时无需在 tb 中调用与 fsdbDumpvars()函数，仅需在脚本中进行设置即可。在运行仿真时，打开 ucli 接口，通过 Tcl 脚本对 fsdb 进行设置，设置 fsdb 文件的文件名，设置 fsdb 文件的集成类型和起始文件： 

```
global env  
```

tcl 脚本引用环境变量，Makefile 中通过 export 定义 

```
fsdbDumpfile "$env(demo_name).fsdb"
```

设置波形文件名，受环境变量 env(demo_name)控制 

demo_name 在 makefile 中使用 exportdemo_name=demo_fifo 

```
fsdbDumpvars 0 "tb_top"     
```
 
设置波形的顶层和层次，表示将 tb_top 作为顶层，Dump 所有层次

```
run
```

```
+fsdb+autoflush 
```

+fsdb+f+autoflush：用于开启一边仿真以一边 Dump 波形的功能，在不开启该功能时，运行完仿真之后，未退出命令行，直接在新终端中启动 Verdi 调用波性文件的话是一个用文件，没有波形，这是因为只有在结束仿真之后，波形才会 Dump 为静态文件供 verdi 调用，没有出现波形的原因是此时的.fsdb 只是一个空文件，波形还未 Dump，如下图所示：

此时可以在仿真的命令行中键入：fsdbDumpflush，启动波形 Dump，在另一个终端中启动 verdi 加载波形，波形正常加载：

verdi 优于 modelsim 也正是因此，通过 tcl 语言的控制，每次设置 run 时间，不断的加载仿真波形，十分方便！

##### **前仿选项**
-   **+nospeicy**  
    在仿真时忽略库文件中指定的延时。
-   **+delay_mode_zero**  
    将标准库单元中定义的延时替换为0。testbench中的 #延时也都被消除。
-   **+notimingcheck**时序检查开关，比如setup/hold/width检查等等，如使用了该option，则仿真时不检查时序，行为类似于RTL仿真。  
    在PR未结束，sdf反标文件还没准备好时，可用该选项忽略延时，可用于功能性的粗略检查。  
    但真正跑后仿真时，不可使用该选项，否则仿真有效性大大降低。

##### **后仿选项**

-   **+sdfverbose**  
    显示所有的sdf反标错误；
-   **+no_notifier**  
    可以关掉时序检查产生的不定态。通过这个命令参数可以使时序检查任务中检测到时序违例后，不影响其参数列表中的notifier的值，从而避免了notifier变化引起udp输出不定态的情况，该命令仅对notifier的值有影响，对于时序检查任务检测到的时序违例不产生任何影响；
-   **+neg_tchk**若要使用负延时检查，在编译设计时必须包含+neg_tchk选项。如果省略此选项，VCS将所有负延迟更改为0。
-   **-negdelay**  
    用于 SDF 文件中有负延迟，如果省略此选项，VCS 将所有负延迟更改为 0。
[芯片后仿及SDF反标 - 知乎](https://zhuanlan.zhihu.com/p/439180974) 上面这段连接

