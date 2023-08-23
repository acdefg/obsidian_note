### 特点
- 只需要两条总线；
- 没有严格的波特率要求，例如使用RS232，主设备生成总线时钟；
- 所有组件之间都存在简单的主/从关系，连接到总线的每个设备均可通过唯一地址进行软件寻址；
- I2C 是真正的多主设备总线，可提供仲裁和冲突检测；
- 最大主设备数：无限制；
- 最大从机数：理论上是 127。
### 模式
1、标准模式：Standard Mode=100 Kbps
2、快速模式：Fast Mode=400 Kbps
3、高速模式：High speed mode=3.4 Mbps
4、超快速模式：Ultra fast mode=5 Mbps

### 硬件
I2C 协议仅需要 SDA 和 SCL 两个引脚。SDA 是串行数据线的缩写，而 SCL 是串行时钟线的缩写。这两条数据线需要接上拉电阻。
I2C 总线（SDA，SCL）内部都使用漏极开路驱动器（开漏驱动），因此 SDA 和 SCL 可以被拉低为低电平，但是不能被驱动为高电平，所以每条线上都要使用一个上拉电阻，默认情况下将其保持在高电平；
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308221559901.png)

[一文搞懂I2C通信 - 知乎](https://zhuanlan.zhihu.com/p/282949543) 一些关于上拉电阻的问题

### 数据传输协议

[I2C介绍及verilog实现（主机/从机可综合）\_i2c verilog代码\_swear蛋的博客-CSDN博客](https://blog.csdn.net/weixin_45863605/article/details/121730144) 这个写的很清楚

1. **总线拓扑**： I2C总线通常由两根线构成：
    - **SDA（Serial Data Line）**：用于传输数据，是双向的。
    - **SCL（Serial Clock Line）**：用于传输时钟信号，由主机控制。

1. **主从关系**： I2C 通信中存在主设备和从设备。主设备（通常是微控制器或处理器）控制总线并发起通信，而从设备则被动地响应主设备的请求。
    
2. **起始和停止条件**：
    - 通信始于主设备发送一个起始条件：S（Start）信号。
    - 通信结束于主设备发送一个停止条件：P（Stop）信号。
3. **地址传输**： 主设备通过发送从设备的唯一7位或10位地址来选择要与之通信的从设备。通常，第一位是从设备的地址，后续位用于指定读写操作。一个I2C事务可以包含多个从设备的地址，使主设备能够与多个设备通信。
    
4. **数据传输**： 数据传输是通过在时钟信号的边缘来同步传输的。数据线（SDA）上的每一位数据都在时钟线（SCL）的每个时钟脉冲上发生变化。通信可以是字节级的，主设备发送一个字节，然后从设备回应一个确认信号（ACK/NACK）。
    
5. **ACK/NACK**：
    - 当主设备发送一个字节后，接收设备（从设备）会回应ACK（应答）信号表示成功接收。
    - 如果接收设备无法接收或处理数据，它会回应 NACK（非应答）信号。

主设备通过将 SDA 线从高电平切换到低电平，再将 SCL 线从高电平切换到低电平，来向每个连接的从机发送启动条件。
**_开始位_*需要满足的条件：SCL 高电平时，SDA 由高变低；**_停止位_**需要满足的条件：SCL 为高电平时，SDA 由低变高

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308231343957.png)
主设备向每个从机发送要与之通信的从机的 7 位或 10 位地址，以及相应的读/写位
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308231344319.png)

如果地址匹配，则从设备通过将 SDA 线拉低一位以表示返回一个 ACK 位。
如果来自主设备的地址与从机自身的地址不匹配，则从设备将 SDA 线拉高，表示返回一个 NACK 位。
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308231344974.png)
主设备发送或接收数据到从设备
**SDA 电平变化不能在 SCL 为高电平时进行，否则可能会被误认为成开始位或者停止位。因此 SDA 只能在 SCL 低电平时改变。**
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308231345045.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308231345939.png)

在传输完每个数据帧后，接收设备将另一个 ACK 位返回给发送方，以确认已成功接收到该帧
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308231346507.png)

为了停止数据传输，主设备将 SCL 切换为高电平，然后再将 SDA 切换为高电平
