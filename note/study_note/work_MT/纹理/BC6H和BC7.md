# **BC7 纹理压缩格式**

## **1. BC7 介绍**
BC7 是 DirectX 11 引入的高质量 RGBA 纹理压缩格式，它将 4×4 像素块(16 像素)压缩为 16 字节(128 位)的数据块。主要特点包括：
- 支持 8 种压缩模式(Mode 0-7)
- 每个块可选择 1-3 个子集(Subset)
- 支持 alpha 通道
- 通过分区和插值实现高质量压缩
- 采用有损压缩，但视觉质量优秀

多种采样模式使得 BC7 可以在不同模式下采用不同精度进行压缩编码，多个子集的存储方式使得在被压缩图片出现颜色变化很大的区域时，能够获得更高的压缩精度。

## **2. BC7 所有模式概括**

BC7 的模式是由最低 byte 的最低 bit 位决定的
>例如，采用“x”表示不包含在模式编号中的 bits，
mode 0 的二进制低字节编码为 xxxxxxx1，mode 5 为 xx100000，mode 7 则为 10000000。低字节编码为零的数值为保留值，在 BC7 纹理编码中禁止使用。硬件解码器处理低字节全为 0 的纹素块时，应为所有纹素的所有通道返回 0 值，但允许在 A 通道返回 1.0。

一些基本的概念：
BC7 为 128bit，支持 RGBA 的压缩纹理格式，包含 8 种不同的模式
内部可以支持最多三个子集，每个子集包含 2 个端点（endpoint），用来描述内部颜色变化比较大的压缩区域
对于每个端点值，可以有附加位，分为两种 Shared P bit（SPB），和 per-endpoint P-bit（EPB），SPB 为每个端点值一个（数量等于端点），SPB 的值会加在不同端点的每个通道的最低位（反量化前），EPB 为每个通道一个（数量为端点* 3），EPB 的值会分别加在不同通道的最低位（反量化前）
mode4 和 mode5 支持两个 index table，分别为 2bit 和 3bit 精度，可以通过 index selection bit（ISB） 选择插值时用哪个 index table
partition bit 用于表示压缩数据存储 pattern，即不同 texel 使用哪个子集，以及锚点的位置

以下是 8 种模式的完整对比：


| Abbreviation | Description |
|--------------|-------------|
| M | Mode identifier bits |
| NS | Number of subsets |
| PB | Partition selection bits |
| RB | Rotation bits |
| ISB | Index selection bit |
| CB | Color bits |
| AB | Alpha bits |
| EPB | Endpoint P-bits (all channels) |
| SPB | Shared P-bits |
| IB | Index bits |
| IB2 | Secondary index bits |

*EPB: 端点位数(Endpoint Bits per component)*  
*SPB: 共享精度位(Shared P-bit)*  
*IB: 索引位数(Index Bits)*

## **3. 端点(Endpoint)编码**

BC7 中可以有 3 个子集，每个子集中有个两个端点值，排列方式为：endpoint，subset，channel，如果有 alpha 以第四个通道的形式，跟在颜色值之后

### **(1) 基础端点存储**
端点存储的是颜色值(RGB/RGBA)，每个分量(component)的位数由 EPB 决定：
- Mode 0: 4 位/分量
- Mode 1: 6 位/分量
- Mode 3: 7 位/分量
- ...

### **(2) P-bit 精度扩展**
P-bit(精度位)用于扩展端点精度：
- **独立 P-bit**：每个端点有自己的 P-bit(EPB)
- **共享 P-bit**：两个端点共享一个 P-bit(SPB)

7 位扩展示例(Mode 5):
```
存储值: 6位 + P-bit
实际值 = (存储值 << 1) | P-bit
```
例如存储值 `010110` (44) + P-bit `1` ⇒ `0101101` (45)

### **(3) 增量编码**
端点 1(EP1)通常存储为相对于端点 0(EP0)的差值：
```
EP1 = EP0 + delta
```
delta 值是有符号数，可以节省存储空间。

### **(4) 反量化**
压缩块中端点值的精度不一，在插值前将端点值扩展到 8 bits

```
uint8_t DequantizeBC7Endpoint(uint8_t stored, int bits) {
    return (stored << (8-bits)) | (stored >> (2*bits-8));
}
// 示例：5-bit值17 → 140

```


| 索引位数 | 权重值数量 | 权重计算 |
|----------|------------|----------|
| 2-bit | 4 | W = index × 64 / 3 |
| 3-bit | 8 | W = index × 64 / 7 |
| 4-bit | 16 | W = index × 64 / 15 |


## **4. 锚点(Anchor Point)**
### **(1) 什么是锚点？**
锚点是每个子集(Subset)中的第一个像素位置，它的索引(index)不存储，默认为 0。

### **(2) 为什么需要锚点？**
1. **节省空间**：不存储锚点索引可以节省 IB 位
2. **优化插值**：自然地选择子集内最具代表性的点作为基准

### **(3) 锚点确定规则**
锚点位置由分区模式(Partition)决定：
- **子集 0**：总是(0,0)位置
- **子集 1**：由 partition bits 查表确定
- **子集 2**：由 partition bits 查表确定

常见锚点位置举例：
- 分区模式 5(NS=2)：子集 1 锚点为(2,1)
- 分区模式 10(NS=3)：子集 1 锚点为(1,1)，子集 2 锚点为(2,2)

### **(4) 锚点对索引存储的影响**
因为锚点索引不存储，所以后续像素的索引存储位置需要调整：
```
索引偏移 = IB×(x+4y) - 已跳过的锚点数量
```

## **5. 完整解码流程示例(Mode 1)**
我们以 Mode 1 为例说明完整的解码过程：

### **步骤 1：解析头部**
1. 读取前 4 位确定 Mode：`0b0001` ⇒ Mode 1
2. Mode 1 特征：
   - 2 个子集(NS=2)
   - 6 位端点(EPB=6)
   - 3 位索引(IB=3)
   - 无 P-bit

### **步骤 2：读取分区**
1. 读取接下来的 6 位分区位：`0b101010` ⇒ 分区 42
2. 查分区表得：
   - 子集 0：(0,0)-(3,1)
   - 子集 1：(2,1)-(3,3)
   - 子集 1 锚点：(2,1)

### **步骤 3：解码端点**
1. 读取子集 0 端点：
   - EP0_RGB: 18 位(6×3)
   - EP1_RGB: 18 位(6×3)
1. 读取子集 1 端点：
   - EP2_RGB: 18 位
   - EP3_RGB: 18 位

### **步骤 4：解码索引**
计算各像素索引位置：
- (1,0): offset=3×1-1=2
- (2,0): offset=3×2-1=5
- ...
- (2,1): 锚点，index=0(不存储)
- (3,1): offset=3×7-1=20
- (2,2): offset=3×10-2=28(跳过 1 锚点)

### **步骤 5：插值计算颜色**
对每个像素：
1. 确定所属子集
2. 获取索引值
3. 计算插值颜色：
   ```
   Color = [(7-index)×EP0 + index×EP1]/7
   ```



# **BC6H 纹理压缩格式**

## **1. BC6H 介绍**
BC6H 是 DirectX 11 引入的高动态范围(HDR)纹理压缩格式，专为 FP16 HDR 纹理设计。它将 4×4 像素块(16 像素)压缩为 16 字节(128 位)的数据块。主要特点包括：
- 支持最多两个subsets
- 支持 14 种不同模式
- 支持 signed/unsigned RGB 颜色输入，alpha 通道保持为 1
- 对于保留格式或者错误格式则返回（0，0，0，1.0）
- 不同于 BC7 的 fix-point， BC6H 会将 16bit integer 解释为 16bit half point，插值为非线性

## **2. BC6H 所有模式详细参数表**
![Pasted image 20251115161730](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129225804064.png)
没有列出的位是保留位，

### Mode
![Pasted image 20251115160425](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129225804065.png)
存的是每一种 mode 数字对应的的二进制表示


## **3. 端点(Endpoint)编码详解**

### **(1) 端点存储格式**
mode： 3，7，11，15 只有一个 subset，其余 mode 都有两个
除了 mode3 和 mode30 endpoint 使用两个完整的 endpoint 存储，其他皆采用增量编码：endpoint0+delta 的形式（transformed endpoints）存储
0，1 为 subset 0 的 endpoints，2，3 代表 subset 1 的 endpoints，对于 signed 格式会进行**符号扩展**，对于 signed 格式得到的 endpoint 的值可能为负，一些 endpoint+delat 的结果可能会存在溢出的情况，所以选取 endpoint 的时候需要小心
对于增量编码形式的 endpoint 计算，会用 E0 作为 offset：
`R1 = (R0 + R1) & ((1 << EPB)−1)`
EPB 为 endpoint 的 bit 数
### 反量化
端点数据会被**反量化**，以充分利用比特位，并确保负值范围正确表示为**二进制补码**，便于插值计算
unsigned：
```
if (EPB >= 15)
	unq = x;
else if (x == 0)
	unq = 0;
else if (x == ((1 << EPB)-1))
	unq = 0xFFFF;
else
	unq = ((x << 15) + 0x4000) >> (EPB-1);
```
signed
```
s = 0;
if (EPB >= 16) {
	unq = x;
} else {
	if (x < 0) {
		s = 1;
		x = -x;
	}

	if (x == 0)
		unq = 0;
	else if (x >= ((1 << (EPB-1))-1))
		unq = 0x7FFF;
	else
		unq = ((x << 15) + 0x4000) >> (EPB-1);
	if (s)
		unq = -unq;
}
```
反量化之后，插值将 treate data as 16bits fix-point

### 插值
对于 unsigned：
由于 half float 和 16-bit integer 表示的范围不同，需要对数据进行 remap，即乘以 31/64
```
out = (i * 31) >> 6;
```

对于 signed：
在反量化阶段限制了数据的范围，remap 通过取数据的绝对值，放缩 31/32，再恢复 sign  bit
```
out = i < 0 ? (((-i) * 31) >> 5) | 0x8000 : (i * 31) >> 5;
```

## Index
对于只有一个分区的 block，mode bits 和 endpoints bits 后面会跟随 63bits 的 index bits（从整个 block 的 bit 65 开始），其中每个 index 4 bits，包含一个 implict bit
对于两个分区的 block，从 bit 82 开始，会有 46bits 的 index data，其中每个 index 3bits，包含每个分区 1bit 的 implicit bit
都是 y-major 以及小端存储
关于 implicit bit：对于每个分区的锚点值，对应的权重都是最小的，这里在存储锚点位置的 index bits 的时候会少存 1bit
## **4. 分区(Partition)系统**

分区，权重以及锚点位置和 BC7 的一致
#HW 可以和 BC7 复用同一存储空间

| 索引位数 | 权重值数量 | 权重计算 |
|----------|------------|----------|
| 2-bit | 4 | W = index × 64 / 3 |
| 3-bit | 8 | W = index × 64 / 7 |
| 4-bit | 16 | W = index × 64 / 15 |


## 数据格式和计算简介
### fix point

### integer

### half float

### posit
