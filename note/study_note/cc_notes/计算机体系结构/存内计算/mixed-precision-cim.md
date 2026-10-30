---
tags: [arch, cim]
updated: 2026-10-19
---

# 混合精度存内计算

## INT8/INT4 量化误差怎么分析？ #arch #cim #quantization

> **速查**
> - 均匀量化：$q=\text{clip}(\text{round}(x/s)+z)$；量化噪声方差约 $\Delta^2/12$
> - 满量程正弦 SNR $\approx 6.02b+1.76$ dB；INT4 比 INT8 少约 24 dB
> - 输出误差 $\lesssim \|X\|\|\Delta W\|+\|\Delta X\|\|W\|$
> - per-channel/per-group scale 降低 outlier 影响；高敏感层用 INT8
> - 器件噪声叠加在量化噪声上，有效位数低于标称位数

### 正文

**① 量化模型**

均匀仿射量化：

$$
q=\text{clip}\left(\text{round}\left(\frac{x}{s}\right)+z,\;-2^{b-1},\;2^{b-1}-1\right),
\qquad \hat{x}=s(q-z)
$$

- $s$：scale，量化步长；
- $z$：zero-point，保证实数 0 能精确表示；
- $b$：位宽。

量化误差 $e=x-\hat{x}$ 在 $[-\Delta/2,\Delta/2]$ 内近似均匀，$\Delta=s$，所以：

$$
\sigma_e^2 \approx \frac{\Delta^2}{12}
$$

**② 一个具体数字**

假设权重范围 $[-1,1]$，先按 $2^b$ 个量化级估算：

- INT8：$\Delta_8=2/256=7.81\times10^{-3}$；
- INT4：$\Delta_4=2/16=0.125$。

噪声方差 $\sigma^2=\Delta^2/12$，所以噪声功率比：

$$
\frac{\sigma_4^2}{\sigma_8^2}=\left(\frac{\Delta_4}{\Delta_8}\right)^2=16^2=256
$$

即约 $24.1$ dB 的 SNR 差距。用满量程正弦公式核对：

$$
\text{SNR}_8\approx6.02\times8+1.76=49.9\ \text{dB},\qquad
\text{SNR}_4\approx6.02\times4+1.76=25.8\ \text{dB}
$$

差 $24.1$ dB，一致。

**一个容易踩的细节**：对称量化常用 $2^b-1$ 个间隔（INT8 255 个、INT4 15 个），此时步长比是 $255/15=17$，噪声功率比 $289$，约 $24.6$ dB。面试说“约 24 dB”即可，但要能解释 16 和 17 的差别来自“按 $2^b$ 级还是 $2^b-1$ 个间隔”。

**③ 误差怎么传到输出**

对 $Y=XW$：

$$
\Delta Y=\Delta X\,W+X\,\Delta W+\Delta X\,\Delta W
$$

一阶界：

$$
\|\Delta Y\|\lesssim\|X\|\,\|\Delta W\|+\|\Delta X\|\,\|W\|
$$

所以：

- 权重量化误差被输入幅值放大；
- 激活量化误差被权重幅值放大；
- 两者都大的层更敏感；
- per-channel scale 能让每个输出通道的 $\Delta$ 更小，尤其是权重分布有 outlier 时。

**④ 器件噪声与有效位数**

实际 CIM 的误差不止量化：

- 电导编程误差、漂移、RTN/1/f 噪声；
- ADC 量化/offset/gain；
- IR drop、sneak path。

这些叠加后，系统的**有效位数（ENOB）**低于标称位宽。面试时要说“标称 INT8”还是“有效 6–7 bit”，不要混。

**⑤ 敏感度分析**

- **逐层**：固定其他层，改变某一层位宽，看端到端精度；
- **Hessian/HAQ**：用二阶信息估计每层对损失的敏感度；
- **块级**：按 crossbar 操作单元（OU）分块，块内同精度；
- **搜索**：敏感度排序 + 强化学习/贪心分配位宽。

相关：[[rram-cim#RRAM 存内计算的基本原理是什么？]]

### 自测

- [ ] 权重范围 $[-1,1]$，INT8 和 INT4 的量化步长、噪声方差、SNR 差各是多少？
- [ ] 写出 $\Delta Y$ 的一阶界，并说明哪一层更敏感。
- [ ] 为什么“标称 8 bit”不等于“有效 8 bit”？

---

## Bit-slicing 怎么把 INT8/INT4 映射到 MVM？ #arch #cim #quantization

> **速查**
> - 权重切片：INT8→2×4-bit 或 8×1-bit；输出按 $2^{4k}$ 移位相加
> - 输入切片：INT8→8 个 1-bit 周期；INT4→4 个周期
> - 权重切片平均噪声/漂移；输入切片对精度影响较小
> - ADC 量程要按 slice 设置，否则精度收益被量化吃掉
> - 切片越多，延迟和 ADC 次数越多

### 正文

**① 为什么需要切片**

RRAM 单元通常只有 1–4 bit 的稳定电导态，而网络权重/激活可能是 INT8。切片就是把高位宽数拆成多个低位宽 slice，分别做 MVM，再按权重移位相加。

**② 权重切片**

INT8 权重 $W$ 拆成两个 4-bit slice：

$$
W=16\,W_H+W_L
$$

两个 slice 分别映射到两组单元（或两个阵列）。MVM 结果：

$$
XW=X(16W_H+W_L)=16\,(XW_H)+(XW_L)
$$

硬件上就是两个部分和，低 slice 结果直接加，高 slice 结果左移 4 位再加。

**③ 输入切片**

INT8 输入 $X$ 拆成 8 个 1-bit 周期：

$$
X=\sum_{k=0}^{7}2^k x_k
$$

每个周期输入一位，阵列做一次 MVM，结果按 $2^k$ 累加。也可以拆成 2 个 4-bit 周期，减少周期数但需要 4-bit DAC。

**④ 手算例子**

$W=90=0x5A$，$x=19=0x13$，拆成 4-bit slice：

- $W=5\times16+10$；
- $x=1\times16+3$；
- 乘积：
  $$
  (5\times16+10)(1\times16+3)
  =5\times1\times256+(5\times3+10\times1)\times16+10\times3
  $$
  $$
  =1280+(15+10)\times16+30=1280+400+30=1710
  $$
- 直接算 $90\times19=1710$。

四个部分积对应四个 slice 组合，硬件按 $2^4$ 移位后相加。

**⑤ 权重切片 vs 输入切片**

| | 权重切片 | 输入切片 |
|---|---|---|
| 拆谁 | 权重 | 激活 |
| 硬件 | 多组单元/多阵列 | 多周期扫描 |
| 噪声 | 切片平均降低编程噪声/漂移 | 对器件噪声平均作用弱 |
| 精度影响 | 通常更有利于精度 | 文献认为影响较小 |
| 代价 | 面积/单元数增加 | 延迟/ADC 次数增加 |

Le Gallo 2022 的结论：

- 权重切片能平均编程误差、噪声、漂移；
- 输入切片对整体精度影响不大；
- **ADC 量程要按切片数重新设置**，否则切片收益被 ADC 量化吃掉；
- 权重切片用等显著性（base 1）更利于噪声平均。

**⑥ 切片与 ADC**

切片后每列电流的动态范围变了。若 ADC 仍按全精度满量程设计，低 significance slice 只用掉很少的码字，等效位数下降。正确做法是每个 slice 用匹配的参考/量程，或者用可配置 ADC。

相关：[[rram-cim#GEMM 怎么映射到 crossbar 阵列？]]

### 自测

- [ ] $W=90$、$x=19$ 拆成 4-bit slice，写出四个部分积和移位相加。
- [ ] 权重切片和输入切片各自解决什么问题、付出什么代价？
- [ ] 为什么 ADC 量程不匹配会吃掉 bit-slicing 的精度收益？

---

## 混合精度怎么分配、硬件怎么支持？ #arch #cim #quantization #mixed-precision

> **速查**
> - 粒度：层 / 块 / 通道；块级通常匹配 crossbar 操作单元
> - 敏感度：Hessian/HAQ/RL 搜索；高敏感 INT8，低敏感 INT4
> - 硬件：可重构 slice、precision-scalable ADC、混合精度数据流
> - 残差修正：低精度主路径 + 高精度残差（MORE-CIM）
> - 评价看精度、延迟、能耗、面积四维，不只看位宽

### 正文

**① 为什么要混合精度**

全局 INT8 浪费能效，全局 INT4 精度不够。不同层/块对量化误差的敏感度不同，所以按敏感度分配位宽：敏感层 INT8，不敏感层 INT4。

**② 分配粒度**

| 粒度 | 优点 | 缺点 |
|---|---|---|
| 层 | 简单，编译器好做 | 层内不均匀，浪费 |
| 块 | 匹配 crossbar 操作单元，硬件好映射 | 需要块级 scale/控制 |
| 通道 | 精细，适合 outlier | 控制/存储开销大 |

BWQ 的做法是把权重块大小匹配到操作单元（OU），块内同精度、块间不同精度，这样同一 crossbar 可以按块配置。

**③ 敏感度与搜索**

- **Hessian 迹/HAQ**：估计每层/块对损失的二阶敏感度；
- **逐层扫描**：固定其他层，改变一层位宽，测端到端精度；
- **强化学习**：把位宽选择建模为序列决策，奖励 = 精度 − 延迟/能耗惩罚；
- **聚类**：把敏感度相近的块聚成几档（如 INT4/INT6/INT8），减少配置种类。

**④ 硬件支持**

- **可重构 slice**：同一阵列按配置拆成 1/2/4 个 slice，支持不同位宽；
- **precision-scalable ADC**：按当前 slice 调整分辨率/量程，避免固定高精度 ADC 的浪费；
- **混合精度数据流**：不同块的结果在数字域按各自 scale 合并；
- **残差路径**：低精度主路径 + 高精度残差分支，修正系统误差。

**⑤ 残差修正**

MORE-CIM 的思路：

- 主路径用 MXINT4 做大部分计算；
- 残差路径用 MXINT8 重建主路径的量化/硬件误差；
- 残差只处理误差，不需要全精度重算。

代价是面积/能量增加，但比全局 INT8 便宜。论文数据：残差路径 +6.5% 面积、+5.31% 动态能量。

**⑥ 评价指标**

不要只说“用了 INT4”。要给出：

- 端到端精度（相对 FP32/INT8 基线）；
- 延迟（slice 数 × 周期）；
- 能耗（ADC/DAC/数字累加/写）；
- 面积（slice 单元、ADC、控制）；
- 编译/校准开销。

相关：[[rram-cim#精度补偿机制分哪几层？]]

### 自测

- [ ] 层/块/通道三种粒度各适合什么场景？
- [ ] 敏感度分析怎么做？为什么不能只看权重量化误差？
- [ ] 残差修正为什么比全局 INT8 便宜？

---

## 论文清单：混合精度 CIM 该读哪几篇？ #arch #cim #quantization #mixed-precision

> **速查**
> - 概念起点：Le Gallo 2018
> - 切片精度：Le Gallo 2022
> - 混合精度分配：BWQ 2024、Sensitivity-aware 2025、MP compiler 2026
> - 硬件/残差：MORE-CIM 2026、MINT 2020
> - 量化误差基础：任何量化教材/论文的 SNR 公式

### 正文

| 论文 | 链接 | 一句话 |
|---|---|---|
| Le Gallo et al., *Mixed-precision in-memory computing*, Nat. Electron. 2018 | https://doi.org/10.1038/s41928-018-0054-8 | 模拟单元做 bulk、数字单元做高精度残差的原始概念 |
| Le Gallo et al., *Precision of bit slicing with in-memory computing*, NCE 2022 | https://doi.org/10.1088/2634-4386/ac4fb7 | 权重切片 vs 输入切片、ADC 量程的系统分析 |
| Wu et al., *Block-Wise Mixed-Precision Quantization*, TCAD 2024 | https://doi.org/10.1109/TCAD.2024.3409193 | 块级混合精度 + ReRAM 加速器映射 |
| Chen et al., *Sensitivity-Aware Mixed-Precision Quantization for ReRAM-based CIM*, arXiv 2025 | https://arxiv.org/abs/2512.19445 | Hessian 敏感度 + 聚类分配位宽 |
| Pelke et al., *Mixed-Precision Training and Compilation for RRAM-based CIM*, arXiv 2026 | https://arxiv.org/abs/2601.21737 | 1–8 bit 混合精度训练/编译，RL 搜索配置 |
| Zhou et al., *MORE-CIM*, TCAS-I 2026 | https://doi.org/10.1109/TCSI.2026.3737166 | MXINT4 主路径 + MXINT8 残差修正 |
| Jiang et al., *MINT*, ISCAS 2020 | https://doi.org/10.1109/ISCAS45731.2020.9181020 | 混合精度 RRAM in-memory training |
| Zhu et al., *Configurable Multi-Precision CNN Framework*, DAC 2019 | https://doi.org/10.1145/3316781.3317739 | 单比特 RRAM 的多精度计算框架 |
| Ghosh et al., *Approximate ADCs for In-Memory Computing*, arXiv 2024 | https://arxiv.org/abs/2408.06390 | 近似 ADC 的精度/能耗权衡 |

**阅读顺序**

1. Le Gallo 2018 → 建立“模拟 bulk + 数字残差”的框架；
2. Le Gallo 2022 → 搞清 bit-slicing 的精度机制；
3. BWQ 2024 → 看块级混合精度怎么落到硬件；
4. MORE-CIM 2026 → 看残差路径的工程实现；
5. MP compiler 2026 → 看编译/训练侧怎么支持 1–8 bit。

### 自测

- [ ] 哪篇论文直接回答“权重切片还是输入切片”？
- [ ] 哪篇论文把块级混合精度映射到 ReRAM 加速器？
- [ ] MORE-CIM 的残差路径解决什么问题、代价是什么？

---

## 面试背诵卡：混合精度 CIM #arch #cim #quantization #mixed-precision

> **速查**
> - 30 秒：分析 INT8/INT4 量化误差，做 bit-slicing 和逐层/逐块精度分配，映射到 MVM 并评估精度-能耗-延迟
> - 必答：INT4 为什么掉点、怎么分析、怎么分配、硬件怎么支持、残差是什么
> - 回答顺序：误差来源 → 量化模型 → 切片映射 → 分配策略 → 硬件代价
> - 不夸大：参与就说参与，仿真就说仿真
> - 每个结论都要有数字或论文支撑

### 正文

**30 秒项目口述**

> 我参与混合精度 CIM：分析 INT8/INT4 的量化误差和器件噪声对 MVM 的影响，做 bit-slicing 和逐层/逐块精度分配，把不同精度映射到阵列，并评估精度、能耗、延迟的 trade-off。

**高频问答**

| 问题 | 回答要点 |
|---|---|
| INT4 为什么掉点？ | 量化步长约是 INT8 的 16–17 倍（取决于按 $2^b$ 级还是 $2^b-1$ 个间隔），噪声功率约 256–289 倍（约 24 dB）；再叠加器件噪声，有效位数更低 |
| 怎么分析量化误差？ | 均匀量化模型 + $\Delta^2/12$ + SNR $6.02b+1.76$；逐层/逐块敏感度；端到端精度验证 |
| bit-slicing 权重 vs 输入？ | 权重切片平均噪声/漂移，利于精度；输入切片对精度影响小、主要增加周期；ADC 量程要匹配 |
| 混合精度怎么分配？ | 层/块/通道；Hessian/HAQ/RL 搜索；高敏感 INT8，低敏感 INT4；块级匹配 OU |
| 硬件怎么支持？ | 可重构 slice、precision-scalable ADC、混合精度数据流、残差路径 |
| 残差修正是什么？ | 低精度主路径 + 高精度残差分支，只重建误差，不全精度重算（MORE-CIM） |
| 为什么不用全局 INT8？ | 能效/延迟浪费；不敏感层用 INT4 可以省 slice/周期 |
| 为什么不用全局 INT4？ | 敏感层精度不够；端到端掉点，需要混合精度/残差 |
| 你的参与边界？ | 说清你负责量化分析、映射、仿真中的哪一块；不要把团队工作全说成自己的 |
| 怎么验证？ | PyTorch QAT + 硬件噪声模型 + 端到端精度；行为级/架构级指标；有实测再单独说 |

**常见陷阱**

- 只讲“INT4 更省”，不讲精度/敏感度；
- 把标称位宽当有效位数；
- 忽略 ADC 量程与切片匹配；
- 把“混合精度”说成“每层随便选”；
- 残差路径的代价（面积/能量）不提。

### 自测

- [ ] 不看笔记，30 秒讲完混合精度项目的定位。
- [ ] 连续回答“INT4 为什么掉点 → 怎么分配 → 硬件怎么支持”三个追问。
- [ ] 给出一个你项目里的精度/延迟/能耗数字，并说清基线。
