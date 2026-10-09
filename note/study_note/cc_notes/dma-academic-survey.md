---
tags: [dma, survey]
updated: 2026-10-10
created: 星期六, 十月 10日 2026, 12:37:43 凌晨
modified: 星期六, 十月 10日 2026, 12:46:05 凌晨
---

# 多 die / chiplet 数据搬运：学术现状综述

> **检索日期 2026-10-10**，覆盖 ISCA / MICRO / HPCA / ASPLOS / DAC / NeurIPS / 期刊与 arXiv。
>
> **可信度约定**
> - **【检索确认】** —— 联网检索中实际看到标题 + 会议/年份（多数有 DOI/URL）的条目。⚠️ **检索返回的是二手摘要片段，未逐篇打开原文核对技术细节**；未能确认的字段（页码、卷期、具体数字）会标注"未确认"。
> - **【背景推断】** —— 文末单列一节，与检索结果**严格分开**。
>
> **本文不编造任何论文标题、作者或结论。查不到就写"未找到"。**

## 先纠正几个网上流传的错误引用 #dma #survey

> **速查**
> - ⚠️ **"CePIDE" 并不存在** —— MICRO 2024 上关于多芯粒 GPU 隐式同步的论文实际叫 **CPElide**
> - ⚠️ **"Tangram HPCA 2023" 不存在** —— Tangram 是 **ASPLOS** 论文；HPCA 上相关的芯粒 DNN 加速器论文是 **Gemini（HPCA 2024）**
> - ⚠️ **"near-memory DMA" / "in-network DMA" 作为专门术语，未检索到以此命名并聚焦的论文**
> - ⚠️ **Hecaton 的正式发表 venue 未能确认** —— 只能确认是 arXiv 预印本（arXiv:2407.05784，清华）。网上流传的 "HPCA 2025" 说法**未找到证据**
> - ⚠️ **"REX 是 HPCA 2024" 不成立** —— REX 发表在 *Future Generation Computer Systems*，vol. 178, 2026
> - ⚠️ **"distance-aware inter-chiplet interconnect" 未找到以此为名的论文** —— MICRO 2023 那篇的主题是**异构（heterogeneous）**die-to-die 接口

> **这一节本身就是有用的方法论提醒**：这个领域的中文二手资料里存在互相抄错的引用。**引用前务必回落原论文。**

---

## 速查：这个领域在做什么 #dma #survey

> **速查**
> - **奠基工作**：**MCM-GPU（ISCA 2017，NVIDIA/普林斯顿）** —— 用硅中介层把多个 GPU 模块拼成一个大 GPU。此后几乎**所有**多芯粒 GPU 论文都以它为基线
> - **Simba（MICRO 2019，Best Paper）** —— 第一个芯粒化 DNN 推理加速器，36 个 chiplet（6×6），NoP 带宽 100 GB/s/chiplet、~20 ns/hop
> - **研究重心已转移**：从"**能不能把多个 die 拼起来**" → "**如何为跨 die 搬运设计正确的抽象与载体**"
> - 现在的三个主战场：**一致性/同步层**（HMG、CPElide）、**显式搬运引擎**（iDMA、XDMA、Memory Fabric 的 DME）、**精确的 D2D PHY 建模**（DICE、Omelet）
> - **三个被反复点名、尚未解决的空白**：① **D2D 带宽/能耗** ② **跨 die 一致性/同步的可扩展性** ③ **可复现、参数统一的芯粒仿真与基准**

---

## 主要工作分类 #dma #survey

### 一、奠基性多芯片 / 芯粒 GPU 与 DNN 加速器

| 工作 | 会议/年份 | 一句话贡献 |
|---|---|---|
| **MCM-GPU** | ISCA 2017 | 硅中介层拼多 GPU 模块 + 必需的内存系统/一致性改造。**后续所有工作的基线** |
| **Simba** | MICRO 2019（Best Paper） | 首个芯粒化 DNN 推理加速器；36 chiplet，NoP 100 GB/s/chiplet、~20 ns/hop |
| **Modular Routing Design** | ISCA 2018 | 多芯粒系统在 active interposer 上的**免死锁模块化路由**（对进出 chiplet 的流量施加 turn restriction） |
| **NN-Baton** | ISCA 2021 | 芯粒粒度与 DNN 映射编排；相对 Simba 省 22.5%–44% 能耗 |
| **Gemini** | HPCA 2024 | **首个把映射与架构联合探索**用于大规模 DNN 芯粒加速器；比 Simba+Tangram 平均 1.98× 性能、1.41× 能效，成本仅高 14.3% |
| **Chiplet Actuary** | DAC 2022 | **成本模型**。结论：<200 mm² 无需芯粒；~800 mm² 以上才有收益 |
| **Enhancing Collective Communication in MCM Accelerators** | HPCA 2024 | 面向 MCM 加速器的 DNN 训练**集合通信**优化 |

### 二、芯粒互连网络 / die-to-die 接口（载体层）

| 工作                                                                                     | 会议/年份           | 一句话贡献                                                                                                          |
| -------------------------------------------------------------------------------------- | --------------- | -------------------------------------------------------------------------------------------------------------- |
| **A Scalable Methodology for Designing Efficient Interconnection Network of Chiplets** | HPCA 2023       | 不同规模用不同拓扑，但接口是"统一串行接口"——**这正是后续异构接口工作的动机**                                                                     |
| **Heterogeneous Die-to-Die Interfaces**                                                | MICRO 2023      | 声称首次讨论**异构 D2D 接口**（Hetero-PHY、Hetero-Channel）+ 免死锁路由与调度                                                       |
| **Ring Road**                                                                          | MICRO 2024      | 隔离多环 + 紧凑路由器 + **极坐标 2D 拓扑描述** + 免死锁路由，隔离片内/片间流量；1.7–2× 带宽/面积                                                  |
| **FlooNoC**                                                                            | IEEE TVLSI 2025 | 开源 NoC，**多流 DMA 引擎**做端到端 AXI4 排序；645 Gb/s/link、0.15 pJ/B/hop                                                   |
| **An IO Die with Collective-Aware Routing and In-Situ Processing**                     | venue 未确认       | 针对 D2D 点对点不适合 broadcast/gather/all-reduce，提出集合感知调度器 + In-Situ Data Processing Engine；同步开销降幅达 86%（数字来自摘要，未核对原文） |

### 三、多芯粒 GPU 的缓存一致性 / 同步 / 内存系统（**搬运"发生"在这一层**）

| 工作 | 会议/年份 | 一句话贡献 |
|---|---|---|
| **HMG** | HPCA 2020 | 面向多 GPU/MCM 的 **VI 类分层一致性协议**，利用 GPU 内存模型的 scoped / non-multi-copy-atomic 性质，避免失效确认与瞬态；4-GPU 上比软件 bulk-invalidation 快 26%，达理想化缓存 97% |
| **CPElide** | MICRO 2024 | 利用 command processor 已有的内核元信息，把保守的"全 GPU 失效"变成**按芯粒按需的隐式 acquire/release**，**无需改硬件**；24 个负载上平均 13% 性能、14% 能耗、14% 网络流量改善 |
| **Barre Chord** | ISCA 2024 | MCM GPU 的**虚拟内存地址翻译可扩展性** |
| **NearFetch** | HPCA 2025 | 提出 **IBPA（Inter-Module Bandwidth per Access）**度量，用路由转发 + 拓扑感知 MSHR 合并让"近处模块"供数；IBPA 降 42.6%，高共享负载性能平均 +52.2% |
| **Optimizing Attention by Exploiting GPU Architectural NUMA Effects** | arXiv（venue 未确认） | attention head 与 GPU NUMA 域对齐；MI300X 上最高 +50% 性能 |
| **Fleet** | arXiv（venue 未确认） | "Chiplet-task" 抽象层，处理"L2 在编程模型里是 device-scoped、但物理上按芯粒切分"的矛盾 |
| **REX** | *FGCS* vol.178, 2026 | ⚠️ **不是 HPCA**。**把线程迁移到数据所在 chiplet**（而非搬数据），面向不规则访存；平均 +13%、最高 +38% |
| **Memory Fabric** | IEEE CAL vol.25 no.2, 2026 | **在 HBM base die 上未使用的 "memory shoreline" 用 D2D 直连相邻 HBM stack**，绕开 compute chiplet 做**内存到内存直搬**；每个 stack 加轻量 **DME（Data Movement Engine）** 承担 MoE dispatch/combine 的布局变换。远程 HBM 带宽最高 +75% |
| **Memory-centric MCM-GPU architecture** | IEEE CAL vol.24 no.1, 2025 | 把 HBM stack 连到中介层而非 GPU，实现片上/远程带宽动态分配；平均 1.36×（最高 1.90×） |

### 四、面向 LLM 的芯粒加速器

| 工作 | 会议/年份 | 一句话贡献 |
|---|---|---|
| **Cambricon-LLM** | MICRO 2024 | NPU + 专用 NAND flash 芯粒混合架构，**用 chiplet D2D 链路直连 NPU 与 flash** 减少搬运；70B 达 3.44 token/s |
| **H²-LLM** | ISCA 2025（Best Paper） | hybrid bonding 异构加速器 + 以数据为中心的数据流抽象；比 in-DRAM NMP 快 2.72× |
| **WSC-LLM** | ISCA 2025 | 晶圆级芯片的 LLM 服务-架构联合探索 |
| **FRED** | ISCA 2025 | 面向 3D 并行的晶圆级 **reduction-distribution 互联**（microswitch 内置归约/广播）；网络流量约 2× 降低，训练时间 1.34–1.87× 改善 |
| **Hecaton** | arXiv:2407.05784（**正式 venue 未确认**） | 面向 LLM 训练/微调的芯粒架构 + 调度；D2D 参数取自 UCIe |
| **Mozart** | NeurIPS 2025 Spotlight | 3.5D 芯粒（逻辑 die + SRAM die hybrid bonding）+ 2.5D NoP-Tree；MoE 训练 1.9×+ 加速 |
| **McPAL** | DAC 2025 | 多芯粒 HBM-PIM，纵向重排 3D-HBM 层级减少长距离搬运，横向用 2.5D I/O chiplet 减少 D2D；1.57–3.12× |
| **Patterns behind Chaos** | 标注 ISCA'26（venue 未独立确认） | 面向晶圆级 GPU 的 MoE 推理，做**数据搬运预测的 command processor**，降低 D2D 流量 |
| **Sangam** | venue 未确认 | 逻辑/DRAM 分拆到不同工艺节点的芯粒 + 中介层互连 + CXL 挂载 |

### 五、显式数据搬运引擎（DMA / DME）—— **对你最直接相关**

| 工作 | 出处 | 一句话贡献 |
|---|---|---|
| **iDMA** | IEEE TC（卷期未确认） | **前端（控制）/中端（描述符切片，N 维张量）/后端（片上网络数据面）分层的参数化 DMA**；16 B 传输即达满总线利用率，<25 kGE。**在 Manticore 双芯粒多核上，每 cluster 加 iDMA 后密集负载最高 1.5×、稀疏 8.4×，面积仅 +2.1%** |
| **XDMA** | arXiv:2508.08396 | 硬件数据流引擎替代软件地址生成器 + **分布式 DMA（配置与搬运分离）** + **在线数据变换插件**；FPGA 原型上链路利用率最高 151× 提升 |
| **Occamy** | arXiv:2501.07330（venue 未确认） | 双芯粒 RISC-V 系统，D2D 链路分窄（1.33 Gbit/s 同步）/宽（64 Gbit/s 批量）两段 |

### 六、GPU 侧搬运（copy engine / TMA）

| 工作 | 出处 | 一句话贡献 |
|---|---|---|
| **Dissecting the NVIDIA Hopper Architecture** | ACM TOCS / arXiv:2501.12084 | 系统微基准 Hopper 存储子系统与 **TMA**、DSM、DPX；大矩阵 GEMM 开 TMA 约 600 vs 450 TFLOPS |
| **Benchmarking and Dissecting the NVIDIA Hopper GPU Architecture** | IEEE IPDPS 2024 | Hopper 微基准 |
| **ACTA** | ACM，DOI 10.1145/3725798 | 为 TMA kernel **自动推断最优 tile 大小与 queue-slot 配置** |

### 七、建模 / 仿真 / 综述

| 工具/工作 | 出处 | 一句话贡献 |
|---|---|---|
| **MGPUSim** | ISCA 2019 | 开源 cycle-level 多 GPU 模拟器（AMD GCN3），与真实硬件平均偏差 5.5%；**被大量多芯粒论文用作 MCM 建模基座** |
| **MuchiSim** | arXiv:2312.10244 | 可扩展到百万级处理单元的并行多芯粒模拟器，逐周期建模搬运 |
| **RapidChiplet** | ACM CF'25 | 毫秒级 ICI 延迟/吞吐/面积/功耗/热/成本**代理模型** + BookSim2 精确校核；延迟代理相对 BookSim2 约 1075× 加速、~2.57% 误差 |
| **Omelet** | ISCA 2026 | 统一建模 NoC（片内）/ NoI（中介层）/ NoL（层间）；link 参数来自 EM 提取与 SPICE 紧凑模型 |
| **DICE** | ISCA 2026 | ⭐ 在 gem5 内建**端到端 PHY 模型**（QC-LDPC FEC、PAM4、AWGN 信道、自适应重传、PHY 级流控） |
| **CHIPSIM** | venue 未确认 | 用 HeteroGarnet 按实测读/写带宽配置异构链路 |
| **Survey of Chiplet Technology** | IEEE JETCAS 2025 | 覆盖 200+ 论文的四领域综述（SoC 架构 / 互连 / EDA / 先进封装） |

---

## 公开设计方案与取舍 #dma #survey

| 方案类别 | 代表工作 | 核心做法 | 取舍 |
|---|---|---|---|
| **统一 D2D 接口 + 多种拓扑** | Feng/Xiang/Ma HPCA 2023 | 不同规模用不同拓扑，接口统一 | 统一串行接口对小规模系统的延迟/能耗偏大 |
| **异构 D2D 接口** | MICRO 2023 | 按场景混用不同 PHY/通道 | 更灵活，但接口不统一、验证与路由复杂度上升 |
| **距离/介质感知的 NoC** | Ring Road (MICRO 2024) | 隔离多环 + 紧凑路由器 + 极坐标拓扑 | 省路由器开销；牺牲通用性，路由理论需扩展 |
| **内存侧直搬（绕开 compute die）** | Memory Fabric (CAL 2026) | HBM base die 上加 D2D I/O + DME | 省 compute chiplet 的 shoreline 与带宽；需定制 HBM base die |
| **就近供数（路由转发）** | NearFetch (HPCA 2025) | 路径上的近端模块供数 + MSHR 合并 | 降 IBPA；**依赖数据局部性，收益随局部性波动** |
| **线程迁到数据侧** | REX (FGCS 2026) | 远程 warp 在数据所在 chiplet 执行 | 省搬数、适合不规则访存；需编译器/运行时支持 |
| **一致性与同步层减少搬运** | HMG / CPElide | 分层 VI 协议；按芯粒按需 acquire/release | 减少无效失效与流量；引入目录/CP 查找开销 |
| **显式搬运引擎** | **iDMA / XDMA / FlooNoC / Occamy D2D** | 分层/参数化 DMA、N 维描述符、在线变换 | **小传输即高利用率；面积/功耗开销小（~2%）** |
| **光互连** | SPACX / SPRINT / ReSiPI / TRINE | 波长复用做广播/单播 | 带宽密度高、能耗低；激光功耗、热漂移、制造良率是代价 |
| **晶圆级集合通信** | FRED (ISCA 2025) | switch 内置归约/广播 | 网络流量 ~2× 降低；只面向晶圆级 |
| **3.5D 逻辑-on-存储** | Mozart (NeurIPS 2025) | 逻辑 die + SRAM die hybrid bonding + NoP-Tree | 垂直带宽高；SRAM 容量受限、switch 高负载时成瓶颈 |
| **片内 TMA/描述符搬运** | ACTA / Hopper TMA | 硬件描述符驱动的 tile 搬运 | 提高 GEMM/attention 吞吐；**提升依赖大型规则 tile，灵活性有限** |

---

## 尚未解决的问题（被反复点名） #dma #survey

> **这一节对选题最直接——被反复点名的 open problem 就是可能的贡献点。**

1. **把 D2D 当"固定延迟链路"会系统性失真。**
   **DICE（ISCA 2026）**明确指出：HeteroGarnet 等固定延迟模型会扭曲 packet 延迟构成，**IPC 平均偏 6.8%、最高 27.6%**。
   → "如何以可负担的成本精确建模 D2D PHY"仍是开放问题。

2. **D2D 带宽与能耗是芯粒加速器的头号瓶颈。**
   Gemini（HPCA 2024）与多篇 2024 综述反复指出：D2D 接口比片内互连**面积更大、功耗更高、带宽更低**——"带宽墙 + 能耗墙"。

3. **跨 D2D 的一致性不可扩展。**
   snoop/MESI 广播在带宽受限的 D2D 链路上会造成 **coherence storm**，趋于目录/混合协议。

4. **片内 L2 私有化导致跨芯粒复用丢失。**
   CPElide 的核心动机：kernel 边界的隐式同步使整个 L2 失效，破坏 inter-kernel reuse。

5. **NUMA 效应从"设备间"下移到"设备内"。**
   跨芯粒访存约 **2× 延迟、~½ 有效带宽**。

6. **大规模芯粒下通信开销主导。**
   转述数据：N≥9 时通信占比从 N=2 的 ~30% 升到 N=18 的 **>70%**——超过某规模后互连成为主项。

7. **仿真与配套参数缺乏统一基准。**
   多篇 2025 综述明确：**缺乏标准化的 AI 流量 trace、floorplan、封装参数、热栈**。

8. **设计空间不可穷举。**
   联合 架构/精度/数据流/硬件 的搜索空间 **>10^18**。RapidChiplet、CHASE、Gemini 都在试图用代理模型/多目标优化压缩它——**"可负担、可复现的芯粒 DSE"本身是开放问题**。

9. **热-搬运-可靠性耦合。**
   3D 堆叠热点 >200 W/cm²、DRAM 85°C 限制会拉低 30–50% 带宽；"热-安全耦合"研究尚处早期。

10. **晶圆/中介层级拓扑与流量匹配。**
    Taming the Tail：**NoI（Network-on-Interposer）**上的突发流量放大尾延迟、违反 SLA——**NoI 拓扑如何随混合负载自适应**是开放问题。

11. **"搬运"的显式管理 vs 隐式透明。**
    "Patterns behind Chaos" 点出该张力：**把优化全交给硬件、软件无法显式管理跨芯片搬运**，因此架构层优化变得关键。

### 自测
- [ ] DICE 那篇指出的"把 D2D 当固定延迟会失真"具体失真是多少？
- [ ] 为什么说一致性协议在 D2D 上不可扩展？症状叫什么？
- [ ] 列出三个"被反复点名但未解决"的空白。

---

## 常用评价方法 #dma #survey

### 仿真器（按被使用频次）

| 工具 | 类型 | 典型用途 |
|---|---|---|
| **gem5** | 全系统 cycle-level | CPElide 用它建 4/6/7-chiplet GPU；DICE 在 gem5 内加 PHY 模型 |
| **MGPUSim** | 多 GPU cycle-level（AMD GCN3） | MCM/多 GPU 建模基座 |
| **GPGPU-Sim** | GPU cycle-level | 与 x86 CMP 联用评估异构芯粒 |
| **BookSim2** | NoC 精确仿真 | RapidChiplet 的精确校核后端 |
| **HeteroGarnet** | 片上网络（含异构链路） | CHIPSIM 配置异构链路；**DICE 的对比基线** |
| **MuchiSim** | 并行多芯粒 manycore | 百万级处理单元、逐周期搬运 |
| **RapidChiplet** | 快速 DSE 工具链 | 毫秒级代理 + BookSim2 校核 |
| **Omelet** | 封装感知分层互连 | NoC/NoI/NoL 统一 |
| **DICE** | gem5 插件 PHY 模型 | 端到端 D2D PHY（LDPC/PAM4/AWGN） |
| **Timeloop / Orojenesis** | 数据搬运下界分析 | ski-slope 图（"Mind the Gap"） |

### Benchmark

- **传统 GPGPU/HPC**：Rodinia、PolyBench、SHOC、HeteroMark
- **ML/DNN**：DNNMark；ResNet-50/152、VGG-16、DenseNet-201、EfficientNet-B7；BERT、GPT、Llama 系列
- **LLM**：Llama2-70B / Llama3.1-405B（Hecaton）、Qwen3-30B-A3B、OLMoE-1B-7B、DeepSeek-MoE-16B（Mozart）
- **图/其他**：Color、FW、SSSP（CPElide）
- **合成/微基准**：Xtreme suite、TMA 微基准

### 常用指标

- **性能/延迟**：执行时间、加速比、IPC、kernel 时间、tail latency / worst-case slowdown
- **带宽/流量**：**IBPA**（NearFetch 自定义）、网络流量、hop 数、链路利用率、achieved 带宽 vs 峰值
- **能耗/能效**：**pJ/bit**、TOPS/W、能量开销比 baseline
- **搬运量**：DRAM 访问次数、L2/L1 流量相对 **algorithmic minimum** 的倍数（"Mind the Gap" 的核心度量）
- **成本/面积/热**：芯片成本（Chiplet Actuary、Gemini 的 MC 增量）、面积开销（iDMA 2.1%、FlooNoC 3.5%）、热点功率密度
- **一致性/同步**：失效次数、远程/本地访问比、同步开销占比

---

## 基于背景知识的推断（**未联网核实，与上文严格分开**）

> ⚠️ **这一节是"待核实线索"，不要当引用。**

1. **Tangram** 的确切会议/年份——检索只确认是 ASPLOS 系列，我记忆中为 ASPLOS 2019（Gao 等）。**需核对原文**
2. **SPACX** 的发表期刊/年份——我记忆倾向 IEEE TC 或 TPDS，2021–2022。**需核对**
3. **HMG** 的 HPCA 页码/DOI——检索给出 pp.582–595，DOI 未直接从检索确认
4. **"Memory-side DMA" 作为一种独立于"近存计算"的体系结构概念，可能并不作为一个专门 term 存在**
5. **NVIDIA/AMD 工业架构细节来自二手分析文章与 AMD 官方演讲稿，不是学术论文**。其中 CU 数量、L2 容量在不同来源间有出入（例如 XCD 的 CU 数在不同来源中为 32/38 不一），**引用前务必以厂商白皮书为准**
6. ⭐ **TMA"提升 D2D / 跨芯粒搬运"的直接学术论证未检索到** —— 现有 TMA 研究基本停留在片内/单 GPU 的存储层次，**跨 die 的 TMA 语义是空白**

> **第 6 条对主人项目最相关**：如果做"跨 die 的 TMA 语义"，检索结果支持"这是空白"这个判断。

---

## 一句话总结

> 本领域当前的研究重心已经从"**能不能把多个 die 拼起来**"（MCM-GPU / Simba 阶段）转向"**如何为跨 die 搬运设计正确的抽象与载体**"：一致性/同步层（HMG、CPElide）、显式搬运引擎（iDMA、XDMA、Memory Fabric 的 DME）、以及精确的 D2D PHY 建模（DICE、Omelet）。
>
> 被反复点名、且尚未解决的三个核心空白是：**① D2D 带宽/能耗 ② 跨 die 一致性/同步的可扩展性 ③ 可复现、参数统一的芯粒仿真与基准。**

---

## 关键来源

- CPElide (MICRO 2024): https://dl.acm.org/doi/abs/10.1109/MICRO61859.2024.00058
- NearFetch (HPCA 2025): https://ieeexplore.ieee.org/abstract/document/10946733
- DICE (ISCA 2026): https://arxiv.org/html/2607.24221v2
- Memory Fabric (IEEE CAL 2026): https://ieeexplore.ieee.org/document/11646988
- Heterogeneous Die-to-Die Interfaces (MICRO 2023): https://dl.acm.org/doi/pdf/10.1145/3613424.3614310
- Gemini (HPCA 2024): http://people.iiis.tsinghua.edu.cn/~gaomy/pubs/gemini.hpca24.pdf
- Mind the Gap (ISCA 2024): https://people.csail.mit.edu/emer/media/papers/2024.06.isca.orojenesis.pdf
- FRED (ISCA 2025): https://dl.acm.org/doi/full/10.1145/3695053.3731055
- Cambricon-LLM (MICRO 2024): https://ieeexplore.ieee.org/abstract/document/10764574
- MGPUSim (ISCA 2019): https://dl.acm.org/doi/abs/10.1145/3307650.3322230
- RapidChiplet (CF'25): https://github.com/spcl/rapidchiplet
- Survey of Chiplet Technology (JETCAS 2025): https://liuyuhang.cc/wp-content/uploads/2025/11/Survey_of_Chiplet_Technology_SoC_Architecture_Interconnect_EDA_and_Advanced_Packaging.pdf
- Chiplet Actuary (DAC 2022): https://ar5iv.labs.arxiv.org/html/2203.12268
