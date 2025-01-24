### Verilog-A Model
[GitHub - google/skywater-pdk-libs-sky130\_fd\_pr\_reram: SKY130 ReRAM and examples (SkyWater Provided)](https://github.com/google/skywater-pdk-libs-sky130_fd_pr_reram/tree/main)

### 仿真 log
#### 参考
[skywater-pdk-libs-sky130_fd_pr_reram/examples/1T1R/schematic.png at main · google/skywater-pdk-libs-sky130_fd_pr_reram (github.com)](https://github.com/google/skywater-pdk-libs-sky130_fd_pr_reram/blob/main/examples/1T1R/schematic.png)
[skywater-pdk-libs-sky130\_fd\_pr\_reram/docs/technology\_specifications.rst at main · google/skywater-pdk-libs-sky130\_fd\_pr\_reram · GitHub](https://github.com/google/skywater-pdk-libs-sky130_fd_pr_reram/blob/main/docs/technology_specifications.rst)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202405081203302.png)
[A SPICE Compact Model of Metal Oxide Resistive Switching Memory With Variations](https://ieeexplore.ieee.org/document/6296677)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202405081203873.png)

#### 过程
##### forming
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202405081207027.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202405081206151.png)

V：2.2-3.1  w：1000ns
R：3M - 400

##### reset/set
[A SPICE Compact Model of Metal Oxide Resistive Switching Memory With Variations](https://ieeexplore.ieee.org/document/6296677)
和 Vreset 的幅值以及脉冲宽度成正比
文中取：3V，-2.4V，-2.7V，-3V
		   50ns，500ns，5us

##### 结果
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202405081212940.png)
