---
tags: [dma, survey]
updated: 2026-10-10
---

# 跨卡互连与数据搬运：工业与标准现状

> **检索日期：2026-10-10。所有数字均附来源。**
>
> **标注约定（先读这个，否则会误用）**
>
> | 标记 | 含义 |
> |---|---|
> | **【官方】** | 厂商 / 标准组织自己的发布物料、规格书、文档 |
> | **【三方】** | 第三方媒体、学术论文、社区逆向分析、行业报告 |
> | **【推测】** | 由公开数据推导，或来源本身标注不确定 |
>
> **凡查不到公开数字的地方，直接写"查不到"，不做填充。** 凡官方未明说方向性（单向/双向）的，也标出。

## 速查：每 GPU 双向总带宽与拓扑

> **速查**
> - **NVLink 5 (B200) = 1.8 TB/s**；NVLink 4 (H100) = 900 GB/s。⚠️ **都是双向合计，单向除以 2**
> - **NVSwitch** 提供卡间**交换**（任意两点满带宽）；**xGMI 是 mesh**（要所有 GPU 一起参与才跑满）
> - **NV-HBI**（Blackwell 双 die 内）约 **10 TB/s**，⚠️ **方向性官方未说明**【三方】
> - **NVLink-C2C = 900 GB/s 且硬件一致**（CPU↔GPU）；⚠️ **NVLink 卡间 P2P 不是硬件一致，靠软件维护**
> - **UALink 1.0**（2025-04 发布）：200 Gbps/通道，x4 单端口每向 800 Gbps，**单 pod 最多 1024 加速器**，目标 RTT < 1 µs
> - **UALink / UEC / CXL / PCIe 是开放规范；NVLink 专有**（NVLink Fusion 有限开放）
> - **最大空白**：NVLink 交换/链路内部协议、NV-HBI 方向性与协议、NVLink Fusion 许可边界、CXL fabric 运行时调度、所有方案的精确延迟口径

### 正文

| 方案 | 每 GPU 双向带宽 | 拓扑能力 | 一致性 | 来源类型 |
|---|---|---|---|---|
| NVLink 5 (B200) | 1.8 TB/s | 交换 all-to-all（NVSwitch） | 卡间非硬件一致 | 官方 |
| NVLink 4 (H100) | 900 GB/s | 同上 | 同上 | 官方 |
| NVLink-C2C | 900 GB/s | 点对点（CPU↔GPU / die↔die） | **硬件一致** | 官方 |
| NV-HBI | 10 TB/s | 双 die 内点对点 | 全一致（视作单 GPU） | 三方 |
| UALink 1.0 | 单端口 x4 = 800 Gbps/向，1–4 端口 | 交换，可达 1024 加速器 | 软件一致 | 官方 |
| xGMI (MI300X) | 平台聚合 896 GB/s | mesh 全连接 8 卡 | 软件一致 | 官方（聚合）/三方（per-link） |
| CXL 3.0 / 4.0 | 64 / 128 GT/s（x16 层） | 交换 fabric（PBR） | 硬件一致（CXL.cache/mem） | 官方 |

---

## NVLink 各代规格 #dma #survey #noc

> **速查**
> - NVIDIA **官方每代只给"每 GPU 双向总带宽"一个数字**，链路层细节（per-lane 速率、编码、FEC）**从不逐代公布**
> - 代际：NVLink 1 (P100) 160 GB/s → 2 (V100) 300 → 3 (A100) 600 → 4 (H100) 900 → 5 (B200) **1.8 TB/s** → 6 (Rubin) 3.6 TB/s
> - ⚠️ **官方给的全是双向合计**，单向需除以 2（H100 每向 450 GB/s）。多份三方整理反复强调这点
> - ⚠️ 三方表格里"per lane / per pair"的定义**不一致**（有的把 NVLink 4 一条链路说成"2×100 Gbps 子链路"）——**引用时必须说清口径**

### 正文

| 代 | 架构 / 代表产品 | 年份 | 每 GPU 双向带宽 | 链路数/GPU |
|---|---|---|---|---|
| NVLink 1 | Pascal / P100 | 2016 | 160 GB/s | 4 |
| NVLink 2 | Volta / V100 | 2017 | 300 GB/s | 6 |
| NVLink 3 | Ampere / A100 | 2020 | 600 GB/s | 12 |
| NVLink 4 | Hopper / H100 | 2022 | 900 GB/s | 18 |
| NVLink 5 | Blackwell / B200 | 2024 | **1.8 TB/s** | 18 |
| NVLink 6 | Rubin | 2026（发布） | 3.6 TB/s | 查不到 |

**信号层（全部【三方】逆向或整理，NVIDIA 不公布）**：

| 代 | 每信号对速率 | 调制 | 每链路双向带宽 |
|---|---|---|---|
| NVLink 1 | 20 Gb/s | NRZ | 40 GB/s |
| NVLink 2 | 25 Gb/s | NRZ | 50 GB/s |
| NVLink 3 | 50 Gb/s | NRZ | 50 GB/s |
| NVLink 4 | 100 Gb/s | PAM4 | 50 GB/s |
| NVLink 5 | 200 Gb/s【三方，可信度有限】 | PAM4 | 100 GB/s |

**来源**：Wikipedia NVLink（三方汇总）· NVIDIA 技术博客（官方，NVLink 5）· TechTarget（三方，NVLink 6）· 《航空兵器》信号层表（三方）

---

## NVSwitch / NV-HBI / NVLink-C2C #dma #survey #noc

> **速查**
> - **NVSwitch = 卡间交换芯片**，实现 all-to-all。DGX H100：4 颗 NVSwitch gen3，整机 GPU-to-GPU 双向 **7.2 TB/s**【官方】
> - NVSwitch gen4（Blackwell）：单颗 **144 个 NVLink 端口、14.4 TB/s 无阻塞交换容量**；GB200 NVL72 用 9 个交换托盘连 72 GPU，域内聚合 **130 TB/s**【官方】
> - **NV-HBI**：B200 双 die 经它互连，**约 10 TB/s**，软件视作单个一致 GPU【三方】。⚠️ **方向性官方未说明**【推测】
> - **NVLink-C2C**：**900 GB/s 硬件一致**带宽（Grace Hopper / Grace Blackwell），比 PCIe Gen5 快约 7×；支持 Arm AMBA CHI 与 CXL【官方】
> - ISSCC 2023 论文标题即 **"40 Gbps/pin single-ended signaling"**——这是 NVIDIA 少见的公开信号层数字

### 正文

**NVSwitch 的历史**：gen1（DGX-2, V100）为 18 端口、18×18 无阻塞 crossbar、每端口每向 25 GB/s【三方逆向】。

**NVLink-C2C 的接口语义**：CPU 与 GPU **共享单一页表**，硬件 ATS + 缓存一致，形成统一地址空间（converged / unified memory）。

**来源**：NVIDIA DGX H100 页 · NVIDIA NVLink 页 · NVIDIA Grace Hopper 页 · NVIDIA 开放 NVLink 新闻稿（均官方）；ISSCC 2023 论文（官方/学术）；The Register · TrendForce（三方，NV-HBI）

---

## UALink（开放 scale-up 标准） #dma #survey #noc

> **速查**
> - 联盟 **2024-05** 成立 promoter group，2024-10 注册为 UALink Consortium Inc.，85+ 成员（AMD、Broadcom、Cisco、Google、HPE、Intel、Meta、Microsoft 等）
> - **1.0（UALink 200G）规范 2025-04-08 发布**【官方】
> - **200 Gbps/通道**（信令 212.5 GT/s，差额留给 FEC/编码）；链路宽度 x1/x2/x4；**x4 时单端口每向 800 Gbps**
> - 物理层基于 **IEEE P802.3dj 以太网 PHY**（200GBASE-KR1/CR1、400GBASE-KR2/CR2、800GBASE-KR4/CR4）
> - 单 pod **最多 1024 个加速器**；目标 **RTT < 1 µs**；64B–256B 事务；峰值带宽效率约 93%
> - **内存语义**操作（read/write/atomic），**软件维护一致性（software coherency）**；协议脱胎自 **AMD Infinity Fabric**
> - ⚠️ 规范以"评估副本"形式公开，**非成员仅获阅读权，不获实现许可**
> - ⚠️ **UALink 2.0（2026-04-07）我未拿到官方一手确认，引用须自行核实**

### 正文

**协议分层细节（评估副本可查）**：读写最大 256B、**不跨 256B 边界**；数据以 **64B beat** 返回；atomic 元素 1/2/4/8B；单拷贝原子性；两种顺序模式（Strict Ordering / Lite）。

**与 UEC 的关系**：**互补**——UEC 管 scale-out，UALink 管 scale-up。

**来源**：UALink 官方白皮书 / 新闻稿 / 评估副本（官方）；Tom's Hardware（三方）；Wikipedia UALink（三方）

---

## UEC（Ultra Ethernet Consortium，开放 scale-out） #dma #survey

> **速查**
> - Linux Foundation（Joint Development Foundation）项目。**UEC Specification 1.0 于 2025-06-11 发布**，后续 1.0.1（2025-09）
> - 目标 scale-out 支持**百万级端点**
> - 关键机制：**packet trimming**（拥塞时裁到只剩头）、in-band telemetry、链路层重传、信用流控、原生硬件加密
> - 采用现代 RDMA-over-Ethernet（UET）
> - **与 UALink 互补**：UEC 管 scale-out，UALink 管 scale-up

**来源**：Linux Foundation 新闻 · UEC 规范 PDF（官方）；Network World（三方）

---

## CXL #dma #survey #iommu

> **速查**
> - 版本：1.0/1.1 (2019) → 2.0 (2020，加 memory pooling + switching) → **3.0 (2022-08-02，速率翻倍到 64 GT/s，引入 fabric / Port-Based Routing (PBR)、memory sharing、GFAM)** → 3.1 (2023-11) → 3.2 (2024-12) → **4.0 (2025-11-18，基于 PCIe 7.0，128 GT/s)**
> - **CXL 4.0**：128 GT/s、保留 256B FLIT；新增 **bundled ports**（x16、128 GT/s 单 bundle 每向约 768 GB/s，四 bundle > 2 TB/s）；retimer 上限从 2 提到 4
> - **Type-2 设备**（含加速器）：CXL.io + CXL.cache + CXL.mem，可选硬件缓存 + Host-Managed Device Memory (HDM)
> - **一致性模型**：CXL.cache 用 **MESI、64B cache line**；⚠️ **host 负责跟踪所有 peer cache 的一致性，device 从不直接与 peer cache 交互**
> - **加速器间 P2P**：**CXL 3.1 通过 PBR 交换增加 "Direct P2P CXL.mem for Accelerators"**——accelerator 可访问 peer Type-3 内存，双方须各自直连 Edge Downstream Port
> - ⚠️ **Fabric 的运行时调度缺席**：Fabric Manager 目前实质是"配置工具"而非运行时调度器，且 host–FM 控制通路在规范之外
> - ⚠️ **面向 GPU 加速器的 CXL Type-2 量产产品基本缺席**——行业归因于 CUDA 软件栈适配难

**来源**：CXL 3.0 新闻稿 · CXL 4.0 官方博客（官方）；Linux 内核文档（官方，Type 划分）；OpenFabrics 2024 workshop（官方，PBR P2P）；ACM 综述 · Netdev 演讲（三方）

---

## GPUDirect（RDMA / Storage） #dma #survey #rdma

> **速查**
> - **GPUDirect RDMA**：让第三方 PCIe 设备（NIC、SSD、FPGA）**绕过 CPU/系统内存直接读写 GPU 显存**
> - 机制核心是 **GPU 的 BAR1 aperture**——把显存暴露到 PCIe 内存空间
> - ⚠️ **显存 pin/unpin 必须走 NVIDIA 内核驱动接口（`nvidia_p2p_get_pages` 等），不同于 CPU 内存走 `get_user_pages`**
> - ⚠️ **内存序：只有 CUDA 的同步/提交 API 才为 GPUDirect RDMA 操作提供内存序**；`CU_POINTER_ATTRIBUTE_SYNC_MEMOPS` 会让操作在 CPU 线程重新获得控制前完成（更保守、可能损性能）
> - **GPUDirect Storage (GDS)**：GPU 显存与存储之间的直接 DMA 路径，免去 CPU 内存 bounce buffer。用户态 `libcufile.so`（cuFile API）+ 内核 `nvidia-fs.ko`（GPLv2）。不满足条件时**回退到 POSIX 兼容模式**
> - ⚠️ **BAR1 P2P 的准入条件多为逆向得来**（需 Resizable BAR、vGPU/CC 模式不支持、PCIe atomic 对 BAR1 P2P 返回 NOT_SUPPORTED），非官方声明

**来源**：NVIDIA GPUDirect RDMA 文档 · GDS Overview · cuFile API（官方）；CUDA Handbook ch9 · geforce-gpudirect-rdma 逆向（三方）

---

## AMD Infinity Fabric / xGMI #dma #survey #noc

> **速查**
> - MI300X 属第 4 代 Infinity Architecture；8 卡 UBB 2.0 平台；**平台级 peer-to-peer 双向聚合 I/O 带宽 896 GB/s**；PCIe Gen5 x16【官方】
> - 8 卡 xGMI 是 **mesh 全连接**（非交换）【官方】
> - ⚠️ **AMD 官方只公布聚合值**——per-link（32 Gbps × 16 lane ≈ 128 GB/s）是【三方/推测】推导
> - ⚠️ **xGMI 是 mesh，须所有 GPU 一起参与才能跑满带宽**（不像 NVSwitch 的交换结构可任意两点满带宽）
> - **HW 布局**：MI300X 在 I/O die 中集成 **16 个 system-DMA 引擎**，同时服务 CPU↔GPU（PCIe Gen5）与 GPU↔GPU（xGMI）【三方】
> - **软件开关**：`HSA_ENABLE_SDMA`（默认 1）、`HSA_ENABLE_PEER_SDMA`。置 0 会让 `hipMemcpy` **回退到 GPU "blit" kernel**（用计算单元做 load/store）
> - ⚠️ **一个反直觉的实测**：MI300A 上 SDMA 拷贝约 116 GB/s，而 blit kernel 可达约 319–586 GB/s——**专用引擎不一定比用计算单元搬更快**

**来源**：AMD MI300X 产品页 / 产品简介 PDF / ROCm 博客（官方）；ROCR-Runtime 环境变量文档（官方）；simllm AMD fabric 分析 · DMA-Latte 论文 · ACL 论文（三方）

---

## 跨卡搬运是怎么实现的（公开资料可见的机制） #dma #survey #noc

> **速查**
> - **地址模型四种**：① **UVA + peer 映射**（CUDA，需显式 `cudaDeviceEnablePeerAccess`，**映射非对称**）② **BAR1 aperture**（GPUDirect RDMA）③ **HDM / DAX**（CXL Type-2）④ **内存语义寻址**（UALink，加速器间直接 load/store/atomic）
> - **一致性归属**：**硬件一致**（NVLink-C2C 共享页表、CXL.cache/mem）vs **软件维护**（GPU 间 NVLink P2P、UALink、xGMI）
> - ⚠️ **GPU 间 NVLink P2P 的内存序只由 CUDA 同步/提交 API 保证**——并发跑着的 kernel 可能读到陈旧/乱序数据
> - **完成通知**：CUDA stream/event、门铃+完成标志（GPUDirect Async / IBGDA）、DMA 引擎完成信号（ROCm SDMA）、RDMA 完成队列
> - ⚠️ **P2P memcpy 不与任何其他操作并发**——发起前须等两侧 pending 操作完成，完成前后续操作不启动
> - **搬运硬件**：NVIDIA **copy engine**（Fermi 起，低端 1 个、中高端 2 个）；AMD **SDMA**（MI300X 集成 16 个）

### 正文

**CUDA 的 P2P 地址模型细节**：两卡 P2P 需显式 `cudaDeviceEnablePeerAccess()` / `cuCtxEnablePeerAccess()`，且**映射是非对称的**（每张卡各自映射对方）。用 `cudaDeviceCanAccessPeer()` 探测。启用后常规 `cudaMemcpy` 家族可直接用；未启用则须 `cudaMemcpyPeer(Async)`。

**NVIDIA copy engine 的历史**：Fermi（2010）起独立于图形引擎。低端卡 1 个（单向传输可与渲染重叠）；中高端 Quadro/Tesla 2 个（双向传输 + 渲染三方重叠）。GF100/GF110 有两个 DMA 引擎但**仅 Tesla 驱动启用第二个**。

**一个重要的设计取舍**：专用 DMA（copy engine / SDMA）省计算资源，但**延迟/带宽不一定最优**；用 CU 做 blit 可能更快——取决于数据与并发。

---

## 设计取舍对比 #dma #survey

| 维度 | NVLink + NVSwitch | UALink | CXL | UEC / IB | GPUDirect | xGMI (AMD) |
|---|---|---|---|---|---|---|
| 定位 | scale-up（卡间） | scale-up（开放） | 内存语义一致互连 | scale-out（节点/机架间） | 数据路径旁路 | scale-up（卡间） |
| 拓扑 | **交换** crossbar，任意两点满带宽 | 交换，"pod" 概念 | 交换 fabric（PBR） | 多层 fat-tree/clos | 依托 PCIe/NVLink 底层 | **mesh**，全节点参与才满带宽 |
| 一致性 | 卡间软件一致；C2C 硬件一致 | 软件一致 | **硬件一致** | 不适用（消息语义） | 依赖 CUDA 同步 | 软件一致 |
| 规模 | NVL72 = 72 GPU（铜）；DGX GH200 = 256 | 1024 加速器/pod | 3.x fabric 规划 4096 节点 | 百万级端点 | 单机上多卡 | 8 卡 UBB |
| 延迟目标 | NVLink 级（三方称 <0.3 µs RTT） | **< 1 µs RTT**（官方） | 与 PCIe 同量级 | 微秒级 | 受 PCIe 拓扑/NUMA 影响 | 机内 fabric 量级 |
| 介质/距离 | NVL72 内铜；跨机架须光 | 铜/光，目标 1–4 机架 | 铜 + 光学（4.0 加强） | 以太网/光 | 依底层 | 铜（UBB 内） |
| 标准属性 | **专有**（NVLink Fusion 有限开放） | **开放**（评估副本公开） | **开放** | **开放**（LF 项目） | 专有 API | 专有，UALink 脱胎于此 |

**五个取舍点（均有来源支撑）**：

1. **交换 vs mesh** —— NVSwitch 让任意两点满带宽（适合 MoE all-to-all），代价是需要专用交换芯片；xGMI mesh 无交换芯片但"要么整节点一起用、要么别用"
2. **一致性的位置** —— CXL/NVLink-C2C 把一致性放进硬件（好编程、但硬件复杂）；UALink/GPU P2P 留给软件（简单、可扩展、但要显式同步）
3. **开放 vs 锁定** —— UALink/UEC/CXL 是开放规范；NVLink 软件层、许可与协议高层访问受限
4. **介质距离** —— 1.8 TB/s 铜信号数英尺即衰减，**硬性把 NVL72 卡在 72 GPU/单机架**；跨机架须转光
5. **DMA 引擎 vs 计算单元搬运** —— 见上节的 AMD 实测

---

## 空白与未公开 #dma #survey

> **这一节对预研选题最直接——空白就是可能的贡献点。**

### NVLink / NVSwitch 内部协议（大面积空白）

【三方】逆向明确标为 **UNDOCUMENTED** 的项：A100/H100 的 **replay buffer 深度、ack 定时器、重传延迟**；**credit 粒度**（一个 credit = flit / packet / 固定字节？）；接收缓冲深度；posted/non-posted 是否走不同虚通道。

**NVSwitch 路由算法、仲裁、gen4/gen5 交换芯片微结构**：无公开规范，结论多来自专利与侧信道实验。

**NVLink 5/6 的 per-lane 权威速率、编码、FEC 细节**：官方不公布。

### 其他空白

| 空白 | 具体缺什么 |
|---|---|
| **NVLink Fusion 的开放边界** | 仅 8 家获有限访问；**连接配置软件仍受 NVIDIA 控制**，无 NVSwitch 许可无法接 GPU；NUMA/atomic 等高阶协议层需额外审批；IP 公司只拿到 PHY 与 link 层；且每个项目必须含至少一件 NVIDIA 组件 |
| **NV-HBI** | **10 TB/s 的方向性官方未说明**；die 间协议、一致性实现、纠错机制无公开规范 |
| **UALink** | 评估副本公开但**实现许可仅成员持有**；每个 op 的精确延迟、拥塞/流控参数、交换芯片路由看不到；**原生 UALink 交换硅尚未出货**，AMD Helios 当前用 **UALoE（UALink over Ethernet）**过渡 |
| **CXL** | Fabric 运行时调度缺席（FM 只是配置工具）；4096 节点是"纸面能力"；**面向 GPU 加速器的 CXL Type-2 量产产品基本缺席**；CXL.cache 的 peer 一致性受限（host 代管） |
| **GPUDirect** | `nvidia-fs.ko` 完整设计文档公开有限；BAR1 P2P 准入矩阵多为逆向；**同步语义与性能的权衡缺少量化数据** |
| **AMD** | 官方只公布聚合带宽，per-link 无官方出处；xGMI 协议层细节（credit、顺序、错误处理）无公开规范；SDMA 调度策略公开有限 |
| **通用** | **跨厂商互操作性**——UALink/UEC/CXL/NVLink 之间没有公开互联规范；**精确端到端延迟**——各家的 "<1 µs"、"<0.3 µs" 多为目标值或三方测量，缺少官方可比的测量口径；**完成通知的微架构**——除 API 语义外基本不公开 |

### 自测
- [ ] NVLink 公布的是单向还是双向带宽？引用时要注意什么？
- [ ] NVSwitch 的交换结构和 xGMI 的 mesh 在"能不能任意两点满带宽"上差在哪？
- [ ] GPU 间 NVLink P2P 是硬件一致还是软件维护？证据是什么？
- [ ] 列出三个"公开资料明确查不到"的 NVLink 内部机制。

---

## 来源清单

**NVLink / NVSwitch / C2C**
- https://www.nvidia.com/en-us/data-center/nvlink/ （官方）
- https://developer.nvidia.com/blog/scaling-ai-inference-performance-and-flexibility-with-nvidia-nvlink-and-nvlink-fusion/ （官方）
- https://www.nvidia.com/en-us/data-center/grace-hopper-superchip/ （官方）
- https://www.nvidia.com/en-us/data-center/dgx-h100/ （官方）
- https://ieeexplore.ieee.org/document/10067395 （ISSCC 2023，NVLink-C2C）
- https://en.wikipedia.org/wiki/NVLink （三方汇总）
- https://www.aeroweaponry.avic.com/fileup/1673-5048/PDF/2023-0138.pdf （三方，信号层表）

**UALink / UEC**
- https://ualinkconsortium.org/wp-content/uploads/2025/04/UALink-1.0-White_Paper_FINAL.pdf （官方）
- https://ualinkconsortium.org/wp-content/uploads/2025/04/UALink-1.0-Specification-PR_FINAL.pdf （官方）
- https://ualinkconsortium.org/wp-content/uploads/2025/04/UALink200_Specification_v1.0_Evaluation_Copy.pdf （官方评估副本）
- https://www.linuxfoundation.org/press/uec-launches-spec-1.0 （官方）
- https://ultraethernet.org/wp-content/uploads/sites/20/2025/06/UE-Specification-6.11.25.pdf （官方）

**CXL**
- http://computeexpresslink.org/wp-content/uploads/2024/01/CXL_3.0-Specification-Release_FINAL-1.pdf （官方）
- https://computeexpresslink.org/blog/introducing-the-cxl-4-0-specification-webinar-qa-recap-4386/ （官方）
- https://docs.kernel.org/6.16/driver-api/cxl/devices/device-types.html （官方）
- https://www.openfabrics.org/wp-content/uploads/2024-workshop/2024-workshop-presentations/session-10.pdf （官方，PBR P2P）

**GPUDirect**
- https://docs.nvidia.com/cuda/archive/12.9.2/gpudirect-rdma/index.html （官方）
- https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html （官方）
- https://www.cudahandbook.com/book/ch9/peer-to-peer （三方）
- https://github.com/mcornea/geforce-gpudirect-rdma （三方逆向）

**AMD**
- https://www.amd.com/en/products/accelerators/instinct/mi300/platform.html （官方）
- https://rocm.blogs.amd.com/software-tools-optimization/mi300x-rccl-xgmi/README.html （官方）
- https://rocm.docs.amd.com/projects/ROCR-Runtime/en/docs-6.3.1/api-reference/environment_variables.html （官方）

**DMA / 逆向 / 其他**
- https://www.nvidia.es/docs/IO/40049/Dual_copy_engines.pdf （官方，copy engine）
- https://github.com/openfabric-systems/simllm/blob/main/docs/design/nvlink-mechanism-reverse-engineering.md （三方逆向）
- https://pcisig.com/blog/progressing-track-pcie-70-specification-version-07-now-available-member-review （官方，PCIe 7.0）
