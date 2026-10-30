---
tags: [arch, memory-hierarchy]
updated: 2026-10-25
created: 星期日, 十月 25日 2026, 11:19:16
modified: 星期日, 十月 25日 2026, 11:28:09
---

# Cache 的硬件结构与访存合并

## Cache 的硬件结构：一次访问到底经过哪些部件？ #arch #memory-hierarchy

> **速查**
> - 地址拆成 **tag / set index / block offset**：index 选组，tag 并行比对定命中，offset 选块内字节
> - 容量 = 组数 × 路数 × 行大小；组相联用 N 个 tag 比较器换更低的冲突 miss
> - 命中：TLB → tag array → data array；miss：MSHR 登记 → 下一级取块 → 填回并唤醒等待者
> - 关键结构：tag/data array、valid/dirty、替换状态位、MSHR、write buffer、victim cache、prefetcher
> - 相联度只影响冲突 miss，不影响 compulsory miss；容量 miss 要靠更大的 cache

### 正文

**一张图看懂**

```text
                        CPU core
                           │
                  VA (virtual address)
                           │
                    ┌──────▼──────┐
                    │     TLB     │  VA → PA
                    └──────┬──────┘
                           │ PA
                           ▼
        ┌──────────────────────────────────────────────┐
        │                  L1 cache                    │
        │                                              │
        │   address: ┌────────┬───────────┬─────────┐  │
        │            │  tag   │ set index │ offset  │  │
        │            └───┬────┴─────┬─────┴────┬────┘  │
        │                │          │          │       │
        │                ▼          ▼          │       │
        │         ┌───────────┐  select set    │       │
        │         │ tag array │◀───────────────┘       │
        │         └─────┬─────┘                        │
        │               │ compare N ways               │
        │         ┌─────▼─────┐                        │
        │         │   hit?    │── hit ──▶ data array ──┼──▶ CPU
        │         └─────┬─────┘        (use offset)    │
        │               │ miss                         │
        └───────────────┼──────────────────────────────┘
                        ▼
                ┌───────────────┐
                │     MSHR      │  primary / secondary
                └───────┬───────┘
                        │ request
                        ▼
                ┌───────────────┐
                │   L2 / LLC    │
                └───────┬───────┘
                        │ miss
                        ▼
                ┌───────────────┐
                │     DRAM      │
                └───────┬───────┘
                        │ data return
                        ▼
          fill data array + valid = 1
          if victim dirty → writeback to L2/DRAM
```

**① 地址怎么拆**

以 32 位地址、32KB、8 路、64B 行为例：

- block offset = log₂(64) = **6 位**
- 块数 = 32KB / 64B = 512
- 组数 = 512 / 8 = 64 → set index = log₂(64) = **6 位**
- tag = 32 − 6 − 6 = **20 位**

公式：

```
offset = log2(line_size)
sets   = capacity / (ways × line_size)
index  = log2(sets)
tag    = address_bits − index − offset
```

**② 三种组织**

| | 直接映射 | 组相联 | 全相联 |
|---|---|---|---|
| 每组路数 | 1 | N | 全部 |
| tag 比较器 | 1 | N | 全部 |
| 冲突 miss | 高 | 中 | 低 |
| 替换算法 | 不需要 | 需要 | 需要 |
| 典型用途 | 小 cache / 低功耗 | L1 / L2 / L3 | TLB / victim cache |

直接映射每组只有一路，所以没有“选谁逐出”的问题——新块只能覆盖唯一的位置。组相联/全相联才需要替换算法。

**③ 一次访问的路径**

1. 用 index 选出组；组内 N 路的 tag 并行比较，同时读 valid 位。
2. 任一路 tag 匹配且 valid=1 → 命中，用 offset 从 data array 取字节。
3. 全不匹配 → miss：
   - 在 MSHR 中分配/查找条目（同一 line 的 miss 可以合并）
   - 向下一级发请求；若该组已满，按替换算法选 victim
   - 数据返回后写入 data array、置 valid；若 victim 是 dirty，先写回
   - 唤醒等待该 line 的请求
4. 写操作还要看写策略：write-back（只写 cache，置 dirty）或 write-through（同时写下一级）；write-allocate 决定 miss 时是否先把块取上来。

**④ 关键结构各自在做什么**

| 结构 | 作用 | 归属 |
|---|---|---|
| tag / data array | 存 tag 与数据；tag 比较决定命中 | 硬件 |
| valid / dirty | valid=块有效；dirty=块被改过、替换时要写回 | 硬件 |
| 替换状态位 | LRU / NRU / RRIP 等算法的状态 | 硬件 |
| MSHR | 记录 outstanding miss，合并同一 line 的 secondary miss | 硬件 |
| write buffer | 暂存待写回/待写下的数据，合并写 | 硬件 |
| victim cache | 少量全相联条目，暂存刚被逐出的块，降低冲突 miss | 硬件 |
| prefetcher | 预测并提前取块；可在硬件或软件（编译器 / 显式指令） | 硬件 / 软件 |

**⑤ 包含关系**

inclusive（L2 包含 L1 的所有块）、exclusive（两级互斥）、non-inclusive（不保证）。包含关系影响一致性失效的范围，也影响压缩/去重能否跨级共享；exclusive 能提高有效容量，但替换路径更复杂。

> 来源：Hennessy & Patterson, *Computer Architecture: A Quantitative Approach*（cache 组织与地址划分）；MSHR 概念见 Kroft, ISCA 1981。

相关：[[hardware#什么场景下压缩是净亏]]

### 自测
- [ ] 32KB、8 路、64B 行、32 位地址：index 和 tag 各几位？
- [ ] 组相联降低的是哪一类 miss？它不能降低哪一类？
- [ ] MSHR 和 victim cache 分别在解决什么问题？
- [ ] write-back 和 write-through 对 dirty 位、写回路径有什么不同？

## 替换算法：LRU 为什么难硬件实现？PLRU / RRIP 怎么折中？ #arch #memory-hierarchy #performance

> **速查**
> - 精确 LRU 要记录每路的完整访问顺序：N 路需 log₂(N!) 位（8 路约 16 位），命中还要更新顺序 → 面积/延迟都贵
> - 实际用近似：tree-PLRU（N−1 位）、NRU / Clock（1 位/路）、RRIP（2 位 RRPV）、Random（0 位）
> - LRU 的病理：C+1 个块的循环访问 → 命中率 0%；FIFO、tree-PLRU 同样归零；反向极端是 MRU（80%，随机访问灾难）；BIP 把新块插到 LRU 位置，保护热工作集
> - RRIP 用 2 位 RRPV 预测再引用距离：SRRIP 新块插到 2，命中置 0，替换找 3，找不到就全体加 1
> - 替换算法只影响冲突/容量 miss，不能减少 compulsory miss

### 正文

**① 为什么需要替换算法**

组相联/全相联的一组满了，必须选一路逐出；选谁，决定未来的 miss。直接映射每组只有一路，没有选择。替换算法不改变“必须放进来”的事实，只改变“把谁赶走”。

**② 精确 LRU 的代价**

LRU 需要知道 N 路中哪一路最久未用。两种实现：

- 每路一个 age counter：N × log₂N 位。8 路 = 8 × 3 = 24 位。
- 记录全部排列：log₂(N!) 位。8 路 = log₂(40320) ≈ 15.3 → 16 位；16 路 = log₂(16!) ≈ 44.25 → 45 位。

代价不只是位宽：**每次命中都要更新顺序/计数器**。在 L1 的 4 周期命中路径上，这是实打实的延迟和功耗。所以大相联度 cache 通常不用精确 LRU，而是用“足够好”的近似。

**③ 常见近似算法**

| 算法          | 每路状态          | 8 路总位宽  | 命中时     | 替换时          | 特点                   |
| ----------- | ------------- | ------- | ------- | ------------ | -------------------- |
| 精确 LRU      | age / 排列      | 16–24   | 更新全序    | 选最久未用        | 理论好，硬件贵              |
| 精确 MRU      | age / 排列      | 16–24   | 更新全序    | **选最新用**     | 抗循环扫描；随机访问灾难，状态同 LRU |
| FIFO        | 0（只维护插入顺序指针）  | 0       | 不动      | 逐出最早插入的      | 不看命中；循环扫描同样归零        |
| tree-PLRU   | 树指针           | N−1 = 7 | 翻转路径上的位 | 沿指针走到叶子      | 便宜，近似 LRU            |
| NRU / Clock | 1 位引用位        | 8       | 置 1     | 扫 0，扫过的清 0   | 更便宜，粒度粗              |
| Random      | 0             | 0       | 无       | 随机           | 极便宜，抗扫描              |
| SRRIP       | 2 位 RRPV      | 16      | 置 0     | 找 3，找不到全体加 1 | 抗扫描，预测再引用            |
| BIP         | 类似 LRU + 插入策略 | 同 LRU   | 同 LRU   | 同 LRU        | 抗 thrashing          |

**④ tree-PLRU 怎么工作**

把 N 路组织成完全二叉树，每个内部节点 1 位，指向“更久未用”的那一半。命中一路时，把从根到该叶路径上的位翻转（指向另一半）；替换时从根沿位走到叶，就是 victim。N 路只需 N−1 位，8 路 7 位。它只记录“哪半边更旧”，不记录完整顺序，所以是近似 LRU。

**⑤ LRU 的病理与 BIP**

考虑 4 路 cache，循环访问 5 个块 `A B C D E A B C D E …`：

- **LRU**：A B C D 填满；E 逐出 A；A 逐出 B；B 逐出 C……**每次访问都 miss**。原因是 LRU 总把“最久未用”的块逐出，而循环模式里“最久未用”恰好是“下一次就要用”的块。
- **FIFO / tree-PLRU**：同一序列同样接近归零——FIFO 的插入序等于访问序，tree-PLRU 仍按新近性排序。**扫描病理不是 LRU 独有，而是所有“偏爱新块”的策略共有。**
- **Random**：cache 里始终有 4/5 的块，下一次访问有相当概率命中，不会必然 0%。
- **LIP / BIP**：LIP 把新块插入到 **LRU 位置**（而不是 MRU），新块自己成为下一个 victim，不会把热块挤走；BIP 以大概率插 LRU、以小概率（如 1/32）插 MRU，兼顾工作集变化。这样能保住一个稳定的热集合，避免 LRU 的“必然逐出下一次要用的块”。

直觉：LRU 假设“最近用过的还会用”；循环扫描没有重用，这个假设失效。BIP 改的是**插入位置**，不是 victim 选择。

反方向的极端是 **MRU**（逐出最新鲜的块）：同一序列命中率 80%，但它对随机访问是灾难，只能作为思路来源，不能当通用策略。逐行推演、以及 GPU 上 evict-first / streaming hint 的落地，见下一条。

**⑥ RRIP 怎么工作**

RRIP 给每块 2 位 **RRPV**（re-reference prediction value，再引用预测值）：

- `0` = 很快会再被引用；`3` = 很久以后才会
- **命中**：RRPV ← 0
- **SRRIP 插入**：RRPV ← 2（“较远”，不是最远，避免一进来就被逐出）
- **替换**：找 RRPV=3 的块；没有就让所有块 RRPV+1（饱和），再找
- **DRRIP**：用 set dueling 在 SRRIP 和 BRRIP 之间动态选择

效果：新块不会一进来就被当成“最近使用”（避免扫描污染），也不会被立刻逐出（保留一定机会）。论文报告只需要 2 位/块；SRRIP/DRRIP 在 2MB LLC 上比 LRU 吞吐高约 4%/10%（单核）、7%/9%（4 核 8MB 共享 LLC）。

**⑦ 怎么选**

- 相联度低、延迟紧（L1）：tree-PLRU / NRU / Random
- 大 LLC、扫描/多核干扰明显：RRIP / BIP / DIP
- 面积几乎不能加：Random 是强 baseline
- 有学习/预测逻辑：Hawkeye 等，但面积和复杂度更高

> 来源：Jaleel et al., "High Performance Cache Replacement Using Re-Reference Interval Prediction (RRIP)", ISCA 2010；Qureshi et al., "Adaptive Insertion Policies for High Performance Caching", ISCA 2007。

### 自测
- [ ] 8 路精确 LRU 和 tree-PLRU 各需要多少状态位？
- [ ] 4 路 cache 循环访问 5 个块，LRU 为什么 0% 命中？BIP 改了什么？
- [ ] SRRIP 的 RRPV 在命中、插入、替换时怎么变？
- [ ] 替换算法能减少 compulsory miss 吗？为什么？

## CPU 侧的访存合并：MSHR merge 和 write combining 在合并什么？ #arch #memory-hierarchy #performance

> **速查**
> - **MSHR merge**：同一 cache line 的多个 outstanding miss 只发一次请求；第一个是 primary，后续同 line 是 secondary
> - **MSHR 上限决定 MLP**：entry 满 → 新 primary miss 无法登记，cache lockup/stall；slot 满 → secondary merge 失败
> - **write combining**：write-back cache 在 cache 内天然合并同一 line 的 store；WC buffer 合并 uncached/non-temporal 写
> - 部分写/跨 line 写会降低 WC 效率，甚至触发 read-modify-write
> - 两者都是**硬件能力**；软件只能靠访问模式、对齐、non-temporal store、减少 false sharing 让合并更容易发生

### 正文

**① MSHR：合并“同一个 line 的 miss”**

场景：4 个核/线程几乎同时 miss 同一 cache line。

- 第一个请求分配 MSHR entry，记为 **primary miss**，向下一级发一次请求。
- 后续同 line 的请求是 **secondary miss**，只挂到该 entry 的 slot，不再发请求。
- 数据返回后，一次唤醒所有等待者。

没有 MSHR 合并：4 次 DRAM 读；有合并：1 次。代价是 MSHR 资源有限：

- **entry 满** → 新的 primary miss 无法登记，cache 可能 lockup / stall
- **slot 满** → 同 line 的 secondary miss 无法合并（merge-full）
- 教学示例常见“4 entry × 4 slot”，但真实核的 MSHR 数量/组织差异很大，不能把这个数字当规格

MSHR 还决定 **MLP（memory-level parallelism，内存级并行）**：MSHR 越多，能同时 outstanding 的 miss 越多，越能掩盖延迟。不同 line 的 miss 不能合并；false sharing 会让多个核的 miss 落在同一 line 上，看似“合并”成功，实际浪费了 MSHR 和带宽。

**② write combining：合并“同一个 line 的 store”**

- **write-back cache**：store 先写 cache，置 dirty；同一 line 的多次写天然在 cache 里合并，替换时才写回一次。
- **write-through / uncached / non-temporal**：store 进入 write buffer 或 WC buffer，多个对同一 line 的写攒成一次下游写事务。
- 部分写（只写 line 的一部分）可能需要 read-modify-write；WC 效率取决于写是否覆盖完整 sector/line。
- 软件可以通过 non-temporal store（如 x86 的 `movnt*`、CUDA 的 streaming store）影响写路径，但**是否合并、合并多少由硬件决定**。

**③ 两种合并的对比**

| 机制 | 合并对象 | 粒度 | 谁做 | 失败后果 |
|---|---|---|---|---|
| MSHR merge | 同一 line 的 outstanding miss | cache line | cache controller | entry 满 → lockup/stall；slot 满 → merge-full |
| Write combining | 同一 line 的 store | cache line / WC buffer | store buffer / WC | buffer 满 → 冲刷；部分写 → RMW |
| Write-back cache | 同一 line 的多次写 | cache line | cache | 替换时才写回；dirty 位记录 |

**④ 什么时候失效**

- 不同 line 的 miss 不能合并；false sharing 会浪费 entry。
- 写分散在不同 line、部分写、需要顺序/持久化（fence/flush）时，WC 收益下降。
- WC buffer 深度有限，写压力大时会提前冲刷。
- 合并只减少**事务数/带宽**，不一定降低**延迟**。

**⑤ 软件 vs 硬件**

- MSHR 合并、write buffer 合并：**硬件能力**。
- 软件能做的是**让合并更容易发生**：连续地址、对齐、减少 false sharing、对确实不复用的数据用 non-temporal store。
- 反过来，软件无法“命令”硬件合并；它只能改变访问模式，让硬件的合并规则命中。

> 来源：Kroft, ISCA 1981 与 lockup-free cache 文献（MSHR primary/secondary miss）；Intel, *64 and IA-32 Architectures Optimization Reference Manual*（write combining / non-temporal store）。

### 自测
- [ ] MSHR 的 primary miss 和 secondary miss 分别是什么？entry 满和 slot 满有什么区别？
- [ ] write-back cache 的写合并和 write buffer 的写合并有什么不同？
- [ ] 为什么部分写可能需要 read-modify-write？
- [ ] false sharing 为什么浪费 MSHR？

## GPU 侧的访存合并：一个 warp 的访问怎么变成 32B sector？ #arch #memory-hierarchy #performance

> **速查**
> - GPU 的合并粒度是 **32B sector**，不是 128B line；一个 warp 的 32 个线程只有落在同一 sector 才合并
> - 32 threads × 4B 连续 = 128B = 4 sectors；每个线程间隔 128B → 32 sectors = 8× 取回字节
> - 合并逻辑在 **LSU / memory pipeline**（硬件）；软件改数据布局和线程映射（SoA、连续、对齐）让合并发生
> - shared memory bank conflict 是另一回事：同一 bank 不同地址被串行化
> - 合并减少事务数/带宽，不一定降低延迟

### 正文

**① warp、line、sector 的关系**

NVIDIA GPU 的 L1/L2 line 通常是 128B，分成 4 个 32B **sector**（可独立取的最小传输单位）。一个 warp 有 32 个线程；执行一条 global load/store 时，硬件看这 32 个地址的并集，按 32B 对齐的 sector 发事务。**sector 是合并的粒度，line 只是 cache 的组织单位。**

**② 两个算例（每个线程 4B）**

| 访问模式 | 地址范围 | sector 数 | 有效字节 / 取回字节 |
|---|---|---|---|
| 连续：`a[tid]` | 32 × 4B = 128B 连续 | 4 | 128 / 128 = 100% |
| 间隔：`a[tid * 32]` | 32 个地址相隔 128B | 32 | 128 / 1024 = 12.5% |

所以“合并”不是把 32 个线程变成一个事务，而是**把落在同一 32B sector 的线程合并**；sector 数才是带宽成本。线程访问 8B/16B 时，一个 warp 可请求更多字节，但仍然按 sector 拆分。

**③ 怎么让访问合并**

- **SoA 而不是 AoS**：同样访问 `x[i]`、`y[i]`、`z[i]`，结构体数组会把三个分量分散到不同 sector；拆成三个连续数组后每个 warp 的访问连续。
- **线程映射**：让相邻线程访问相邻元素；2D 数组的宽度和 block 宽度取 warp size 的倍数。
- **对齐**：`cudaMalloc` 至少 256B 对齐；结构体/数组起始地址对齐到 sector 边界。
- **向量化**：`float4` 等 16B 访问能减少指令数，但每个线程的 16B 仍要落在合适的 sector 里。
- **避免发散**：warp 内地址跨度越大，sector 越多。

**④ 写合并与部分 sector**

GPU 的 store 也按 sector 合并。只写一个 sector 的一部分时，可能需要 read-modify-write（取决于写策略和 ECC）；写满整个 sector/line 最划算。streaming/non-temporal store 可以绕过 cache 污染，但不会改变 sector 合并的基本规则。

**⑤ 和 shared memory bank conflict 的区别**

- **Global coalescing**：一个 warp 的 32 个线程访问 global memory，硬件按 32B sector 合并；跨 sector 越多，事务越多。
- **Shared memory bank conflict**：shared memory 按 32 个 bank、4B 宽度组织；同一 bank 的不同地址在同一周期被访问时串行化。它是 **bank 级冲突**，不是“合并”。

两者经常一起出现，但机制不同：前者是“能不能合并成一个 sector 事务”，后者是“能不能在一个周期内访问不同的 bank”。

**⑥ 什么时候失效**

- 地址发散、跨 sector、非对齐、每线程访问大小不同，都会增加 sector 数。
- 合并只减少**事务数/带宽**，不一定降低**延迟**；命中率、MLP、DRAM 行局部性仍然重要。
- 用 profiler 看 **sectors per request**（每请求 sector 数）比看“是否 coalesced”更直接。

> 来源：NVIDIA, *CUDA C++ Programming Guide*（global memory coalescing、32B sector）；NVIDIA Technical Blog, "Unlock GPU Performance: Global Memory Access in CUDA"。

### 自测
- [ ] 32 个线程各访问 4B：连续 128B 和间隔 128B，各产生多少 32B sector？
- [ ] sector 和 cache line 是什么关系？合并的粒度为什么是 sector 而不是 line？
- [ ] AoS 改成 SoA 为什么能改善合并？
- [ ] shared memory bank conflict 和 global memory coalescing 的区别是什么？

## MRU 是什么？evict-first / streaming hint 和它是什么关系？ #arch #memory-hierarchy #performance

> **速查**
> - **MRU = Most Recently Used**，最近被访问过的那一项，LRU 的对偶（中文常误写成“最近最久未使用”，那是 LRU）
> - 作为策略：**逐出最新鲜的块**。4 行 cache 循环访问 5 块时，LRU / FIFO / tree-PLRU 命中率 0%，MRU / Random / SRRIP 约 80%
> - MRU 的状态开销与精确 LRU **相同**（都要记完整顺序），随机访问时是灾难 → 只能当思路来源，工业界不用它
> - tree-PLRU 的硬保证是 `victim ≠ MRU`；4 路时还能排除第 2 新的块，但**可以**逐出第 3 新（第二旧）的块
> - GPU 的 evict-first（PTX `ld.global.cs`）思想同源、实现不同：**不是**跟踪顺序，而是给每条 line 一个 policy 位，替换时优先挑这些行

### 正文

**① 先分清两个词性**

- **名词**：“MRU 块”指最近刚被访问的那一个。“tree-PLRU 不淘汰 MRU”用的是名词。
- **策略**：“MRU 算法”指替换时**故意选** MRU 块做 victim，与 LRU 正好相反。

两者容易混。面试问“PLRU 和 LRU 差在哪”，考的是名词性质（保证不淘汰 MRU）；问“有没有比 LRU 更抗扫描的策略”，考的才是策略。

顺带把一个常见说法纠正掉：4 路 tree-PLRU **不是**“只保证不淘汰 MRU”。victim 一定落在 MRU 所在半区的**对侧**；若第 2 新的块与 MRU 同侧，victim 在对侧自然不是它；若第 2 新的块在对侧，则对侧那一位已经指向远离它的方向，victim 也不是它。所以 4 路时 victim 只在“第 3 新”和“最旧”之间选。相联度更高时这个保证会变弱。

**② 4 行 cache、循环访问 5 个块（可逐行验算）**

访问序列 `A B C D E` 无限循环，cache 4 行，**MRU 策略**：

| 访问 | 结果 | cache 内容 | 当前 MRU |
|---|---|---|---|
| A B C D | 4 次 miss，填满 | A B C D | D |
| **E** | miss，逐出 MRU=D | A B C **E** | E |
| A B C | 3 次命中 | A B C E | C |
| **D** | miss，逐出 MRU=C | A B **D** E | D |
| E A B | 3 次命中 | A B D E | B |
| **C** | miss，逐出 MRU=B | A **C** D E | C |
| D E A | 3 次命中 | A C D E | A |
| **B** | miss，逐出 MRU=A | **B** C D E | B |

稳态：**每 5 次访问 1 次 miss，命中率 80%**。缺失的那一位在 5 个块之间轮转，每轮只丢一个。

机制：MRU 保住的始终是“较早的那 4 个”，被逐出的永远是刚进来的那个——于是下一轮里唯一缺失的就是它，构成固定的 1 次义务 miss。

**③ 同一序列上各策略的命中率**

| 策略 | 命中率 | 机理 |
|---|---|---|
| LRU | 0% | 逐出的正好是下一轮第一个要用的 |
| FIFO | 0% | 插入序 = 访问序，行为与 LRU 相同 |
| tree-PLRU | 0%（稳态） | 仍是“偏爱新块”的策略，只是顺序用树近似 |
| MRU | 80% | 保留较早的 4 个，缺失位每轮只 miss 一次 |
| Random | ~80% | 缺失位在 5 个块中均匀轮转 |
| SRRIP | ~80% | 新插入 RRPV=2，扫描块不会霸占“最该保留”的位置 |

**关键结论**：扫描病理不是 LRU 独有的，而是**所有偏爱新块的策略共有**——FIFO、tree-PLRU 同样归零。反过来，MRU 和 Random 之所以不归零，恰恰因为**它们不保留刚用过的那批**。

**④ 那为什么工业界不用 MRU**

1. **状态开销与 LRU 相同**：要精确知道“哪个最新”，就得记完整顺序（8 路 16–24 位），一位都省不下来。
2. **随机访问时是灾难**：时间局部性说“刚用过的马上还会用”，MRU 专门逐出它，等于主动放弃局部性。
3. **Random 拿到同样的抗扫描效果，代价是 0 位、0 延迟**。既然 Random 已经不归零，MRU 没有存在理由。
4. **MRU 不是 stack algorithm**：不满足包含性质（大 cache 的内容不一定是小 cache 的超集），无法用“小 cache 反推大 cache”做分析。

所以工业界的解法是改**插入位置**（BIP / LIP 把新块插到 LRU 位置）或改**预测**（RRIP），而不是把 victim 选择反过来。

**⑤ evict-first / streaming hint：同源思想，不同实现**

GPU 里确实有“不保留刚用过的”机制，但它不是 MRU 算法。以 PTX 的 cache operator 为例：

| operator | 语义 | 典型用途 |
|---|---|---|
| `ld.global.ca` | 各级都缓存（默认） | 普通读 |
| `ld.global.cg` | 只缓存到 L2，**绕过 L1** | 确定不复用、不想污染 L1 |
| `ld.global.cs` | **evict-first**：在 L1/L2 标成“下一个被逐出” | 流式、只读一次的数据 |
| `ld.global.lu` | last use：读后在 local 窗口内失效 | local 空间的最后一次读 |

三类机制的程度差别（都是“不保留”，但代价不同）：

| 机制 | 流式块进 L1？ | L1 热块受影响？ | 流式块自身能命中？ |
|---|---|---|---|
| 默认 `.ca` | 进，按 LRU 竞争 | 会被逐个挤出 | 能 |
| **evict-first** `.cs` | 进，但带“优先逐出”标记 | 不受影响 | 可能（短时间内） |
| **bypass** `.cg` | 不进 | 完全不受影响 | 不能 |

**软硬件分工**（这条必须讲清）：

| 环节 | 谁做 |
|---|---|
| 判断“这段数据只读一次” | **软件**：程序员 / 编译器分析 / profiling |
| 选择 `cs`、`cg`、`prfm pldl1strm`、NTA 等 hint | **软件**：编译器、运行时、驱动 |
| 把 hint 记录成 line 上的 policy 位 | **硬件**：L1 控制器在 fill 时打标 |
| 替换时优先从 policy=evict-first 的行里挑 victim | **硬件**：替换逻辑 |
| 兜底（policy 位用尽、多路都是 evict-first） | **硬件**：退回普通替换策略 |

**⑥ 一个能算的例子：3 个热块 + 流式数据**

L1 4 行，热块 H1 H2 H3 反复访问，同时不断来只读一次的流式块 S1、S2……

- **无 hint**：S1 填进来成为最新，被逐出的是最老的 H 块；S2 再挤掉一个 H。热块被逐个清空，热访问几乎全 miss（具体比例取决于流式与热访问的交错比）。
- **evict-first**：S 进 L1 时打标；S2 到来时，victim 优先从 evict-first 里挑 → 挤掉 S1，H1–H3 全部留下。
- **bypass（`.cg`）**：S 不进 L1，热块完全不受影响；代价是同一个 S 被第二次读时也没有 L1 命中（本来就预期不复用，可接受）。

**⑦ 接回纹理与 L1**

纹理侧的对应现象：大 mip level 或单次采样的 streaming texture，每个压缩块只解一次，属于典型 `.cs` 语义。可观察的信号是——**L1 命中率随 mip level 增大而骤降，且 miss 分布与 texture descriptor 的 sampling pattern 强相关**。正确动作顺序：

1. 给 streaming 请求单独打标 + evict-first（或走 bypass 通路）；
2. 在 benchmark 里显式加入“工作集略大于 L1”的 streaming pattern，否则线上测不出来；
3. 最后才考虑加大 L1——最贵，而且不解决污染问题。

> 这正是“benchmark 体系”该覆盖的一类：只跑随机和顺序两种模式，看不到这个坑。

### 自测
- [ ] 4 行 cache 循环访问 5 个块：LRU、FIFO、tree-PLRU、MRU、Random 各是多少命中率？为什么 PLRU 也是 0%？
- [ ] MRU 和 evict-first 的本质区别是什么？为什么后者能落地、前者不能？
- [ ] `ld.global.cs` 这类 hint，哪些环节是软件决定、哪些是硬件实现？
- [ ] benchmark 里 L1 命中率随 mip level 增大骤降，你先怀疑什么、按什么顺序排查？
