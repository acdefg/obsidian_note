---
tags: [index, taxonomy]
updated: 2026-10-27
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
| CUDA | `CUDA/memory.md` | `[cuda, memory]` | **主机内存视角**：pinned、pageable、分页、UVA、zero-copy、异步拷贝、stream |
| CUDA | `CUDA/synchronization.md` | `[cuda, memory]` | **并发内存模型视角**：coherence vs consistency、relaxed/release/acquire、fence·barrier 三分、MESI、GPU scope 与 mbarrier、跨卡一致性 |
| CUDA | `CUDA/execution.md` | `[cuda, execution]` | warp、occupancy、bank conflict、**内存层级（register file / shared / local / global）与变量·栈的分配**。**SIMT 的范式辨析归 `计算机体系结构/`** |
| 计算机体系结构 | `计算机体系结构/computer-arch.md` | `[arch, gpu]` | 流水线、分支预测、乱序执行；GPU 微架构（SM/TMA/L2）、memory hierarchy、roofline；**SIMD / SIMT / 脉动阵列的范式对比**；**DNN 加速器的数据流（WS/OS/IS/RS）与复用因子、tiling 的流量代价** |
| 计算机体系结构 | `计算机体系结构/soc-architecture.md` | `[arch, soc]` | **SoC 架构岗位能力图谱**：JD 拆解（模块 owner vs 架构师）、产品定义与 PPA 预算、VMSA·MMU·SMMU·ASID/nG、TrustZone·安全启动、总线·crossbar·NoC 与带宽估算、UPF·VCLP·低功耗、存储层级与 tiling 预算、子系统划分与架构文档、后端·DFT·验证分工、缺口清单 |
| 计算机体系结构 | `计算机体系结构/建模/systemc-kernel.md` | `[arch]` | **SystemC 仿真内核**：离散事件调度、求值/更新两阶段、delta cycle 与时间推进、`sc_signal` vs `sc_buffer`、`SC_METHOD`/`SC_THREAD` 语义、零延迟环与多驱动 |
| 计算机体系结构 | `计算机体系结构/建模/tlm.md` | `[arch]` | **TLM-2.0 事务级建模**：LT vs AT、`b_transport`/`nb_transport` 四相、`tlm_generic_payload`、时间解耦 quantum、DMI、五个常见坑 |
| 计算机体系结构 | `计算机体系结构/建模/modeling-levels.md` | `[arch]` | **性能建模粒度**：解析 / trace-driven / LT·AT / 周期近似 / 周期精确；周期级状态语义（倒序执行 vs current-next 两阶段）；TLM-2.0 不含 cycle-accurate 标准 |
| 计算机体系结构 | `计算机体系结构/cache-microarchitecture.md` | `[arch, memory-hierarchy]` | **cache 硬件结构**：tag/index/offset、组相联、MSHR、write buffer、victim cache、包含关系；**替换算法**：LRU/MRU/FIFO/PLRU/NRU/RRIP/BIP 的硬件代价与扫描病理；**evict-first / streaming hint** 的软硬件分工；**访存合并**：CPU 侧 MSHR merge / write combining；GPU 侧 warp coalescing / 32B sector |
| 计算机体系结构 | `计算机体系结构/中断架构/risc-v-aia.md` | `[arch, interrupt]` | **RISC-V AIA**：AIA CSRs / IMSIC 中断文件·MSI·claim 语义 / APLIC 域·direct·MSI / PLIC 对比 / 软件栈（Linux·OpenSBI·QEMU·KVM·Spike）/ 开源 RTL 盘点 / 自研拆分与验证路线 |
| 计算机体系结构 | `计算机体系结构/中断架构/interrupt-basics.md` | `[arch, interrupt]` | **中断基础**：MMIO / hart / MSI / CSR（含间接 CSR）/ APLIC domain 分层 / direct（类 PLIC）/ direct·MSI·IPI·虚拟化完整 trap 流程 / 面试速查与常见误解 |
| 计算机体系结构 | `计算机体系结构/存内计算/rram-cim.md` | `[arch, cim]` | **RRAM 存内计算**：器件/阵列原理、GEMM 与矩阵求导映射、**tiling 与阵列映射的分层、MNK/batch 与复用因子**、精度补偿四层、仿真验证（含旧 RRAM 仿真笔记案例）、论文清单、面试背诵卡 |
| 计算机体系结构 | `计算机体系结构/存内计算/mixed-precision-cim.md` | `[arch, cim]` | **混合精度 CIM**：INT8/INT4 量化误差、bit-slicing 与 MVM 映射、混合精度分配与硬件支持、论文清单、面试背诵卡 |
| 项目经历 | `项目经历/华为/huawei-mpu.md` | `[arch, memory-hierarchy]` | **华为 MPU 项目案例**：message process：key/lockid、乱序分配、shift queue、64-key 冲突判断、dispatch、提前释放、删拍、自动时钟门控；**Leading-one RTL**（优先链 / 二分树 / lockid 位图） |
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
| C++ | `C++/STL/hash-table.md` | `[cpp]` | **STL 哈希表**：六种实现写法 / `unordered_map` 访问语义与坑 / 从声明到遍历的用法。领域其余范围（语言特性、模板、内存模型、并发）待建子类 |
| C++ | `C++/STL/string.md` | `[cpp]` | **STL 字符串**：声明·初始化 / `>>` 与 `getline` 的分工 / 常用操作表 / 遍历·切分·查找 / 七个坑 / SSO 与扩容 |
| C++ | `C++/0-1背包 完全背包.md` | `[cpp]` | **背包 DP**：0-1 / 完全 / 多重 / 分组 / 二维费用；二维→一维的逆序·正序 / 初始化与比较符 / 组合与排列。算法类笔记暂放 `C++/` 根目录，未开子类 |
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
- 2026-10-11 — 新建 `CUDA/synchronization.md`（内存模型与同步）。原 `CUDA/memory.md` 的「内存模型」四个字划走，它只剩主机内存视角（pinned / 分页 / UVA / zero-copy / 异步拷贝）；并发视角独立成文件，避免单文件越 400 行。两文件共用 `[cuda, memory]` frontmatter（同 `压缩/原理/*` 三文件共用 `[compression, theory]` 的先例）。
- 2026-10-11 — 新建 `CUDA/execution.md`（此前已在表中预留）。覆盖范围从「SIMT / warp / occupancy / bank conflict」扩到含**设备侧内存层级与变量·栈的分配**。曾考虑归入 `计算机体系结构/computer-arch.md`（该行覆盖范围写了「memory hierarchy」），但用户选择留在 CUDA 领域——`计算机体系结构` 领域文件夹**仍未建立**。
- 2026-10-17 — `computer-arch.md` 追加「数据流怎么选？复用因子怎么算？」。覆盖范围补上 **数据流四分类与复用因子**——此前只有脉动阵列的机制与利用率，缺"映射层"（哪份数据留片上、复用怎么算）。归入体系结构而非新开领域，因为回查角度是"这个架构为什么这么设计"，不是"我的 kernel 为什么慢"。
- 2026-10-12 — **建立 `计算机体系结构/` 领域**（首个文件 `computer-arch.md`），此前一直是 `<arch>` 占位。同时把 `CUDA/execution.md` 那行的覆盖范围收窄：**SIMT 的范式辨析归体系结构，CUDA 侧只留 warp / occupancy / bank conflict 与设备内存层级**。判据是「从哪个角度回查」——问「SIMT 和脉动阵列差在哪」是体系结构问题，问「我的 kernel 占用率为什么低」才是 CUDA 问题。
- 2026-10-18 — 新建 `计算机体系结构/cache-microarchitecture.md`（cache 硬件结构 / 替换算法 / 访存合并）。文件名不用 `cache.md`：vault 内 `study_note/pku-note/数字集成/cache.md` 已占用，`[[cache]]` 会歧义。`computer-arch.md` 的覆盖范围去掉「cache」，改由本文件覆盖。
- 2026-10-18 — 新建 `计算机体系结构/项目/huawei-mpu.md`（华为 MPU 项目案例）。TAXONOMY 暂无「项目经历」领域，暂归到计算机体系结构下的「项目」子类；若后续项目笔记增多，再单开领域。
- 2026-10-18 — 单开「项目经历」领域，`huawei-mpu.md` 从 `计算机体系结构/项目/` 迁到 `项目经历/华为/`。项目笔记按公司分子类；后续摩尔线程、算能、润石、北大项目沿用此结构。知识概念仍留在各领域，项目案例单独沉淀。
- 2026-10-19 — 新建 `计算机体系结构/存内计算/rram-cim.md` 与 `mixed-precision-cim.md`。TAXONOMY 暂无独立「存内计算」领域，按“先归最接近领域”的规则暂放 `计算机体系结构/存内计算/`；若后续 CIM 笔记继续增多，再单开领域。两文件共用 `[arch, cim]` frontmatter，细粒度用 `#rram` / `#quantization` / `#mixed-precision`。
- 2026-10-24 — `rram-cim.md` 追加「Tiling 和阵列映射是什么关系？」与「MNK、batch 和复用因子怎么对应？」，补上映射层与复用因子的分层；`computer-arch.md` 的数据流条目加交叉链接。
- 2026-10-24 — 新建 `计算机体系结构/中断架构/risc-v-aia.md`。TAXONOMY 暂无「中断架构」子类，按“先归最接近领域”的规则放在 `计算机体系结构/` 下；覆盖 AIA 规范、IMSIC/APLIC 机制、软件栈与开源 RTL 实现盘点。若后续中断/异常笔记增多，再考虑单开领域。
- 2026-10-24 — 新建 `计算机体系结构/中断架构/interrupt-basics.md`。把 MMIO / hart / MSI / CSR / APLIC domain / direct / 完整 trap 流程从对话整理成基础层，和 `risc-v-aia.md` 分工：本篇讲“词是什么、路径怎么走”，AIA 篇讲“规范细节、软件栈、开源 RTL”。两篇互相链接。
- 2026-10-24 — **建立 `C++/` 领域**（首个文件 `C++/STL/hash-table.md`），此前一直是 `<cpp>` 占位。子类先只开 `STL/`；语言特性、模板、内存模型、并发后续各按子类归入。覆盖范围补上 **STL 容器与用法**。
- 2026-10-25 — 新建 `C++/STL/string.md`。C++ 行拆成两行：哈希表与字符串各占一行（同一 `[cpp]` frontmatter）；领域其余范围仍标注「待建子类」。
- 2026-10-26 — 追加 `C++/0-1背包 完全背包.md`（背包 DP 全情况矩阵）。这是 C++ 领域第一份**算法类**笔记，暂放根目录；若后续 DP/图论/字符串算法继续累积，再开 `C++/算法/` 子类并迁移。新增标签 `#dp`（见条目行尾）。
- 2026-10-25 — `计算机体系结构/cache-microarchitecture.md` 追加「MRU 是什么？evict-first / streaming hint 和它是什么关系？」，覆盖范围补上 **MRU/FIFO 与 evict-first·streaming hint**。不新建文件：同属“替换算法”回查路径，与已有条目共享同一主题。同时给旧条目补 MRU/FIFO 两行对照，并修正一个常见说法——4 路 tree-PLRU 的 victim 也不可能是第 2 新的块，扫描病理属于所有“偏爱新块”的策略（含 FIFO、tree-PLRU），不是 LRU 独有。
- 2026-10-27 — 新建 `计算机体系结构/soc-architecture.md`（SoC 架构岗位知识图谱）。由两份 JD（SoC 前端设计 / SoC 架构师）反推知识域，9 个条目：JD 对比、VMSA·SMMU、TrustZone、互连、低功耗、存储、子系统、后端验证、缺口清单。新增标签 `#soc`、`#security`、`#low-power`。
- 2026-10-27 — 新建 `计算机体系结构/建模/systemc-kernel.md`（SystemC 离散事件内核 / delta cycle）。TAXONOMY 暂无「建模」子类，暂放 `计算机体系结构/` 下；文件名用 `systemc-kernel` 而非 `systemc`，避免与 `pku-note/数字集成/systemC.md` 仅大小写不同。未新增标签（沿用 `#arch`）。
- 2026-10-27 — 新建 `计算机体系结构/建模/tlm.md`（TLM-2.0 事务级建模）。与 `systemc-kernel.md` 分工：内核讲调度语义，TLM 讲事务层怎么建模时间；两篇互链。未新增标签（沿用 `#arch`）。
- 2026-10-27 — 新建 `计算机体系结构/建模/modeling-levels.md`（性能建模粒度与周期级状态语义）。把「解析 / trace-driven / LT·AT / 周期近似 / 周期精确」分层，并说明倒序执行 vs current-next 两阶段；与 `tlm.md`、`systemc-kernel.md` 互链。未新增标签（沿用 `#arch`）。
