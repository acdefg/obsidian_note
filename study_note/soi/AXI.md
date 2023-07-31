[Verilog AXI Components Readme [Alex Forencich]](http://alexforencich.com/wiki/en/verilog/axi/readme)
[test\_axi\_cdma.py](https://github.com/alexforencich/verilog-axi/blob/master/tb/axi_cdma/test_axi_cdma.py) 
[AMBA总线协议（五）—— AXI3 协议接口信号介绍1\_axi3接口\_摆渡沧桑的博客-CSDN博客](https://blog.csdn.net/vivid117/article/details/110871257)    ----------接口信号介绍的比较详细
[NoC总线架构拓扑介绍-电子工程专辑](https://www.eet-china.com/mp/a73963.html)   ---------这篇关于总线的整体介绍不错
[[UVM]]
[[AXI——pytest测试]]
## 类型
AXI 是 ARM AMBA（AdvancedMicrocontroller Bus Architecture，高级微控制器总线架构）的一部分，AXI4 是 2010 年发行的一个 AXI 版本。主要有三种类型：
• AXI4: 一般用于高性能存储映射需求。
• AXI4-Lite: 一般用于简单的，低吞吐量的存储映射（例如控制与状态寄存器之间的映射）。
• AXI4-Stream: 一般用于高速的数据流。
AXI4-Lite 与 AXI4 的总线逻辑基本是一样的，仅端口数量上有差异（AXI4 端口数量更多，对应功能更多），AXI4-Stream 与以上两者差别相对大一些，因此仅梳理 AXI4 以及 AXI4-Stream 实际上就可以对 AXI4 的整体协议框架有一个比较好的认识。

## 概念
先搞清楚几个概念 Transaction、Burst、Beat（Transfer）：
- Transaction：一个 AXI Master 启动一个 Transaction 来与一个 AXI Slave 通信，一般情况下 Transaction 在多个通道上进行 Master 与 Slave 之间的信息交换，这一整套的信息交换构成了AXI Transaction。
- Burst：是一种根据**单个地址**完成多个数据项传输的过程，每一个传输的数据项都被称为 Beat（Transfer）。由于只有一个地址传输，**突发中每个 Beat （Transfer）的地址都是基于传输类型 (INCR、FIXED或WRAP) 计算得到的**。
- Beat（Transfer）：是AXI突发中的单个数据传输。

简单来说，AXI Transaction 就是传输一段数据（AXI burst）所需要的一整套操作，而 AXI burst 就是待传数据，AXI burst 由 AXI Beats 组成，一个 Beat 就是一个 transfer。

## 通道
AXI 协议定义了 5 条通道：
其中 2 条用于读取传输事务
- 读地址
- 读数据
另 3 条用于写入传输事务
- 写地址
- 写数据
- 写响应
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231842804.png)
任一通道上发射的每一条数据都称为一次传输 (transfer)。当 VALID 和 READY 信号均居高不下并且时钟存在上升沿时，就会发生传输。

### 写通道
AXI4 的写过程主要分为三个通道：
Write address channel：写地址通道，用来写地址以及控制信息，包含写地址信号图中的信号。
Write data channel：写数据通道，包含写数据信号图中的信号。
Write response channel：写响应通道，包含写响应信号图中的信号。
这三个通道相互独立，信息传递之间互不干扰
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231834454.png)

### 读通道
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231836803.png)

## 信号
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230727025438.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231808460.png)
- AXI 的组件都使用相同的时钟信号 ACLK。输入信号都在 ACLK 的上升沿被采样，输出信号的变化都发生在 ACLK 上升沿之后。
- ARESETn采用低电平复位。需要注意的是ARVALID，AWVALID，WVALID，RVALID，BVALID（这几个信号的含义会在后面说明）在复位时必须保证是处于低电平的。

### 写地址
AXI4 在进行数据读写时主要使用通道（channel）方式，这种方式保证了读写可以同步进行。
- AWLEN[7:0]：AXI4 支持 INCR 突发类型的突发长度在 1 ~ 256 之间，其余类型的突发长度在 1~16 之间。突发长度 Burst_Length = AxLEN[7:0] + 1，**需要注意的是突发不能超过 4KB 的地址界限，WRAP 类型的突发长度仅能取 2, 4, 8, 或者 16**。
- AWSIZE[2:0]：表示一个 transfer 中的 Bytes 个数。 `3'b000=>1` ，`3'b001=>2` ，`3'b010=>4` ，`3'b011=>8` ，`3'b100=>16`，`3'b101=>32` ，`3'b110=>64` ，`3'b111=>128` 。
- AWBURST[1:0]：定义了突发的类型 `2'b00=>FIXED` ，`2'b01=>INCR`，`2'b10=>WRAP`。
	- FIXED ：固定突发模式，burst 中的每一个 transfer 的地址都相同。
	- INCR：递增突发模式，burst 中的每一个 transfer 的地址都是在前一个传输地址的增量上增加一个 transfer 的大小。例如在一个 transfer 的 Bytes = 4 的突发中，每个 transfer 的传输的地址都是前一个地址加 4。
	- WRAP：环绕突发模式类似于递增突发模式。不同的是，如果达到了地址上限，地址会重新回到起始地址。实际上**这种方式和数据结构中的用数组构建的循环链表是很类似的**。

### 写数据
WSTRB[n:0]：用来指示总线上哪些字节的数据是有效的，WSTRB 的每一个 bit 位对应 WDATA 的每一个字节，即 WSTRB[n]对应于 WDATA[8n+7:8n]。

### 写响应
- BRESP[1:0]：定义了突发的类型 `2'b00=>OKAY` ，`2'b01=>EXOKAY` , `2'b10=>SLVERR`，`2'b11=>DECERR`。
	- OKAY：正常访问成功，还可以指示独占访问失败。
	- EXOKAY：指示独占访问的部分已成功。
	- SLVERR：主机正常发送但是从机没有正常接收。
	- DECERR：主机没找到从机。

### 读地址
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231836793.png)

### 读数据
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231837463.png)

## 过程

### 读过程
AXI 读传输事务需要在 2 条读取信道上发生多次传输。
- 首先，地址读通道 (Address Read Channel) 从主设备 (Master) 发送到从设备 (Slave)，以便设置地址和部分控制信号。  
- 然后，此地址的数据通过读数据通道 (Read data channel) 从从设备发送到主设备。  
请注意，根据下图所示，每个地址中可发生多次数据传输。此类型的传输事务称为突发 (burst)。
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231846679.png)

### 写过程
AXI 写入传输事务需要在 3 条读取信道上存在多次传输。
- 首先，写地址通道 (Address Write Channel) 从主设备发送到从设备，以便设置地址和部分控制信号。
- 然后，此地址的数据通过写数据通道 (Write data channel) 从主设备发射到从设备。
- 最后，写入响应通过写响应通道 (Write Response Channel) 从从设备发送到主设备，以指示传输是否成功。

## 注意
值得注意的是：
- 断言 VALID (AxVALID/xVALID) 信号时，它必须保持处于已断言状态直至从设备发出 AxREADY/xREADY 断言后出现上升时钟沿为止。
- 发送信息的 AXI 接口的 VALID 信号不得从属于接收该信息的 AXI 接口的 READY 信号。

但是，READY 信号的状态可从属于 VALID 信号
- 写响应必须始终位于所属的写入传输事务中最后一次写入传输之后
- 读数据必须始终位于数据相关的地址之后
- 从设备必须等待发出 ARVALID 和 ARREADY 断言后，才能发出 RVALID 断言以指示该有效数据可用

### 类型

AXI-DMA：实现从 PS 内存到 PL 高速传输高速通道 AXI-HP<---->AXI-Stream 的转换
AXI-FIFO-MM2S：实现从PS内存到PL通用传输通道AXI-GP<----->AXI-Stream的转换
AXI-DataMover：实现从PS内存到PL高速传输高速通道AXI-HP<---->AXI-Stream的转换，只不过这次是完全由PL控制的，PS是完全被动的。
AXI-VDMA：实现从PS内存到PL高速传输高速通道AXI-HP<---->AXI-Stream的转换，只不过是专门针对视频、图像等二维数据的。
除了上面的还有一个AXI-CDMA IP核，这个是由PL完成的将数据从内存的一个位置搬移到另一个位置，无需CPU来插手。

