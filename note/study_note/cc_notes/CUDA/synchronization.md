---
tags: [cuda, memory]
updated: 2026-10-11
---

# CUDA 同步与内存模型

## 生产者-消费者为什么需要内存模型？relaxed / release / acquire 各保证什么？ #memory-model #cuda

> **速查**
> - 内存模型回答的是**两个不同**问题：coherence（单地址，硬件免费）vs consistency（跨地址，不免费）
> - 破坏生产者-消费者的有**三条独立的路**：编译器重排（软件）/ store buffer 重排（硬件，弱序机器）/ cache 副本时序（硬件，MESI 那层）
> - 唯一重要的规则：**release 写 + acquire 读出同一个值 ⇒ synchronizes-with**
> - 只有「这个原子的值要带着**别的内存位置**一起可见」时才要 release/acquire；否则 `relaxed` 就够，多写反而变慢
> - 「barrier」是三样东西共用：编译器屏障（**不生成指令**）/ 内存 fence（硬件执行）/ 执行屏障（会合，真的停线程）

### 正文

**① 先分清两个中文都译作「一致性」的词**

| | **Coherence（一致性）** | **Consistency（内存一致性模型）** |
|---|---|---|
| 问的是 | 单个地址 | 多个地址之间的顺序 |
| 例 | 所有 SM 对 `flag` 的写有没有统一全序？会不会读到撕裂值？ | 我看见 `flag == 1` 时，是否一定看见 `data == 42`？ |
| 谁解决 | **硬件**（MESI 类协议 / L2 这个串行化点） | **编译器 + fence 指令** |
| 代价 | 免费，协议自己跑 | 不免费，必须显式写出来 |

生产者-消费者**同时需要两列**——这是它当第一教学例的原因。

**② 问题长什么样**

```c
// 共享： int data;  atomic<int> flag{0};
// T1 生产者              // T2 消费者
data = 42;                while (flag.load() == 0) {}
flag.store(1);            r = data;          // 期望 42
```

`r == 42` **默认没有保证**，而且是三条互相独立的路都能弄坏它：

**a) 编译器重排（软件）**

编译器不知道有别的线程。在它眼里 `data = 42` 和 `flag.store(1)` 是两条无关的写，可以调换；`r = data` 是**循环不变量**，能直接被提到循环外面去。`-O2` 下看汇编就能验证：消费者那个自旋循环里根本没有对 `data` 的读。

**b) Store buffer 重排（硬件）**

一条 store 指令 retire 了不代表「写完成了」。它进 **store buffer**（每个核私有的、存放待完成写的队列），在那里等自己那条 cache line 拿到所有权（M 或 E 状态）才真正落地。

- **弱内存序机器**（ARM、Power、GPU）：store buffer 里的写**排空顺序不保证等于程序序**，所以 `flag=1` 可能先到。
- **x86-TSO**：store buffer 是 FIFO，按程序序排空。所以 `data=42; flag=1` 这个 **StoreStore** 顺序在 x86 上被保证。

这就是同一个 C++ 程序在 x86 上「碰巧能跑」、在 ARM / GPU 上就坏的原因。

**c) Cache 副本的时序（硬件，MESI 那一层）**

就算排空顺序对，T2 的 L1 里可能还留着 `data` 那行的**旧副本**。`flag` 那行的失效消息和 `data` 那行的失效消息是两笔独立事务，到达 T2 的先后没有约束。

**③ 五档 memory order**

| C++ | PTX | 保证什么 | x86 编译成 | ARMv8 编译成 |
|---|---|---|---|---|
| `relaxed` | `.relaxed` | **只有原子性**（不撕裂），无任何顺序 | 普通 `mov` | 普通 `ldr`/`str` |
| `acquire`（load） | `ld.acquire` | 它**之后**的访存不能被提到它**之前** | 普通 `mov` | `ldar` |
| `release`（store） | `st.release` | 它**之前**的访存不能被落到它**之后** | 普通 `mov` | `stlr` |
| `acq_rel` | `atom.acq_rel` | 两者兼有 | `lock` 前缀 | `ldaxr`/`stlxr` |
| `seq_cst` | `fence.sc` | 所有 seq_cst 操作存在单一全序 | `mfence` 或 `lock` | 额外 `dmb ish` |

注意 x86 那一列：**acquire load 和 release store 在 x86 上编译成普通 `mov`，零指令开销**——因为 TSO 本身就禁止 LoadLoad / StoreStore 重排，release/acquire 要的那部分顺序 x86 白送。这是「硬件已有能力、软件只是去用」的干净例子。

**④ 唯一重要的规则：synchronizes-with**

> 一个 **release store** 把值 V 写进原子变量 X，之后某个 **acquire load** 从 X 读到了 V —— 那么**该 release store 之前的所有访存（程序序）** happens-before **该 acquire load 之后的所有访存**。

三个必须说准的点：

1. **这是跨线程的配对，靠「值」中介。** 不是「acquire 和 release 各自在自己线程上当栅栏」——两个 release 碰在一起什么也不发生。必须是 release **写**、acquire **读出同一个值**。
2. **`sequenced-before`** = 同一线程内的程序序；**`happens-before`** = 由 sequenced-before 与 synchronizes-with 传递闭包出来的跨线程可见性保证。前者是语法，后者是语义。
3. 单向且成对：producer 需要 release，consumer 需要 acquire。只做一边不成立。

**手工验算——成立的：**

```c
T1:  data = 42;  flag.store(1, release);
T2:  while (flag.load(acquire) == 0) {}
     r = data;                        // 必为 42
```

release store 在程序序上排在 `data = 42` 之后；acquire load 读到了它写的值 ⇒ synchronizes-with ⇒ `data = 42` happens-before `r = data` ⇒ **r == 42 有保证**。

**当场失败的反例（两边都 relaxed）：**

```c
T1:  data = 42;  flag.store(1, relaxed);
T2:  while (flag.load(relaxed) == 0) {}
     r = data;                        // 可以是 0
```

`relaxed` **不建立任何 synchronizes-with**，`data = 42` 和 `r = data` 之间连一条边都没有。合法执行：T1 的 `flag=1` 先被排空到 L2（`data=42` 还堵在 store buffer 里等行所有权），T2 读到 1 退出循环，再读 `data` → **读到 0**。这个交错本身就在规范允许的集合里，不用跑机器。

**反向的反例——过度同步同样是错的：**

```c
atomic<int> cnt;
cnt.fetch_add(1, relaxed);      // 完全正确
```

没有任何别的东西需要跟它一起可见，用 `acq_rel` 只会给每次自增塞一个 `mfence`/`lock`，收益为零。

> **诊断问句**（比背规则有用）：*「有没有别的内存位置，需要跟着这个原子的值一起被看见？」* —— 有 → release/acquire；没有 → relaxed。

**⑤「barrier」这个词被三样东西共用，必须拆开**

| 名字 | 是什么 | 作用域 | 软/硬件 | 生成指令吗 |
|---|---|---|---|---|
| 编译器屏障 `atomic_signal_fence` / `asm volatile("":::"memory")` | 只禁止**编译器**挪动访存 | 本线程 | 纯软件 | **不生成** |
| 内存 fence `atomic_thread_fence` / `__threadfence()` | 约束**硬件**的可观察顺序 | cta / gpu / sys | 硬件执行 | 生成（x86 上可能退化成零） |
| 执行屏障 `__syncthreads()` / `bar.sync` | **线程互相等待**（会合 rendezvous） | block | 硬件 barrier 单元 + 软件计数 | 生成 |

要害区别：前两个是**顺序**原语，它们不阻塞任何东西，只约束「操作以什么顺序变得可见」；第三个是**会合**原语，真的把线程停住。

而且 `__syncthreads()` 是复合的：

> **`__syncthreads()` = 会合 + block 级 fence。**

所以同一个 block 内、走 shared memory 的生产者-消费者，一条 `__syncthreads()` 就够：

```cuda
__shared__ int sdata;
sdata = 42;           // 普通写
__syncthreads();      // 会合 + threadfence_block
r = sdata;            // 保证看到 42
```

而**跨 block** 通信时 `__syncthreads()` 完全无用——它只同步一个 block 内的线程。跨 block 的做法见下一条。

### 自测

- [ ] coherence 和 consistency 各管什么？生产者-消费者为什么两样都要？
- [ ] 破坏 `data=42; flag=1` 的三条路分别属于软件还是硬件？x86 上哪一条被硬件挡住了？
- [ ] `relaxed` 用在计数器上为什么是对的？用在生产者-消费者上为什么当场就错？
- [ ] 编译器屏障和内存 fence 的区别是什么？`__syncthreads()` 属于哪一种、作用域多大？

## MESI 协议保证了什么、又没保证什么？ #memory-model #arch

> **速查**
> - 四态 M/E/S/I，不变式：任意时刻**一行最多只有一个 cache 处于 M 或 E**，S 可以有多个
> - 机制是**侦听（snooping）**：每个 cache 都监看互连；写之前必须先让别的副本失效，读时由持有者 cache-to-cache 回填
> - **内存可以一直是旧的**——一致性点是 cache，不是 DRAM；只有 M 态行被驱逐时才写回
> - **MESI 把一个地址的访问排成全序；两个不同地址之间的顺序它一无所知** ← 这就是内存模型存在的理由

### 正文

**① 四个状态**（每 cache line、每 cache 一份）

| 状态 | 这份副本 | 别的 cache 还有副本吗 | 内存是最新的吗 |
|---|---|---|---|
| **M** Modified | 唯一有效副本，**脏**（和内存不同） | 没有 | 否 |
| **E** Exclusive | 唯一有效副本，**干净** | 没有 | 是 |
| **S** Shared | 有效，**干净** | 有 | 是 |
| **I** Invalid | 无效，**不可读** | — | — |

**② 机制**（硬件）：每个 cache 都在监看互连总线（snooping），只需要两类消息——「我要读这行」「我要作废这行」。

**手工推一遍状态转移：**

```
初始：行 X 在 L1_A 和 L1_B 都是 S，内存里是干净值 v0

(1) A 要写 X
    A 发 Invalidate(X)
    B 侦听到 → 把自己那份置 I（丢弃）
    B 回 Ack
    A 收齐 Ack → 置 M，写入 v1
    现在：A=M(v1)   B=I   内存=v0（已过期）

(2) B 要读 X
    B 发 Read(X)
    A 侦听到（它持 M，是唯一有效副本）
    A 把 v1 直接回填给 B（cache-to-cache transfer），自己置 S
    B 置 S，拿到 v1
    现在：A=S(v1)  B=S(v1)  内存仍可能是 v0
```

最后一行值得停一下：**内存可以一直是旧的**。coherence 点是 *cache*，不是 DRAM——只有 M 态的行被驱逐时才写回。这就是架构上「cache 是一致性点（point of coherence）」的实际含义。

**③ 边界——最关键的一句**

> **MESI 把「对同一个地址的访问」排成了一个全序；它对「两个不同地址之间」的顺序一无所知。**

`data` 在行 X，`flag` 在行 Y：

- MESI **保证**：T2 不会读到撕裂的 `flag`，且最终一定会看到 `flag == 1`。
- MESI **不保证**：T2 看到 `flag == 1` 时也看到 `data == 42`——X 的失效事务和 Y 的失效事务是两笔独立消息，到达 T2 的先后无约束。

**这就是内存模型存在的理由。** coherence 是单地址、硬件、免费；consistency 是跨地址、要 fence、不免费。

**④ 两个常见变体**

- **MESIF**（Intel）：多一个 F=Forward，让持 F 态的 cache 转发数据而不变成第二个数据源，减少一致性流量。
- **MOESI**（AMD）：多一个 O=Owned——脏但可共享，避免「为了共享先写回内存」的往返，多路系统上收益明显。

### 自测

- [ ] 为什么一个 cache line 不可能同时有两个 cache 处于 M 态？
- [ ] 手工推一遍：行 X 在 A、B 都是 S，A 写、然后 B 读，四个实体的状态怎么变？内存什么时候变？
- [ ] MESI 保证了什么、没保证什么？用 `data` / `flag` 两个地址说清楚。
- [ ] MOESI 的 O 态解决的是什么问题？

## GPU 上跨 SM 通信怎么保证一致？scope 和 mbarrier 是什么？ #memory-model #cuda #mbarrier

> **速查**
> - GPU 的 L1 **之间不侦听**；Volta 起 L1 缓存 load，但 store 是 **write-through**
> - **L2 是全 GPU 的一致性点**：所有 global 访问在 L2 串行化，global 原子操作**在 L2 执行** → 天然全 GPU 原子
> - 跨 SM 通信必须绕过 L1：老写法 `volatile` + `__threadfence()`；新写法 `cuda::atomic`。`volatile` 只绕 L1，**不排任何序**
> - PTX 的 acquire/release 带 **scope**（`.cta` / `.cluster` / `.gpu` / `.sys`）：选窄了是**正确性** bug，选宽了是**性能** bug
> - `mbarrier` = release/acquire **加**一个可拆分的会合 → 搬运引擎能当参与者，`bar.sync` 不能

### 正文

**① GPU 的 coherence 和 CPU 不一样在哪**

- **L1 之间不做侦听式 coherence。** 没有「SM0 写 → SM1 的 L1 副本失效」这条路径。
- **L1 是 write-through 的**（Volta 起）：SM0 的 store 打到 L2，并更新 SM0 自己的 L1；**但 SM1 的 L1 可能还留着一份只读的旧副本**。
- **L2 是全 GPU 的一致性点**。所有 global 访问在 L2 串行化，由此推出：

> **GPU 范围的原子性住在 L2。** `atomicAdd()` 打在 global 上，是在 L2 的 ALU/ROP 单元执行的，所以它天然全 GPU 原子，且不需要任何 coherence 流量。

**② 跨 SM 通信的完整写法**

```cuda
__device__ int data;
__device__ unsigned flag = 0;

__global__ void producer() {
    data = 42;
    __threadfence();              // fence.acq_rel.gpu：把 data=42 推到 L2
    atomicExch(&flag, 1);         // 原子写，在 L2 执行 → 全 GPU 立即可见
}

__global__ void consumer() {
    while (atomicAdd(&flag, 0) == 0) {}   // 原子读，从 L2 取
    __threadfence();              // 防止后面的读被提前
    if (data != 42) printf("BROKEN\n");
}
```

逐条对照「少了哪一步会坏」：

| 少掉的部分 | 后果 |
|---|---|
| producer 的 `__threadfence()` | `data=42` 可能还堵在 store buffer / 只在 SM0 的 L1 里，而 `flag` 已经到 L2 → consumer 读到旧 `data` |
| `flag` 用普通 store 而非原子写 | 普通 store 落在 L1，之后才慢慢到 L2，没有「立即可见」保证。**只有在 L2 执行的原子操作才有这个性质** |
| consumer 的 `__threadfence()` | 编译器/硬件可能把 `data` 的 load 提到 `flag` 的 load 之前 |
| 只加 `volatile` 不加 fence | 绕过了 L1，但**没排任何序**——`volatile` 在 CUDA 里是「绕 L1」限定符，不是内存序标注 |

**③ PTX 的 scope——GPU 比 CPU 多出来的一整个维度**

| PTX | 语义 | 谁执行 |
|---|---|---|
| `ld.relaxed.gpu` / `st.relaxed.gpu` | 同 relaxed | 普通 LSU |
| `ld.acquire.gpu` | 之后的访存不能提前 | LSU，**不需要全 fence** |
| `st.release.gpu` | 之前的访存不能推后 | LSU |
| `fence.acq_rel.gpu` / `fence.sc.gpu` | 独立 fence | 更贵 |
| `atom.add.release.gpu` | 原子 + release | **L2** |

**scope 取值**：`.cta`（一个 block）、`.cluster`（thread block cluster，sm_90+）、`.gpu`（整个设备）、`.sys`（设备 + 主机 + 对端设备）。

scope 是 x86/ARM 的 C++ 内存模型里**没有**的概念（C++ 里要 `cuda::thread_scope` 才有）。选错是**两类严重程度不同**的 bug：

- **scope 太窄**（该 `.gpu` 却写 `.cta`）→ **不正确**。fence 不花钱，但也没跟别的 SM 排任何序。
- **scope 太宽**（该 `.gpu` 却写 `.sys`）→ **很慢**。`fence.sys` 比 `fence.gpu` 贵得多。

PTX 的一致性模型有正式规范（PTX ISA 的 Memory Consistency Model 一章；学术形式化见 Lustig et al., ASPLOS 2019）。它比 C++ 模型多出来的核心关系就是 **scope 限定的 happens-before**，以及一对操作要能建立因果必须「morally strong」（同地址、scope 匹配、不是双方都 relaxed）。硬件之所以能承受这么细的粒度，是因为它不需要真把流水线排空——它只需要追踪「哪些操作已经越过了这道 fence」。

**④ mbarrier：把 release/acquire 和一个可拆分的会合打包**

`bar.sync`（`__syncthreads()` 的底层）是**纯会合**：block 内**每个**线程都必须到达。它**表达不了**「搬运引擎搬完了」——引擎不是 block 里的线程，没法参与 `bar.sync` 的计数。

`mbarrier`（PTX）是一个**活在 shared memory 里的屏障对象**，把 arrive 和 wait 拆成两个可以**不同角色、不同时间**执行的操作：

- producer（或 TMA 引擎）搬完数据后 `mbarrier.arrive`；
- consumer `mbarrier.try_wait(phase)`，phase 翻转后去读数据。

关键：**phase 翻转本身携带 release-acquire 语义**——在 arrive 之前完成的写，对所有观察到 phase 翻转的线程可见。所以 `mbarrier` 不是新概念，它**就是**上面那对 release/acquire，外加一个会合；只不过配对的两端可以由**不同角色**发出，包括由 TMA 引擎在 `cp.async.bulk` 完成时自动发出的 arrive。

这就是 `bar.sync` vs `mbarrier` 的真正差别：**不在「同步不同步」，而在参与者能不能被拆开。** `bar.sync` 要求全员到齐；`mbarrier` 让「谁到达」和「谁等待」解耦——这正是异步搬运引擎能融进流水线的前提。

**⑤ 跨卡：硬件层面根本没有一致性**

- **L2 是 per-GPU 的**，一致性域止步于单卡。
- **跨 GPU（NVLink / PCIe）不是 cache-coherent 的**，硬件不提供。`fence.sys`（`__threadfence_system()`）给的是**顺序**，但顺序 ≠ 一致性——还得保证那块内存在对端可映射（peer-mapped）且平台支持。两件事都要，缺一不可。
- **唯一例外是 coherent NVLink**（Grace Hopper / GH200 这一代）：CPU 和 GPU 共享 L2、共享同一个一致性域。

所以「跨卡读写怎么保证正确」在 GPU 上的答案不是「靠协议」，而是「**显式搬运 + 显式 fence + 正确的 scope**」——和 CPU 多路系统靠 MOESI 硬件兜住，是两种不同的工程。

相关：[[engine#现代 DMA 引擎有哪些技术？]] · [[c2c-crosscard#跨卡 DMA 放在哪里？三条路径]] · [[academic-survey#尚未解决的问题（被反复点名）]]

### 自测

- [ ] GPU 的 L1 为什么不需要侦听？那跨 SM 通信靠什么保证看到新值？
- [ ] `volatile` 和 `__threadfence()` 各做了什么？只加 `volatile` 为什么还是错的？
- [ ] scope 写窄了和写宽了分别是什么性质的问题？
- [ ] `mbarrier` 凭什么能表达「搬运引擎搬完了」，而 `bar.sync` 不能？
- [ ] 跨卡为什么不能靠硬件一致性？`fence.sys` 给了什么、没给什么？
