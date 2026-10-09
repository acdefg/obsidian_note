---
tags: [dma, pcie]
updated: 2026-10-10
---

# PCIe 分层与 IOMMU 规格

> **来源说明（重要）**：PCI-SIG 的 Base Spec / ATS / VT-d spec 正文是**会员制付费文档**。本笔记中的位域细节多来自厂商公开数据手册、Linux 内核源码与驱动注释、Arm 公开文档与第三方资料。
> **凡未直接核对 PCI-SIG 原文处，全部集中在文末「存疑清单」一节标出——不编造规格细节。**

## PCIe 三层协议栈 #pcie #dma

> **速查**
> - **Transaction Layer**：产生/解析 **TLP**（Transaction Layer Packet）；拆 Posted / Non-Posted；管流控信用与排序。【软硬协同】
> - **Data Link Layer**：用 **LCRC**（Link CRC）检错 + **序列号/Ack/Nak/重传**修复；控制信息走 **DLLP**（Data Link Layer Packet）。【硬件】
> - **Physical Layer**：编码解码、字节条带化到各 lane、加扰、SERDES；链路训练 **LTSSM**；有序集 TS1/TS2/SKP 等。【硬件】
> - **编码演进**：Gen1/2 用 8b/10b → Gen3/4/5 用 128b/130b → **Gen6 起用 PAM4 + FEC**
> - **Flit 模式**（PCIe 6.0 引入）：Gen6 用 PAM4 导致原始误码率从 ~1e-12 升到 **~1e-6**，需要 FEC；而 FEC 只对定长包有效 → 引入定长 **Flit**（Flow Control Unit）

### 正文

| 层 | 单元 | 作用 | 谁做 |
|---|---|---|---|
| Transaction | TLP | 事务语义（读/写/完成/消息） | 软硬协同 |
| Data Link | DLLP | 可靠传输（Ack/Nak/流控/PM）+ LCRC | 硬件 |
| Physical | Flit（6.0+）/ 有序集 | 编解码、训练、错误纠正 | 硬件 |

**Flit 固定 256 字节**，常见分解（⚠️ 见存疑清单）：

```
236 字节 TLP + 6 字节 DLP + 8 字节 CRC + 6 字节 FEC = 256
```

- DLP 前 2 字节承载 Flit 级序列号 / Ack/Nak / 重传信息
- FEC 的 6 字节分 3 个交织组（每组 2 字节），抑制短突发错误
- 进入 Flit 模式后，链路只要不 down 就一直保持 Flit 模式（即使降速）

### 自测
- [ ] PCIe 三层各管什么？哪些是纯硬件、哪些是软硬协同？
- [ ] PCIe 6.0 为什么要引入 Flit？

## TLP 格式要点 #pcie #dma

> **速查**
> - **第一个 DW（字节 0–3）是公共字段**：Fmt(3) + Type(5) + TC(3) + TD(1) + EP(1) + Attr(2) + AT(2) + Length(10)
> - **Attr[0] = Relaxed Ordering（RO）**，**Attr[1] = No Snoop（NS）**；PCIe 3.0 起扩展 Attr[2] = IDO
> - **AT[1:0]**（Address Type，配合 ATS）：`00` = 未翻译，`01` = Translation Request，`10` = 已翻译
> - **Length 是 10 位，以 DW 为单位**，`0` 编码 **1024 DW = 4096 字节**——所以**单个 TLP 载荷上限 4 KB**
> - **三类事务**：Posted（发出即完成，不需完成包）/ Non-Posted（必须返回 Completion）/ Completion
> - **Transaction ID = Requester ID + Tag**，用于匹配 NP 事务与其完成
> - ⚠️ **TLP 头内位域的确切位号，不同公开资料口径不一**（见存疑清单）

### 正文

**DW0 字段（按传输字节序，byte0 先传）**：

| 位置 | 字段 | 位宽 | 含义 |
|---|---|---|---|
| byte0 [7:5] | Fmt | 3 | 头长度（3DW/4DW）与是否带 data payload；Fmt[2]=1 表示带 TLP Prefix |
| byte0 [4:0] | Type | 5 | Mem / IO / Cfg / Msg / Cpl / AtomicOp |
| byte1 [7:5] | TC | 3 | Traffic Class，调度优先级 |
| byte1 [3] | TD | 1 | TLP Digest 存在位（尾部有 1 DW 的 ECRC） |
| byte1 [2] | EP | 1 | Poisoned，1 = 该 TLP 数据被标记为坏 |
| byte1 [1:0] | Attr[1:0] | 2 | **Attr[0]=RO，Attr[1]=NS** |
| byte2 [7:6] | AT[1:0] | 2 | Address Type（配合 ATS） |
| byte2 [5:4] + byte3 [7:0] | Length[9:0] | 10 | 数据长度（DW 计） |

**Fmt 取值**：`000`=3DW 无数据，`001`=4DW 无数据，`010`=3DW 带数据，`011`=4DW 带数据，`100`=TLP Prefix 第一 DW。

**三类事务对照**：

| 类别 | 含义 | 典型 TLP |
|---|---|---|
| Posted（P） | 发出即认为完成，**不需要**完成包 | Memory Write、Message |
| Non-Posted（NP） | 必须由目标返回 **Completion** | Memory Read、I/O R/W、Config R/W、AtomicOp |
| Completion（Cpl） | 对 NP 的应答 | Cpl（无数据）、CplD（带数据） |

**Completion status（4 位）**：`000b`=SC（成功）、`001b`=UR（Unsupported Request）、`010b`=CRS（仅配置请求）、`100b`=CA（Completer Abort）。

**Transaction ID 的构成**：

- **Requester ID（16 位）** = `Bus[7:0] : Device[4:0] : Function[2:0]`。启用 **ARI**（Alternative Routing-ID Interpretation）时为 `Bus[7:0] : Function[7:0]`
- **Tag（8 位）**：默认只有 **Tag[4:0]** 有效（**5 位**，每 Function 每 TC 最多 **32** 个未完成 NP）；**Extended Tag**（Device Control bit 8）后 8 位全用（最多 **256**）；**10-Bit Tag** ECN 再扩展到 10 位

**Split transaction**：NP 请求发出后链路可继续处理其它事务，目标稍后单独回完成包——避免像老 PCI 那样阻塞总线。

### 自测
- [ ] 单个 TLP 的载荷上限是多少？为什么？
- [ ] Posted 和 Non-Posted 的本质区别是什么？
- [ ] Transaction ID 由哪两部分组成？Tag 默认是几位？

## PCIe Ordering 模型 #pcie #dma

> **速查**
> - 总原则：保持 **Producer/Consumer（生产者-消费者）** 语义——写进去的数据必须先于"写已完成"的标志被观察到
> - **默认是"强有序"**，通过 **RO / IDO** 位放松（性能优化）
> - **三条核心规则**：① Posted **不能**超越 Posted（写写保序）② NP 请求**不能**超越 Posted ③ Completion **不能**超越 Posted
> - ⚠️ **RO 只放松"写超越写"和"完成超越写"，不放松读请求**；NP 超越 Posted 那一格**只有 IDO 能放松**，RO 不适用
> - **同一 Transaction ID 的多个 Completion 不得互相超越**（保证一次读的多个分片按地址顺序返回）；**不同事务的 Completion 之间可乱序**
> - **RO 位必须原样转发**

### 正文

| passer ↓ / passed → | Posted Req | Non-Posted Req | Completion |
|---|---|---|---|
| Posted Req | **不超越**（除非 RO/IDO） | 可超越 | 可超越 |
| Non-Posted Req | **不超越**（除非 IDO） | 可超越 | 可超越 |
| Completion | **不超越**（除非 RO/IDO） | 可超越 | 可超越* |

**Relaxed Ordering（RO）**：位置 TLP 头 `Attr[0]`；软件侧使能位在 **Device Control 寄存器 bit 4**。只有"被超越的写"和"超越者"访问**不同地址/无依赖**时才语义安全——**是否设 RO 由软件策略决定**（所以是软硬协同）。

**No Snoop（NS）**：位置 TLP 头 `Attr[1]`；软件侧对应 **Device Control bit 11**。含义：设备声明该内存访问**不需要被一致性窥探**，平台可跳过对 CPU cache 的一致性检查。⚠️ **排序规则与事务是否一致（coherent / No Snoop）无关**，均适用同一张表。

### 自测
- [ ] 默认排序的三条核心规则是什么？
- [ ] RO 位放松了哪些格？没有放松哪些？
- [ ] 为什么同一 Transaction ID 的多个 Completion 不能互相超越？

## MPS / MRRS #pcie #dma

> **速查**
> - **MPS**（Max Payload Size）：一个 Function 允许收发的、**带数据 TLP** 的最大载荷。⚠️ **只限制带数据的 TLP**；Memory Read 的长度由 Length 字段控制，**不受 MPS 限制**
> - **MRRS**（Max Read Request Size）：作为 Requester 时，**单次 Read Request** 的最大长度
> - 两者都是 **3 位编码**，值 = `128 << encoding`：128 / 256 / 512 / 1024 / 2048 / 4096 字节（`110b`/`111b` 保留）
> - 寄存器：**MPSS**（Device Capabilities bit[2:0]，只读，硬件能力上限）、**MPS**（Device Control **bit[7:5]**，软件可写，必须 ≤ MPSS）、**MRRS**（Device Control **bit[14:12]**，默认 **512 B**）
> - ⚠️ **默认 MPS = 128 字节**；若软件写成大于 DCAP 支持的值，硬件**回绕为 128 B**
> - ⚠️ **系统内 MPS 必须全局一致**——所有 Endpoint、Switch 端口、Root Port 都要支持该值；由软件在**链路静默**（无 TLP 在途）时统一协商、自顶向下编程（Linux `pci_configure_mps()`）
> - ⚠️ 若 **MRRS > MPS**，响应端必须把读应答**切成多个 MPS 大小的 TLP**

### 正文

**性能影响**：

| | 调大 | 代价 |
|---|---|---|
| **MPS** | 同样数据量用的 TLP 越少 → 头开销占比越低 → 有效带宽越高 | 收发缓冲更大、延迟略增 |
| **MRRS** | 单次 Memory Read 抓取更多数据 → 更好利用带宽、减少读请求往返 | 占用更多 outstanding 缓冲，增加延迟抖动 |

**调优惯例**：MPS 取路径所支持的最大值；MRRS 取设备支持的最大值且不超过 MPS。

相关：[[engine#分割、合并与顺序管理]]

### 自测
- [ ] MPS 和 MRRS 各限制什么？为什么说 MPS 只限制"带数据 TLP"？
- [ ] 为什么系统内 MPS 必须全局一致？协商时机有什么约束？
- [ ] MRRS > MPS 时会发生什么？

## ATS（Address Translation Services） #pcie #iommu #dma

> **速查**
> - **四个关键术语**：**TA**（Translation Agent，翻译代理，通常就是 IOMMU）、**ATC**（Address Translation Cache，**设备侧**的翻译缓存）、**ATPT**（翻译与保护表，即页表）、**STU**（Smallest Translation Unit，最小翻译粒度）
> - ⚠️ **ATC 在设备侧，不是 IOMMU 侧的 IOTLB**——这是两个独立缓存
> - **翻译请求**形式上**像一次 Memory Read**，靠 TLP 头 **AT = 01b** 标识。地址字段是**未翻译地址**
> - **翻译完成**形式上**像 CplD**，数据里含：翻译后地址 + 大小（以 STU 为单位）+ 内存保护属性（Global / Priv / Exe / U / W / R）。**一次 Completion 可返回多个已翻译地址**
> - 设备拿到结果后缓存进 ATC，之后对该地址发 **AT = 10b（已翻译）** 的 TLP，IOMMU 可直接透传
> - ⚠️ **若地址未从 ATS 获得，不得擅自置 AT 位**
> - **失效**：TA 向设备发 **Invalidate Request**（**MsgD**，Posted，载荷 64 位 = 要失效的未翻译地址范围，带 **ITag**）
> - 设备回 **Invalidate Completion**（消息），含对应 Request 的 Requester ID
> - **Invalidation Completion Combining**：32 位的 **ITag Vector**（bit0↔ITag 0 … bit31↔ITag 31）可把多个失效完成合并成一条消息

### 正文

**失效失败的状态**：返回无数据的 Cpl（完整状态 → UR，TA 内部错误 → CA）。**CRS 不允许**出现在对 Function 发起的请求的 Completion 中。

**设备侧约束**：收到 Invalidate 后**不要求立即清空在途请求**——可以先把受影响地址的在途事务收尾再应答。

⚠️ **无论 BME（Bus Master Enable）位是 Set 还是 Clear，支持 ATS 的 Function 都必须回 Invalidate Completion。**

**ATS 寄存器字段**：

| 寄存器 | 字段 | 位 |
|---|---|---|
| ATS Capability | Invalidate Queue Depth | bit[4:0]（0 表示 32） |
| | Page Aligned Request | bit5 |
| | Global Invalidate Supported | bit6 |
| ATS Control | STU | bit[4:0]（4096 字节的 2 的幂倍） |
| | Enable | bit15 |

### 自测
- [ ] ATC 和 IOTLB 分别在谁身上？为什么必须分别失效？
- [ ] 翻译请求和翻译完成分别"伪装"成什么 TLP？
- [ ] 设备拿到翻译结果后，后续 TLP 的 AT 字段应该是什么？

## PRI（Page Request Interface） #pcie #iommu #dma

> **速查**
> - **定位：PRI 是 ATS 的补充**——当设备需要某地址但 ATS 无法给出翻译（页不在内存），用 PRI 请主机把页调进来，随后再用 ATS 取翻译
> - ⚠️ **PRI 建立在 ATS 之上**（先有 ATS 才能用 PRI）
> - **三类消息**：Page Request Message（设备→RC/IOMMU）、PRG Response Message（RC/IOMMU→设备）、Stop Marker Message（设备声明某 PASID 停用）
> - **PRG**（Page Request Group，页请求组）：**PRGI 是 9 位**，由 Function 分配；**响应按组返回（每组一个响应，不是每页一个）**；组内**最后一个请求置 L 位（Last）**
> - ⚠️ **信用按"页请求"计，不是按组计**——Function 有静态页请求额度，不得超订
> - ⚠️ **同一 PRG 内若有页带 PASID，则全组必须 PASID 相同**，否则行为未定义
> - **响应码**：Success / Invalid Request / **Response Failure**（致命错误，**停用该 Function 的 PRI 直到重新使能**）
> - ⚠️ **无部分完成**——组内任一页失败，整组算失败

### 正文

**Page Request Message 字段（一种位布局）**：PRG Index 在 bit[75:67]、L 在 bit[66]、W 在 bit[65]、R 在 bit[64]。

**主机/IOMMU 处理**：查 device-context 判断是否使能 ATS/PRI（EN_ATS、EN_PRI）；使能则把请求排入内存中的 **page-request-queue（PQ）**，由**软件 handler** 处理并生成 PRG Response。

**IOMMU 自行生成错误响应**：

| 情况 | 响应 |
|---|---|
| 队列关闭 / 内存访问错误 | Response Failure |
| PRI 禁用 / 找不到 device-context | Invalid Request |
| 队列满/溢出而无法入队 | **Success**（表示"已收下，无需再请求"） |

> ⚠️ PRI Capability/Control 寄存器的**逐位字段未查证**（见存疑清单）。

### 自测
- [ ] PRI 和 ATS 是什么关系？为什么说 PRI 建立在 ATS 之上？
- [ ] 一个 PRG 里 8 个页请求，会收到几个响应？其中一个失败会怎样？
- [ ] 队列满时 IOMMU 为什么回 Success 而不是失败？

## PASID TLP Prefix #pcie #iommu #dma

> **速查**
> - **PASID（Process Address Space ID）是 20 位**，由 OS 分配与管理。它把 I/O 事务与**某个进程的地址空间**绑定——IOMMU 用 **(RID, PASID)** 而非仅 RID 来区分翻译上下文
> - **每个 Function 有独立的一套 PASID 取值**
> - 通过 **PASID TLP Prefix** 承载——TLP Prefix 是**前置在 TLP 前的 1 DW**，首 DW 的 **Fmt[2:0] = 100b** 标识"这是前缀"
> - **无 PASID 前缀时**，未翻译地址属于 **Requester ID** 对应的地址空间；**有 PASID 前缀时**，属于 **(Requester ID, PASID)** 共同确定的地址空间
> - ⚠️ 20 位是**名义宽度**，**实际有效宽度 = min(RC/TA 支持宽度, Function 支持宽度)**；未用的高位必须为 0
> - **SVA（Shared Virtual Addressing）** = CPU 与设备共用同一套虚拟地址/页表，**省去 DMA 前的翻译与页锁定（pin pages）**
> - **SVA 依赖三件：ATS + PRI + 支持 SVA 的 IOMMU**
> - **PASID 是 SVA 的"地址空间标签"**——没有它，IOMMU 只能按设备整体翻译

### 正文

**PASID Prefix 允许出现在**：带未翻译地址的 Memory Request（含 Untranslated AtomicOp）、Translation Request、Page Request Message、ATS Invalidation Request、PRG Response Message。

**IOMMU 如何维护同步**：用 `mmu_notifier()` 保持设备 TLB/ATC 与 CPU 页表同步。ATS 查不到 → 设备用 PRI 请主机把页调入 → 再 ATS 取翻译。

**Linux 侧**：进程首次访问 SVA 设备时分配一个 PASID。一个进程一个 PASID，供其所有线程使用；`fork`/`exec` 时移除，`clone` 时共享，`mm` 退出时才释放。

> ⚠️ **PASID TLP Prefix 的类型编码值与 DW 内 20 位 PASID 的精确位偏移未查证**（见存疑清单）。

### 自测
- [ ] PASID 解决什么问题？没有它 IOMMU 只能按什么粒度翻译？
- [ ] SVA 依赖哪三样东西？
- [ ] 为什么一个进程只需要一个 PASID？

## IOMMU：VT-d / AMD-Vi / SMMUv3 #iommu #dma

> **速查**
> - **两级翻译**：**Stage 1** = GVA→GPA（或 IOVA→GPA），由使用方拥有；**Stage 2** = GPA→HPA（最终物理地址），由宿主内核拥有
> - ⚠️ **只有 stage 2 时（无虚拟化）= 单级翻译**，直接把设备地址翻到物理地址
> - **Stage 2 是安全边界——用户态不得控制物理映射**
> - 三家定位设备条目的方式不同：**VT-d** 用 Root→Context Table→(Scalable)PASID Table；**AMD-Vi** 用 **Device Table → DTE**（按 DeviceID 索引）；**SMMUv3** 用 **Stream Table → STE**（按 StreamID，32 位）
> - ⚠️ **IOTLB 和 ATC 是两个独立缓存**：映射变化时**都要**失效——只清 IOTLB 不清 ATC 会留下设备侧旧翻译
> - 三家都用**内存描述符命令队列 + 一个"完成等待"命令**确认失效生效：VT-d QI completion / AMD `COMPLETION_WAIT` / SMMU `CMD_SYNC`
> - ⚠️ **失效必须等上在途 DMA 才能保证安全**

### 正文

**三家页表/表结构对照**：

| 维度 | Intel VT-d | AMD-Vi | Arm SMMUv3 |
|---|---|---|---|
| 定位设备条目 | Root→Context Table→(Scalable)PASID Table | **Device Table → DTE** | **Stream Table → STE**（StreamID 32 位） |
| 按 PASID 的条目 | Scalable 模式 PASID Table Entry（**64 B**） | **GCR3 Table** | **Context Descriptor / CD**（SSID=PASID，**64 B**） |
| 第一阶段页表 | 第一级页表（FLPTR，x86 风格，专有格式） | Guest 翻译（guest CR3 表） | **VMSAv8-64** stage 1（TTB0/TTB1） |
| 第二阶段页表 | 第二级页表（SLPTPTR/SLPTR，EPT 风格） | I/O 页表（**兼容 AMD64 long mode**） | **VMSAv8-64** stage 2 |
| AGAW/地址宽度 | 30/39/**48**/57 位（4 级=48 位） | 48/57 位 | stage1 48 位（v3.1+ 可 52 位） |
| 大页 | 2MB/1GB/512GB/1TB（SLLPS 报告） | 2MB/1GB | 依 granule（4/16/64KB）的 block 描述符 |
| 粒度 | 4KB 基页 | 4KB 基页 | 4/16/64KB |
| 失效机制 | IOTLB_REG/IVA_REG + **Queued Invalidation（QI）** | **命令队列** | **CMDQ** + `CMD_SYNC` |

**关键实现要点**：

- **VT-d（Scalable Mode, rev 3.x）**：每 PASID Table Entry **64 字节**，其中 **PGTT**（PASID Granular Translation Type）决定翻译类型（仅一级/仅二级/嵌套/直通）。二级大页由 **SLLPS** 报告：bit0=2MB、bit1=1GB、bit2=512GB、bit3=1TB
- **AMD-Vi**：**DTE** 按 DeviceID 索引，**GCR3 Table** 按 PASID 索引。I/O 页表**刻意做成与 AMD64 long mode 兼容**。PTE 关键位：**IW=bit62**（写）、**IR=bit61**（读）、**FC=bit60**（Force Coherent）、**U=bit59**、Page Address[51:12]、**D=bit6**（Dirty）、**A=bit5**（Accessed）。**有效读写权限 = 各级 IW/IR 的逻辑与**
- **SMMUv3**：配置查找**不使用输入地址**，仅由 **StreamID(32) + SubstreamID(20，即 PASID)** 决定。**STE 是 64 字节**；`STE.S1ContextPtr + SSID × 64` 定位 **CD**。页表用 **VMSAv8-64**（表描述符 type 11 / 块描述符 type 01 / 页描述符 L3）。`STE.V=0` 为无效，设备访问被 abort 并记 `C_BAD_STE` 事件

**失效机制细节**：

| | VT-d | AMD-Vi | SMMUv3 |
|---|---|---|---|
| 形式 | 寄存器式（IOTLB_REG + IVA_REG）或 **QI 队列** | 内存命令队列 | **CMDQ** |
| 典型命令 | QI 描述符 | `INVALIDATE_DEVTAB_ENTRY`、`INVALIDATE_IOMMU_PAGES`、`INVALIDATE_IOTLB_PAGES`、`INVALIDATE_INTERRUPT_TABLE` | `CMD_CFGI_STE`、`CMD_CFGI_ALL`、**`CMD_ATC_INV`**、`CMD_TLBI_*` |
| 完成等待 | QI completion | `COMPLETION_WAIT` | **`CMD_SYNC`** |

**AMD-Vi 的排序要求**：收到 `INVALIDATE_IOMMU_PAGES` 时须先失效此前 `INVALIDATE_DEVTAB_ENTRY` 相关缓存；收到 `INVALIDATE_IOTLB_PAGES` 时须先失效此前 DEVTAB 与 IOMMU_PAGES 缓存。

**AMD-Vi 的完成条件**：失效命令只有在**不再有依赖旧缓存内容在途的 DMA**（含已发出的读已完成、已翻译的写已推到 host bridge）后才算完成。

**SMMUv3 的 `CMD_ATC_INV`**：⚠️ 仅在 SMMU 实现 ATS（`ARM_SMMU_FEAT_ATS`）时合法；地址须按范围大小对齐（2 的幂个页），Linux 用 `fls(start ^ end)` 求最小覆盖幂次。
⚠️ **ATC 失效超时在随后的 `CMD_SYNC` 才报告**，而非在出错的 `CMD_ATC_INV`；驱动会重新单独下发每个 ATS SID 以定位无响应设备并屏蔽其 ATS（如清 `STE.EATS`）。

相关：[[address-path#MMU 和 IOMMU 有什么区别？]] · [[address-path#三层地址视图：谁能在哪一层合并？]]

### 自测
- [ ] Stage 1 和 Stage 2 各自是谁拥有的？为什么 Stage 2 是安全边界？
- [ ] IOTLB 和 ATC 为什么要分别失效？只清一个会怎样？
- [ ] 三家 IOMMU 用什么机制确认"失效已生效"？为什么必须等上在途 DMA？

---

## ⚠️ 存疑 / 未查证清单

> **以下内容未直接核对 PCI-SIG / Intel / AMD / Arm 官方规格原文。用于正式引用前须再验证。**

1. **TLP 头 byte1/byte2 内位域的确切位号**（TC、TD、EP、Attr、AT、Length[9:8] 的具体 bit 位置）——不同公开资料口径不一。**字段位宽可确定，位号存疑**
2. **PASID TLP Prefix 的类型编码值**与 DW 内 **20 位 PASID 的精确位偏移**
3. **PASID Prefix 的 Execute/Privileged 位**的精确位置（仅确认存在）
4. **PRI Capability/Control 寄存器**的逐位字段
5. **`IA32_PASID` MSR 的编号**（SRC 里 Linux 内核文档提到 0xD93，但本次未核对 Intel 规格）
6. **默认 MPS = 128 字节**——被广泛引用，但"默认取 MPSS 与 128 较小者"的说法亦有出现
7. **PCIe 6.0 Flit 的字段级位图**——`256 = 236+6+8+6` 被多来源引用，但存在其它口径（如 242 字节 payload）
8. **10-Bit Tag** 的确切编码与上限
9. **VT-d 一级/二级页表的逐位 PTE 位域**——本笔记只给了结构定位与大页/AGAW 层面
10. **SMMU CD 各字段位号**——仅给了字段名与语义，位偏移未逐一列出

---

## 来源

**PCI-SIG 公开资料**
- What is Flit Mode — https://pcisig.com/what-flit-mode-and-why-did-pci-sig-move-unit-data-exchange
- PCI-SIG PCIe 7.0 Webinar（FEC/CRC）— https://pcisig.com/sites/default/files/files/PCI-SIG%20PCIe%207.0%20Webinar_Rev5_FINAL.pdf
- **PASID-ATS ECN** — http://pcisig.com/sites/default/files/specification_documents/ECN-PASID-ATS-2011-03-31.pdf
- **ATS Errata 1.0**（ITag Vector、BME、寄存器字段）— http://www.pcisig.com/specifications/iov/ats/ATS_Errata_1_0-11302007.pdf

**Linux 内核文档 / 源码**
- Shared Virtual Addressing (SVA) with ENQCMD — https://docs.kernel.org/6.5/arch/x86/sva.html
- `drivers/pci/pci.c`（MPS/MRRS、`128 << n`）— https://github.com/torvalds/linux/blob/master/drivers/pci/pci.c
- `arm-smmu-v3.c`（`CMD_ATC_INV`、`STE.EATS`、超时处理）
- iommufd / VT-d 嵌套翻译补丁（SLPTR、PASID entry、`IOMMU_HWPT_INVALIDATE`）

**Intel / AMD / Arm**
- Intel VT-d Architecture Specification（scalable mode、SLPTPTR、NIU/IVO、SLLPS）
- **AMD I/O Virtualization Technology (IOMMU) Specification, Rev 3.09, 48882** — https://www.amd.com/content/dam/amd/en/documents/processor-tech-docs/specifications/48882_IOMMU.pdf
- Arm SMMUv3 架构文档 — https://documentation-service.arm.com/

**第三方交叉参考**（字段细节仍以官方规格为准）
- Cadence：Demystify ATS in PCIe 6.0
- ACM Computing Surveys：An Introduction to CXL（Flit 格式）
- Linux Plumbers Conf：MPS
