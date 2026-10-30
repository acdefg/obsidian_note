---
tags: [index]
updated: 2026-10-27
created: 星期日, 十月 18日 2026, 5:05:52
modified: 星期一, 十月 19日 2026, 22:45:00
---

# 笔记索引

> 这张表由 `learn-note` 维护，条目数与标签表都是实算的，不要手改数字。
> 主题分类约定见 [TAXONOMY.md](TAXONOMY.md)。
>
> 目录结构：`cc_notes/<大类>/<子类>/<主题文件>.md`。大类就是 TAXONOMY 里的「领域」列。

## 压缩

> **按层读**：L1 → L2 → L3 → L4 → L5，每层建立在前一层上。

**原理 · L1–L2** · `压缩/原理/`

- [[information-theory]] — **L1** 信息论根基：熵 / 香农源编码定理 / 熵率 / Kolmogorov 复杂度 / 实际数据的熵 · 12 条
- [[entropy-coding]] — **L2** 熵编码：Huffman / Canonical / 算术·区间编码 / ANS（rANS·tANS）/ PPM·CM · 8 条
- [[dictionary-transform]] — **L2** 字典与变换：LZ77·LZ78·LZW·LZSS / BWT·MTF·RLE / 变换编码通性 · 8 条

**格式 · L3** · `压缩/格式/`

- [[formats]] — DEFLATE·zlib·gzip·zip / LZMA·xz / zstd / brotli / LZ4·Snappy·LZO / seekable · 11 条

**有损 · L4** · `压缩/有损/`

- [[lossy]] — 量化 / 率失真 / DPCM / DCT·JPEG / 小波·JPEG 2000 / 感知模型 / 运动补偿 / RDO · 10 条

**硬件 · L5** · `压缩/硬件/`

- [[hardware]] — 能量论证 / 硬件友好特征 / QAT / IAA / **链路在线压缩** / 内存压缩 / 解压瓶颈 / 度量 · 11 条

## DMA

**引擎与机制** · `DMA/引擎与机制/`

- [[engine]] — 技术分类（六轴）/ SGL / 引擎实际在做什么 / 分割·合并·顺序管理 · 4 条
- [[c2c-crosscard]] — 片内 D2D 与跨卡 / 延迟吸收 / 链路控制器分层 / 三条路径 · 4 条

**地址与翻译** · `DMA/地址与翻译/`

- [[address-path]] — MMU vs IOMMU / 三层地址视图（VA·PA·IOVA）/ 谁能在哪一层合并 · 2 条
- [[pcie-iommu]] — PCIe 三层 / TLP / Ordering / MPS·MRRS / ATS·PRI·PASID / VT-d·AMD-Vi·SMMUv3 · 10 条

**互连与 RDMA** · `DMA/互连与RDMA/`

- [[rdma]] — 内存注册 / 队列对 / 完成队列 / 单边 vs 双边 · 1 条

**软件栈** · `DMA/软件栈/`

- [[linux-dma]] — DMA-API / coherent·streaming / scatterlist / SWIOTLB / dmaengine / dma-buf / P2PDMA / GUP · 10 条
- [[x86-io-primitives]] — WC·MTRR·PAT / non-temporal store / movdir64b / enqcmd / flush 指令 / 门铃与工作队列 · 8 条
- [[zero-copy]] — sendfile·splice / MSG_ZEROCOPY·AF_XDP / io_uring / GPUDirect / DPDK·SPDK 及各自代价 · 11 条

**预研综述** · `DMA/预研综述/`

- [[interconnect-survey]] — 工业与标准：NVLink·NVSwitch·NV-HBI / UALink / UEC / CXL / GPUDirect / xGMI（逐条标来源） · 12 条
- [[academic-survey]] — 学术：多芯粒 GPU 搬运 / 一致性·同步 / 显式搬运引擎 / 仿真器 / 开放问题 · 9 条

## CUDA

- [[memory]] — **主机内存视角**：pinned / pageable / 分页 / UVA / zero-copy / 异步拷贝 · 5 条
- [[synchronization]] — **并发内存模型视角**：coherence vs consistency / relaxed·release·acquire / fence 三分 / MESI / GPU scope / mbarrier / 跨卡 · 3 条
- [[execution]] — **设备侧执行与存储**：内存层级（register file·shared·local·global）/ 变量与栈的分配 / local memory 判定 / occupancy 算术 / 数据放哪 · 3 条

## 计算机体系结构

- [[soc-architecture]] — **SoC 架构岗位能力图谱**：JD 对比（模块 owner vs 架构师）/ 产品定义与 PPA 预算 / VMSA·SMMU·ASID·nG / TrustZone·安全启动 / 总线·crossbar·NoC 带宽估算 / UPF·VCLP·DVFS / 存储层级与 tiling 预算 / 子系统与架构文档 / 后端·DFT·EMU / 缺口清单 · 9 条 · 2026-10-27
- [[computer-arch]] — SIMD·SIMT·脉动阵列的范式对比（指令流 vs 数据流）/ 脉动阵列 PE 机制与峰值算术 / 利用率死穴 / Tensor Core 与 VPU 的融合 / **数据流四分类与复用因子（能手算）** · 4 条
- [[cache-microarchitecture]] — cache 硬件结构（tag/index/offset·组相联·MSHR）/ 替换算法（LRU·MRU·FIFO·PLRU·RRIP·BIP）/ **MRU 与 evict-first·streaming hint** / **CPU 访存合并**（MSHR·write combining）/ **GPU 访存合并**（warp coalescing·32B sector） · 5 条 · 2026-10-25
- [[rram-cim]] — **RRAM 存内计算**：器件/阵列原理 / GEMM 与矩阵求导映射 / **tiling 与阵列映射分层** / **MNK·batch·复用因子** / 精度补偿四层 / 仿真验证 / 论文清单 / 面试背诵卡 · 9 条 · 2026-10-24
- [[mixed-precision-cim]] — **混合精度 CIM**：INT8·INT4 量化误差 / bit-slicing 与 MVM 映射 / 混合精度分配 / 论文清单 / 面试背诵卡 · 5 条 · 2026-10-19
- [[interrupt-basics]] — **中断基础**：MMIO / hart / MSI / CSR（含间接 CSR）/ APLIC domain 分层 / direct（类 PLIC）/ direct·MSI·IPI·虚拟化完整 trap 流程 / 面试速查与常见误解 · 8 条 · 2026-10-24
- [[risc-v-aia]] — **RISC-V AIA**：AIA CSRs / IMSIC 中断文件与 MSI / APLIC 域与 direct·MSI / PLIC 对比 / 软件栈（Linux·OpenSBI·QEMU·KVM·Spike）/ 开源 RTL 盘点 / 自研拆分与验证路线 · 6 条 · 2026-10-24
- [[systemc-kernel]] — **SystemC 仿真内核**：离散事件调度 / 求值·更新两阶段 / delta cycle 与时间推进 / `sc_signal` vs `sc_buffer` / `SC_METHOD`·`SC_THREAD` 语义 / 零延迟环与多驱动 · 1 条 · 2026-10-27
- [[tlm]] — **TLM-2.0**：LT vs AT / `b_transport`·`nb_transport` 四相 / `tlm_generic_payload` / 时间解耦 quantum / DMI / 五个常见坑 · 1 条 · 2026-10-27
- [[modeling-levels]] — **性能建模粒度**：解析 / trace-driven / LT·AT / 周期近似 / 周期精确 / 周期级倒序 vs current-next / TLM-2.0 不含 cycle-accurate 标准 · 1 条 · 2026-10-27

## 项目经历

- [[huawei-mpu]] — **华为 MPU**：message process / key·lockid / 乱序分配 / shift queue·64-key 冲突 / dispatch / 提前释放 / 删拍 / 时钟门控 / **Leading-one RTL** · 2 条 · 2026-10-18

## C++

**STL** · `C++/STL/`

- [[hash-table]] — 哈希表的几种写法（unordered_map / map / flat_hash_map / gp_hash_table / 手写拉链 / 离散化）/ `mp->second` 与访问方式 / 五个必踩的坑 / **从声明到遍历的用法流程** / **`unordered_set` 用法（纯用法）** · 4 条 · 2026-10-25
- [[string]] — **STL 字符串**：声明·初始化 / `>>` 与 `getline` 的分工 / 常用操作表 / 遍历·切分·查找 / 七个坑 / SSO 与扩容 · 1 条 · 2026-10-25

**算法** · `C++/`

- [[0-1背包 完全背包]] — **背包 DP**：0-1 / 完全 / 多重 / 分组 / 二维费用；二维→一维的逆序与正序 / 初始化与比较符 / 组合与排列 / 从递归到 DP · 5 条 · 2026-10-26

## 标签

- `#dma` — 64 条（含 CUDA 与压缩领域里跨标的条目）
- `#arch` — 59 条
- `#compression` — 48 条
- `#survey` — 18 条
- `#kernel` — 17 条
- `#cim` — 14 条
- `#hardware` — 12 条
- `#interrupt` — 11 条
- `#cuda` — 11 条
- `#theory` — 10 条
- `#performance` — 10 条
- `#noc` — 10 条
- `#memory-hierarchy` — 10 条
- `#iommu` — 10 条
- `#cpu` — 10 条
- `#rram` — 9 条
- `#format` — 9 条
- `#pcie` — 8 条
- `#lossy` — 8 条
- `#pinned-memory` — 6 条
- `#entropy` — 6 条
- `#stl` — 5 条
- `#quantization` — 5 条
- `#dp` — 5 条
- `#cpp` — 5 条
- `#memory` — 4 条
- `#soc` — 3 条
- `#rdma` — 3 条
- `#mixed-precision` — 3 条
- `#memory-model` — 3 条
- `#lz` — 3 条
- `#virtual-memory` — 2 条
- `#systolic-array` — 2 条
- `#sgl` — 2 条
- `#dataflow` — 2 条
- `#bwt` — 2 条
- `#uva` — 1 条
- `#security` — 1 条
- `#occupancy` — 1 条
- `#mbarrier` — 1 条
- `#low-power` — 1 条
- `#data` — 1 条

> 共 42 个标签 —— **已过收敛阈值（~30）**。本次新增 `#soc`（3 条）、`#security`（1 条）、`#low-power`（1 条），均来自 SoC 架构笔记；`#arch`、`#noc`、`#iommu` 等既有标签只增计数。下次再引入新标签前先看能不能复用，否则该做一轮合并。
>
> 统计口径提醒：`#include` 在 `hash-table.md` 的代码块里出现过 2 次，Obsidian 不解析代码块内的 `#`，计数脚本会误抓，**已排除**。
