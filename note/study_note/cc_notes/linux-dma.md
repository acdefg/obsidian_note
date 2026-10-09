---
tags: [dma, kernel]
updated: 2026-10-10
---

# Linux DMA 子系统

> 全文三个关键区分：
> - **DMA address（`dma_addr_t`）**：交给设备的地址，经 IOMMU 翻译到物理地址；无 IOMMU 时通常等于物理地址。**不是 CPU 虚拟地址，CPU 不能直接解引用。**
> - **IOMMU**：设备侧地址翻译硬件（x86 的 VT-d、ARM 的 SMMU）。
> - **cache 维护（clean / invalidate）**：CPU 指令，把脏 cache line 写回内存 / 作废 cache line。硬件提供能力，软件决定何时执行。

## Linux DMA-API 是怎么工作的？ #dma #kernel #iommu

> **速查**
> - 解决的问题：驱动手里是 CPU 虚拟地址，设备要的是总线地址；且 CPU cache 与设备看到的内存可能不一致
> - 三个动作：**映射**（`dma_map_*`）→ **同步**（`dma_sync_*`，仅当 CPU 与设备分段复用同一缓冲区）→ **解映射**（`dma_unmap_*`）
> - `dma_map_*` 在**把缓冲区交给设备之前**调用；`dma_unmap_*` 在**设备用完（完成中断/轮询到完成）之后**
> - **方向 `enum dma_data_direction` 决定要做 clean 还是 invalidate**，不是建议：`DMA_TO_DEVICE` / `DMA_FROM_DEVICE` / `DMA_BIDIRECTIONAL` / `DMA_NONE`
> - **一致架构（x86）**：总线做 cache 监听，映射/同步基本都是空操作（有 IOMMU 时只剩 IOMMU 映射）
> - **非一致架构（多数嵌入式 ARM/PowerPC）**：不处理就是错——CPU 的脏 cache line 设备读不到，设备的写 CPU 看不到
> - 归属：【软硬协同】内核填参数、做 cache 维护、编程 IOMMU；硬件做翻译与一致性协议

### 正文

**主要函数**：

```c
/* 映射：准备给设备用 */
dma_addr_t dma_map_single(struct device *dev, void *ptr, size_t size,
                          enum dma_data_direction dir);
dma_addr_t dma_map_page(struct device *dev, struct page *page,
                        size_t offset, size_t size, enum dma_data_direction dir);
int        dma_map_sg(struct device *dev, struct scatterlist *sg,
                      int nents, enum dma_data_direction dir);

/* 同步：所有权在 CPU 与设备之间转移 */
void dma_sync_single_for_cpu(struct device *dev, dma_addr_t addr,
                             size_t size, enum dma_data_direction dir);
void dma_sync_single_for_device(struct device *dev, dma_addr_t addr,
                                size_t size, enum dma_data_direction dir);

/* 解映射 */
void dma_unmap_single(struct device *dev, dma_addr_t addr, size_t size,
                      enum dma_data_direction dir);
void dma_unmap_sg(struct device *dev, struct scatterlist *sg,
                  int nents, enum dma_data_direction dir);
```

**调用时机（顺序不能错）**：

| 时机 | 调用 |
|---|---|
| 把缓冲区交给设备之前 | `dma_map_*` |
| 设备用完（收到完成中断 / 轮询到完成）之后 | `dma_unmap_*` |
| 同一缓冲区被 CPU 和设备**分段复用**时 | `dma_sync_*` |

典型分段序列：设备写完 → `sync_for_cpu` → CPU 读 → CPU 改 → `sync_for_device` → 设备再读。

**为什么需要 sync 系列**：

- **一致架构**：设备 DMA 会让 CPU cache 自动失效/更新（硬件监听），map/sync/unmap 基本空操作。
- **非一致架构**：设备 DMA **绕过** CPU cache。不处理的三种后果：
  - CPU 刚写的数据还在脏 cache line 里没进内存 → 设备读到**旧数据**
  - 设备写进内存，CPU cache 里还是旧行 → CPU 读到旧数据
  - **最糟**：map 期间 cache line 被回写，覆盖设备刚写的数据

  处理方式：`for_device` 对 `DMA_TO_DEVICE`/`BIDIRECTIONAL` 做 **clean**；`for_cpu` 对 `DMA_FROM_DEVICE`/`BIDIRECTIONAL` 做 **invalidate**。

**一个容易踩的坑**：非一致架构上，映射区**必须对齐 cache line**，否则 invalidate 会连带作废相邻数据（cache line 撕裂）。

**设备地址宽度**用 `dma_set_mask_and_coherent(dev, mask)` 设置；设备寻址不到的内存会走 bounce buffer（见下文）。

### 自测
- [ ] `dma_map_*` / `dma_sync_*` / `dma_unmap_*` 各自在什么时机调用？
- [ ] 在一致架构上这三个函数基本是空操作，为什么还要写？
- [ ] 非一致架构上不做 sync，会出现哪几种错误？哪种最隐蔽？

## Coherent mapping 和 streaming mapping 怎么选？ #dma #kernel

> **速查**
> - **Coherent**（`dma_alloc_coherent`）：CPU 与设备可同时访问，**无需 sync**。代价是 CPU 访问慢（非一致架构上分配不可 cache 内存）、预留区稀缺
> - **Streaming**（`dma_map_single/page/sg` + sync + unmap）：映射期间缓冲区**归设备所有**，CPU 要访问必须先 sync。复用普通可 cache 内存
> - **选择规则**：**控制面**（长期存在、频繁小改：命令环、描述符环、mailbox）用 coherent；**数据面**（大数据量、一次性：网卡收发帧、块设备 I/O）用 streaming
> - **两者可混用**：描述符环 coherent，环里指向的数据缓冲 streaming
> - ⚠️ **`dma_alloc_coherent` 的内存不能拿去 `dma_map_single`** —— 语义冲突，会触发 DMA-API debug 告警
> - 归属：【软硬协同】接口由内核 DMA-API 实现；"一致性"由硬件监听保证还是靠软件 cache 维护，取决于架构

### 正文

| | Coherent | Streaming |
|---|---|---|
| 接口 | `dma_alloc_coherent(dev, size, &dma_handle, gfp)`<br>成对 `dma_free_coherent()` | `dma_map_*` + `dma_sync_*` + `dma_unmap_*` |
| 语义 | CPU 与设备可同时访问，免 sync | 映射期间归设备所有（单向） |
| 实现 | 一致架构：普通可 cache 内存 + 硬件监听<br>非一致架构：**不可 cache 内存**或预留区/CMA | 映射已有内存，每次传输做映射/解映射 |
| 代价 | CPU 访问慢；缓存属性/预留区稀缺；分配粒度受限（通常按页） | 每次映射有开销（IOMMU 表项增删、cache 维护、地址计算） |
| 适用 | 命令环、描述符环、mailbox、固件共享结构 | 网卡收发帧、块设备 I/O、采集卡数据流 |

`dma_alloc_coherent` 返回**两个值**：CPU 虚拟地址 + DMA 地址。

### 自测
- [ ] 描述符环为什么用 coherent 而不是 streaming？
- [ ] streaming 映射期间，CPU 想读缓冲区必须先做什么？
- [ ] 为什么 coherent 内存在非一致架构上 CPU 访问慢？

## scatterlist 与 dma_map_sg 的合并 #dma #kernel #sgl

> **速查**
> - `struct scatterlist` 描述**一段**内存；数组用于拼接非连续缓冲区
> - `page_link` / `offset` / `length` 是 **CPU 视角**；`dma_address` / `dma_length` **只有 `dma_map_sg` 之后才有意义**
> - **`dma_map_sg` 的返回值可以 ≤ 输入的 nents** —— 底层实现**有权合并相邻段**（有 IOMMU 时把相邻物理页分到连续 IOVA）
> - 合并后**必须用 `sg_dma_address(s)` / `sg_dma_len(s)`**，不能再用 `sg->length`（那是 CPU 视角）；`sg_dma_len` 可能大于原 `sg->length`
> - ⚠️ **`dma_unmap_sg` / `dma_sync_sg_*` 必须用原始 nents，不能用返回值** —— 用错会导致映射状态错乱、IOMMU 表项泄漏或提前释放
> - ⚠️ **按原始 nents 遍历映射后的列表** → 读到未映射/已合并掉的项 → 有 IOMMU 时触发 DMA fault，无 IOMMU 时 **DMA 到错误物理地址（静默内存破坏）**
> - 归属：【软件】结构与映射逻辑是内核代码；"能否合并"取决于硬件能力（IOMMU 的 IOVA 连续分配、设备的段数上限）

### 正文

```c
struct scatterlist {
    unsigned long  page_link;    /* struct page* + SG_END/chain 标志 */
    unsigned int   offset;
    unsigned int   length;       /* CPU 视角 */
    dma_addr_t     dma_address;  /* 映射后填充 */
    unsigned int   dma_length;   /* 映射后填充 */
};
```

`struct sg_table` = `scatterlist` 数组 + `nents` + `orig_nents`，用于管理生命周期。

**为什么会合并**：有 IOMMU 时（`iommu_dma_map_sg`），IOVA 分配器可能给相邻物理页分配一段**连续的 IOVA**，于是多个段可合并成一条 DMA 段。无 IOMMU 但物理连续时也可能合并。合并也用于把段数压到设备的 `max_segments` 以内。

**正确写法**：

```c
nents = dma_map_sg(dev, sg, nents, dir);
if (!nents) return -EIO;
for_each_sg(sg, s, nents, i) {          /* 用返回值遍历 */
    dma_addr_t a = sg_dma_address(s);
    unsigned int n = sg_dma_len(s);
    /* 填进硬件描述符 */
}
```

**三种典型误用**：

1. 用返回的 count 去调 `dma_unmap_sg` / `dma_sync_sg_*` → 映射状态错乱、IOMMU 表项泄漏
2. 按**原始** nents 遍历映射后的列表 → 读到未映射项 → DMA fault 或静默内存破坏
3. 把返回值赋给 `sg_table->nents` → `nents` 与 `orig_nents` 混用（DRM 等子系统批量修过的 bug）

**建议用 `dma_map_sgtable()` / `dma_unmap_sgtable()`** —— 内部区分 `orig_nents` 与映射后的 `nents`。

相关：[[dma-address-path#三层地址视图：谁能在哪一层合并？]]

### 自测
- [ ] `dma_map_sg` 的返回值为什么可能小于传入的 nents？
- [ ] unmap 应该用原始 nents 还是返回值？用错的后果是什么？
- [ ] 合并之后，为什么不能再用 `sg->length`？

## Bounce buffer 与 SWIOTLB #dma #kernel

> **速查**
> - 解决的问题：**设备 DMA 只能访问它地址宽度以内的内存**。目标缓冲区超出范围时不能直接映射
> - **Bounce buffer** = 一块"设备可寻址内存"里的临时缓冲区；设备读方向先拷进去，设备写方向完成后拷回来
> - **SWIOTLB** = 内核维护的 bounce buffer **池** + 一张软件 IO-TLB 复用它们（**纯数据结构，不是硬件 TLB**）
> - 触发条件：32 位设备 + >4GB 物理内存、32 位系统的 highmem、**内存加密场景**（SEV/TDX，设备只能访问未加密的共享内存）
> - **有真正的 IOMMU 且已启用时，通常不需要 bounce**
> - 代价：**每个不可达缓冲区多做一次内存拷贝** + 池占用的预留物理内存 + 池满时的竞争
> - 归属：【软件】——纯内核代码（`kernel/dma/swiotlb.c`、`kernel/dma/direct.c`）；由**硬件限制**触发

### 正文

**路径**：没有 IOMMU ops 时内核走 `dma-direct`，它在地址不可达时回退到 SWIOTLB。

**调试手段**：内核参数 `swiotlb=force` 强制所有 DMA 走 bounce，用来复现场景。

### 自测
- [ ] 什么条件下会触发 bounce buffer？
- [ ] 有 IOMMU 之后为什么通常不需要 bounce？
- [ ] SWIOTLB 名字里有 TLB，它是硬件吗？

## dmaengine API 的异步提交模型 #dma #kernel

> **速查**
> - 解决的问题：不同厂商的 DMA 控制器寄存器/描述符格式不同，把它抽象成统一的**通道 + 描述符 + cookie** 模型
> - **四步**：请求通道（`dma_request_chan`）→ 配置（`dmaengine_slave_config`）→ 准备描述符（`dmaengine_prep_slave_sg`）→ **提交与启动**（`dmaengine_submit` + `dmaengine_issue_pending`）
> - ⚠️ **`dmaengine_submit` 只是入队；`dmaengine_issue_pending` 才真正推给控制器** —— 漏了后一步传输永远不开始
> - **`dma_cookie_t`** 是每通道**单调递增**的事务标识，用于追踪
> - 完成通知两种：**回调**（`tx->callback`）或**轮询 cookie**（`dma_async_is_tx_complete` / `dma_sync_wait`）
> - ⚠️ **回调不能睡眠** —— 它通常跑在 tasklet/软中断里，只能做轻量收尾
> - 归属：【软硬协同】内核准备描述符、管理队列、回调；DMA 控制器（硬件）执行搬运

### 正文

```c
/* 1. 请求通道（驱动 probe 时） */
struct dma_chan *dma_request_chan(struct device *dev, const char *name);

/* 2. 配置从设备 */
struct dma_slave_config cfg = {
    .direction = DMA_MEM_TO_DEV,
    .dst_addr = reg_base + FIFO,
    .dst_addr_width = DMA_SLAVE_BUSWIDTH_4_BYTES,
    .dst_maxburst = 16,
};
dmaengine_slave_config(chan, &cfg);

/* 3. 准备描述符 */
struct dma_async_tx_descriptor *tx =
    dmaengine_prep_slave_sg(chan, sgl, sg_len, DMA_MEM_TO_DEV, DMA_PREP_INTERRUPT);

/* 4. 设置回调 → 提交 → 启动 */
tx->callback = my_done_cb;
tx->callback_param = my_ctx;
dma_cookie_t cookie = dmaengine_submit(tx);
dmaengine_issue_pending(chan);          /* ← 少了这步传输不会开始 */
```

**其他 prep 变体**：`dmaengine_prep_dma_memcpy`（内存到内存）、`_memset`、`_cyclic`（循环缓冲，音频常用）、`_sg`（SG 到 SG）、`_xor`。

**清理**：`dmaengine_terminate_sync(chan)` / `dmaengine_terminate_all(chan)`，`dma_release_channel(chan)`。

### 自测
- [ ] `dmaengine_submit` 和 `dmaengine_issue_pending` 分别做什么？漏掉一个会怎样？
- [ ] `dma_cookie_t` 的作用是什么？
- [ ] 为什么完成回调里不能睡眠？

## dma-buf：跨驱动共享缓冲区 #dma #kernel

> **速查**
> - 解决的问题：不同驱动（GPU / 显示控制器 / 视频编解码 / 摄像头）要共享**同一块物理内存**，不拷贝、也不互相暴露私有分配器
> - 机制：**导出方**（exporter）+ **导入方**（importer）通过**文件描述符**引用同一 buffer
> - `dma_buf_attach(dmabuf, dev)` → `dma_buf_map_attachment()` 产出 `sg_table` → 再走 `dma_map_sg` 得到本设备视角的 DMA 地址
> - **每个导入设备各有一份 attachment** —— 同一 buffer 在不同设备/IOMMU 域里的 DMA 地址不同
> - **`dma_fence`**（带 `context` + `seqno`）表示某个异步操作的完成，支持回调 / 阻塞等待 / 硬件信号
> - **`dma_resv`** 维护一组共享 fence（读，可多个）+ 一个独占 fence（写，唯一）
> - **隐式同步**（驱动自动查 dma_resv 上的 fence）vs **显式同步**（用户态通过 `sync_file` fd 传递依赖）
> - 归属：【软件】——跨驱动框架；fence 可对应硬件完成信号，但机制由内核实现

### 正文

```c
struct dma_buf_attachment *a = dma_buf_attach(dmabuf, dev);
struct sg_table *sgt = dma_buf_map_attachment(a, DMA_BIDIRECTIONAL);
/* ... 交给本驱动走 dma_map_sg ... */
dma_buf_unmap_attachment(a, sgt, DMA_BIDIRECTIONAL);
dma_buf_detach(dmabuf, a);
```

**`struct dma_buf_ops` 的关键项**：`map_dma_buf` / `unmap_dma_buf`（产出 sg_table）、`attach` / `detach`、`begin_cpu_access` / `end_cpu_access`（CPU 访问前后的 cache 同步钩子）、`mmap`、`vmap`、`release`。

### 自测
- [ ] 为什么每个导入设备都需要一份独立的 attachment？
- [ ] `dma_resv` 里为什么读 fence 可以多个、写 fence 只能一个？
- [ ] 显式同步相比隐式同步，把什么责任交给了用户态？

## P2PDMA：PCIe 设备间直连 DMA #dma #kernel #pcie

> **速查**
> - 让一个 PCIe 端点**直接 DMA 到另一个端点的 BAR 内存**，数据不经系统内存往返
> - 收益：低延迟 + 省内存带宽。典型场景：GPU 直读 NVMe、网卡直写显存、加速卡之间直连
> - **三个前提条件**（缺一不可）：
>   1. **PCIe 拓扑可达**——通常要在**同一个 PCIe 交换机下**（或同一 root port 下，取决于主机桥能力）
>   2. **ACS 配置正确**——路径上的下游端口**不能开 P2P Request Redirect / P2P Completion Redirect**，否则 P2P TLP 被重定向到 root complex，P2P 失效
>   3. **IOMMU 配置正确**——直连 BAR 物理地址（`MAP_BUS_ADDR`）时 IOMMU 必须不拦截
> - 映射类型：`PCI_P2PDMA_MAP_BUS_ADDR`（直接用 BAR 总线地址）/ `THRU_HOST_BRIDGE`（经主机桥）/ `NOT_SUPPORTED`
> - ⚠️ **内核为了 P2P 会清掉交换机下游端口的 ACS 重定向位**，这会把交换机下所有设备并入**同一个 IOMMU group**，**削弱隔离性/安全性** —— 所以发行版通常默认不开 `CONFIG_PCI_P2PDMA`
> - ⚠️ **历史上多数 x86 平台上 P2P 需要关闭 IOMMU**（NVIDIA P2P 长期如此要求）；AMD IOMMU 有部分 P2P 直连支持
> - 归属：【软硬协同】可达性判定与映射选择在内核（`drivers/pci/p2pdma.c`）；实际直连由 PCIe 拓扑与交换机/主机桥完成

### 正文

**关键接口**：

- 导出方：`pci_p2pdma_add_resource()` 登记 BAR 为 P2P 内存；`pci_p2pmem_publish()` 允许他人 `pci_p2pmem_find*()`；`pci_p2pmem_alloc_sgl()` 分配 P2P 内存的 scatterlist
- 消费方：`pci_p2pdma_distance_many()` 判断可达性；`pci_p2pdma_map_sg()` / `pci_p2pdma_map_type()`

**相关启动参数**：`pcie_acs_override=downstream,multifunction`、`disable_acs_redir`、`pci=config_acs=<flags>@<dev>`、`iommu=pt`。**ACS 必须在设备枚举早期配置好，IOMMU group 一旦划分不能热改。**

> ⚠️ **不确定项**：ATC/ATS 型 P2P（ACS "Direct Translated P2P" 位）的具体要求，随 CPU/芯片组与内核版本差异大，本笔记未逐一核实。上文的"通常需关闭 IOMMU"是历史常见结论，**非普适**。实际以目标平台与内核版本为准。

### 自测
- [ ] P2P 的三个前提条件分别是什么？
- [ ] 为什么为了 P2P 要关闭 ACS 重定向位？代价是什么？
- [ ] 为什么很多平台上 P2P 要求关掉 IOMMU？

## get_user_pages / FOLL_PIN 与 dma_map_* 的分工 #dma #kernel #pinned-memory

> **速查**
> - 二者是**前后衔接的两步，职责不重叠**：GUP 回答"**哪些页**"，`dma_map_*` 回答"**这些页的 DMA 地址是什么**"
> - **DMA 场景应当用 `pin_user_pages*`（内部 `FOLL_PIN`），不是 `get_user_pages`（`FOLL_GET`）**
> - 释放要配对：`FOLL_PIN` 用 `unpin_user_page()`；`FOLL_GET` 用 `put_page()`
> - **顺序不可颠倒**：先 pin 再 map；先 unmap 再 unpin
>   - 先 unpin 再 unmap → 页可能在映射仍有效时被迁移，DMA 地址失效
> - pin 保证的是：pin 期间页**不会被回收 / 迁移 / 换出**（`FOLL_PIN` 页不可驱逐）
> - `FOLL_PIN` 与 `FOLL_GET` 在单次调用里**互斥**；`FOLL_LONGTERM` 要求 `FOLL_PIN`
> - 归属：【软件】——纯内核内存管理（`mm/gup.c`），本身不做 DMA，只负责锁定用户态页

### 正文

```c
/* 典型完整序列 —— 顺序不可颠倒 */
pin_user_pages_fast(uaddr, npages, FOLL_WRITE, pages);   /* 1. 钉住 */
nents = dma_map_sg(dev, sg, nents, DMA_FROM_DEVICE);     /* 2. 映射 */
/* ... 启动设备 DMA，等完成中断 ... */
dma_unmap_sg(dev, sg, nents, DMA_FROM_DEVICE);           /* 3. 解映射 */
for (i = 0; i < npages; i++) unpin_user_page(pages[i]);  /* 4. 解除钉住 */
```

**为什么 pin 住还不够**：pin 解决"页的**物理地址**稳定"，但没解决"设备要用的**总线地址**是什么"。后者要经 `dma_map_*`（IOMMU 翻译 + cache 维护 + 可能的 bounce）。

`pin_user_pages*` 通过 `GUP_PIN_COUNTING_BIAS`（初值 1024）在 `page->_refcount` 高位累计 pin 计数，使内核能识别"该页被长期 pin 住"。

> ⚠️ **不确定项**：`FOLL_PIN` / `FOLL_LONGTERM` 的语义在内核版本间有演进（长期 pin 的回收/迁移限制细节），以目标版本代码为准。

相关：[[cuda-memory#pin 是软件机制还是硬件机制？]] · [[cuda-memory#为什么操作系统一定要分页？]]

### 自测
- [ ] GUP 和 `dma_map_*` 各自负责什么？为什么不能合并成一步？
- [ ] 为什么必须先 unmap 再 unpin，不能反过来？
- [ ] DMA 场景为什么该用 `pin_user_pages` 而不是 `get_user_pages`？

---

## 术语速查

| 术语 | 是什么 | 层次 |
|---|---|---|
| DMA address (`dma_addr_t`) | 交设备的地址，经 IOMMU 翻译到物理地址 | 内核 DMA-API 概念 |
| IOMMU | 设备侧地址翻译硬件（VT-d / SMMU） | 硬件 |
| cache 维护（clean / invalidate） | CPU 指令，把脏 cache line 写回 / 作废 | 硬件能力，软件触发 |
| SWIOTLB | 软件 bounce buffer 池 + 软件 IO-TLB | 内核代码 |
| scatterlist | 描述非连续内存段的结构 | 内核数据结构 |
| dmaengine | DMA 控制器的统一抽象 | 软硬协同 |
| dma-buf | 跨驱动共享 buffer 框架 | 内核框架 |
| dma_fence | 异步操作完成同步原语 | 内核框架（可映射硬件中断） |
| P2PDMA | PCIe 设备间直连 DMA | 软硬协同 |
| FOLL_PIN | gup 的钉页语义，用于 DMA pin | 内核内存管理 |
| ACS | PCIe P2P 流量控制/隔离扩展能力 | 硬件能力，内核配置 |

## 来源

- Linux 内核文档：`Documentation/core-api/dma-api.rst`、`dma-api-howto.rst`
- `dma_map_sg` 返回值与 unmap 必须用原始 nents：https://lkml.org/lkml/2014/1/16/116
- `nents` vs `orig_nents` 误用修复系列：https://lkml.org/lkml/2020/4/28/551
- `pin_user_pages` / FOLL_PIN 文档：https://www.kernel.org/doc/html/v6.10/core-api/pin_user_pages.html
- P2PDMA ACS 清除补丁：https://lkml.org/lkml/2018/3/12/836
- P2PDMA 与 ACS/IOMMU 关系（LPC 材料）：https://lpc.events/event/18/contributions/1685/attachments/1424/3047/ATS%20vs%20IOMMU-lyan.pdf
