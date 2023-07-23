[Verilog AXI Components Readme [Alex Forencich]](http://alexforencich.com/wiki/en/verilog/axi/readme)
[test\_axi\_cdma.py](https://github.com/alexforencich/verilog-axi/blob/master/tb/axi_cdma/test_axi_cdma.py)

[[UVM]]

AXI 是 ARM AMBA（AdvancedMicrocontroller Bus Architecture，高级微控制器总线架构）的一部分，AXI4 是 2010 年发行的一个 AXI 版本。主要有三种类型：
• AXI4: 一般用于高性能存储映射需求。
• AXI4-Lite: 一般用于简单的，低吞吐量的存储映射（例如控制与状态寄存器之间的映射）。
• AXI4-Stream: 一般用于高速的数据流。