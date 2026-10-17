---
tags: [index]
updated: 2026-10-12
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

- [[computer-arch]] — SIMD·SIMT·脉动阵列的范式对比（指令流 vs 数据流）/ 脉动阵列 PE 机制与峰值算术 / 利用率死穴 / Tensor Core 与 VPU 的融合 · 3 条

## 标签

- `#dma` — 64 条（含 CUDA 与压缩领域里跨标的条目）
- `#compression` — 48 条
- `#survey` — 16 条
- `#kernel` — 16 条
- `#arch` — 13 条
- `#cuda` — 11 条
- `#theory` — 10 条
- `#hardware` — 10 条
- `#noc` — 9 条
- `#iommu` — 9 条
- `#format` — 9 条
- `#pcie` — 8 条
- `#lossy` — 8 条
- `#cpu` — 7 条
- `#pinned-memory` — 6 条
- `#entropy` — 6 条
- `#memory` — 4 条
- `#rdma` — 3 条
- `#performance` — 3 条
- `#memory-model` — 3 条
- `#memory-hierarchy` — 3 条
- `#lz` — 3 条
- `#systolic-array` — 2 条
- `#sgl` — 2 条
- `#bwt` — 2 条
- `#virtual-memory` — 1 条
- `#uva` — 1 条
- `#occupancy` — 1 条
- `#mbarrier` — 1 条
- `#data` — 1 条

> 共 30 个标签 —— **已到收敛阈值（~30）**：下次再引入新标签前先看能不能复用，否则该做一轮合并。
