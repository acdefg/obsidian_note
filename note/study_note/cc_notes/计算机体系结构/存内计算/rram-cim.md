---
tags: [arch, cim]
updated: 2026-10-24
created: 星期一, 十月 19日 2026, 10:41:36
modified: 星期六, 十月 24日 2026, 10:03:31
---

# RRAM 存内计算

## RRAM 存内计算的基本原理是什么？ #arch #cim #rram

> **速查**
> - 权重用电导 $G$ 表示；行加电压、列收电流，$I_j=\sum_i V_iG_{ij}$ 就是点积
> - 1T1R 用晶体管选通、抑制 sneak path；1R 密度高但有串扰
> - 有符号权重用差分对 $G^+-G^-$；输入/输出符号用差分电压或差分 ADC
> - 瓶颈在 DAC/ADC、IR drop、器件噪声，不在乘法本身
> - ADC 分辨率越高，CIM 的能效优势越容易被外围吃掉

### 正文

**① 器件层：电导就是权重**

RRAM 是两端阻变器件（MIM 结构）：加 SET 电压形成导电细丝，电导升高；加 RESET 电压细丝部分断裂，电导降低。电导 $G$ 在 $[G_{\min},G_{\max}]$ 内连续可调，所以可以当模拟权重。

- **1T1R**：每个单元串一个晶体管。晶体管做选通，限制编程电流，抑制未选单元的漏电（sneak path）。代价是面积大、单元电流受晶体管驱动能力限制。
- **1R**：只有电阻，密度最高；但未选单元会通过共享的 BL/WL 形成漏电通路，读出的列电流里混入串扰。
- **多值**：通过 compliance current、脉冲幅值/脉宽/个数编程出多个电导态。多值的线性度、对称性、良率都比二值差。

**② 阵列层：两条定律**

Crossbar 的列电流：

$$
I_j = \sum_i V_i G_{ij}
$$

这就是输入电压向量 $V$ 与第 $j$ 列电导向量 $G_{\cdot j}$ 的点积。整个阵列一次完成 MVM，不需要逐元素乘加。

**算一个具体例子**：64 行 × 64 列，输入电压 $V_i=0.1\,\text{V}$，每个单元 $G=1\,\mu\text{S}$。

- 单元电流：$0.1\,\text{V}\times1\,\mu\text{S}=0.1\,\mu\text{A}$
- 列电流：$64\times0.1\,\mu\text{A}=6.4\,\mu\text{A}$
- 若电导范围 $1$–$10\,\mu\text{S}$，列电流满量程约 $64\,\mu\text{A}$

ADC 要覆盖 $0$–$64\,\mu\text{A}$，并按这个量程设计参考电压/电流。如果 ADC 只有 4 bit，最小分辨电流是 $64/16=4\,\mu\text{A}$，而每个单元只贡献 $0.1\,\mu\text{A}$，相当于要 40 个单元同时导通才能跨过一个 LSB——这就是“模拟计算做了乘法，但精度被 ADC 截断”的具体含义。

**③ 外围链路**

```text
数字输入
  │
  ▼
DAC / 行驱动 ──► WL（行电压）
                    │
                    ▼
              ┌───────────────┐
              │  crossbar     │  G_ij = 权重
              │  1T1R / 1R    │
              └───────┬───────┘
                      │ BL 列电流求和
                      ▼
              采样保持 / 电流镜 / 参考列
                      │
                      ▼
                    ADC
                      │
                      ▼
              数字移位 / 累加 / 校准
```

**④ 为什么不是“免费的计算”**

- **DAC**：把输入激活转成电压，非线性、建立时间、驱动能力都会引入误差；
- **ADC**：每列一个，分辨率和采样率决定功耗/面积；flash ADC 的面积/功耗随位数指数增长，SAR 更省但慢；
- **IR drop**：BL/SL 导线有电阻，电流越大压降越大，靠近驱动端和远端的单元实际电压不同；
- **sneak path**：1R 阵列中未选单元漏电，列电流不再等于目标点积；
- **器件噪声/漂移**：RTN、1/f 噪声、电导漂移、编程误差。

所以 CIM 的能效优势通常出现在**低精度、低 ADC 分辨率、权重可复用**的场景；精度一高，外围功耗会吃掉收益。

**⑤ 1T1R 与 1R 的取舍**

| | 1T1R | 1R |
|---|---|---|
| 选通 | 有晶体管，抑制 sneak path | 无，依赖阵列偏置 |
| 密度 | 低（晶体管占面积） | 高 |
| 编程电流 | 受晶体管限制，可控 | 难限制，易过写 |
| 读线性度 | 较好 | 受串扰/IR drop 影响大 |
| 适用 | 高精度、小阵列、研究/量产 macro | 高密度、低精度、存内计算原型 |

### 自测

- [ ] 不看公式，用欧姆定律和 KCL 推出 $I_j=\sum_i V_iG_{ij}$。
- [ ] 64×64 阵列、$V=0.1\,\text{V}$、$G=1\,\mu\text{S}$，列电流是多少？ADC 4 bit 时最小分辨电流是多少？
- [ ] 1T1R 和 1R 各解决什么问题、各自付出什么代价？

---

## GEMM 怎么映射到 crossbar 阵列？ #arch #cim #rram

> **速查**
> - $C=AB$：B 映射成电导 $G$，A 的行做电压，列电流得到 C 的一行
> - K 超行数→切 K 累加；N 超列数→切 N；M 超并行度→分批
> - 部分和可在模拟域相加，也可 ADC 后数字累加
> - INT8 权重拆 2×4-bit slice；输入拆 8 个 1-bit 周期；输出移位相加
> - 利用率由 padding、分块、转置读决定

### 正文

**① 基本映射**

设 $C=AB$，$A\in\mathbb{R}^{M\times K}$，$B\in\mathbb{R}^{K\times N}$。把 $B$ 映射成电导矩阵：

$$
G_{kj}=B_{kj}
$$

输入 $A$ 的第 $m$ 行 $a_m$ 变成行电压：

$$
V_k=a_{mk}
$$

第 $j$ 列输出电流：

$$
I_j=\sum_k a_{mk}G_{kj}=C_{mj}
$$

逐行处理 $M$ 行，得到整个 $C$。这就是 **weight-stationary**：权重留在阵列里，激活流过阵列。

**② 一个 2×2 的手算例子**

$$
A=\begin{bmatrix}1&2\end{bmatrix},\quad
B=\begin{bmatrix}3&4\\5&6\end{bmatrix}
$$

映射：第 1 行电导 $[3,4]$，第 2 行电导 $[5,6]$。输入电压 $[1,2]$：

- 第 1 列：$1\times3+2\times5=13$
- 第 2 列：$1\times4+2\times6=16$

结果 $C=[13,16]$，与 $AB$ 一致。这个例子说明：**行是 K 维，列是 N 维，输入沿行方向加电压**。

**③ 大矩阵怎么切**

| 维度 | 超过阵列时 | 代价 |
|---|---|---|
| K（行） | 切 K 块，多个阵列/多次扫描，部分和累加 | 累加器、额外 ADC/数字加 |
| N（列） | 切 N 块，输出分块 | 输出缓冲、写回 |
| M（并行度） | 分批处理 | 时间复用，延迟增加 |

部分和两种做法：

- **模拟域相加**：多个阵列的列电流直接汇总到同一 BL，省 ADC 次数；但动态范围、IR drop、线性度更差。
- **数字域相加**：每块先 ADC，再数字累加；灵活、精度好，但 ADC 次数和数字功耗增加。

**④ 多比特怎么映射**

如果单元只有 4 bit，而权重是 INT8：

- **权重切片**：INT8 权重拆成高 4 bit / 低 4 bit 两个单元，输出按 $2^4$ 移位相加；
- **输入切片**：INT8 输入拆成 8 个 1-bit 周期（或 2 个 4-bit 周期），逐周期扫描并移位累加。

以 $W=171=0xAB$、$x=18=0x12$ 为例，拆成 4-bit slice：

- $W=10\times16+11$，两个 slice 是 10 和 11；
- $x=1\times16+2$，两个 slice 是 1 和 2；
- 乘积 $=(10\times16+11)(1\times16+2)$
  $=10\times1\times256+(10\times2+11\times1)\times16+11\times2$
  $=2560+496+22=3078$；
- 直接算 $171\times18=3078$。

这就是移位相加的硬件含义：每个 slice 的 MVM 结果按 $2^{4k}$ 移位后累加。

**⑤ ADC 量程与切片**

切片会改变列电流的动态范围。若 ADC 量程仍按全精度设置，低 significance slice 的有效位数会被浪费；若按 slice 单独设置，又要增加参考/校准复杂度。文献结论：**bit-slicing 的精度收益很大程度取决于 ADC 量程是否匹配**。

**⑥ 利用率**

面试常问“阵列利用率”。要能拆开说：

- 有用单元 / 总单元；
- padding（K、N 不是阵列尺寸整数倍）；
- 分块后的部分和开销；
- 转置读是否需要复制阵列；
- ADC 数量与列数的比例。

相关：[[computer-arch#数据流怎么选？复用因子怎么算？]]

### 自测

- [ ] 用 2×2 例子手算一遍 $C=AB$，并说明行/列各对应哪个维度。
- [ ] K 超过阵列行数时，部分和在哪累加？模拟域和数字域各有什么代价？
- [ ] $W=171$、$x=18$ 拆成 4-bit slice，写出移位相加的每一步。

---

## 矩阵求导算子怎么映射到存内计算阵列？ #arch #cim #rram

> **速查**
> - $Y=XW$ → $dX=dYW^T$，$dW=X^TdY$
> - $dX$ 要转置读或复制 $W^T$ 阵列；$dW$ 是外积累加
> - 外积更新：行加 $x_i$、列加 $\delta_j$，单元更新 $G_{ij}\leftarrow G_{ij}-\eta x_i\delta_j$
> - 更新难点：非线性/不对称、写验证、耐久、写路径 IR drop
> - 前向可低精度，反向/更新通常更高精度（MINT）

### 正文

**① 反向传播的两个 GEMM**

对 $Y=XW$，$X\in\mathbb{R}^{M\times K}$，$W\in\mathbb{R}^{K\times N}$：

$$
dX=dY\,W^T,\qquad dW=X^T dY
$$

- $dX$：$M\times N$ 的 $dY$ 乘 $N\times K$ 的 $W^T$，输出 $M\times K$；
- $dW$：$K\times M$ 的 $X^T$ 乘 $M\times N$ 的 $dY$，输出 $K\times N$。

两者都是 GEMM，但映射方式不同：$dX$ 需要 $W^T$，$dW$ 是外积。

**② $dX$：转置怎么来**

Crossbar 里权重按 $W$（K 行、N 列）存。要算 $dY W^T$，需要按列方向使用同一阵列，或者：

- **可转置阵列**：行/列驱动和读出都能切换，读列得到 $W^T$；
- **复制阵列**：再存一份 $W^T$，面积翻倍但控制简单；
- **数字转置**：先读出再在数字域转置，灵活但增加搬运和延迟。

1T1R 阵列的 WL/BL 驱动和读出通常不对称，所以“转置读”不是免费的，需要专门设计。

**③ $dW$：外积更新**

$$
dW_{ij}=\sum_m X_{mi}\,dY_{mj}
$$

把 $X$ 的一列和 $dY$ 的一列分别加到行/列上，每个单元得到 $x_i\delta_j$。如果硬件支持并行更新，一次可以更新整个 $K\times N$ 权重块。

**手算例子**：

$$
X=\begin{bmatrix}1&2\end{bmatrix},\quad dY=\begin{bmatrix}3&4\end{bmatrix}
$$

$$
dW=X^TdY=\begin{bmatrix}1\\2\end{bmatrix}\begin{bmatrix}3&4\end{bmatrix}
=\begin{bmatrix}3&4\\6&8\end{bmatrix}
$$

- 第 1 行：$x_1=1$ 乘 $dY=[3,4]$ → $[3,4]$
- 第 2 行：$x_2=2$ 乘 $dY=[3,4]$ → $[6,8]$

**④ In-situ 更新流程**

```text
前向：X → crossbar(W) → Y
损失：Y, label → dY
反向：dY × W^T → dX
更新：X^T × dY → dW，写回 W
```

更新时：

$$
G_{ij}\leftarrow G_{ij}-\eta\,x_i\,\delta_j
$$

其中 $\eta$ 是学习率，$x_i$ 是输入，$\delta_j$ 是误差。实现上用脉冲数/脉宽调制，把乘积转成电导变化量。

**⑤ 难点**

- **更新非线性/不对称**：SET 和 RESET 的响应不同，脉冲数→电导变化不是线性的；
- **写验证**：program-read-compare 迭代，慢；
- **耐久/写干扰**：反复更新会退化、漂移；
- **写路径 IR drop**：并行写时不同位置的更新量不同；
- **梯度精度**：前向可以 INT8/INT4，反向和更新通常要更高精度（MINT 这类混合精度训练架构）。

**⑥ 如果“矩阵求导”指 elementwise Jacobian**

ReLU、softmax、LayerNorm 的 Jacobian 通常是对角/低秩结构，阵列不直接参与，主要在数字域做。面试时要先确认是哪一类，不要把 elementwise 也说成 crossbar GEMM。

相关：[[mixed-precision-cim#混合精度怎么分配、硬件怎么支持？]]

### 自测

- [ ] 写出 $dX$ 和 $dW$ 的公式，说明各自对应哪个矩阵维度。
- [ ] 用 $X=[1,2]$、$dY=[3,4]$ 手算 $dW$，并说明外积的每一行怎么来。
- [ ] 为什么 $dX$ 的转置读在 1T1R 阵列里不是免费的？

---

## Tiling 和阵列映射是什么关系？ #arch #cim #rram

> **速查**
> - 两者是不同层：tiling 决定大 GEMM 怎么切块、部分和怎么累；阵列映射决定一个 tile 怎么绑到 crossbar 的行列
> - 算子映射是总称，等于 tiling + 数据流选择 + 阵列绑定 + 调度
> - tiling 救不了转置：$W^T$ 的转置读取决于阵列的 WL/BL 驱动与读出能力，切块不改变这个需求
> - CIM 的 tiling 额外受 crossbar 尺寸、IR drop、ADC 数量、权重写次数约束
> - 判断问题在哪一层：换 tile 大小能解决的是 tiling；要改电路/阵列结构的是阵列映射

### 正文

**① 四层分工**

| 层 | 回答的问题 | 交付物 |
|---|---|---|
| 算子级 | 算哪个 GEMM | $Y=XW$、$dX=dYW^T$、$dW=X^TdY$ |
| tiling | 切哪个维度、切多大、循环顺序、部分和怎么累 | tile 形状、循环嵌套、片上缓冲分配 |
| 阵列映射 | tile 的哪个维度放行、哪个放列、转置怎么读、权重怎么切 bit | 阵列绑定、转置读/复制、bit-slicing、差分对 |
| 电路级 | 驱动和读出怎么实现 | DAC/ADC、WL/BL 驱动、IR drop、写验证 |

前两层通常由编译器/调度器在软件侧决定（切块参数、循环顺序），后两层由硬件结构决定（阵列尺寸、驱动能力）；软件只能使用硬件提供的能力。

**② 一个判断方法**

问：“把 tile 从 128 改成 64，问题会消失吗？”

- 会 → tiling 问题（容量、复用、部分和）
- 不会 → 阵列映射或电路问题（转置读、IR drop、ADC 量程）

**③ 两个反向 GEMM 卡在不同层**

| GEMM | 主要卡点 | 属于哪一层 |
|---|---|---|
| $dX=dYW^T$ | 需要 $W^T$：转置读 / 复制阵列 / 数字转置 | **阵列映射**（tiling 改不了） |
| $dW=X^TdY$ | 外积累加；沿 $M$ 切几块就产生几次部分更新 | **tiling**（切法决定写次数） |

$dW$ 的例子：$M=512$、tile 128，沿 $M$ 切 4 块，每个权重 tile 得到 4 个部分 $dW$。可以在数字域累加后写 1 次，也可以原地更新 4 次；前者省耐久、要 SRAM，后者省 SRAM、伤耐久。这是 tiling 策略，不是阵列映射能决定的。

**④ CIM 的 tiling 比数字 NPU 多四个约束**

| 约束 | 对 tiling 的影响 |
|---|---|
| crossbar 物理尺寸 | tile 不能超过阵列行列数；超出必须切 $K$ 并累加部分和 |
| IR drop / 同时打开的 WL·BL 数 | 一次并行的行/列数受限，tile 的并行度被压低 |
| ADC 数量与面积 | 每列一个 ADC 还是分时复用，决定一次能读多少列 |
| 权重写次数与耐久 | $dW$ 沿 $M$ 的切法直接决定每个单元被写几次 |

数字 NPU 的 tiling 主要优化复用和带宽；CIM 还要把模拟非理想当成一等约束。

**⑤ 边界**

不要把所有“映射”都说成 tiling。论文里 mapping 常是总称，已经把 tiling 包进去；但如果问题出在转置读、IR drop、ADC 量程，换 tiling 不会有用。

### 自测

- [ ] 用一句话区分 tiling 和阵列映射，并各举一个只有它能解决的问题。
- [ ] 为什么说 tiling 救不了 $dX=dYW^T$ 的转置问题？
- [ ] $dW$ 沿 $M$ 切 4 块，权重可能被写几次？两种策略各付出什么代价？
- [ ] CIM 的 tiling 比数字 NPU 多哪四个约束？

---

## MNK、batch 和复用因子怎么对应？ #arch #cim #rram #dataflow

> **速查**
> - $M$ = A 的行数 = C 的行数（输出行），$N$ = B 的列数 = C 的列数（输出列），$K$ = 被消掉的归约维
> - batch 不是第四个维度：通常折进 $M$（$M=B\times S$），或作为 batched GEMM 的外层循环
> - 复用因子：左矩阵复用 = 输出列数，右矩阵复用 = 输出行数；切归约维产生部分和，不叫复用
> - 三个 GEMM 的归约维：前向 $K$、$dX$ 是 $N$、$dW$ 是 $M$
> - 留谁在片上不只看复用次数，要看“复用 × 体积”

### 正文

**① 标准 GEMM 与三围**

```
C = A·B,  A: M×K,  B: K×N,  C: M×N
```

| 维度 | 含义 | 深度学习里 |
|---|---|---|
| $M$ | A 的行数 = C 的行数 | token 数、$B\times S$、输出空间位置数 |
| $N$ | B 的列数 = C 的列数 | 输出通道数、hidden dim、vocab size |
| $K$ | 归约维（内积深度） | 输入通道数、in_features、head_dim |

$$
\text{FLOPs}=2MNK,\qquad \text{数据量}=MK+KN+MN
$$

**② batch 在哪**

- **折进 $M$**：Transformer 的 `[B, S, H]` flatten 成 `[B·S, H]`，于是 $M=B\cdot S$。prefill 的 $M$ 是 batch×seq；decode 每序列每步只有 1 个 token，$M$ 等于并发序列数（continuous batching 下可以 >1）。
- **外层循环**：多个独立 GEMM（attention 的每个 head、每组卷积）用 strided batched GEMM，M/N/K 不变，外面套 batchCount。

batch 维不参与内积，它只是并行/循环维；$K$ 才是被累加的维。

**③ 复用的定义与 2×2 手算**

复用因子 = 一个元素参与了多少次 MAC。

```
dY = [a b]      Wᵀ = [e f]      dX = dY·Wᵀ = [ae+bg  af+bh]
     [c d]            [g h]                    [ce+dg  cf+dh]
```

- `a` 在 `dX[0,0]`、`dX[0,1]` 各用一次 → 2 次 = $K$
- `e` 在 `dX[0,0]`、`dX[1,0]` 各用一次 → 2 次 = $M$

所以：**左矩阵沿输出列复用，右矩阵沿输出行复用。**

**④ 通用规则与三个 GEMM**

```
C = A·B：
  A 的复用 = C 的列数
  B 的复用 = C 的行数
  归约维   = 被消掉的那个维度
```

| GEMM | 输出 | 归约维 | 左矩阵复用 | 右矩阵复用 |
|---|---|---|---|---|
| $Y=XW$ | M×N | $K$ | $X$：$N$ | $W$：$M$ |
| $dX=dYW^T$ | M×K | $N$ | $dY$：$K$ | $W^T$：$M$ |
| $dW=X^TdY$ | K×N | $M$ | $X^T$：$N$ | $dY$：$K$ |

前向那条与 computer-arch 的结论一致：$M=512,N=4096$ 时，$X$ 复用 4096 次、$W$ 复用 512 次。

**⑤ 回到 $dX$ 的 tile 例子**

$dY$：$512(M)\times4096(N)$，$W^T$：$4096(N)\times1024(K)$，$dX$：$512(M)\times1024(K)$；tile 128：

| 维度 | 块数 | 角色 |
|---|---|---|
| $M$ | 4 | 输出行 |
| $K$ | 8 | 输出列 |
| $N$ | 32 | **归约维（部分和）** |

- 输出 tile $(m,k)$ 遍历 32 个 $N$ 块 → **32 个部分和**
- $dY$ tile $(m,n)$ 与所有 8 个 $k$ 相乘 → **复用 8 次**
- $W^T$ tile $(n,k)$ 与所有 4 个 $m$ 相乘 → **复用 4 次**

容易说反的一组数：**32 是归约次数，不是复用次数。** 复用发生在输出维度上；归约发生在被消掉的维度上。

**⑥ 留谁在片上**

省下的 DRAM 流量 ≈ 复用次数 × 数据大小：

| 留谁 | 复用 | 大小 | 省下的重读量 |
|---|---|---|---|
| $dY$ | 8 | 2.1 MB | ~14.7 MB |
| $W^T$ | 4 | 4.19 MB | ~12.6 MB |

这里两个判据同向：$dY$ 复用更高、体积更小，留 $dY$。

但如果复用高的是大矩阵（例如 8 次 × 20 MB），硬留会挤掉别的数据；此时要按“复用 × 体积”比较，不能只看次数。

CIM 里权重物理驻留在 crossbar 中，不发生“重读权重”；复用分析主要针对激活。权重侧的核心问题变成能不能转置读。

相关：[[computer-arch#数据流怎么选？复用因子怎么算？]]

### 自测

- [ ] 写出 $C=AB$ 中 A、B 的复用因子，并说明由哪个输出维度决定。
- [ ] $dX=dYW^T$ 的归约维是谁？切它产生几个部分和、复用次数分别是多少？
- [ ] 为什么“32 个 N tile”不是 $dY$ 的复用次数？
- [ ] 留谁在片上为什么不能只看复用次数？举一个反例。

---

## 精度补偿机制分哪几层？ #arch #cim #rram

> **速查**
> - 器件：差分对、写验证、多值、ECC
> - 电路：参考列、CDS/auto-zero/chopping、电流钳位、阻抗提升
> - 架构：IR-drop-aware 映射、输出相关补偿、校准表/背景校准
> - 算法：噪声感知训练、QAT、chip-in-the-loop、残差修正
> - 面试要说清补的是哪层、静态/动态、开销、剩余误差

### 正文

**① 误差来源**

| 层级 | 典型误差 | 机制 | 影响 |
|---|---|---|---|
| 器件 | 编程误差、漂移、RTN/1/f 噪声、保持/耐久 | 导电细丝随机性、氧空位迁移、缺陷 | 权重偏差、时间漂移 |
| 阵列 | IR drop、sneak path、寄生 RC | BL/SL 导线电阻、未选单元漏电 | 输入/位置相关系统误差 |
| 外围 | DAC 非线性、ADC offset/gain、采样电荷注入、电流镜失配 | 模拟电路失配、时钟馈通 | 固定/慢变偏移 |
| 系统 | 量化误差、映射/padding、部分和截断 | 有限位宽、分块 | 精度损失 |

**② 四层补偿**

**器件层**

- 差分对 $G^+-G^-$：抵消共模漂移；
- 写验证：编程-读回-修正；
- 多值编程 + 冗余/ECC；
- 漂移感知刷新。

**电路层**

- 参考列/复制列：跟踪 IR drop、共模、温度；
- 相关双采样（CDS）、auto-zero、chopping：消 ADC/放大器 offset；
- 电流钳位、阻抗提升：减小 IR drop 影响；
- 低 cell bias：降低非线性；
- in-ADC computing：在数字化前做部分补偿。

**架构层**

- IR-drop-aware 映射：把大电流模式分散；
- 输出相关补偿（X-CODA）：预表征输出相关的平均误差，片上查表/加性修正；
- ADC 校准表、背景校准；
- 分块/分时，限制同时打开的 WL/BL 数。

**算法层**

- 噪声感知训练；
- QAT + 混合精度；
- chip-in-the-loop：用芯片实测输出微调软件层；
- 残差修正：低精度主路径 + 高精度残差路径。

**③ 代表工作与“补什么”**

| 工作 | 关键技术 | 补什么 | 结果/看点 |
|---|---|---|---|
| [REF-CIM](https://doi.org/10.1109/TCSI.2026.3682398)（TCAS-I 2026） | CMIU、PCSC 电流缩放钳位、DTS-ADC、可配置精度 | IR drop、器件 variation、模拟噪声 | 8b/8b/16b，10% variation 下 CIFAR-10 92% |
| [VLSI 2023 2.38 MCells/mm² macro](https://doi.org/10.23919/VLSITechnologyandCir57934.2023.10185424) | hybrid offset/IOFF cancellation、$I_{CELL}R_{BL/SL}$ drop mitigation、低 cell bias | ADC offset、关态电流、IR drop | 9.81–350 TOPS/W，post-ECC BER < 5e-8 |
| [X-CODA](https://doi.org/10.1145/3816440.3818583)（ISLPED 2026） | 预表征 output-dependent mean error + 片上加性补偿、ADC binning | IR drop 等输出相关误差 | 接近软件 1–2% |
| [IR-drop 分析](https://doi.org/10.23919/DATE69613.2026.11539412)（DATE 2026） | 映射优化 + ADC 校准 | IR drop | 低开销缓解 |
| [IIB/In-ADC](https://doi.org/10.1109/JXCDC.2023.3255788)（JXCDC 2023） | intrinsic impedance boosting、in-ADC computing | 读出失真、ADC 参考调校 | 面向 calibration-free SAR |
| [NeuRRAM](https://doi.org/10.1038/s41586-022-04992-8)（Nature 2022） | noise-aware training + chip-in-the-loop 逐层微调 | 器件非理想、量化 | CIFAR-10 85.66% |
| [BPAP/AOSC](https://doi.org/10.11999/JEIT251174)（JEIT 2026） | 反向传播辅助定位失败单元、模拟域 offset 消除、可转置 macro | 编程误差、参考漂移、IR drop | 0.64% 全并行 RMS 误差 |
| [CIM-SECDED](https://doi.org/10.1109/A-SSCC53895.2021.9634742)（A-SSCC 2021） | ECC | 可靠性/误码 | 64Kb macro |
| [Yoon ISSCC 2021](https://doi.org/10.1109/ISSCC42613.2021.9365926) | active-feedback read、in-situ write verify | read disturb、写验证 | 56.67 TOPS/W |

**④ 面试回答框架**

不要说“做了校准”，按这个顺序说：

1. **误差源**：器件/阵列/外围/量化，哪一类；
2. **静态还是动态**：offset 是静态，IR drop 随输入变，漂移随时间变；
3. **补偿层级**：器件/电路/架构/算法；
4. **开销**：面积、功耗、延迟、校准时间；
5. **剩余误差**：补偿后还差多少，怎么验证。

**⑤ 一个具体的 IR drop 直觉**

BL 导线有电阻。越远的单元，实际 BL 电压越低；列电流越大，压降越大。所以误差不是固定的 offset，而是**输入相关、位置相关**的。参考列只能跟踪共模，不能完全消除这种空间分布；这也是为什么需要 IR-drop-aware 映射和输出相关补偿。

### 自测

- [ ] 把误差源按器件/阵列/外围/系统四层各举一个例子。
- [ ] 参考列补偿什么、不能补偿什么？
- [ ] 为什么 IR drop 是输入相关误差，而 ADC offset 更接近静态误差？

---

## 存内计算的仿真验证怎么做？ #arch #cim #rram

> **速查**
> - 四层：器件 SPICE / 阵列行为级 / 架构 NeuroSim / 算法 PyTorch
> - SPICE 验 I-V、SET/RESET、Monte Carlo；行为级验大阵列、IR drop、ADC
> - 架构级算面积/能耗/延迟；算法级验端到端精度
> - 必须说清是仿真还是硅实测；不要暗示流片
> - 验证清单：功能 golden、精度、非理想、性能、可靠性、边界

### 正文

**① 四个层级**

| 层级 | 工具/方法 | 能验证什么 | 局限 |
|---|---|---|---|
| 器件 | RRAM compact model + SPICE（JART/Stanford/ASU 等） | I-V、SET/RESET、变异性、瞬态 | 大阵列跑不动 |
| 阵列 | 行为级：电导矩阵 + IR drop 求解 + ADC/DAC 量化 + 噪声 | 大阵列功能、精度、IR drop | 依赖模型准确性 |
| 架构 | NeuroSim / DNN+NeuroSim / MemIntelli | 面积、能耗、延迟、精度 | 抽象层级较高 |
| 算法 | PyTorch + QAT + 噪声注入 + chip-in-the-loop | 端到端精度 | 需要硬件校准数据 |

**② 旧笔记里的仿真案例**

你的旧笔记 [[rram model仿真]] 和 [[rram 仿真]] 里有一组可以直接在面试里讲的细节：

- 用 SkyWater 的 Verilog-A ReRAM 模型（`sky130_fd_pr_reram`）做 1T1R 仿真；
- forming：$2.2$–$3.1\,\text{V}$、$1000\,\text{ns}$，电阻从约 $3\,\text{M}\Omega$ 降到约 $400\,\Omega$；
- reset/set 与电压幅值、脉冲宽度相关（例如 $3\,\text{V}$、$-2.4/-2.7/-3\,\text{V}$，$50\,\text{ns}/500\,\text{ns}/5\,\mu\text{s}$）；
- 3×3 crossbar 同值单元测试：预期 $500\,\text{mV}/620.7\,\text{k}\Omega\times3=2.41\,\mu\text{A}$，实测 $498.8\,\text{mV}$ vs 预测 $498.629\,\text{mV}$；
- 漏电问题：MOS 源漏高场 + 正反馈导致大电压，运放不理想、开关管导致不收敛；加负反馈通路后求解 $(A+1)x+b=0$。

这些细节比“做过仿真”有说服力，面试时要能解释**为什么误差这么小、漏电怎么定位、负反馈怎么加**。

**③ 验证清单**

- **功能**：GEMM/反向算子 vs 软件 golden model；数字部分 bit-exact，模拟部分看误差分布；
- **精度**：MNIST/CIFAR/ImageNet；逐层误差 + 端到端；
- **非理想**：Monte Carlo 器件 variation、IR drop、漂移、噪声、温度、电源；
- **性能**：TOPS/W、TOPS/mm²、延迟、ADC 能耗占比；
- **可靠性**：BER、耐久、保持、read disturb；
- **边界**：全 0/全 1、最大/最小电流、切片进位、转置读。

**④ 怎么讲仿真层级**

明确说：

> 器件和关键模拟链路用 SPICE + Monte Carlo；大阵列和端到端精度用行为级模型；架构指标用 NeuroSim 类框架；如果有实测数据，再单独说明实测与仿真的偏差。

不要用“仿真验证”暗示流片；如果只是行为级，就说行为级。

相关：[[mixed-precision-cim#混合精度怎么分配、硬件怎么支持？]]

### 自测

- [ ] 四个仿真层级各验证什么、各自不能验证什么？
- [ ] 旧笔记里的 3×3 crossbar 测试，预期电流和实测电压怎么对上？
- [ ] 漏电问题的根因链是什么？负反馈解决了哪一环？

---

## 论文清单：RRAM CIM 该读哪几篇？ #arch #cim #rram

> **速查**
> - 综述先读 Ielmini 2018、Sebastian 2020、Wei 2022
> - 架构读 ISAAC、PRIME、PipeLayer、PUMA
> - 补偿读 REF-CIM、VLSI 2023 macro、X-CODA、NeuRRAM
> - 训练读 Prezioso 2015、Yao 2020、MINT
> - 仿真读 NeuroSim、DNN+NeuroSim、MemIntelli

### 正文

**综述与基础**

| 论文                                                                                                  | 链接                                         | 一句话                      |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------ | ------------------------ |
| Ielmini & Wong, *In-memory computing with resistive switching devices*, Nat. Electron. 2018         | https://doi.org/10.1038/s41928-018-0092-2  | RRAM 器件 + 阵列 + CIM 的入门综述 |
| Sebastian et al., *Memory devices and applications for in-memory computing*, Nat. Nanotechnol. 2020 | https://doi.org/10.1038/s41565-020-0655-z  | 各类非易失器件的 IMC 全景          |
| Wei, Gao, Wu, *Trends and challenges in the circuit and macro of RRAM-based CIM*, Chip 2022         | https://doi.org/10.1016/j.chip.2022.100004 | 电路/macro 层面的真问题清单        |
| Le Gallo et al., *Mixed-precision in-memory computing*, Nat. Electron. 2018                         | https://doi.org/10.1038/s41928-018-0054-8  | 混合精度 IMC 的原始概念           |

**架构与映射**

| 论文                                        | 链接                                      | 一句话                         |
| ----------------------------------------- | --------------------------------------- | --------------------------- |
| Shafiee et al., *ISAAC*, ISCA 2016        | https://doi.org/10.1109/ISCA.2016.12    | 流水化 RRAM crossbar CNN 加速器   |
| Chi et al., *PRIME*, ISCA 2016            | https://doi.org/10.1109/ISCA.2016.13    | ReRAM 主存内计算，计算/存储模式切换       |
| Song et al., *PipeLayer*, HPCA 2017       | https://doi.org/10.1109/HPCA.2017.55    | 流水化训练/推理，spike 编码省 ADC      |
| Ankit et al., *PUMA*, ASPLOS 2019         | https://arxiv.org/abs/1901.10351        | 可编程 memristor 加速器 + ISA/编译器 |
| Hu et al., *Dot-product engine*, DAC 2016 | https://doi.org/10.1145/2897937.2898010 | 1T1M crossbar 做 MVM 的早期代表   |

**非理想与补偿**

| 论文 | 链接 | 一句话 |
|---|---|---|
| *REF-CIM*, TCAS-I 2026 | https://doi.org/10.1109/TCSI.2026.3682398 | 非理想容忍 + 可配置精度 macro |
| *2.38 MCells/mm² macro*, VLSI 2023 | https://doi.org/10.23919/VLSITechnologyandCir57934.2023.10185424 | offset/IOFF 消除 + IR drop 缓解 |
| *X-CODA*, ISLPED 2026 | https://doi.org/10.1145/3816440.3818583 | 输出相关误差的片上补偿 |
| *IR drop 分析*, DATE 2026 | https://doi.org/10.23919/DATE69613.2026.11539412 | IR drop 建模 + 映射/ADC 校准 |
| Xie, Yu, Li, *High-Parallelism RRAM CIM Macro*, JXCDC 2023 | https://doi.org/10.1109/JXCDC.2023.3255788 | 阻抗提升 + in-ADC 计算 |
| Wan et al., *NeuRRAM*, Nature 2022 | https://doi.org/10.1038/s41586-022-04992-8 | 芯片在环微调 + 噪声感知训练 |
| *BPAP/AOSC*, JEIT 2026 | https://doi.org/10.11999/JEIT251174 | 编程策略 + offset 消除 + 可转置 macro |

**训练与矩阵求导**

| 论文 | 链接 | 一句话 |
|---|---|---|
| Prezioso et al., Nature 2015 | https://doi.org/10.1038/nature14441 | 金属氧化物 memristor 网络的 in-situ 训练 |
| Yao et al., Nature 2020 | https://doi.org/10.1038/s41586-020-1942-4 | 全硬件 memristor CNN |
| Jiang et al., *MINT*, ISCAS 2020 | https://doi.org/10.1109/ISCAS45731.2020.9181020 | 混合精度 RRAM in-memory training |
| Prajapat et al., SOCC 2022 | https://doi.org/10.1109/SOCC56010.2022.9908131 | 外积/权重更新 $O(1)$ 的 in-situ 训练 |
| Li et al., Frontiers Neurosci. 2021 | https://doi.org/10.3389/fnins.2021.636127 | 并行模拟训练 |

**仿真与建模**

| 工具/论文 | 链接 | 一句话 |
|---|---|---|
| Chen, Peng, Yu, *NeuroSim*, TCAD 2018 | https://doi.org/10.1109/TCAD.2018.2789723 | 电路级 macro 模型 + 在线学习基准 |
| Peng et al., *DNN+NeuroSim*, IEDM 2019 | https://doi.org/10.1109/IEDM19573.2019.8993491 | 端到端 CIM 基准框架 |
| Zhou et al., *MemIntelli*, arXiv 2025 | https://arxiv.org/abs/2511.17418 | 可变精度、器件/电路/架构联合仿真 |
| Ghosh et al., *Approximate ADCs for IMC*, arXiv 2024 | https://arxiv.org/abs/2408.06390 | 近似 ADC 的精度/能耗分析 |

### 自测

- [ ] 如果面试官只让你推荐三篇，你选哪三篇？为什么？
- [ ] 哪篇论文直接对应“IR drop 补偿”？哪篇对应“混合精度训练”？
- [ ] 你的仿真工作可以挂在哪个工具/论文的谱系上？

---

## 面试背诵卡：RRAM 存内计算 #arch #cim #rram

> **速查**
> - 30 秒：RRAM 用电导存权重，crossbar 用欧姆+KCL 做 MVM；我负责数模混合链路、GEMM/求导映射、精度补偿和仿真
> - 五个必答：为什么省能耗、非理想、GEMM/求导映射、精度补偿、仿真层级
> - 回答顺序：动机 → 机制 → trade-off → 量化 → 边界
> - 不夸大：仿真就说仿真，参与就说参与
> - 每个数字都要能说清基线

### 正文

**30 秒项目口述**

> 我独立负责 RRAM 存内计算 macro 的数模混合链路：把 GEMM 和反向传播的矩阵求导算子映射到 crossbar，设计/选型 DAC、ADC、采样保持和参考列，并针对编程误差、IR drop、ADC offset 等非理想做分层补偿；用 SPICE 加行为级模型做仿真验证，和软件 golden model 对比精度与能效。

**高频问答**

| 问题 | 回答要点 |
|---|---|
| 为什么 RRAM 能做 MVM？ | 电导存权重，行电压、列电流；$I_j=\sum_i V_iG_{ij}$ 是点积 |
| 为什么省能耗？ | 一次模拟 MVM 替代大量数字乘加；权重非易失、无搬运；但 ADC/DAC 会吃掉收益 |
| 精度怎么保证？ | 器件（差分/写验证）、电路（参考列/offset 消除/钳位）、架构（IR-drop-aware 映射/输出补偿）、算法（噪声感知训练/混合精度/残差） |
| IR drop 怎么补？ | 低 cell bias、阻抗提升、参考列、映射优化、输出相关修正；它是输入/位置相关误差 |
| 矩阵求导怎么映射？ | $dX=dYW^T$ 转置 GEMM；$dW=X^TdY$ 外积累加；转置读/复制阵列，更新用脉冲编程 |
| ADC 为什么是瓶颈？ | 每列一个；flash ADC 面积/功耗随位数指数增长；SAR 省但慢；精度高时外围功耗占比大 |
| 为什么不用 SRAM CIM？ | RRAM 密度高、非易失、低漏电；但写能量、耐久、变异性差；SRAM 精度高、工艺成熟但面积大 |
| 你的仿真是哪一级？ | 器件/关键链路 SPICE + Monte Carlo；大阵列行为级；架构指标 NeuroSim 类；不暗示流片 |
| 你独立负责什么？ | 按模块讲：阵列映射、外围电路、补偿、仿真；团队工作不揽功 |
| 如果再优化 20%，从哪入手？ | 先看能耗拆分：ADC/DAC、IR drop 导致的精度余量、映射利用率；优先降 ADC 次数/分辨率或提高利用率 |
| 项目的量化指标？ | 说清工艺、阵列大小、单元位数、ADC 位数、频率、TOPS/W、精度、基线 |
| 失败案例？ | 例如漏电/不收敛：定位到正反馈 + 理想运放 + 开关管，加负反馈解决；讲定位过程 |

**常见陷阱**

- 把“模拟 8-bit”说成理所当然，不提 ENOB 和补偿；
- 忽略 ADC/DAC 功耗，只讲 crossbar 的 O(1)；
- 把 in-situ training 说成已成熟，不提更新非线性/耐久；
- 把 elementwise Jacobian 和 GEMM 混为一谈；
- 只说“做了校准”，说不清校准对象和开销。

### 自测

- [ ] 不看笔记，30 秒讲完项目定位。
- [ ] 连续回答“为什么省能耗 → 精度怎么保证 → IR drop 怎么补”三个追问。
- [ ] 给一个你项目里的具体数字，并说清它的基线。
