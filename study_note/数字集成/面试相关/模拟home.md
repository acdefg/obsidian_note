## 噪声
包括热噪声和闪烁噪声？
热噪声：电子随机运动对导体两端电压影响产生的，与温度成正比
闪烁噪声：也称粉红噪声，由于电荷载流子在材料界面之间随机捕获和释放引起的

## 失配和失调
[[运放失调调整]]
- 失配是指由于工艺或者设计问题使得电路出现不匹配，影响电路性能。
- 运放的失调电压分为随机失调和系统失调
	- 随机失调是由于工艺偏差和版图的失配导致的，是不可完全消除的；
		成因可以是：阈值电压失配、跨导失配、源漏电压及λ失配、工艺版图失配等
	- 随着晶体管面积(WL)的增加，所有的随机失配都减小
	（原理就是：随着晶体管面积增加，相当于多个晶体管并联，这些失调量有正有负，平均下来使得整体失调减小）
- 而系统失调是在没有随机失调的情况下，由于电路的结构导致的，系统失调是可以通过电路结构的优化消除掉的。


### 01 什么是运放的失调电压，它是怎么定义的？

对于一个理想的全差分运放，当差分输入Vin=0时，其差分输出Vout=0，然而在实际运放中这是不可能实现的。举个例子，如果我们有一个真实的运放，当Vin=0时，你会发现其Vout≠0，接着当你慢慢改变Vin的值，终于在某个值，比如Vin=10mV时Vout变为了0，那么此时的Vin值即为运放的输入失调电压，一般用Vos表示。也就是说，你手里的这个实际运放的输入失调电压Vos=10mV。

也就是说，**全差分OP的失调电压可以这么定义：可以使差分输出等于0时对应的差分输入即为失调电压。**

那对于单端运放，失调应该怎么定义呢？书上一般是这么说的：可以使差分输出等于 vdd/2 时对应的差分输入即为失调电压。这里的“vdd/2”不应该做僵化的理解，其实可以认为是不考虑 mismatch 时的输出 DC 值，它是一个基准值，有失配时输出肯定不等于基准值，而能令输出回归到基准值的输入即为失调电压，所以单端运放的定义可以是：可以使输出等于设定的基准值时对应的输入即为失调电压。

### 02 置信区间
mismatch是跑蒙特卡洛仿真，每一次仿真失配的参数都不一样，则每次得到的offset都不一样。跑1000次蒙特卡洛，得到1000个offset，用统计学的方法来处理数据，得到一个sigma值和mean值，那么可以认为实际的电路，offset在[mean +/- 3*sigma]这个区间范围内的可信度有96%左右（具体多少忘了，有3sgima和6sigma的置信区间，可以了解一下）

### 03 电流镜失配
[电流镜的失调及消除 - Analog/RF IC 设计讨论 - EETOP 创芯网论坛 (原名：电子顶级开发网) -](https://bbs.eetop.cn/thread-662292-1-1.html)
[电流镜失配总结 - Analog/RF IC 设计讨论 - EETOP 创芯网论坛 (原名：电子顶级开发网) -](https://bbs.eetop.cn/thread-850890-1-1.html)
Cascode 基本结构讲的不错：
[https://zhuanlan.zhihu.com/p/567596915](https://zhuanlan.zhihu.com/p/567596915)
各种结构的电流镜：
[www.tup.tsinghua.edu.cn/upload/books/yz/095507-01.pdf](http://www.tup.tsinghua.edu.cn/upload/books/yz/095507-01.pdf)
### reference
1、一篇讲的很详细的总结帖
[​一篇关于Analog IC中失调的总结性文章：以运放为例 - chen\_ww1993的日志 - EETOP 创芯网论坛 (原名：电子顶级开发网) -](https://blog.eetop.cn/blog-1615674-6952843.html)

## LDO
[电子电路学习笔记（14）——LDO(低压差线性稳压器)\_ldo电路-CSDN博客](https://blog.csdn.net/qq_36347513/article/details/121019508)
### 分类
PMOS LDO：电流小
NMOS LDO：电流大
PNP LDO：压降小
NPN LDO：负输出？
### 基本结构
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272231476.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272232938.png)
以 PMOS 为例反馈分析：
![650](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272233472.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272233350.png)

### 参数
1. 输入电压范围：稳压器输入端可以输入的电压范围。
2. 输出电压：稳压器输出端的输出电压值。
3. 最大输出电流：稳压器输出端的最大输出电流值。
4. 线性调整率：稳压器输入变化对输出的影响，即在负载一定的情况下，输出电压变化量和输入电压变化量之比。线性调整率越小越好。
5. 负载调整率：是指在给定负载变化下的输出电压的变化，这里的负载变化通常是从无负载到满负载。负载调整率越小越好。
6. 电源纹波抑制比(PSRR)：表示稳压器抑制由输入电压造成的输出电压波动的能力。线性调整率只有在直流电时才需要考虑，但是电源抑制比必须在宽频率范围上考虑。PSRR 是一个用来描述输出信号受电源影响的参量，PSRR 越大，输出信号受到电源的影响越小。
7. 瞬态响应：表示负载电流突变时引起的输出电压的最大变化，它是输出电容及其等效串联电阻和旁路电容的函数。其中输出电容的作用是提高负载瞬态响应的能力，也起到了高频旁路的作用。
8. 静态电流：又叫接地电流，是通路元件的偏流和驱动电流的组合，通常保持尽可能低的水平。静态电流越大，稳压器的效率越低。
9. 最大耗散功率：为了确保 LDO 节点温度不至于过高而损坏，LDO 都必须计算最大耗散功率。LDO 的实际耗散功耗要小于最大耗散功率，否则可能损坏 LDO 芯片。
### reference
[彻底弄明白LDO\_ldo结构-CSDN博客](https://blog.csdn.net/tanguohua_666/article/details/103860320)
[电子电路学习笔记（14）——LDO(低压差线性稳压器)\_ldo电路-CSDN博客](https://blog.csdn.net/qq_36347513/article/details/121019508)

## DCDC 电路原理
[DC-DC开关电源 拓扑结构（BUCK BOOST BUCK-BOOST）电路\_buck-boost电路-CSDN博客](https://blog.csdn.net/qq_41451521/article/details/100925249)
### buck
降压
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272250235.png)
图中器件 T 为  N-mos 管
当 PWM 驱动高电平使得 NMOS 管 T 导通的时候，忽略 MOS 管的导通压降，等效如图 2，电感电流呈线性上升，MOS 导通时电感正向伏秒为：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272251498.png)
当 PWM 驱动低电平的时候，MOS 管截止，电感电流不能突变，经过续流二极管形成回路（忽略二极管电压），给输出负载供电，此时电感电流下降，如下图 3 所示，MOS 截止时电感反向伏秒为：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272251711.png)

**什么是电感的伏秒平衡呐？**
处于稳定状态的电感，开关导通时间(电流上升段)的伏秒数须与开关关断(电流下降段)时的伏秒数在数值上相等，尽管两者符号相反。这也表示，绘出电感电压对时间的曲线，导通时段曲线的面积必须等于关断时段曲线的面积。
(电容两极板上的电压不能突变？电感两端电流不能突变？电子总量？)
公式推导：
[硬件设计:电源设计--DC/DC工作原理及芯片详解\_dcdc工作原理-CSDN博客](https://blog.csdn.net/chenhuanqiangnihao/article/details/110680989)
可得 dI(t)=1/L∫V(t)dt, 对于稳态的一个功率变换器，其应保证在一个周期内电感中的能量充放相等，反映在 V-t 图中即表示在一个周期内其面积之和为 0，所以得出电感电压伏秒平衡定律

### boost
Boost 升压型电路拓扑，有时又称为 step-up 电路，其典型的电路结构如下图 4 所示：

　　[![](https://i-blog.csdnimg.cn/blog_migrate/60b054f117fb409bf65f5ce55a6a78c9.png)](http://www.elecfans.com/uploads/allimg/160307/1442114949-3.jpg)

　　同样地，根据Buck电路的分析方式，Boost电路的工作原理为：

　　[![](https://i-blog.csdnimg.cn/blog_migrate/39e37cf6477fb9fc0e6c698e01efcfef.png)](http://www.elecfans.com/uploads/allimg/160307/1442113154-4.jpg)
### buck-boost
Buck-Boost 电路拓扑，有时又称为 Inverting，其典型的电路结构如下图 5 所示：

　　[![](https://i-blog.csdnimg.cn/blog_migrate/fc160aee2599ce7958fc7330bf2d4a75.png)](http://www.elecfans.com/uploads/allimg/160307/1442116032-5.jpg)

　　同样地，根据Buck电路的分析方式，Buck-Boost电路的工作原理为：

　　[![](https://i-blog.csdnimg.cn/blog_migrate/4f388b3388fa477e5f1aeb686aa1a3ef.png)](http://www.elecfans.com/uploads/allimg/160307/1442113K7-6.jpg)
###  Buck 与 Boost 组合

Buck 与 Boost 两者相结合，会得到什么样的电路和应用呢？根据不同的控制，可以让电源从高压降到低压，也可以将低压升到高压，可以称之为双向 DC-DC 变换器之一，典型的应用电路如下图：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202408272255764.png)

DC-DC 双向变换器目前主要应用在各大充放电系统中，随着储能器件的发展得到了广泛地应用，主要的行业在汽车电子，电梯节能系统等应用行业。

当 T2 管截止时，T1 管与 D1、L 等器件构成了 Buck 型降压电路，可以实现对后级的负载进行供电; 反之，当 T1 管截止，T2 管与 D2 二极管、L 等器件构成了 Boost 升压电路，对前端电源进行能量补充。目前对 T1 和 T2 管的控制以模拟方式控制相对还是比较困难，均是以数字控制方式为主。
     
原文链接：https://blog.csdn.net/qq_41451521/article/details/100925249

## PMOS 和 NMOS 在使用上的区别
**P 沟道 MOS 晶体管的空穴迁移率低**, 因而在 MOS 晶体管的几何尺寸和工作电压绝对值相等的情况下，**PMOS 晶体管的跨导小于 N 沟道 MOS 晶体管**。此外，**P 沟道 MOS 晶体管阈值电压的绝对值一般偏高，要求有较高的工作电压**。它的供电电源的电压大小和极性, 与双极型晶体管——晶体管逻辑电路不兼容。PMOS 因**逻辑摆幅大，充电放电过程长，加之器件跨导小，所以工作速度更低**，在 NMOS 电路(见 N 沟道金属—氧化物—半导体集成电路)出现之后，多数已为 NMOS 电路所取代。只是, 因 PMOS 电路工艺简单, 价格便宜，有些中规模和小规模数字控制电路仍采用 PMOS 电路技术。

PMOS 集成电路是一种适合在低速、低频领域内应用的器件。PMOS 集成电路采用-24V 电压供电。

原文链接：https://blog.csdn.net/sinat_26528193/article/details/114965721