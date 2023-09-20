## 总结

| 单位：um² | AXI_32x32 | AXI_128x128 | AXI_256x256 | AXI_512x512 | rram_top  |
| --------- | --------- | ----------- | ----------- | ----------- | --------- |
| 综合面积  | 1701.97   | 3792.10     | 6611.48     | 12349.64    | 97073.40  |
| 后端面积  | 2440.75   | 5422.14     | 9452.85     | 17651.29    | 138021.20 |
| density  |0.72726 |     0.72006         | 0.72268           | 0.73423            |   0.80775        |

## 具体设置

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



