## blake2b 算法的安全性
**一、单向性**
1.  从计算过程来看：
    -   Blake2b 将任意长度的输入数据经过一系列复杂的数学运算转换为固定长度的输出，这个过程是易于计算的。
    -   然而，要从给定的哈希值逆向推导出原始输入数据在计算上是极其困难的，甚至在实际应用中被认为是不可行的。
2.  数学特性决定：
    -   Blake2b 的设计基于复杂的位运算和循环迭代，使得输入数据的微小变化会导致输出哈希值的巨大变化。
    -   这种雪崩效应增加了逆向推导的难度，因为即使知道输出哈希值以及输入数据的一些特征，也很难确定具体的原始输入。

**二、抗碰撞性**
1.  弱抗碰撞性：
    -   对于给定的一个输入数据，很难找到另一个不同的输入数据，使得它们产生相同的 Blake2b 哈希值。
    -   这意味着即使攻击者试图通过尝试不同的输入来找到与目标哈希值匹配的输入，成功的概率也是非常低的。
2.  强抗碰撞性：
    -   更难找到任意两个不同的输入数据，使得它们的 Blake2b 哈希值相同。
    -   这种特性进一步保证了哈希值的唯一性，使得逆向推导更加困难。
综上所述，虽然 Blake2b 不是真正的非可逆加密算法，但由于其单向性和抗碰撞性等特点，使得从哈希值逆向推导出原始输入数据非常困难，从而在实际应用中表现出类似非可逆加密的特性。

## 算法关键
### 计算 round
(1) 对 16 个中间状态进行连续 12 个 round 的转换，每个 round 的操作包括：
![](file:///C:\Users\ASUS\AppData\Local\Temp\ksohtml30480\wps1.jpg) 

其中每一个G函数包含如下内容：
其中![](file:///C:\Users\ASUS\AppData\Local\Temp\ksohtml30480\wps2.jpg)分别代表每次输入G函数的四个向量
![](file:///C:\Users\ASUS\AppData\Local\Temp\ksohtml30480\wps3.jpg)

根据 round 轮次查 sigma，得到 index，再根据 index 查 m_hreg 得到m
G 拆分成两部分，前四个一起，后四个一起
每一部分，需要给出两个 index 去 sigma，然后拿回来两个 index，送到 m_hreg 中去查找对应的 message
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202409081636817.png)
column0 = 2i+0    column1 = 2i+1

## 硬件实现
G 函数的操作概括为：逻辑计算，根据 round 轮次查 sigma，得到 index，再根据 index 查 m_hreg 得到 两个m
需要计算 12 次 G 函数，每个 G 函数包含 8 个对向量的计算
1、对于复杂的 G 函数，观察可以得到，前四个和后四个计算相互之间没有交叉，可以并行
2、每一个 round 的数据？
3、复用？
	原来设置了 12 个 round(包括 round a 和 round b)，和 12 个 mreg，改为使用一个 round 和一个 mreg，两组中间寄存器，计算完半轮之后就会将 A 的结果给 B，B 的结果给 A，前一种可以不断地输入新的数据计算，后一种每次只能计算一个message
### ref clock
400Mhz
 (4 clocks per 'G' x 2 'G' per round x 12 rounds)
 96 clocks
 24 级流水
###  quiz
输入是 0- 256byte ，每一个  128byte 为一组计算一轮
中间寄存器：24 * V[16* 64]
sigma：12* 4bit lookup table   
4bit MUX：5 级？
12 个：4 bit mux 5 级
m_hreg：8 个：16* 64 bit output 64 bit
V：a，b，c，d 64bit
一次处理 1024bit = 128 byte 的数据
超过的话就算两次