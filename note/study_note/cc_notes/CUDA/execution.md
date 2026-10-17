---
tags: [cuda, execution]
updated: 2026-10-11
---

# CUDA 执行与内存层级

## CUDA 的内存层级是什么？五块物理存储和六个地址空间怎么对应？ #cuda #memory-hierarchy #arch #occupancy

> **速查**
> - 「层级」是两轴：**在不在片上**（register / shared / L1 / L2 vs DRAM） × **谁是主人**（线程 / block / grid / 应用）
> - **L1 和 shared memory 是同一块 SRAM 的两种切法**，carveout 由软件在运行期可调
> - 64K × 32-bit 寄存器 / SM = 256 KB；**寄存器按 warp 分配**，不按线程
> - **寄存器不可寻址**——不能取地址、不能用运行期计算的下标索引它。这一条是后面所有分配规则的根
> - **`.local` 叫 local，但物理上在板载 DRAM**——它只是"线程私有"，不是"离得近"

### 正文

**① 先拆掉"层级"这个说法**

「register ↔ shared ↔ global」这个梯子漏掉了两个正交维度，而实际困惑几乎全出在这两维上：

| 维度 | 取值 |
|---|---|
| **在不在片上** | 片上（register / shared / L1 / L2） vs 片外（DRAM） |
| **谁是主人** | 单线程 / 单个 block / 整个 grid / 整个应用 |

一次 global load 慢，**不是因为它"层级低"，而是因为它在片外**。shared memory 之所以要显式声明、要 `__syncthreads()`，**不是因为它"在中间层"，而是因为它的主人是 block 而不是线程**。两轴分开，后面的规则就都是推论。

**② 物理上只有五块存储**

| 物理存储 | 位置 | 容量 | 主人 | 典型延迟 |
|---|---|---|---|---|
| Register file | SM 内 | 64K × 32-bit = 256 KB / SM | 单线程（按 warp 分配） | ~1 cycle（直接做指令操作数） |
| Shared memory | SM 内 | 与 L1 共用同一块 SRAM | 整个 block | ~20–30 cycle |
| L1 cache | SM 内 | 同上，carveout 可配 | SM 私有 | ~30 cycle |
| L2 cache | 卡上 | 几十 MB | 整卡共享，**一致性点** | ~200 cycle |
| DRAM | 板载 | 设备显存 | 整卡 | ~400–600 cycle |

> 具体容量随架构变：数据中心卡上 L1+shared 合并容量明显更大（A100 192 KB、H100 256 KB 每 SM）。寄存器文件 64K/SM 这一条从 Fermi 起就没变过。

**③ 四个主要存储的正面对比**

| 特性 | Register | Shared Memory | Local Memory | Global Memory |
| ---- | ---------- | ------------- | -------------------- | --------------- |
| 所在位置 | SM 内的寄存器文件 | SM 片上 SRAM | 显存 DRAM（经 L1/L2 缓存） | 显存 DRAM |
| 生命周期 | 线程 | CTA | 线程 | 程序/主机管理 |
| 访问权限 | 线程私有 | CTA 内共享 | 线程私有 | 全 grid + 主机 |
| 延迟 | ~1 cycle | ~20-30 cycles | ~400+ cycles（未命中 L1） | ~400-600 cycles |

> **CTA**（Cooperative Thread Array）= thread block，就是 `blockIdx` 指的那个单位；同一个 CTA 内的线程可以用 `__syncthreads()` 会合。
>
> Local 那格的「未命中 L1」要当真：**命中 L1 时它也是 ~30 cycle**，和 shared 同一个量级。所以 local 慢不慢，取决于访问模式能不能把它留在 L1，而不是它"天生慢"。

**④ 三个必须说清的点**

**a) L1 和 shared memory 是同一块 SRAM 的两种切法**，不是两块硬件。有一个运行期可配的 **carveout**（切分）决定多大归 shared、多大归 L1。
- **硬件**：一块物理 SRAM 阵列。
- **软件**：`cudaFuncSetAttribute(..., cudaFuncAttributePreferredSharedMemoryCarveout, ...)` 决定怎么切。
这是"硬件给一块阵列、软件决定怎么切"的干净例子。

**b) register file 不是"内存"，因为寄存器不可寻址。**
指令编码里寄存器的编号**直接写在指令里**，硬件没有"按地址找寄存器"这条通路。所以一个值一旦被要求可寻址（取地址、或用一个运行期算出来的下标索引），它**必须离开寄存器**。这一条推出下一整条笔记的内容。

**c) 寄存器按 warp 分配，不按线程。** 一个 warp 32 条线程拿一样多的寄存器，占用是 `每线程寄存器数 × 32`，向上取整到分配粒度。

**⑤ 占用率的算术是能手算的**

每 SM 64K 个寄存器。但**还要被「每 SM 最多多少线程」卡住，而这个数按架构不同**——实测 RTX 5060（sm_120）是 **1536 线程 = 48 warp**，不是常被引用的 2048 / 64。

按 sm_120 算：

| 每线程寄存器数 | 每 warp 占用 | 寄存器允许的 warp 数 | 被 48 warp 上限截断 | 占用率 |
|---|---|---|---|---|
| 32 | 32 × 32 = 1024 | 65536 / 1024 = 64 | 48 | 100% |
| 40 | 40 × 32 = 1280 | 65536 / 1280 = 51 | 48 | 100% |
| 43 | 43 × 32 = 1376 | 65536 / 1376 = 47 | 47 | 98% |
| 255（ptxas 顶格） | 255 × 32 = 8160 | 65536 / 8160 = 8 | 8 | 17% |

**分界点在 42.6 个寄存器**（`65536 ÷ 48 ÷ 32`）——43 个以上才开始掉占用率。所以「寄存器用得越多占用率越低」在 40 个以内**不成立**：48 warp 的天花板先到。

> 用 `cudaDeviceProp.maxThreadsPerMultiProcessor` 和 `regsPerMultiprocessor` 量自己卡上的真实值，别套别人的数字。

**⑥ 六个"地址空间"和物理落点**

地址空间由**声明位置**决定，不是数据类型决定：

| 声明位置 | 地址空间 | 物理上真正在哪 |
|---|---|---|
| 函数内局部变量（不取地址） | 寄存器 | SM 的 register file |
| 函数内局部变量（取地址 / 溢出） | `.local` | **DRAM**，经 L1/L2 缓存 |
| `__shared__` / `extern __shared__` | `.shared` | SM 的 L1/shared SRAM |
| `__constant__` | `.const` | DRAM + constant cache |
| 函数外 `__device__` | `.global` | DRAM |
| kernel 参数 | `.param`（常量 bank 0） | DRAM，经 constant cache |

**为什么 local 不能放片上**：它**每线程一份、且编译期大小无上限**。一个线程 1 KB 局部数组，每 SM 2048 线程就是 2 MB，塞不进那块 SRAM。硬件给的方案是：给它一个 DRAM 里的 per-thread 区域，靠 L1/L2 缓存它——快靠缓存，装得下靠 DRAM。

相关：[[synchronization#生产者-消费者为什么需要内存模型？relaxed / release / acquire 各保证什么？]]

### 自测

- [ ] 为什么说 global 慢的原因不是"层级低"？两轴框架怎么解释 shared memory 要显式声明？
- [ ] 一个 kernel 每线程用 40 个寄存器，每 SM 能常驻多少 warp？写出算式。
- [ ] 为什么寄存器不能像内存一样被寻址？这条限制推出什么后果？
- [ ] `.local` 到底在哪块物理存储上？为什么它不能放片上？

## CUDA 怎么给变量和栈分配空间？什么会让变量掉进 local memory？ #cuda #memory-hierarchy #arch

> **速查**
> - 除了 `cudaMalloc` 和动态 shared memory，**几乎没有一样是运行期分配的**——寄存器 / local / 常数 / 静态 shared 全在编译期定
> - 决定性判据**不是"动态下标"**，而是「**编译器能不能证明它不需要地址**」
> - 实测：`b[4]` + 任意运行期下标 → **0 字节栈帧**（编译器用选择链代掉了）；地址逃逸进 `__noinline__` → 栈帧恰好 **4×N**
> - `-Xptxas -v` 的栈帧 + spill 字节数是最快的体检；**`-O0` 下以上全部失效**，永远在 `-O3` 下判断
> - 栈帧住在 local memory（= DRAM）；只有**非内联**的函数调用才真的用栈

### 正文

**① 分配时机——CUDA 和 CPU 差别最大的地方**

| 空间 | **谁**决定 | **什么时候** | 大小从哪来 |
|---|---|---|---|
| 寄存器 | **ptxas**（编译器后端） | **编译期** | 寄存器分配的结果 |
| 静态 shared | **编译器**（`__shared__` 声明） | 编译期定大小 | block 上到 SM 时，硬件从 SRAM 切 |
| 动态 shared | **host 代码**（launch 第三个参数） | **内核启动时** | 同上；默认上限 48 KB，要更多必须显式 opt-in |
| local | **编译器**定大小 | 编译期定，运行期按 tid 算地址 | DRAM 里的 per-thread 区域 |
| global（动态） | **host 代码**（`cudaMalloc`） | **运行期** | DRAM |
| global（静态） | `__device__` 声明 | 加载期 | DRAM |
| 栈 | 编译器算帧布局 + `cudaLimitStackSize` | 上限在 launch 前设 | local memory |

kernel 里没有裸 `malloc` 意义上的堆——device 侧 `malloc` / `new` 走一个独立的 **device heap**（默认约 8 MB，`cudaDeviceSetLimit(cudaLimitMallocHeapSize, …)` 可调），本质是个 global memory 分配器。

**② 判定链**

```
某个值 V 需要一个地址吗？
├─ 不取地址，且下标要么是常量、要么范围可证 ──→ 寄存器
│   └─ 可证时编译器生成「选择链」（n 路选择），代价 ~n 条指令
├─ 取地址且指针逃逸（传进非内联函数 / 存进内存 / 下标范围不可证）──→ .local
│   └─ 选择链太贵时也退化成 .local（这就是"动态下标 → local"说法的来源）
└─ ptxas 寄存器不够用 ──→ .local（这叫 spill，和上面是两条不同的路径）
```

**③ 实测证据**（本机 CUDA 13.0，编译到 `sm_120`，`nvcc -O3 -Xptxas -v` 读栈帧）

| 代码 | 栈帧 | 真正发生了什么 |
|---|---|---|
| 纯标量 `float a`，不取地址 | **0 B** | 寄存器 |
| `float b[4]`，值全是编译期常量，`b[tid & 3]` | **0 B** | 常量不占寄存器 → **选择链** |
| `float b[64]`，`b[j]=out[j]`，`b[n]`（n 任意） | **0 B** | 数组被**完全消除**（`b[n]` 就是 `out[n]`）：SASS 里 64 条 `LDG`、**0 条 local 指令** |
| `float b[4]`，值来自 `out[]`，`b[n]`（n 任意） | **0 B** | 4 个值进 R8/R7/R6/R0，再用**选择链**挑 |
| `float b[4]`，地址传进 `__noinline__` 函数 | **16 B** | → local |
| `float b[8]` / `[16]` / `[32]`，地址逃逸 | **32 / 64 / 128 B** | → local，**恰好 4×N** |

教科书说「动态下标 → local memory」。**实测这条太粗**：第三行那个 `b[64]` 配任意下标，栈帧仍是 0——因为编译器把整个数组消掉了；第四行 `b[4]` 配任意下标也是 0——因为它用了选择链。真正让它掉进 local 的是**地址逃逸**，那时栈帧精确等于 `4×N`。

> **结论：不是数据类型决定位置，也不是"动态下标"本身决定，而是「编译器能不能证明它不需要地址」。** 选择链就是一个"不需要地址也能满足动态下标"的证明；它太贵时，编译器才退回 `.local`。

**④ 机制：动态下标是怎么变成地址的**

地址逃逸那条路的 SASS（`cuobjdump -sass`，sm_120）：

```
STL.128 [R1], R4        ; 把 b[0..3] 一条 128 位指令写进 local（编译器还顺手向量化了）
IMAD    R0, R3, 0x4, R0 ; local 地址 = 基址 + n*4   ← 动态下标变成了地址算术
LDL     R5, [R0]        ; 再按地址读回来
```

**`IMAD` 那一行就是全部机制**：动态下标本来是想索引寄存器，寄存器不能索引，所以编译器把它编译成「先算地址、再按地址读内存」。而那条内存，就是 DRAM。

**⑤ 栈**

- **栈帧住在 local memory 里——也就是 DRAM。** 上表那个 `16 B` 栈帧，就是 `float b[4]` 在 DRAM 里的位置。
- **只有非内联的函数调用才真的用栈。** nvcc 默认激进内联，绝大多数 `__device__` 函数没有自己的栈帧（实测里那个 `sink` 函数报的是 `0 bytes stack frame`，栈帧记在**调用者**头上）。要真的看到栈，得用 `__noinline__`。
- **上限 `cudaDeviceSetLimit(cudaLimitStackSize, N)`，是每线程的，必须在 launch 之前设。** 设小了报 `unspecified launch failure`（很隐晦）；设大了吃掉 local 容量、隐式占用 global 带宽。用 `cudaDeviceGetLimit` 读当前值。
- **别混淆另外两个独立的"栈"**：device 侧 `printf` 的格式化缓冲走 `cudaLimitPrintfFifoSize`；CDP（dynamic parallelism，内核里再启动内核）的 device runtime 栈走 `cudaLimitDevRuntimeSyncDepth` / `cudaLimitDevRuntimePendingLaunchCount`。

**⑥ 怎么自己查**

```bash
nvcc -arch=sm_120 -O3 -Xptxas -v -cubin -o x.cubin x.cu   # 看栈帧与 spill
cuobjdump -sass x.cubin                                    # 看指令
```

SASS 助记符直接对应空间，不用猜：

| SASS | 空间 |
|---|---|
| （寄存器直接做操作数） | register |
| `LDL` / `STL` | **local** |
| `LDS` / `STS` | shared |
| `LDG` / `STG` | global |
| `LDC` | constant |

`-Xptxas -v` 报告里 **`bytes spill stores/loads` 非零**，或者**栈帧非零而你没写递归**，就是有东西掉进 local 了。

> ⚠️ **`-O0` 下以上全部失效**——所有局部变量都进 local memory。调试版本慢几个数量级，一大半原因在这里。**永远在 `-O3` 下判断分配。**

### 自测

- [ ] 列出决定每个地址空间大小的角色与时机。哪几样是运行期才定的？
- [ ] `float b[64]` 配任意运行期下标，为什么栈帧可以是 0？编译器做了什么？
- [ ] 什么条件下数组一定会进 local memory？栈帧大小怎么算？
- [ ] 为什么 `sink` 函数自己报 0 字节栈帧，但调用它的 kernel 报了 8 字节？

## 什么样的数据该放哪个空间？ #cuda #memory-hierarchy #performance

> **速查**
> - `__constant__` 是**广播**不是缓存：全 warp 读**同一地址** 1 次；读**不同地址**最多串行 32 次 → 适合统一系数，**极不适合** LUT
> - 每线程查表读**不同项** → global + 手动搬进 shared memory
> - **数据类型本身不决定位置**，只通过"大小"间接影响（越大越容易溢出寄存器）
> - 三个例外：寄存器是 **32 位粒度**（`char` 占满一个）、向量类型是 **ISA 原生**、`__half` / `__nv_bfloat16` 两个挤一个

### 正文

**① 按用途对照**

| 数据 | 通常放哪 | 理由 |
|---|---|---|
| 累加器、循环变量、坐标 | 寄存器 | 不取地址、下标全静态 |
| `float4` / `int4`、小矩阵分块 | 寄存器（下标全静态时） | 向量类型是 ISA 原生，能生成 `LDG.128` 这类宽访存 |
| warp 内要共享的中间结果、归约暂存、tile | shared | 需要**跨线程**复用——这是 shared 存在的唯一理由 |
| 每线程的局部数组，地址逃逸 | local（= DRAM） | 编译器被迫 |
| **所有线程读同一个地址**的系数、滤波器 | `__constant__` | 见下节的广播 |
| **各线程读不同项**的查找表 | global + 手动搬进 shared | 见下节 |
| 模型权重、大数组 | global | 只有它装得下 |

**② 一个必踩的坑：`__constant__` 是广播，不是缓存**

constant cache 的行为是**广播**，不是"随机只读内存"：

- warp 里 32 条线程读**同一个**地址 → **1 次**访问，一个周期内广播给全 warp。
- warp 里 32 条线程读**不同**地址 → 每个**不同**地址各串行一次，最坏 **32 次**。

所以 `__constant__` 适合「所有线程用同一组系数」，**极不适合**「每线程查表不同项」的 LUT——后者要搬到 shared memory。这是把 `__constant__` 当"快速只读内存"用的经典翻车点。

**③ 数据类型本身的三个例外**

数据类型**不**决定位置，只通过大小间接影响（越大的结构体越容易溢出寄存器）。但有三条值得单独记：

- **寄存器文件是 32 位粒度的**，`char` / `short` / `bool` 通常各占**一整个** 32 位寄存器——别指望用 `char` 省寄存器。
- **向量类型是 ISA 原生的**，`float4` 不是"结构体"，它能生成单条 128 位访存指令（上面那条 `STL.128` 就是）。
- **`__half` / `__nv_bfloat16` 能两个挤进一个 32 位寄存器**——这是它们在 AI kernel 里真实的收益来源之一。

### 自测

- [ ] 为什么 `__constant__` 不适合做 LUT？全 warp 读同一地址和读不同地址分别几次访问？
- [ ] 一个 warp 里各线程都要查一张表的**不同**项，该放哪？怎么放？
- [ ] 用 `char` 存局部变量能省寄存器吗？为什么？
- [ ] `float4` 和 `struct {float a,b,c,d;}` 在寄存器/访存层面有什么实际差别？
