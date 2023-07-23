[Verilog AXI Components Readme [Alex Forencich]](http://alexforencich.com/wiki/en/verilog/axi/readme)
[test\_axi\_cdma.py](https://github.com/alexforencich/verilog-axi/blob/master/tb/axi_cdma/test_axi_cdma.py)

[[UVM]]

AXI 是 ARM AMBA（AdvancedMicrocontroller Bus Architecture，高级微控制器总线架构）的一部分，AXI4 是 2010 年发行的一个 AXI 版本。主要有三种类型：
• AXI4: 一般用于高性能存储映射需求。
• AXI4-Lite: 一般用于简单的，低吞吐量的存储映射（例如控制与状态寄存器之间的映射）。
• AXI4-Stream: 一般用于高速的数据流。
AXI4-Lite 与 AXI4 的总线逻辑基本是一样的，仅端口数量上有差异（AXI4 端口数量更多，对应功能更多），AXI4-Stream 与以上两者差别相对大一些，因此仅梳理 AXI4 以及 AXI4-Stream 实际上就可以对 AXI4 的整体协议框架有一个比较好的认识。

### 信号
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231808460.png)


### 写
AXI4 在进行数据读写时主要使用通道（channel）方式，这种方式保证了读写可以同步进行。
