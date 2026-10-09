---
tags: [dma, cpu]
updated: 2026-10-10
---

# x86 上与设备交互的原语

> 范围：内存类型（MTRR/PAT）→ 写指令（`movnt*` / `movdir64b` / `enqcmd`）→ 缓存行写回与屏障 → 队列/门铃协议 → 在 DMA 里的定位。
> 每条末尾标【软件】【硬件】【软硬协同】，并写明各自负责什么。

## Write-combining memory（WC） #dma #cpu

> **速查**
> - WC 是**内存类型**之一；写先进 **write-combining buffer**，凑满一条 cache line（64B）后以**整行突发**写到内存/设备
> - **只对写做合并**，读不合并
> - **为什么弱序**：WC buffer **不参与 cache 一致性协议**，硬件不跟踪其他 agent，buffer 里的写还没"全局可见"→ 可重排，需要顺序处必须显式 fence
> - **为什么读慢**：WC 区域的读**不进 cache**，每次都是到内存/设备的事务，没有缓存命中可摊薄；若读地址落在当前 buffer 内的 pending 写上，还得**先刷 buffer** 才能读到正确值
> - **buffer 何时刷出**：填满 64B 行 / 执行序列化指令（`SFENCE`、`MFENCE`、locked 指令、`CPUID`、`IRET`）/ 访问不同内存类型 / 对该区域的读 / I/O 指令 / 中断异常任务切换。**硬件也可随时主动刷出——buffer 驻留不被保证**
> - 定位：WC 面向**写多读少**（典型是 framebuffer）
> - 归属：【软硬协同】硬件提供 MTRR/PAT 表、写合并缓冲电路；软件写 MTRR、写 PAT 与 PTE 属性位、用 fence 维护顺序

### 正文

**两层的类型控制**：

| 机制 | 粒度 | 在哪 | 谁设 |
|---|---|---|---|
| **MTRR**（Memory Type Range Registers） | **物理地址区间**（粗） | 一组 MSR，条数有限 | 固件/BIOS 或早期内核 |
| **PAT**（Page Attribute Table） | **每个 4KB 页**（细） | `IA32_PAT` MSR（0x277），8 个 entry | OS |

**页表里怎么选 PAT entry**：PTE 的 bit 3（PWT）+ bit 4（PCD）+ bit 7（PAT），共 3 bit 索引 8 个 entry。大页（PDE/PDPTE）时 PAT 位对应 bit 12。

**有效类型的优先级**：`UC > WC > WT > WP > WB`。两个要点：
- MTRR 声明 UC 时，PAT **无法**把它放宽成 WB/WC
- MTRR 为 WB 时，PAT **可以**把该页设成 WC

（完整组合表以 Intel SDM Vol.3 内存类型章节为准。）

**MTRR 相关寄存器**：`IA32_MTRRCAP`(0xFE)、`IA32_MTRR_DEF_TYPE`(0x2FF)、可变区间 `IA32_MTRR_PHYSBASEi`(0x200+2i) / `PHYSMASKi`(0x201+2i)、固定区间 `IA32_MTRR_FIX*`(0x250–0x26F)。

### 自测
- [ ] WC 为什么是弱序的？硬件为什么不便宜地强制强序？
- [ ] WC 区域为什么读很慢？
- [ ] write-combining buffer 会在哪些时机被刷出？为什么它的驻留不被保证？
- [ ] MTRR 和 PAT 各自控制什么粒度？冲突时谁赢？

## Non-temporal / streaming stores #dma #cpu

> **速查**
> - 指令：`MOVNTI`(m32/m64)、`MOVNTQ`(m64)、`MOVNTDQ`(m128)、`MOVNTPS`/`MOVNTPD`、`VMOVNTDQ`(m256)、AVX-512 下可到 m512
> - 对应的"流式 load"是 `MOVNTDQA` —— ⚠️ **它的非临时读语义只在目标内存类型为 WC 时才被保证**
> - 区别：NT store 是**对单条指令**施加 WC 语义；WC **内存类型**是对**一个地址区间**施加。两者落到同一套 write-combining buffer 硬件
> - 即使目标是 WB 内存，NT store 也按 WC 协议走：**进 WC buffer 直写内存，不分配 cache line，不触发 RFO**（read-for-ownership，写前先把行读进来的那次读）
> - **什么时候赢**：大块流式写且写完短期不再读；省掉 RFO 读、不污染 cache；顺序写满整 64B 行
> - **什么时候输**：数据很快被读（放弃缓存后每次回内存）；写不连续/不满行（拼不出整行，每条变 partial 事务）；弱序带来额外 fence 开销；小规模拷贝或 cache 空闲时未必比普通 store 好
> - 归属：【软硬协同】硬件提供绕过 cache、不分配行的写路径；软件选发哪条指令、何时加 fence

### 正文

`SFENCE` 是保证 NT store 对后续读/其他 agent 可见的必要条件。

### 自测
- [ ] NT store 和 WC 内存类型的关系是什么？
- [ ] RFO 是什么？NT store 为什么省掉它？
- [ ] 什么情况下 NT store 会比普通 store 更慢？

## MOVDIR64B #dma #cpu

> **速查**
> - Intel 的 "direct store" 指令。形式 `66 0F 38 F8 /r`，汇编 `MOVDIR64B r16/r32/r64, m512`
> - **目的地址由通用寄存器给出**（因此可为 MMIO）；**目的必须 64B 对齐**，不对齐触发 `#GP(0)`；**源 m512 无对齐限制**
> - 解决什么问题：普通 64B 写由多条 store 组成，对设备 MMIO 表现为**多次 posted 写，设备可能读到撕裂的描述符**；对 UC MMIO 的逐次 store 还会串行化
> - 它保证 **64 字节 write-completion 原子性**——数据以单个不可拆分的 64B 写事务到达目的（对 MMIO 即一条 non-torn posted write）
> - ⚠️ **源 64B 读不保证原子**（实现可用多次 load）；只有写保证原子
> - **忽略目的内存类型**（即使 UC），自身按弱序处理；需顺序时显式 `SFENCE`/`MFENCE`
> - **与 ENQCMD 的差异**：MOVDIR64B **不填 PASID**、不回传设备状态（posted，不等设备响应）
> - 可用性检测：CPUID leaf 07H，ECX bit 28
> - 典型用途：向设备投递 64B 描述符/门铃（DSA 的 dedicated WQ）；iomem 大块拷贝；MKTME/TDX 内存清零
> - 归属：【软硬协同】硬件提供 direct-store 引擎与 64B 原子写保证；软件选指令、给 64B 对齐的目的地址

### 自测
- [ ] MOVDIR64B 保证了什么原子性？**没有**保证什么？
- [ ] 它对目的地址有什么对齐要求？源呢？
- [ ] 和 ENQCMD 的关键差异是什么？

## ENQCMD / ENQCMDS #dma #cpu #iommu

> **速查**
> - 向**共享工作队列（SWQ，Shared Work Queue）**的 **portal**（64B MMIO 区域）提交 64B 描述符的指令。**non-posted**，等待设备响应
> - **状态反馈在 `EFLAGS.ZF`**：0 = 描述符被接受（**不等于已执行**），1 = 重试（队列满 / 未使能 / portal 非法）
> - **PASID** = 20 bit 的**进程地址空间 ID**，标识设备该用哪套进程页表做翻译；**SVA**（Shared Virtual Addressing）让设备直接用进程虚拟地址，PASID 把设备侧翻译绑定到具体进程
> - **ENQCMD**：任意特权级；把当前线程的 PASID 从 `IA32_PASID` MSR（**0xD93**，bit31 = Valid，bit19:0 = PASID）拷进描述符**首 4 字节**
> - **ENQCMDS**：仅 CPL0（内核）；由软件显式初始化首 4 字节
> - **为什么需要**：用户态不能直接写设备门铃（权限/安全，且门铃不携带进程身份）；传统路径是"写描述符到内存 → 再敲门铃"两步，且描述符还要设备 DMA fetch
> - **一条指令合并了三件事**：提交 + 通知 + 携带进程身份。设备立即看到描述符并用 PASID 翻译，**无需 per-submission syscall，无需描述符环的 DMA fetch**
> - ⚠️ **顺序约束**：每逻辑处理器同时**最多一个** enqueue store pending；enqueue store 之间不可互相超越；对不同地址的普通/WC store、`CLFLUSHOPT`、`CLWB` **不**保序 → 需要 fence
> - 归属：【软硬协同】硬件提供 SWQ portal、PASID 翻译、64B 原子 non-posted enqueue；软件选指令、设 `IA32_PASID`、重试逻辑、VMX 下 PASID 虚拟化配置

### 正文

**批量**：DSA 的 **Batch descriptor** 含"描述符数组地址 + 计数"，一次 enqueue 提交 N 条（batch 内逐条有 Fence 位控制顺序）。

> ⚠️ **不确定项**：DSA 描述符**首 4 字节之外**的具体位域（opcode、flags、completion 字段偏移）未逐位列出，以 Intel DSA spec（341204）与 ISEE（319433）为准。本文只确证 PASID[19:0]、PRIV(bit 31) 两个位。

### 自测
- [ ] ENQCMD 相比"写描述符 + 敲门铃"两步传统路径，省掉了什么？
- [ ] `EFLAGS.ZF = 0` 表示什么？它不等于什么？
- [ ] PASID 是干什么的？它和 SVA 的关系是什么？

## 缓存行写回与屏障 #dma #cpu

> **速查**
> - **`CLFLUSH`**：把含操作数的 cache line 写回内存**并失效**（行被移除）。⚠️ 历史上**串行化**，多条顺序执行，慢
> - **`CLFLUSHOPT`**：同样写回 + 失效，但**去掉串行化**，多条可并发/流水。⚠️ **仍需其后 `SFENCE`** 才保证完成
> - **`CLWB`**：写回（若脏）但**保留**该行在 cache 中（变 clean）。再次访问易命中，**优于前两者**；仍需 `SFENCE`
> - **`SFENCE`**：只对 **store** 排序——其前所有 store 全局可见后，其后 store 才可见。**不对 load 排序**
> - **`MFENCE`**：全屏障，之前 load+store 全部完成后之后才开。比 SFENCE 强，更贵
> - 操作数是 m8（任意指向该行的地址）。刷一段要每 64B 一条 + 末尾一条 `SFENCE`
> - ⚠️ **重要边界**：x86 上普通**一致性 DMA 由硬件 snoop CPU cache**，设备读主机内存前**通常不需要**手动 `CLFLUSH`。`CLFLUSH`/`CLWB` 主要面向**持久内存**与**非一致性访问者**
> - 归属：【软硬协同】硬件实现 cache 写回与顺序模型；软件决定刷哪条行、在哪里 fence

### 正文

- `CLFLUSH`/`CLFLUSHOPT` 不需要前置 barrier；`CLWB` 对同地址的旧 store 隐式有序
- **后置 `SFENCE`** 用来保证写回在后续动作前完成

**用途**：让 CPU 写的行到达内存（持久内存的持久化；对非一致性访问者可见）；与 WC/NT store 配合（fence 刷 WC buffer）。

### 自测
- [ ] `CLFLUSH` / `CLFLUSHOPT` / `CLWB` 三者的区别是什么？为什么 CLWB 通常最好？
- [ ] `SFENCE` 和 `MFENCE` 的区别是什么？
- [ ] 在一台一致性 DMA 的 x86 机器上，DMA 前需要手动 CLFLUSH 吗？为什么？

## Command queue / work queue 模式 #dma #cpu

> **速查**
> - **NVMe 的 SQ/CQ**：队列位于**主机内存**。SQ entry = **64B**，CQ entry = **16B**
> - 提交流程：主机写 SQ entry → 写 **SQ Tail Doorbell**（MMIO）→ 控制器据此 **DMA fetch** 命令 → 执行 → DMA 写 CQ entry → （可选）MSI-X 中断 → 主机消费后写 **CQ Head Doorbell** 归还条目
> - **门铃 = 一次 MMIO 写**（uncached、有序代价、一次总线事务），**远贵于内存 store**
> - **Intel DSA 的 SWQ**：多进程共享；**portal 是 64B MMIO**；提交用 `ENQCMD`（用户）/ `ENQCMDS`（内核）/ `MOVDIR64B`（dedicated WQ）
> - **关键差异**：DSA 门外**没有内存描述符环**——指令直接向 portal 投递，**提交动作本身即时被设备看到**
> - **门铃批处理为什么必要**：每敲一次门铃 = 一次 MMIO。队列语义（tail 指针表达"新增 N 条"）允许写完 N 条命令后**只敲一次** → 通知成本摊销为 1/N。代价是**批内首条命令延迟上升**
> - 更强形态：`ENQCMD`/`MOVDIR64B` 把"提交"本身变成通知，**取消了分离门铃**
> - Linux 侧实现：`iosubmit_cmds512()`（基于 `movdir64b`）和 `enqcmds()`
> - 归属：【软硬协同】硬件提供门铃寄存器 / portal、设备侧队列消费；软件维护队列环结构、tail/head、批处理策略

### 正文

**NVMe 门铃寄存器**：起始偏移 **0x1000**；`SQyTDBL` / `CQyHDBL`；步长 = `4 << CAP.DSTRD`（CAP 在偏移 0x00）。

### 自测
- [ ] NVMe 的一次提交，从主机写 SQ 到控制器拿到命令，中间经过哪些步骤？
- [ ] 门铃为什么贵？批处理怎么摊销它？代价是什么？
- [ ] DSA 的提交路径和 NVMe 的关键差异是什么？

## 这些原语在 DMA 路径里的位置 #dma #cpu

> **速查**
> - 把原语按**作用在哪条路径**分三类，这是理解它们分工的关键
> - **① 补充 DMA（不改数据面）**：WC / NT store（优化 CPU→内存 的写路径）、`CLFLUSH`/`CLWB`/`SFENCE`（建立 CPU 写对设备/持久介质的**可见点与顺序**）、memory-based SQ/CQ ring（描述符结构本身）
> - **② 部分替代传统模型（命令提交面）**：`MOVDIR64B` / `ENQCMD(S)` 把"写描述符到内存 → 再敲 MMIO 门铃"两步**合并成一条指令**，设备**无需 DMA fetch 描述符**
> - **③ 仍在设备数据路径（未被上述原语触及）**：bulk 数据搬运本身
> - **边界很重要**：`ENQCMD`/`MOVDIR64B` 替代的是**提交路径**，**没有**替代数据搬运——DSA 的 memory-move、NVMe 的数据传输，bulk 数据仍由设备 DMA 引擎搬
> - 一句话定位：WC/NT/flush/fence 在 **CPU 写路径**；SQ/CQ+门铃 与 ENQCMD/MOVDIR64B 在 **命令提交路径**；bulk 搬运在 **设备 DMA 数据路径**
> - 对「统一引擎 vs 两条路径」的参考价值：**ENQCMD/MOVDIR64B 是"提交路径可被指令合并"的论据**；而 x86 DMA 的 cache-coherent 特性意味着**片内场景与跨卡场景的可见性维护需求不同**

### 正文

对主人 DMA 预研（片内 C2C/D2D + 跨卡）的直接含义：

- **提交路径**可以用一条指令合并（ENQCMD 模式），不需要内存描述符环
- **可见性维护**：x86 一致性 DMA 靠硬件 snoop，片内场景可能完全不需要显式 flush；跨卡场景则需要——**这是两域又一处结构性差异**

### 自测
- [ ] 三类原语各自作用在哪条路径上？
- [ ] ENQCMD 替代了什么？**没有**替代什么？
- [ ] 为什么说 x86 的 cache-coherent 特性让片内和跨卡的可见性需求不同？

---

## 参考

- Intel SDM Vol.3（内存类型与顺序、PAT/MTRR）
- Intel ISEE 319433-038（ENQCMD/ENQCMDS、PASID 翻译）
- Intel DSA 规范 341204（SWQ、portal、Batch descriptor）
- Linux `iosubmit_cmds512()` / `enqcmds()`：https://git.kernel.dk/cgit/linux/commit/?id=232bb01bb8ad2f88aefbd88b0d3fe3f9a253502b
- Linux `IA32_PASID` MSR 定义（0xD93、bit31 Valid）：https://lkml.org/lkml/2020/3/30/913
- MOVDIR64B 指令形式与对齐/原子性：https://github.com/HJLebbink/asm-dude/wiki/MOVDIR64B
- CLFLUSH / CLFLUSHOPT / CLWB / SFENCE 差异：https://escholarship.org/content/qt4983x8dc/qt4983x8dc.pdf
