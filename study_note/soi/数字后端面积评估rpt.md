## 总结


## 具体步骤

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
（潘泽伦的 tcl）
### PIN
手动排 pin
clk 改信号类型
source placement.Tcl

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920151510.png)

### 报告生成
summaryreport
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230920151809.png)



