---
tags: [cuda, memory]
updated: 2026-10-09
---

# CUDA 内存模型

## Pinned Memory 是什么？为什么是异步 DMA 的前提？ #cuda #memory #dma

> **速查**
> - 页锁定（page-locked）主机内存：物理页在 DMA 期间不可换出、不可搬移
> - 价值：让 `cudaMemcpyAsync` 真正异步；pageable 源会**静默退化**为同步拷贝
> - 代价：分配昂贵（改页表 + 锁物理页）+ 占不可回收物理内存 → 工程上做 Pinned Pool
> - 「能安全提交异步 DMA」是**必要条件**，≠「传输一定被计算隐藏」

### 正文

**① 为什么分页内存做不了 DMA**

虚拟内存的页表允许 OS 随时换出 / 搬移页面。这对 CPU 是透明的（CPU 走 MMU，拿到的是虚拟地址），但对 GPU 的 DMA 引擎致命：DMA 直接按**物理地址**搬数据，若传输途中页面被换出或搬走，数据就会损坏。

所以 pageable（普通 `malloc`）内存不能直接交给 DMA 引擎 —— 这是问题的根，不是性能问题。

**② 两条拷贝路径**

| | Pageable（`malloc`） | Pinned（`cudaMallocHost` / `cudaHostAlloc`） |
|---|---|---|
| 物理页 | 可换出 / 可搬移 | 锁定、地址稳定 |
| 拷贝路径 | 驱动先拷到内部 staging buffer，再 DMA | 直接 DMA |
| `cudaMemcpyAsync` | **必然阻塞**（退化成同步） | 立即返回、真正异步 |
| 分配 | 便宜 | 昂贵（改页表 + 锁物理页） |

关键：pageable 源之所以不报错却没有异步效果，正是因为驱动在背后替你做了「拷进 staging buffer」这一步，而这一步是同步的。

**③ 必要条件 ≠ 充分条件**

教材截图那句：「它解决『能否安全提交异步 DMA』的条件问题，并不保证传输一定被计算隐藏。」

- pinned 保证的是**能安全地**把 DMA 交给异步引擎（地址在整个传输期间稳定）——这是**前提条件**。
- 「能异步」≠「被隐藏」：异步提交只是把传输排进了 stream 队列，是否真正与计算重叠，还取决于 stream 编排、依赖顺序、以及计算是否足够久到盖住传输时间。

**④ 代价与工程取舍**

- 分配慢：需要改页表 + 锁物理页。
- 占不可回收物理内存，**过量会拖慢整个系统**（挤压其它进程的可用内存）。
- 因此工程上做 **Pinned Pool**：初始化时一次性分配一大块 pinned 内存，运行时只管理 offset / Block ID（控制面与数据面分离）。截图里「本章沿用上一章初始化时一次性分配的 Pinned Pool，运行时只分配 Block ID」正是这个取舍。

### 自测
- [ ] 为什么 `cudaMemcpyAsync` 传 pageable 指针不报错，却没有异步效果？
- [ ] Pinned Pool 解决了什么、没解决什么？

## Pinned 一定比 Pageable 快吗？异步拷贝能「完全隐藏」传输吗？ #cuda #memory #dma #performance #uva

> **速查**
> - **不一定，甚至可能更慢**：小传输时 pinned 的分配/管理开销可能超过重叠收益，pageable 的 staging 拷贝（CPU 侧内存到内存）反而更省事
> - 「能异步」≠「被隐藏」：kernel 短、或真依赖该数据时，stream 里仍是串行，异步提交无效
> - GPU 侧始终走同一条 PCIe —— pinned 省的是 CPU 侧路径，不减少传输量本身
> - pinned 页不可回收，过度使用挤压其它进程 → 拖慢整个系统
> - 第三条路：`cudaHostAllocMapped` + `cudaHostGetDevicePointer`（zero-copy / UVA），省显式拷贝，但每次访问走 PCIe、延迟高，仅适合只读一次且访问规律的数据

### 正文

**① 小传输：开销可能 > 收益**

pinned 的优势来自两点：DMA 能直接读，且能异步。但传输量很小时（几 KB）：

- 异步重叠省下的时间，小于 pinned 内存分配 / 管理的额外开销；
- GPU 侧还是得走同一条 PCIe，带宽没变；
- pageable 那次 staging buffer 拷贝是 CPU 侧的内存到内存拷贝，很快，可能比显式维护 pinned pool 更省事。

所以「小传输 + 一次性」场景，pinned 未必划算。

**② 「完全隐藏」是理想情况**

要隐藏传输，两个条件得同时成立：计算真的能和 DMA 并行，且计算时间长到盖住传输时间。

- kernel 很短 → 盖不住；
- kernel 依赖这块数据（真依赖）→ stream 里它必然排在拷贝之后，串行。

异步提交只是把传输排进 stream 队列，并不改变依赖关系。**能异步 ≠ 被隐藏。**

**③ 过度使用 pinned 伤整个系统**

pinned 页不可回收，分配过多会挤压其它进程的可用内存，系统整体变慢。这正是教程用 **Pinned Pool** 一次性分配、运行时只切 offset 的原因。

**④ 第三条路：zero-copy（UVA mapped）**

用 `cudaHostAlloc` 带 `cudaHostAllocMapped` 标志分配，再用 `cudaHostGetDevicePointer` 拿到设备侧指针，kernel 就能通过 UVA 直接访问主机内存 —— 省掉显式拷贝。

代价：每次访问都走 PCIe，延迟高。只在「数据只读一次、且访问模式规律」时划算。

### 自测
- [ ] 为什么「能异步提交」不等于「传输被计算隐藏」？
- [ ] 小传输场景下 pinned 为什么可能反而更慢？
- [ ] zero-copy（`cudaHostAllocMapped`）省掉了什么？代价是什么？
