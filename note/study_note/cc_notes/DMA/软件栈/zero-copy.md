---
tags: [dma, kernel]
updated: 2026-10-10
---

# 零拷贝（zero-copy）技术谱系

> **计数口径（先定义，否则"省了几次拷贝"没法讨论）**
>
> | 符号 | 含义 |
> |---|---|
> | **C** | **数据拷贝**——CPU 执行的字节搬运（`memcpy` / `copy_to_user` / `copy_from_user`） |
> | **D** | **DMA 传输**——设备 DMA 引擎执行的搬运，不占 CPU。单独计数，**不混进"拷贝数"** |
>
> **判据：数据字节本身有没有经过 CPU 执行的搬运。经手过 = 有拷贝；只处理地址/描述符 = 无拷贝。**
>
> 一个贯穿全文的事实：绝大多数"零拷贝"方案**不是让数据少走物理链路**，而是**去掉 CPU 在中间当搬运工那一步**，把搬运交给 DMA 引擎，CPU 只处理元数据。

## 零拷贝的基线是什么？ #dma #kernel

> **速查**
> - 基线：`read(fd, buf, n)` + `write(sock, buf, n)` 把文件发给 socket = **2C + 2D**
> - 四次搬运：① DMA 磁盘→page cache ② **CPU 拷贝 page cache→用户 buf** ③ **CPU 拷贝 用户 buf→socket 缓冲 skb** ④ DMA skb→NIC
> - 外加 4 次用户/内核态切换
> - **②③ 这两次 CPU 拷贝就是所有内核内零拷贝 API 要消灭的对象**
> - 硬件底座：**DMA 引擎 + SGL**（scatter-gather list，一组 `(地址,长度)` 描述符，让一次 DMA 覆盖物理上不连续的多个页）

### 正文

```
磁盘 --D--> page cache --C--> 用户buf --C--> socket buf(skb) --D--> NIC
                          拷贝(1)           拷贝(2)
```

**pin（钉住 / 内存注册）**：把一段虚拟内存的物理页固定住，保证某个时间窗内不被换出、迁移、COW 重映射。是**软硬协同的契约**——软件（内核/驱动）执行 pin，硬件（设备 MMU / IOMMU）依赖这个保证去做 DMA。

### 自测
- [ ] 基线的四次搬运里，哪两次是 CPU 拷贝？为什么它们是消灭对象？
- [ ] C 和 D 为什么要分开计数？

## 内核内的零拷贝：sendfile / splice / vmsplice / tee / mmap #dma #kernel

> **速查**
> - **`sendfile`**：file→socket，**省掉拷贝 (1) 和 (2)**，剩 0C + 2D。⚠️ **需要网卡支持 SGL 才是 0 CPU 拷贝**，老网卡会退化成 1C
> - **`splice`**：在两个 fd 之间搬，**至少一端必须是管道（pipe）**，否则 `EINVAL`。接的是**页引用**不是页内容
> - 关系：**现代内核里 `sendfile` 内部就是用 `splice` 实现的**（用一个内部 pipe 把两 fd 接起来），sendfile 是 file→socket 的特化接口
> - **`vmsplice`**：user→pipe，省掉"用户 buf→pipe"那次拷贝。⚠️ **以失去缓冲区写权限为代价**——不加 `SPLICE_F_GIFT` 时用户不得改写内存直到消费完
> - **`tee`**：pipe→pipe，**不消费源**（分流/广播），只复制页引用 → **0C + 0D**
> - **`mmap`**：省掉 page cache↔用户 buf 的中间拷贝，但 **CPU 仍直接 load/store 数据**——是"消除中间人"，不是"消除 CPU 参与"
> - ⚠️ **`mmap` + 普通 `write` 到 socket，拷贝一次没少**
> - 归属：【软件】为主——内核 pipes 子系统持有页引用；配合硬件 DMA + SGL

### 正文

**splice 限制的历史变化**：Linux 2.6.30 及更早要求**恰好一个** fd 是 pipe；2.6.31 起**两个都可以**是 pipe，但**至少一个仍是硬要求**。所以 socket↔普通文件不能单次 splice 直达，标准做法是 `socket → pipe → 文件` 两次 splice。

**一个容易踩的点**：splice 的管道缓冲区受 `F_SETPIPE_SZ` 限制（默认约 64KB，上限受 `/proc/sys/fs/pipe-max-size` 约束）。

**mmap 的代价**：

- **缺页开销**：首次触碰每页都要陷入内核。随机访问大文件可能比顺序 `read` 慢；小文件/顺序读常不如 `read()`（避免缺页 + TLB 抖动）
- **错误检测被推迟**：写 mmap 是写内存，真正的 I/O 错误延后到 writeback 才暴露，可能触发 `SIGBUS`；**失去 `write()` 的同步错误返回**
- ⚠️ **只有 `MAP_SHARED` 才真正共享 page cache；`MAP_PRIVATE` 写时触发 COW，那一刻零拷贝性质消失**

### 自测
- [ ] `sendfile` 省掉了哪两次拷贝？什么情况下退化？
- [ ] `splice` 为什么必须至少一端是 pipe？
- [ ] `mmap` 是"零拷贝"吗？它省掉了什么、没省掉什么？
- [ ] `MAP_PRIVATE` 的 mmap 在什么时候失去零拷贝性质？

## 网络侧的零拷贝：MSG_ZEROCOPY / XDP / AF_XDP #dma #kernel

> **速查**
> - **`MSG_ZEROCOPY` / `SO_ZEROCOPY`（发送方向）**：内核 **pin 住用户页**，让 skb 的 frag 直接指向用户页（把 SGL 填给 NIC）。**只覆盖 TX**
> - 完成通知：DMA 真把数据读走后，内核往 socket 的**错误队列**投 `SO_EE_ORIGIN_ZEROCOPY` 事件，用户用 `recvmsg(MSG_ERRQUEUE)` 收割
> - ⚠️ **收到通知前，用户不得改写该缓冲区**——核心契约
> - ⚠️ **收益门槛**：官方建议单次发送 **> 约 10KB** 才划算（小包时 pin + 通知开销超过拷贝收益）
> - **XDP** 是**钩子位置**，不是零拷贝本身——价值在于让包**还没被复制进 skb 时**就被处理（`XDP_DROP` / `XDP_TX` / `XDP_REDIRECT`）
> - **`AF_XDP` 有两种模式**：**零拷贝模式**（NIC DMA 直接把包写进用户注册的 UMEM，**真零拷贝**，⚠️ **需驱动支持 XDP zero-copy**）vs **拷贝模式**（先到内核缓冲再 memcpy 进 UMEM，**仍有 1 次 CPU 拷贝，不是真零拷贝**）
> - ⚠️ **RX 和 TX 是分开的**：MSG_ZEROCOPY 只有 TX；AF_XDP 主打 RX。**零拷贝网络栈通常是两者拼出来的，没有单一开关**
> - 归属：【软硬协同】

### 正文

**AF_XDP 的 UMEM**：用户态预分配并注册的大块内存（**通常配 hugepage**）。描述符环（FILL / RX / TX / COMPLETION）在用户态与内核间共享。

### 自测
- [ ] `MSG_ZEROCOPY` 的完成通知走什么通道？为什么必须等通知才能改写缓冲区？
- [ ] AF_XDP 的零拷贝模式和拷贝模式差在哪？为什么零拷贝模式要驱动支持？
- [ ] 为什么没有"一个开关搞定双向零拷贝"的 API？

## io_uring 的注册缓冲区：它**不是**数据零拷贝 #dma #kernel

> **速查**
> - ⚠️ **`IORING_REGISTER_BUFFERS` 省掉的不是数据拷贝，而是每次 I/O 的页表遍历/校验开销（元数据成本）**
> - 机制：一次性把一批缓冲区交给内核，内核**执行一次 pin + 建立映射**，之后引用它们的操作（`READ_FIXED` / `WRITE_FIXED`）直接用预建映射，**不再每次 `get_user_pages`**
> - `IORING_REGISTER_FILES`：预注册 fd，省掉每次 fd→file 查找。同样是元数据优化
> - **与真正零拷贝合流的操作**：`IORING_OP_SEND_ZC`（等价 MSG_ZEROCOPY，用 CQE 替代错误队列）、`IORING_OP_SPLICE` / `TEE`
> - ⚠️ **`PROVIDE_BUFFERS` 和 `REGISTER_BUFFERS` 是两回事**：前者是给**接收方向**的环形缓冲组，配合 multi-shot recv，减少每次提交传地址
> - 代价：注册的缓冲区/文件在内核被 pin/引用，同样吃 pin 的额度
> - 归属：【软件】（可叠加硬件零拷贝）

### 正文

**这是全篇最容易误读的一条**——名字里有"注册"，机制也确实和 pin 有关，但它**一次数据拷贝都没省**。它优化的是"每次 I/O 都要重新做的那套页表/查表动作"。

### 自测
- [ ] `IORING_REGISTER_BUFFERS` 省掉的是什么？为什么不能叫它数据零拷贝？
- [ ] `PROVIDE_BUFFERS` 和 `REGISTER_BUFFERS` 的区别是什么？

## RDMA 与 GPUDirect #dma #kernel #rdma

> **速查**
> - **RDMA**（Remote Direct Memory Access）：本机 HCA（Host Channel Adapter，即 RDMA 网卡）直接读写对端内存，**绕过两端内核**
> - **三件套**：内存注册（`ibv_reg_mr`，pin + 把翻译表交给 HCA，换 `lkey/rkey`）、队列对 QP、完成队列 CQ
> - **省掉的**：常规网络"应用 buf→内核 socket buf→NIC"整条链上的 CPU 拷贝，以及对端"NIC→内核→应用 buf"的拷贝。数据全程只走 **2 次 DMA**，**0C**。**跨主机也是真零拷贝**
> - **双边（SEND/RECV）** vs **单边（RDMA READ/WRITE）**——单边**远端 CPU 完全不参与**
> - **GPUDirect RDMA**：让 RDMA 网卡**直接 DMA 进/出 GPU 显存**，省掉"GPU 显存→主机内存→NIC"的中转
> - **GPUDirect Storage (GDS)**：存储设备**直接 DMA 进/出 GPU 显存**，接口是 **cuFile**（`cuFileRead` / `cuFileWrite`）
> - 归属：【软硬协同】硬件提供 HCA 内置地址翻译、PCIe P2P；软件负责 pin、建表、填描述符

### 正文

**GPUDirect RDMA 的机制演化**：

- 早期：`nv_peer_mem`（NVIDIA 专有内核模块）+ `ibv_reg_mr`
- 现在：`nvidia-peermem` + **dma-buf**（内核通用的跨驱动缓冲区共享框架），NIC 驱动对 dma-buf 导出者做 DMA mapping

**前提**：PCIe 支持 P2P——NIC 与 GPU 在同一 PCIe 域/switch 下。**对拓扑敏感**（跨 NUMA/跨 switch 差别大）。

**GDS 的两条路径**：本地 NVMe（PCIe P2P）与远端存储（NVMe-oF / RDMA）。

> ⚠️ **不确定项**：HCA 的地址翻译结构（页表还是 TLB 式缓存、粒度）随厂商而异，需查具体文档（如 mlx5 的 MTT/MPT）。

相关：[[rdma#RDMA 是什么？]]

### 自测
- [ ] RDMA 单边操作"绕过远端 CPU"意味着什么？这带来什么能力，又带来什么使用约束？
- [ ] GPUDirect RDMA 省掉的到底是哪一段搬运？
- [ ] 为什么 GPUDirect 对 PCIe 拓扑敏感？

## DPDK / SPDK 的 kernel bypass #dma #kernel

> **速查**
> - ⚠️ **准确名称是 kernel bypass（绕过内核），不是"零拷贝"**。它把设备驱动搬到用户态；**顺带**实现了零拷贝，但主收益是"不走内核路径"
> - **硬件提供**：设备寄存器与 DMA 队列可被用户态映射（NIC 的 queue pair 寄存器、NVMe 的 SQ/CQ 都在 BAR 里）
> - **软件提供**：用户态 PMD（Poll Mode Driver）、大页内存池、无锁环
> - **DPDK**：NIC 把包 DMA 进预分配的 `rte_mbuf` 缓冲，用户态直接读 → **RX 零拷贝（0C）**。省掉内核对每包的 **skb 分配 + 拷贝**、系统调用、中断
> - **SPDK**：NVMe 的 SQ/CQ 直接映射到用户态，NVMe DMA 直接读写用户**大页**里的缓冲。省掉块设备层请求队列、内核缓冲、中断
> - ⚠️ **代价很重**：**独占整核做轮询（100% CPU）**、**独占设备**（内核栈用不了这张卡）、必须大页 + pin、受大页配额与 mlock 限制
> - ⚠️ **必须大页的原因**：减少描述符数量与 TLB 压力（一页 2MB/1GB，同样缓冲区的描述符数少几十/几百倍）
> - 归属：【软硬协同】

### 自测
- [ ] 为什么说 DPDK 的主收益是 kernel bypass 而不是零拷贝？
- [ ] DPDK/SPDK 为什么强制要求大页？
- [ ] 它们的代价主要是什么？

## 零拷贝的代价 #dma #kernel #pinned-memory

> **速查**
> - **pin 是所有零拷贝的共同税**——设备 DMA 用物理地址，**"给设备的地址"必须有一个稳定期契约**
> - **限额**：历史上受 `RLIMIT_MEMLOCK` 限制。⚠️ **已核实**：**BPF 子系统自 Linux 5.11 起改用 memcg 记账**，不再受 `RLIMIT_MEMLOCK`。其他子系统（RDMA/io_uring/DPDK/GPUDirect）走哪套**随内核版本而异**
> - **pin 本身的开销**：`get_user_pages` 要遍历页表、对每页加计数；**一次 pin 几 GB 会阻塞很久** → 高性能系统一律**启动时一次性注册、长期持有**
> - ⚠️ **长期 pin（`FOLL_LONGTERM`）的副作用**：被 pin 的页**不能迁移** → 妨碍内存规整（compaction）、透明大页（THP）合并、NUMA balancing、内存热插拔、虚拟机 live migration；妨碍页回收，**pin 住大内存会推高整体内存压力，甚至引发 OOM**
> - ⚠️ **失去 COW 语义**：pin 住的页不能被 COW 重映射——设备手里握着**旧物理页**。`fork` + 大缓冲 + 异步零拷贝发送时，容易出现"进程已改 buf，设备还在读写旧页"
> - **复杂度**：DMA 完成 ≠ 发送成功（TCP 还可能重传，重传期间缓冲区仍被引用）；生命周期跨到 I/O 完成；驱动/硬件支持矩阵参差，必须有探测与回退分支；**可观测性下降**（数据不经用户态/内核时，tcpdump / strace / perf 失灵）
> - **收益门槛**：小数据量下 pin + 通知开销常**大于**省下的拷贝（MSG_ZEROCOPY 约 10KB 阈值即明证）
> - **一句话**：零拷贝不是免费午餐，是"**用元数据开销换数据搬运开销**"，只在数据量大或路径深时划算

### 正文

**统一推论**：所有零拷贝 API 都要求"**提交后、完成通知前，用户不得改写/释放缓冲区**"。这是用便利性换零拷贝的典型对价。

`vmsplice` 不带 `SPLICE_F_GIFT` 时用户不能改缓冲区，本质是同一约束的显式化。

相关：[[memory#pin 是软件机制还是硬件机制？]]

### 自测
- [ ] 为什么 pin 是零拷贝的必然代价？
- [ ] 长期 pin 会妨碍哪些内核机制？
- [ ] 为什么零拷贝在数据量小时反而不划算？

## 逐个技术"省了几次拷贝"（总表） #dma #kernel

> **速查**
> - 这张表回答唯一真正重要的问题：**每个技术到底省掉了哪一次拷贝、还剩几次**
> - **三类名词要分清**：**真·零拷贝**（字节从不经过 CPU）/ **少一次拷贝**（减少但未清零）/ **名字里有"零拷贝"但一次没省**
> - 基线 = **2C + 2D**

### 正文

| 技术 | 针对的那一次拷贝 | 省掉 | 剩下 | 真·零拷贝？ |
|---|---|---|---|---|
| **sendfile** | page cache→user→skb | C(1) + C(2) | 0C + 2D（有 SGL）；老网卡退化 1C | **是**（CPU 层面） |
| **splice** | fd↔pipe 间字节搬运 | 该 C（改用页引用） | 0C（跨设备才有 D） | **是** |
| **vmsplice** | user buf→pipe | 这 1 次 C | 0C | **是**（但失去缓冲区写权） |
| **tee** | pipe→pipe 字节复制 | 全部 C | 0C + 0D | **是** |
| **mmap** | page cache↔user buf | read 的 C(1) / write 的 C(2) | 本地文件 0C；写回仍 D | **局部是**：消灭中间人，CPU 仍直接访问数据 |
| **MSG_ZEROCOPY** | user buf→skb | C(2) | 发送路径 0C + 2D | **是**（TX） |
| **AF_XDP 零拷贝** | NIC→kernel→user | RX 路径全部 C | 0C + 1D | **是**（RX，需驱动支持） |
| **AF_XDP 拷贝模式** | — | 只省部分内核开销 | 仍 **1C**（kernel→UMEM） | **否**：只是"少一次" |
| **XDP（DROP/TX/REDIRECT）** | 避免构造 skb 的拷贝 | 构造 skb 那次 C | 视 action 而定 | **视情况** |
| **io_uring 注册缓冲** | **不是数据拷贝**，是页表查找 | 每次 `get_user_pages` 的**元数据** | 数据搬运不变 | **否** |
| **io_uring SEND_ZC** | user buf→skb | C(2) | 0C + 2D | **是**（TX） |
| **RDMA 单边 WRITE** | 应用buf→内核buf→NIC 全程中转 | 两端所有 C + 内核中转 D | **2D**，0C | **是**（跨主机也是） |
| **RDMA 内存注册** | 不是数据拷贝 | — | — | **否**：是使零拷贝可行的前置 pin |
| **GPUDirect RDMA** | GPU→host bounce buffer | 经主存的中转拷贝 | 1D（NIC→显存） | **是** |
| **GPUDirect Storage** | storage→host buf→GPU | 经主机内存的中转拷贝 | 1D（存储→显存） | **是** |
| **DPDK / SPDK** | 内核 skb/块层缓冲那几跳 | 每包/每请求的内核 C + 系统调用 | **0C** | **是**（更准确叫 kernel bypass） |

**三类辨伪**：

- **（A）真·零拷贝**：sendfile（有 SGL NIC）、splice / vmsplice / tee、MSG_ZEROCOPY、AF_XDP 零拷贝、RDMA、GPUDirect、DPDK/SPDK
- **（B）少一次拷贝**：`mmap`、AF_XDP 拷贝模式、无 SGL 网卡上的 sendfile
- **（C）名字里有"零拷贝/注册"但一次没省**：`IORING_REGISTER_BUFFERS`、RDMA 内存注册、pin 本身

**一句话判据**：

1. **字节有没有进过 CPU 的 load/store 循环？** 进了 → 有拷贝
2. **剩余搬运是 DMA 引擎做的还是 CPU 做的？** 是 DMA → 这一步不算拷贝

### 自测
- [ ] 说出三个"名字里有零拷贝但一次拷贝都没省"的东西。
- [ ] `mmap` 和 `sendfile` 在"零拷贝"这件事上的本质差别是什么？
- [ ] 用那两句判据判断：`AF_XDP` 拷贝模式算零拷贝吗？

---

## 横切要点

- **pin 是所有零拷贝的共同税**：sendfile/splice 只 pin page cache 页（内核已有引用，成本低）；RDMA/DPDK/GPUDirect/io_uring 要 pin 用户内存（成本高，受额度限制）
- **大页是减少描述符与 TLB 压力的通用手段**：DPDK、SPDK、AF_XDP 都强依赖
- **SGL 是硬件能力，填 SGL 是软件职责** —— 看到"网卡支持零拷贝"，要追问是硬件有 DMA/SGL 能力，还是驱动已实现填表与回退路径
- **异步化是零拷贝的孪生代价**：数据不再同步拷走，就必须有完成通知（CQ / 错误队列 / io_uring CQE），用户被迫管理缓冲区生命周期

## 已核实 / 不确定

- **已核实**：splice 仍需至少一端为 pipe（现行内核，`EINVAL`；2.6.30 前要求恰好一个，2.6.31 起两个都可为 pipe）；BPF 自 Linux 5.11 起用 memcg 记账取代 `RLIMIT_MEMLOCK`
- **不确定**：通用 mlock 记账向 memcg 迁移的确切版本（疑为 5.14，未证实）；各子系统限额走 rlimit 还是 memcg 的具体行为随内核版本而异；RDMA HCA 地址翻译表的具体结构随厂商而异

## 来源

- splice(2) man page
- libbpf: Auto-bump RLIMIT_MEMLOCK if kernel needs it for BPF
- ebpf-go: Resource Limits
