---
tags: [index]
updated: 2026-10-10
---

# 笔记索引

> 这张表由 `learn-note` 维护，条目数与标签表都是实算的，不要手改数字。
> 主题分类约定见 [TAXONOMY.md](TAXONOMY.md)。
>
> 目录结构：`cc_notes/<大类>/<子类>/<主题文件>.md`。大类就是 TAXONOMY 里的「领域」列。

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

- [[memory]] — 内存模型 / pinned / pageable / 分页 / zero-copy / 异步拷贝 · 5 条 · 2026-10-09

## 标签

- `#dma` — 63 条
- `#survey` — 16 条
- `#kernel` — 16 条
- `#noc` — 9 条
- `#iommu` — 9 条
- `#pcie` — 8 条
- `#cpu` — 7 条
- `#arch` — 7 条
- `#pinned-memory` — 6 条
- `#cuda` — 5 条
- `#memory` — 4 条
- `#rdma` — 3 条
- `#sgl` — 2 条
- `#virtual-memory` — 1 条
- `#uva` — 1 条
- `#performance` — 1 条
