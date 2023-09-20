### init_design
Innovus
Import design Lef 
VDD、VSS、MMMC
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920145143.png)
综合库用的 smic40ll

### floorplan
rram_top 根据综合面积的 1.5 倍手动设置，AXI 使用自动生成的值
ratio：0.9 utilization：0.7
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920145211.png)

### power+route
power -> connect global nets
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920145551.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920145748.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920145952.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920150051.png)

route -> special route
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920150103.png)

source init.tcl
source power.Tcl

排 pin
clk 改信号类型

save design  .enc

source placement.Tcl（这个 tcl 有点问题，没设 contrainmode 吧）

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920151510.png)

summaryreport
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920151809.png)

### ref
[基于innovus的全加器数字芯片物理实现(step by step) - 知乎](https://zhuanlan.zhihu.com/p/56259682)


### 面积评估
[计算cell block的面积 - 简书](https://www.jianshu.com/p/17fd59369383)

