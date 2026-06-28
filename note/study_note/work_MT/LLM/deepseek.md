---
title: deepseek
tags: ["note"]
created: 星期日, 六月 28日 2026, 2:58:15 下午
modified: 星期日, 六月 28日 2026, 5:29:56 下午
status: todo
---

# Base Model

## 24.01.05 DeepSeekLLM 7B & 67B

论文链接：[https://arxiv.org/abs/2401.02954](https://arxiv.org/abs/2401.02954)

### Abstract

- 开源 DeepSeek LLM 7B 和 67B，可以认为是 LLaMA-2 的复现
- 训练基于 2T Token，SFT + DPO 做 post training
- 评测结果：67B 表现超过 LLaMA-2 70B

### 两个亮点

1. **learning rate scheduler**

	**小改动**，multi-step scheduler 让训练过程更容易复用（因为 cosine scheduler 依赖确定的训练用的 token 数，而在实际训练的过程中，用于训练的数据是会变化的），而模型的表现和使用 cosine scheduler 基本一致。

> A multi-step learning rate scheduler is employed during pre-training instead of the typical cosine scheduler.

1. **针对 scaling law 的详细 " 富有科学精神 " 的研究：确保高效扩容算力的路线正确**

- 业界之前的研究：Chinchilla Scaling Law [https://arxiv.org/abs/2203.15556](https://arxiv.org/abs/2203.15556)

	大体理解成：给定算力地前提下，LLM 参数规模和训练用的数据量的最近分配策略。

> optimal model size and number of tokens for training a transformer language model under a given compute budget.

- DeepSeek 提出了三点改进

	1. 扩展了这个 scaling law 的范畴，讨论了在给定算力的前提下，不同的训练参数（超参数）的最佳分配策略。
	2. 使用 non-embedding FLOPs/token M 来代表模型规模，而不是传统的模型参数 N：直接量化每个 token 在模型核心计算部分（Attention, FFN）的算力消耗
	3. 同时，**伴随着训练过程中数据质量的改进**，DeepSeek 还发现数据质量也显著地影响 LLM 参数规模和训练用的数据量的最近分配策略。

- 产生的结果：

	1. non-embedding FLOPs/token M 是一个更合适的模型规模的表达，对于 MoE 模型和 Dense 模型通用
	2. 基于对于超参数的 scaling law 帮助 DeepSeek 实现了最优的 batch size 和 learning rate 的设置（训练验证的结果也符合他们预测）

### 解读

1. 没有仅仅停留在 " 复现 " LLaMA-2，而是在过程中去探索 Scaling Law 的规律（而不是直接使用经验值）
	- 解读：相比于 " 完成一个 KPI 去复现 LLaMA-2" 的动机，DeepSeek 更关心 how to do it right/efficiently
2. multi-step scheduler 和对于 Scaling Law 的探索其实都是在为了更高效率训练目标服务。
3. why scaling law matters?
	 - 想象你只有一次机会去花几百万刀上千万刀去训练一个大模型，你要怎么去分配你的资源到模型规模，数据集规模，你该如何去找到最符合你的训练目标的超参数？
	- 用小的算力/代价得到一个超参数的拟合函数，从而推导出大算力下最适合的超参数

## 24.01.11 DeepSeekMoE（DeepSeek V2 前序核心工作）

论文链接：[https://arxiv.org/abs/2401.06066](https://arxiv.org/abs/2401.06066)

### MoE 架构发展时间线

1. **2017**：首次将 MoE 与 Transformer 结合，提出 Sparsely-Gated MoE 层
	[https://arxiv.org/abs/1701.06538](https://arxiv.org/abs/1701.06538)
2. **2020**：GShard，首次将 MoE 扩展至 600B 参数，支持超大规模多语言翻译
	[https://arxiv.org/abs/2006.16668](https://arxiv.org/abs/2006.16668)
3. **2023.12**：Mixtral 8x7B，工业界主流开源稀疏 MoE，性能对标 GPT-3.5
	[https://arxiv.org/abs/2401.04088](https://arxiv.org/abs/2401.04088)
4. **2024.01**：DeepSeekMoE，面向极致专家专业化的新一代 MoE 架构，最大规模 145B
	[https://arxiv.org/abs/2401.06066](https://arxiv.org/abs/2401.06066)

### Abstract 核心概述

在大模型扩容阶段，MoE（混合专家）是控制计算成本的主流架构；但传统 GShard 类 Top-K 路由 MoE 存在**专家专业化不足**问题：各专家知识重叠、分工模糊。

本文提出 DeepSeekMoE 架构，两大核心改进实现极致专家细分：

#### 两大核心创新

1. **细粒度专家拆分**

	将专家切分为 `mN` 个细分专家，每层激活 `mK` 个，专家组合更灵活，提升单一专家知识专一性

> finely segmenting the experts into mN ones and activating mK from them, allowing for a more flexible combination of activated experts

1. **隔离共享专家**

	固定保留 Ks​ 个全程激活的共享专家，统一承载通用基础常识，大幅降低路由专家间知识冗余

> isolating Ks​ experts as shared ones, aiming at capturing common knowledge and mitigating redundancy in routed experts

### 分层缩放实验结果（2B → 16B → 145B）

1. **2B 小规模验证**

	DeepSeekMoE-2B 性能对标 GShard-2.9B；后者专家参数量、计算量是前者 1.5 倍。

	同等总参数量下，DeepSeekMoE-2B 性能逼近同规模稠密模型（MoE 性能理论上限）。

2. **16B 中规模验证**

	DeepSeekMoE-16B 综合性能持平 LLaMA2-7B，仅需约 **40%** 计算量。

3. **145B 大规模验证**

	145B DeepSeekMoE 性能与稠密模型 DeepSeek-67B 对齐，仅消耗 **28.5%** 计算量，最优场景可低至 **18.2%**。

	全尺寸实验持续验证：相比传统 GShard 架构，DeepSeekMoE 在参数效率、推理成本上具备显著优势。

