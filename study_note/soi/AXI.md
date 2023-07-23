[Verilog AXI Components Readme [Alex Forencich]](http://alexforencich.com/wiki/en/verilog/axi/readme)
[test\_axi\_cdma.py](https://github.com/alexforencich/verilog-axi/blob/master/tb/axi_cdma/test_axi_cdma.py)

[[UVM]]

AXI 是 ARM AMBA（AdvancedMicrocontroller Bus Architecture，高级微控制器总线架构）的一部分，AXI4 是 2010 年发行的一个 AXI 版本。主要有三种类型：
• AXI4: 一般用于高性能存储映射需求。
• AXI4-Lite: 一般用于简单的，低吞吐量的存储映射（例如控制与状态寄存器之间的映射）。
• AXI4-Stream: 一般用于高速的数据流。
AXI4-Lite 与 AXI4 的总线逻辑基本是一样的，仅端口数量上有差异（AXI4 端口数量更多，对应功能更多），AXI4-Stream 与以上两者差别相对大一些，因此仅梳理 AXI4 以及 AXI4-Stream 实际上就可以对 AXI4 的整体协议框架有一个比较好的认识。

## 信号
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231808460.png)
- AXI 的组件都使用相同的时钟信号 ACLK。输入信号都在 ACLK 的上升沿被采样，输出信号的变化都发生在 ACLK 上升沿之后。
- ARESETn采用低电平复位。需要注意的是ARVALID，AWVALID，WVALID，RVALID，BVALID（这几个信号的含义会在后面说明）在复位时必须保证是处于低电平的。

### 写
AXI4 在进行数据读写时主要使用通道（channel）方式，这种方式保证了读写可以同步进行。
- AWLEN[7:0]：AXI4 支持 INCR 突发类型的突发长度在 1 ~ 256 之间，其余类型的突发长度在 1~16 之间。突发长度 Burst_Length = AxLEN[7:0] + 1，**需要注意的是突发不能超过 4KB 的地址界限，WRAP 类型的突发长度仅能取 2, 4, 8, 或者 16**。
- AWSIZE[2:0]：表示一个 transfer 中的 Bytes 个数。 `3'b000=>1` ，`3'b001=>2` ，`3'b010=>4` ，`3'b011=>8` ，`3'b100=>16`，`3'b101=>32` ，`3'b110=>64` ，`3'b111=>128` 。
- AWBURST[1:0]：定义了突发的类型 `2'b00=>FIXED` ，`2'b01=>INCR`，`2'b10=>WRAP`。
	- FIXED ：固定突发模式，burst 中的每一个 transfer 的地址都相同。
	- INCR：递增突发模式，burst 中的每一个 transfer 的地址都是在前一个传输地址的增量上增加一个 transfer 的大小。例如在一个 transfer 的 Bytes = 4 的突发中，每个 transfer 的传输的地址都是前一个地址加 4。
	- WRAP：环绕突发模式类似于递增突发模式。不同的是，如果达到了地址上限，地址会重新回到起始地址。实际上**这种方式和数据结构中的用数组构建的循环链表是很类似的**。

