---
tags: [index, taxonomy]
updated: 2026-10-10
---

# 主题分类表

`learn-note` 每次归类和写标签前都会读这张表。**这张表是知识库的唯一分类权威**——
改表 = 改分类法。新建主题文件前若发现表里没有对应领域，说明该改表了，而不是即兴造个文件名。

## 目录结构约定

```
cc_notes/
├─ INDEX.md
├─ TAXONOMY.md
└─ <领域>/<子类>/<主题文件>.md
```

- **第一层 = 本表的「领域」列**（`DMA/`、`CUDA/`、`图形学/`……）
- **第二层 = 子类**（同一领域内的分组，如 `DMA/引擎与机制/`）
- **文件名不带领域前缀** —— 因为文件夹已经表达了领域

⚠️ **改文件名会断开所有 `[[双链]]`**（Obsidian 按文件名解析，跨文件夹）。
改名前必须全库检查两件事，改后必须跑死链检查：

1. **新文件名在全 vault 内唯一**（`find . -name "<新名>.md"`，别处有同名会让双链变歧义）
2. **没有来自 `cc_notes/` 之外的入链**（否则那些链接会断）

## 分类表

| 领域 | 路径 | frontmatter tags | 覆盖范围 |
|---|---|---|---|
| CUDA | `CUDA/memory.md` | `[cuda, memory]` | 内存模型、pinned、UVA、异步拷贝、stream |
| CUDA | `CUDA/execution.md` | `[cuda, execution]` | SIMT、warp、occupancy、bank conflict |
| 计算机体系结构 | `<arch>/computer-arch.md` | `[arch, gpu]` | cache、流水线、分支预测、乱序执行；GPU 微架构（SM/TMA/L2）、memory hierarchy、roofline |
| DMA | `DMA/引擎与机制/engine.md` | `[dma, arch]` | DMA 引擎本体：技术分类（六轴）、SGL、引擎实际在做什么、分割 / 合并 / 顺序管理 |
| DMA | `DMA/引擎与机制/c2c-crosscard.md` | `[dma, noc]` | 片内 C2C/D2D 与跨卡：两域设计轴、延迟吸收、链路控制器分层、跨卡三条路径 |
| DMA | `DMA/地址与翻译/address-path.md` | `[dma, iommu]` | 地址空间与翻译：MMU vs IOMMU、三层地址视图（VA / PA / IOVA）、谁能在哪一层合并 |
| DMA | `DMA/地址与翻译/pcie-iommu.md` | `[dma, pcie]` | PCIe 三层 / TLP / Ordering / MPS·MRRS、ATS·PRI·PASID、VT-d·AMD-Vi·SMMUv3 两级翻译与失效 |
| DMA | `DMA/互连与RDMA/rdma.md` | `[dma, rdma]` | RDMA：内存注册 / 队列对 / 完成队列、单边 vs 双边、与跨卡 DMA 的关系 |
| DMA | `DMA/软件栈/linux-dma.md` | `[dma, kernel]` | Linux DMA 子系统：DMA-API、coherent/streaming、scatterlist、SWIOTLB、dmaengine、dma-buf、P2PDMA、GUP/FOLL_PIN |
| DMA | `DMA/软件栈/x86-io-primitives.md` | `[dma, cpu]` | WC/MTRR/PAT、non-temporal store、`movdir64b`、`enqcmd(s)`、CLFLUSH/CLWB/SFENCE、门铃与工作队列 |
| DMA | `DMA/软件栈/zero-copy.md` | `[dma, kernel]` | 零拷贝谱系：sendfile/splice、MSG_ZEROCOPY、AF_XDP、io_uring、GPUDirect、DPDK/SPDK 及各自代价 |
| DMA | `DMA/预研综述/interconnect-survey.md` | `[dma, survey]` | **工业与标准现状**：NVLink/NVSwitch/NV-HBI、UALink、UEC、CXL、GPUDirect、xGMI 的规格与公开空白（逐条标来源） |
| DMA | `DMA/预研综述/academic-survey.md` | `[dma, survey]` | **学术现状**：多芯粒 GPU 搬运、一致性/同步、显式搬运引擎（iDMA/XDMA/DME）、仿真器、开放问题 |
| 压缩 | `压缩/原理/information-theory.md` | `[compression, theory]` | **L1** 信息论根基：熵、香农源编码定理、熵率、Kolmogorov 复杂度、典型数据的实际熵 |
| 压缩 | `压缩/原理/entropy-coding.md` | `[compression, theory]` | **L2** 熵编码：Huffman、Canonical Huffman、算术/区间编码、ANS（rANS·tANS）、PPM·CM 上下文建模 |
| 压缩 | `压缩/原理/dictionary-transform.md` | `[compression, theory]` | **L2** 字典与变换：LZ77·LZ78·LZW·LZSS、BWT·MTF·RLE、变换编码通性 |
| 压缩 | `压缩/格式/formats.md` | `[compression, format]` | **L3** 现代格式：DEFLATE·zlib·gzip·zip、LZMA·xz、zstd、brotli、LZ4·Snappy·LZO、seekable |
| 压缩 | `压缩/有损/lossy.md` | `[compression, lossy]` | **L4** 有损与感知：量化、率失真、DPCM、DCT·JPEG、小波·JPEG 2000、感知模型、运动补偿、RDO |
| 压缩 | `压缩/硬件/hardware.md` | `[compression, hardware]` | **L5** 硬件与系统：能量论证、硬件友好特征、QAT、IAA、链路在线压缩、内存压缩、解压瓶颈、度量 |
| 图形学 | `<graphics>/graphics-rendering.md` | `[graphics, rendering]` | 光栅化、光追、shading、GAMES101 |
| 图形学 | `<graphics>/geometry-processing.md` | `[graphics, geometry]` | 网格、细分、参数化 |
| 图形学 | `<graphics>/graphics-gaussian-splatting.md` | `[graphics, 3dgs]` | 3DGS / 4DGS、可逆渲染、splat 表示与压缩 |
| 深度学习系统 | `<dl>/dl-systems.md` | `[dl, systems]` | 训练/推理系统、算子、并行策略 |
| 具身智能 | `<embodied>/embodied-ai.md` | `[embodied, robotics]` | 机器人、仿真、感知–决策–控制 |
| 数字电路 | `<digital>/digital-design.md` | `[digital, verilog]` | Verilog、时序、综合、Zynq |
| C++ | `<cpp>/cpp.md` | `[cpp]` | 语言特性、模板、内存模型、并发 |
| 工具链 | `<toolchain>/toolchain.md` | `[toolchain]` | 构建、调试、profiler、环境 |

> `<...>` 表示该领域的文件夹尚未建立。**第一次往这个领域写笔记时建文件夹**，
> 名字用领域名（如 `图形学/`、`体系结构/`），并把这张表里的路径补齐。

## 归类歧义怎么办

一个概念同时沾两个领域时（例：pinned memory 既是 CUDA 也是 OS 的虚拟内存话题）：

1. **先查覆盖范围**列的措辞，哪边写了就归哪边。
2. 两边都没写 → 归到**你更可能从哪个角度回查**的那个领域。
3. 仍然模棱两可 → 归主领域，在另一个领域开一行 `[[链接]]` 占位。这是唯一该开占位的情形。
4. **把决定补进覆盖范围列**，下次就不用再想。

不做的事：**不要为了一次归类新建一个领域**。领域是慢慢长出来的，不是每次提问都长一个。

## 怎么改这张表

直接改，不用问谁：

- **加领域** → 建文件夹 + 加一行；`路径` 用 `<领域>/<子类>/<kebab-case>.md`
- **加子类** → 就在该领域文件夹下建子目录，改对应行的路径
- **合领域** → 删行、移动文件、更新 `INDEX.md`
- **改文件名** → ⚠️ **会断双链**。改名前后必须做上面「目录结构约定」里那两项检查

## 变更记录

- 2026-10-09 — 初版。由 10 行草稿定稿：GPU 架构并入 `computer-arch.md`；图形学拆三片（4DGS 单列）；补 `embodied-ai.md`。
- 2026-10-09 — 从 `D:\code\notes\` 迁到 vault 内 `note\study_note\cc_notes\`，双链与标签面板自此生效。
- 2026-10-10 — **改为两层目录**（`<领域>/<子类>/`），文件名去掉领域前缀。11 个文件按子类归位；改名前后均做了重名检查与死链检查。DMA 领域拆出 5 个子类；`cuda-memory.md` → `CUDA/memory.md`。
