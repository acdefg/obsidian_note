down:: [[AXI——pytest测试]]
## ref
[Verilog AXI Components Readme [Alex Forencich]](http://alexforencich.com/wiki/en/verilog/axi/readme)
[test\_axi\_cdma.py](https://github.com/alexforencich/verilog-axi/blob/master/tb/axi_cdma/test_axi_cdma.py) 
[AXI4协议学习：架构、信号定义、工作时序和握手机制-CSDN博客](https://blog.csdn.net/lum250/article/details/120912567)  --握手信号依赖
[AXI4（AXI-full）总线详细介绍\_axi4总线-CSDN博客](https://blog.csdn.net/weixin_46136963/article/details/118003005)
[AMBA总线协议（五）—— AXI3 协议接口信号介绍1\_axi3接口\_摆渡沧桑的博客-CSDN博客](https://blog.csdn.net/vivid117/article/details/110871257)    ----------接口信号介绍的比较详细
[NoC总线架构拓扑介绍-电子工程专辑](https://www.eet-china.com/mp/a73963.html)   ---------这篇关于总线的整体介绍不错
[[UVM]]
[[AXI——pytest测试]]

[AMBA总线协议\_apple\_ttt的博客-CSDN博客](https://blog.csdn.net/apple_53311083/category_12416130.html) 专栏深入浅出 AXI 协议部分讲的非常详细

## 特点
AXI 是一种面向高性能、高带宽、低延迟的片内总线，它具有如下的几个特点：

        （1）总线的地址/控制和数据通道是分离的；
        （2）支持不对齐的数据传输；
        （3）支持突发传输，突发传输过程中只需要首地址；
        （4）具有分离的读/写数据通道；
        （5）支持显著传输访问和乱序访问；
        （6）更加容易进行时序收敛。


## 类型
AXI 是 ARM AMBA（AdvancedMicrocontroller Bus Architecture，高级微控制器总线架构）的一部分，AXI4 是 2010 年发行的一个 AXI 版本。主要有三种类型：
• AXI4: （For high-performance memory-mapped requirements），主要面向高性能地址映射通信的需求，是面向地址映射的接口，允许最大 256 轮的数据突发传输。
AXI4 协议支持突发传输，主要用于处理器访问存储器等需要指定地址的高速数据传输场景
• AXI4-Lite: （For simple, low-throughput memory-mapped communication），是一个轻量级的地址映射单次传输接口，占用很少的逻辑单元。适用于吞吐量较小的地址映射通信总线。
AXI-Stream 接口则像 FIFO 一样，数据传输时不需要地址，在主从设备之间直接连续读写数据，主要用于如视频、高速 AD、PCIe、DMA 接口等需要高速数据传输的场合。
• AXI4-Stream: For high-speed streaming data.）面向高速流数据传输；去掉了地址项，允许无限制的数据突发传输规模。
AXI-Lite为外设提供单个数据传输，主要用于访问一些低速外设中的寄存器。
>AXI4-Lite 与 AXI4 的总线逻辑基本是一样的，仅端口数量上有差异（AXI4 端口数量更多，对应功能更多），AXI4-Stream 与以上两者差别相对大一些，因此仅梳理 AXI4 以及 AXI4-Stream 实际上就可以对 AXI4 的整体协议框架有一个比较好的认识。

> [!info] 注意
> 
需要注意的是，AXI4：面向地址映射的接口，在单地址传输的情况下最大允许256个时钟周期的数据突发长度；AXI4-Lite：一个轻量级的地址映射单次传输接口，占用较少的资源；AXI4-Stream：去掉了地址传输的功能，允许无限制的数据突发传输，无需考虑地址映射。
## 概念
先搞清楚几个概念 Transaction、Burst、Beat（Transfer）：
- Transaction：一个 AXI Master 启动一个 Transaction 来与一个 AXI Slave 通信，一般情况下 Transaction 在多个通道上进行 Master 与 Slave 之间的信息交换，这一整套的信息交换构成了AXI Transaction。
- Burst：是一种根据**单个地址**完成多个数据项传输的过程，每一个传输的数据项都被称为 Beat（Transfer）。由于只有一个地址传输，**突发中每个 Beat （Transfer）的地址都是基于传输类型 (INCR、FIXED或WRAP) 计算得到的**。
- Beat（Transfer）：是AXI突发中的单个数据传输。

简单来说，AXI Transaction 就是传输一段数据（AXI burst）所需要的一整套操作，而 AXI burst 就是待传数据，AXI burst 由 AXI Beats 组成，一个 Beat 就是一个 transfer。

Asize：一次传输字节大小 
Asize^2 x8 < Wdata
Alen：一次 transfer 传输数据次数

#### 突发类型

AXI 支持三种突发类型：
  FIXED（AxBURST[1:0]=0b00）：固定突发模式，每次突发传输的地址相同；
  INCR（AxBURST[1:0]=0b01）：增量突发模式，突发传输地址递增，递增与突发尺寸相关。
  WRAP（AxBURST[1:0]=0b10）：回卷突发模式，突发传输地址可溢出性递增，突发长度仅支持 2,4,8,16。地址空间被划分为长度【突发尺寸 awlen x 突发长度 awsize】的块，传输地址不会超出起始地址所在的块，一旦递增超出，则回到该块的起始地址。

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
任一通道上发射的每一条数据都称为一次传输 (transfer)。**当 VALID 和 READY 信号均居高不下并且时钟存在上升沿时**，就会发生传输。
#### 握手信号
AXI 握手协议
  AXI 协议的五个通道都有各自的 VALID/READY 握手信号对。每个通道握手信号对的名称如下图所示：
  ![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202402221932280.png)
  AXI4 所采用的是一种 READY，VALID 握手通信机制。即主从模块进行数据通信前，根据操作对各所用到的数据、地址通道进行握手。主要操作包括传输发送者 A 等到传输接受者 B 的 READY 信号后，A 将数据与 VALID 信号同时发送给 B。
  简单来说主从双方进行数据通信前，有一个握手的过程。传输源产生 VLAID 信号来指明何时数据或控制信息有效。而目地源产生 READY 信号来指明已经准备好接受数据或控制信息。传输发生在 VALID 和 READY 信号同时为高的时候，如下图所示：
  ![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202402221932403.png)
                        
原文链接：https://blog.csdn.net/weixin_46136963/article/details/118003005
### 写通道
AXI4 的写过程主要分为三个通道：
Write address channel：写地址通道，用来写地址以及控制信息，包含写地址信号图中的信号。
Write data channel：写数据通道，包含写数据信号图中的信号。
Write response channel：写响应通道，包含写响应信号图中的信号。
这三个通道相互独立，信息传递之间互不干扰
	![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231834454.png)

### 读通道
从上图中可以看到，在一个读传输过程中，主机首先在读地址通道给出读地址和控制信号，然后从机由读数据通道返回读出的数据。另外我们需要注意的是，这是一次突发读操作，主机只给出一个地址，从该地址连续突发读出四个数据。
 ![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307231836803.png)

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
WSTRB[n:0]：用来指示总线上哪些字节的数据是有效的，WSTRB 的每一个 bit 位对应 WDATA 的每一个字节，比如 WSTRB[n]对应于 WDATA[8n+7:8n]。

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

### GITHUB 相关项目
[AMBA\_AXI\_AHB\_APB/slides/16\_amba\_axi\_intro.pdf at master · adki/AMBA\_AXI\_AHB\_APB · GitHub](https://github.com/adki/AMBA_AXI_AHB_APB/blob/master/slides/16_amba_axi_intro.pdf)  --slides + code
[GitHub - pulp-platform/axi: AXI SystemVerilog synthesizable IP modules and verification infrastructure for high-performance on-chip communication](https://github.com/pulp-platform/axi)
[verilog-axi/rtl/axi\_ram.v at master · alexforencich/verilog-axi · GitHub](https://github.com/alexforencich/verilog-axi/blob/master/rtl/axi_ram.v)
[https://mp.weixin.qq.com/s/W532h0fyOe1A6Yxw2duw-g](https://mp.weixin.qq.com/s/W532h0fyOe1A6Yxw2duw-g)

