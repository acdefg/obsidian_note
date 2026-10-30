---
tags: [arch, interrupt]
updated: 2026-10-24
---

# RISC-V 中断架构（AIA）

> 基础概念（MMIO / hart / MSI / CSR / domain / direct / 完整流程）见 [[interrupt-basics]]。

## AIA 是什么？为什么它要取代 PLIC？ #arch #interrupt

> **速查**
> - AIA = **AIA CSRs**（hart 本地中断扩展）+ **IMSIC**（每 hart 的 MSI 控制器）+ **APLIC**（有线中断控制器）
> - PLIC 的硬伤：全局共享寄存器、无 MSI、无 per-hart 虚拟化、源类型不可配、扩展性差
> - AIA 的 MSI 路径把外部中断从「拉一根线」变成「写一次内存」：地址选中断文件，数据低 11 位是 EIID
> - APLIC 可 direct（类 PLIC）或 MSI（转给 IMSIC）投递；两种模式可在不同 domain 共存
> - 结果：中断注入不再依赖全局 claim/complete，VS-file 让虚拟机中断可以硬件直投

### 正文

**先厘清三个词**

- **hart**：硬件线程（hardware thread），一个能独立取指执行的核内上下文；一个核可以有多个 hart。
- **MSI**（message-signaled interrupt，消息信号中断）：设备或控制器不再拉一根中断线，而是向一个约定地址写一个约定数据；目标控制器把这次写解释为「某个中断号挂起」。
- **EIID**（external interrupt identity，外部中断号）：IMSIC 里标识一个外部中断的 11 位编号，范围 1–2047；0 表示「没有中断」。

**PLIC 的结构性限制**

PLIC（Platform-Level Interrupt Controller）是 AIA 之前的平台中断控制器。它的模型是：

```text
device IRQ lines ──▶ PLIC ──▶ per-context external interrupt line ──▶ hart
                        │
                        ├─ 全局 priority 数组
                        ├─ 全局 pending 数组
                        ├─ 全局 enable 数组（按 context 复制）
                        └─ per-context claim/complete（MMIO）
```

问题不在「慢」，而在结构：

| 维度 | PLIC | AIA |
|---|---|---|
| 中断注入 | 一根电平线 + MMIO claim/complete | MSI 写内存；IMSIC 本地挂起 |
| 共享 | 全局 priority/pending 数组被所有 context 共享，M/S 无独立虚拟化隔离 | 每 hart 独立中断文件，M/S/VS 分开 |
| 虚拟化 |  guest 中断要 trap 到 hypervisor 模拟 | VS-file + guest index，硬件直投 |
| 源配置 | 基本只有电平 | edge rising/falling、level high/low 可配 |
| 扩展 | 全局仲裁，核多时 MMIO 争用明显 | 每 hart 本地仲裁，MSI 无共享锁 |

**AIA 的三层结构**

```text
 wired device ──▶ APLIC domain ──┬─ direct mode ──▶ hart external interrupt input
                                 │
                                 └─ MSI mode ──▶ write to IMSIC file ──▶ hart local ext irq
 device MSI ───────────────────────────────────▶ write to IMSIC file ──▶ hart local ext irq
```

- **AIA CSRs**：hart 侧新增的本地中断 CSR，负责「本 hart 怎么看见、怎么 claim 外部中断」，例如 `stopi`/`stopei`（S 模式）、`mtopei`/`mtopi`（M 模式），以及虚拟化用的 `mvien`/`mvip`、`hvictl`/`hviprio`。
- **IMSIC**：Incoming MSI Controller，每 hart 一个；内部是若干 **interrupt file**（中断文件），分别给 M 模式、S 模式、以及每个 guest 的 VS 模式使用。
- **APLIC**：Advanced Platform-Level Interrupt Controller，管理有线中断源；可以按 **domain** 分层，把源委派给子 domain。

**不是「所有平台都必须三件套」**

AIA 允许平台只实现其中一部分：例如只有 APLIC direct mode（替代 PLIC）而没有 IMSIC；或者有 IMSIC 但没有 APLIC。软件通过设备树/ACPI 和 ISA 扩展（Smaia/Ssaia）发现实际能力，不能假设三件套一定齐全。

> vault 里已有的 [[AIA]] 记录的是单核流水线里的中断响应流程（mepc/mcause/mstatus），属于「hart 收到中断之后怎么做」；本条讲的是「中断怎么送到 hart」。两者可以对照看。

### 自测

- [ ] 为什么 PLIC 在虚拟化场景下会频繁 trap，而 IMSIC 的 MSI 模式不会？
- [ ] AIA 的三个组成部分各自负责哪一段路径？哪一段是可选的？
- [ ] 如果平台只有 APLIC direct mode，hart 侧还需要 IMSIC 吗？

---

## IMSIC 的中断文件是怎么工作的？ #arch #interrupt

> **速查**
> - 每个 hart 一个 IMSIC，内含 M-file / S-file / 每个 guest 一个 VS-file；每个文件占一个 4 KiB 页
> - 页内**只有** `seteipnum_le`（0x000）和 `seteipnum_be`（0x004）两个 MSI 写端口，其余偏移保留
> - 控制面走间接 CSR：`eidelivery` 0x70、`eithreshold` 0x72、`eip0–63` 0x80–0xBF、`eie0–63` 0xC0–0xFF
> - **EIID 越小优先级越高**：1 最高、2047 最低、0 无效；`eithreshold=P` 时只放行 EIID < P
> - claim 用一条 `csrrw rd, stopei, x0`：写 `stopei` 的副作用清 pending，读本身不清

### 正文

**中断文件是什么**

一个 IMSIC 中断文件（interrupt file）就是「某个 hart 在某个特权级/某个 guest 下的一组 pending + enable 位」。它包含：

- 2048 个 pending 位（EIP）
- 2048 个 enable 位（EIE）
- 一个投递开关 `eidelivery`
- 一个优先级阈值 `eithreshold`

文件按 4 KiB 页对齐。M-file、S-file、以及每个 guest 的 VS-file 各占一页；S 与 guest 页通常连续排列为 `S, G1, G2, …`。

**访问路径：控制面是 CSR，数据面是内存写**

这是实现时最容易踩的坑：

| 访问方式 | 用途 | 关键点 |
|---|---|---|
| 间接 CSR `*iselect` / `*ireg` | 本 hart 读写自己的中断文件 | S 模式 `siselect/sireg`；M 模式 `miselect/mireg`；VS 模式 `vsiselect/vsireg` |
| MMIO 页偏移 0x000 / 0x004 | 其他 hart、设备、APLIC 发 MSI | 写数据 = EIID；读返回 0；其余页偏移保留 |
| `stopei` / `mtopei` | claim 当前最高优先级外部中断 | 写 side effect，不是普通 MMIO |

**寄存器表（间接 CSR 选择号）**

| 选择号 | 名称 | 作用 |
|---:|---|---|
| 0x70 | `eidelivery` | 0 = 关闭投递；1 = 打开；0x40000000 = 改为从 PLIC/APLIC direct 投递 |
| 0x72 | `eithreshold` | 优先级阈值；0 = 不设阈值；P≠0 时只放行 EIID < P |
| 0x80–0xBF | `eip0`–`eip63` | pending 位，每个 32 位寄存器管 32 个 EIID |
| 0xC0–0xFF | `eie0`–`eie63` | enable 位，与 EIP 一一对应 |

注意：**0x80–0xBF 是选择号，不是 4 KiB 页内的字节偏移**。选择号每个寄存器 +1；如果某个实现把它们映射成 MMIO，字节偏移要按 4 字节步进，不能照抄这些数字。

**XLEN=64 时只有偶数号存在**

- XLEN=32：`eip0`–`eip63` 全部存在，`eipk` 管 EIID `32k`–`32k+31`。
- XLEN=64：奇数号 `eip1, eip3, …, eip63` 不存在；选择奇数号并访问 `*ireg` 会触发 illegal-instruction（VS 模式下是 virtual-instruction）。偶数号 `eipk` 管 EIID `64k`–`64k+63`。

EIE 同理。RTL 实现时如果按 64 位 CSR 组织，就直接做 32 个 64 位寄存器，别做 64 个 32 位。

**投递条件**

一个中断文件向 hart 拉高外部中断信号，当且仅当：

```text
eidelivery = 1
AND 存在 i 使得 EIP[i] = 1 且 EIE[i] = 1
AND (eithreshold = 0 OR i < eithreshold)
```

满足后，hart 的 `mip.SEIP` / `sip.SEIP`（S 文件）或 `mip.MEIP`（M 文件）被置起。`stopi` 是只读的「当前最高优先级」视图，读它不会 claim。

**优先级方向**

IMSIC 没有单独的优先级寄存器，**EIID 本身就是优先级**：

- EIID = 1 最高，2047 最低。
- 多个 pending+enabled 时，选数值最小的 EIID。
- `eithreshold = P` 时，EIID ≥ P 的全部被挡住；所以 `eithreshold = 1` 等价于「全部关闭」。

**MSI 地址与数据**

一次 MSI 写由两部分组成：

```text
地址：IMSIC 基址 + (group_index, hart_index, guest_index 拼成的页号) << 12
      低 12 位必须为 0（页对齐）
数据：低 11 位 = EIID；其余位必须为 0（非 guest 场景）
```

字段宽度和位置由平台决定，设备树用这些属性描述：

- `riscv,guest-index-bits`（0–7，默认 0）
- `riscv,hart-index-bits`（0–15）
- `riscv,group-index-bits`（0–7，默认 0）
- `riscv,group-index-shift`（默认 24）

APLIC 在 MSI 模式下生成地址时，必须使用同一套几何；两边对不上，MSI 就会写到错误的中断文件，表现为「中断丢失」或「打到别的 hart」。

**claim / complete**

AIA 不允许「读 CSR 产生副作用」，所以 claim 改成「写 side effect」：

```asm
csrrw rd, stopei, x0    # rd = 当前最高优先级外部中断；写 0 触发 claim
```

- 读 `stopei` 返回 IID（bits 27:16）和 IPRIO（bits 7:0）。
- 写 `stopei` 时**写入值被忽略**；硬件看的是寄存器当前可读值，把对应 EIID 的 pending 位清掉。
- 如果当前值为 0，写无效。
- 读和写必须在同一条 CSR 指令里完成；分两次做，中间可能插入更高优先级中断，第二次写会清错对象。

**IPI 与虚拟化**

- **IPI**：SBI 的 `sbi_send_ipi` 在 AIA 平台上就是向目标 hart 的 IMSIC 页 0x000 写一个约定 EIID；不需要 PLIC 的 MMIO claim。
- **虚拟化**：每个 guest 一个 VS-file；guest 的 MSI 地址里带 guest index，硬件直接投到对应文件。KVM 可以选择硬件加速（VS-file 由硬件保存/恢复）或 trap 模拟（SW-file）。

### 自测

- [ ] 为什么 `eip0` 的选择号是 0x80，但不能把 0x80 直接当成 4 KiB 页内的字节偏移？
- [ ] XLEN=64 时，访问 `eip1` 会发生什么？RTL 应该实现多少个 64 位 pending 寄存器？
- [ ] `eithreshold = 1` 为什么等价于关闭全部投递？
- [ ] 为什么 claim 必须用 `csrrw` 一条指令，而不能先读再写？

---

## APLIC 的域和两种投递模式怎么选？ #arch #interrupt

> **速查**
> - APLIC 管理有线中断源；domain 分层，root 把源委派给 child，child 0 通常给 S 模式
> - 每个源可配 inactive / detached / edge rising/falling / level high/low
> - **direct mode**：APLIC 拉 hart 外部中断线，hart 读 IDC 的 `CLAIMI` 取中断（类 PLIC）
> - **MSI mode**：APLIC 把源转成 MSI 写 IMSIC，地址/数据由 `TARGET[i]` 决定
> - 优先级方向与 IMSIC 一致：**IPRIO 越小越高**；`ithreshold=P` 只放行 IPRIO < P

### 正文

**domain 结构**

APLIC 不是「一个大数组」，而是「一棵 domain 树」：

```text
root domain
├─ child 0  (通常是 S-mode domain)
├─ child 1  (另一个 OS / 安全域)
└─ child 2  ...
```

- 所有源先接到 root domain。
- root 通过 `SOURCECFG[i].D`（delegate）把某个源委派给子 domain，`CHILD_INDEX` 指定子 domain 编号。
- 每个 domain 有自己的 MMIO 基址和一套寄存器；子 domain 只能看到被委派给它的源。
- 一个源在某个 domain 里要么 inactive、要么 detached、要么 direct/MSI 投递。

**每个 domain 的关键寄存器**

| 寄存器 | 偏移（domain 内） | 作用 |
|---|---:|---|
| `DOMAINCFG` | 0x0000 | bit 8 `IE` 全局投递使能；bit 2 `DM`（0=direct，1=MSI）；bit 0 `BE` 端序 |
| `SOURCECFG[i]` | 0x0004 + 4×(i−1) | `D`/`CHILD_INDEX` 委派；否则 `SM` 源模式 |
| `TARGET[i]` | 0x3004 + 4×(i−1) | direct：目标 hart + 优先级；MSI：hart/guest/EIID |
| `SETIPNUM` / `CLRIPNUM` | 0x1CDC 附近 | 软件置/清 pending |
| `SETIENUM` / `CLRIENUM` | 0x1F00 附近 | 软件置/清 enable |
| `GENMSI` | 0x3000 附近 | 软件主动生成一条 MSI（Busy 位指示进行中） |

`SOURCECFG[i].SM` 的编码：

| SM | 含义 |
|---:|---|
| 0 | inactive（源不参与） |
| 1 | detached（源被检测但暂不投递） |
| 4 | edge rising（上升沿） |
| 5 | edge falling（下降沿） |
| 6 | level high（高电平） |
| 7 | level low（低电平） |

**direct mode**

- `DOMAINCFG.DM = 0`。
- APLIC 为每个目标 hart 维护一个 **IDC**（interrupt delivery controller）块，通常 32 字节：

| IDC 偏移 | 寄存器 | 作用 |
|---:|---|---|
| 0x00 | `IDELIVERY` | 该 hart 的投递开关 |
| 0x04 | `IFORCE` | 强制投递一个指定源（调试/软件触发） |
| 0x08 | `ITHRESHOLD` | 优先级阈值 |
| 0x18 | `TOPI` | 只读，当前最高优先级中断 |
| 0x1C | `CLAIMI` | 读 = claim，返回最高优先级并清 pending（如果可能） |

- hart 侧只看到一根外部中断线；进入 handler 后读 `CLAIMI` 取源号，循环读到 0。
- **level 源的 pending 就是整流后输入电平的副本**：输入高就 pending，claim 也清不掉；只有输入变低才不 pending。所以 level 源在 handler 返回前必须让设备 deassert，否则会立刻重进。
- **edge 源的 pending 由 gateway 置位，claim 可以清掉**；清掉后要等下一个边沿。
- `CLAIMI` 没有对应的 complete 写操作；completion 的语义由源类型决定，不像 PLIC 那样统一写 claim/complete。

**MSI mode**

- `DOMAINCFG.DM = 1`。
- 每个源有一个 `TARGET[i]`，编码目标 hart、guest、EIID（具体位域随实现/spec 版本略有差异，常见布局是 EIID bits 10:0、guest bits 17:12、hart bits 31:18）。
- 源变 pending 且 enable 后，APLIC 生成一次 MSI 写，把 EIID 送到目标 IMSIC 文件。
- 没有 per-hart 中断线；也不需要 handler 里 MMIO claim（claim 在 IMSIC 的 `stopei`）。
- `GENMSI` 允许软件直接生成一条 MSI，用于 IPI 或调试。

**direct vs MSI 的选择**

| 维度 | direct | MSI |
|---|---|---|
| 硬件成本 | 低，类 PLIC | 需要 IMSIC + MSI 生成逻辑 |
| 中断延迟 | 一根线 + MMIO claim | MSI 写 + IMSIC 本地仲裁 |
| 虚拟化 | guest 中断通常要 trap | guest index 直投，无 trap |
| 亲和性 | 可按源改目标 hart | `TARGET[i]` 固定，改亲和性要重配 |
| 适用 | 简单 MCU、无 IMSIC、类 PLIC 替换 | Linux/虚拟化平台、多 hart、MSI 设备 |

一个平台可以同时有 direct domain 和 MSI domain（例如 M 模式用 direct，S 模式用 MSI），但**同一个 domain 内 DM 是全局的**，不能逐源混用。

**优先级**

APLIC 的优先级是每个源的 `IPRIO`，位宽 `IPRIOLEN`（1–8 位）：

- `IPRIO` 取值 1 到 2^IPRIOLEN − 1；0 不允许。
- **数值越小优先级越高**；同优先级时源号小的优先。
- `ITHRESHOLD = P` 时，只有 `IPRIO < P` 的源能参与投递；`P = 0` 不设阈值。
- 很多实现把 `IPRIO` 数组做成只读 0（即所有源同优先级），软件不能假设一定可配。

> 注意：APLIC 的 IPRIO 与 IMSIC 的 EIID 是两套优先级。MSI 模式下，APLIC 选出一个源后，IMSIC 只按 EIID 再排一次。如果希望「APLIC 里更高优先级的源在 IMSIC 里也更高」，就要让 EIID 的分配反映优先级，或者使用实现提供的优先级传递机制。

### 自测

- [ ] root domain 和 child domain 之间，源是怎么「移动」的？
- [ ] 为什么 level 源在 direct mode 下 claim 之后还可能立刻再触发？
- [ ] MSI 模式下，`TARGET[i]` 和 IMSIC 的 MSI 地址几何为什么必须一致？
- [ ] APLIC 的 IPRIO 与 IMSIC 的 EIID，哪个数值大表示优先级高？

---

## AIA 的软件栈怎么分工？ #kernel #survey

> **速查**
> - Linux 6.10 起合入 IMSIC/APLIC irqchip 驱动；DT bindings 是软硬件接口契约
> - OpenSBI v1.2 起支持 AIA 1.0：M 模式初始化 IMSIC/APLIC，并把 S 域委派给 S 模式
> - QEMU `virt`：`aia=aplic` / `aia=aplic-imsic` / `aia-guests=N`；Linux 测试基线 QEMU 7.2+
> - KVM RISC-V：IMSIC guest file 可硬件加速（VS-file）或 trap 模拟；APLIC 只支持 MSI mode
> - Spike 有 `imsic_t` / `aplic_t` 参考模型；Zephyr、Xen 也在补 AIA 驱动

### 正文

**分层**

```text
┌─────────────────────────────────────────────┐
│ 应用 / 驱动：request_irq、PCI MSI、IPI       │
├─────────────────────────────────────────────┤
│ Linux irqchip：IMSIC driver / APLIC driver   │
│  - 建立 MSI domain、irq domain、亲和性        │
├─────────────────────────────────────────────┤
│ OpenSBI（M 模式）：初始化 APLIC/IMSIC、委派   │
├─────────────────────────────────────────────┤
│ 硬件：AIA CSRs + IMSIC + APLIC               │
└─────────────────────────────────────────────┘
```

**Linux**

- 合入版本：Linux 6.10 起。
- 关键文件：
  - `drivers/irqchip/irq-riscv-imsic.c`（公共）
  - `drivers/irqchip/irq-riscv-imsic-early.c`（early driver，负责 IPI 和本地状态）
  - `drivers/irqchip/irq-riscv-imsic-platform.c`（platform MSI domain）
  - `drivers/irqchip/irq-riscv-aplic.c`
  - `drivers/irqchip/irq-riscv-aplic-direct.c`
  - `drivers/irqchip/irq-riscv-aplic-msi.c`
- DT bindings：`riscv,imsics.yaml`、`riscv,aplic.yaml`。这两个 YAML 是「硬件寄存器/地址几何」的正式契约，写 RTL 时应该对照它们定义参数。
- 驱动职责：探测 IMSIC/APLIC；建立 MSI domain；把 APLIC 的源映射成 Linux irq；处理亲和性、热插拔、IPI。

**OpenSBI**

- v1.1 支持 AIA v0.3.0 草案；v1.2 支持 AIA v1.0。
- 关键文件：`lib/utils/irqchip/imsic.c`、`lib/utils/irqchip/aplic.c`。
- M 模式职责：
  - 从设备树发现 IMSIC/APLIC 的地址和几何。
  - 初始化 IMSIC（清 pending/enable、设置 `eidelivery`/`eithreshold`）。
  - 初始化 APLIC root domain，把 S 模式需要的源委派给 child 0。
  - 配置 `mideleg`/`medeleg` 和 AIA CSR，让 S 模式能直接处理外部中断。
- 2026 年仍有「APLIC/IMSIC 接入 `sbi_irqchip` framework」的补丁在评审，说明上游 M 模式实现还在收敛，读代码时要注意版本差异。

**QEMU**

- `virt` 机器选项：
  - `aia=none`：默认，使用 SiFive PLIC。
  - `aia=aplic`：只加 APLIC（direct）。
  - `aia=aplic-imsic`：APLIC + 每 CPU 一个 IMSIC。
  - `aia-guests=N`：每个 hart 模拟 N 个 VS-level IMSIC 页。
- TCG：APLIC/IMSIC 全部用户态模拟。
- KVM：默认使用内核 irqchip；`kernel-irqchip=split` 时 S 模式 APLIC 用户态模拟、IMSIC 走内核；`kernel-irqchip=off` 时两者都用户态模拟。
- 用途：在没有 RTL 的情况下验证 OpenSBI/Linux/KVM 软件栈；也可以作为 RTL 的对拍参考。

**KVM RISC-V**

- in-kernel AIA irqchip；IMSIC guest file 有两种模式：
  - **HWACCEL**：guest 直接使用硬件 VS-file，上下文切换由硬件/内核配合。
  - **EMUL**：guest 访问被 trap，由 KVM 用软件文件模拟。
  - **AUTO**：按硬件能力选择。
- APLIC 在 KVM 里只支持 MSI mode；direct mode 的 guest 中断需要别的模拟路径。
- guest AIA 状态通过 `ONE_REG` 接口保存/恢复；前提是 host 暴露 Ssaia。

**Spike**

- `imsic_t` / `imsic_file_t`：M/S/VS 文件、EIP/EIE、MSI 写、`stopei` 语义。
- `aplic_t` / `aplic_m_t` / `aplic_s_t`：domain、源模式、direct/MSI。
- ISA 字符串里出现 `Smaia`/`Ssaia` 时创建 IMSIC。
- 用途：指令级参考模型，适合做 RTL 的逐指令对拍。

**其他**

- Zephyr 4.x 有 APLIC direct/MSI 和 IMSIC 驱动，适合裸机/RTOS 场景。
- Xen 在 2025–2026 年补 RISC-V AIA 的虚拟 APLIC/IMSIC 模拟，说明虚拟化栈仍在完善。

**启动时序（简化）**

```text
reset
  → OpenSBI: 读 DT，初始化 IMSIC（清状态、设 eidelivery/eithreshold）
  → OpenSBI: 初始化 APLIC root，委派 S 源到 child 0
  → OpenSBI: 设置 mideleg/medeleg，进入 S 模式
  → Linux: 探测 imsic/aplic DT 节点，建立 irq domain / MSI domain
  → Linux: 驱动 request_irq / pci_alloc_irq_vectors
  → 中断：设备 → APLIC → MSI → IMSIC → hart → Linux handler
```

### 自测

- [ ] OpenSBI 和 Linux 在 AIA 初始化里各做什么？为什么不能让 Linux 直接初始化 APLIC root？
- [ ] `aia=aplic` 与 `aia=aplic-imsic` 的设备树和中断路径差在哪？
- [ ] KVM 的 HWACCEL 与 EMUL 两种 IMSIC guest file 模式，各自依赖什么硬件能力？
- [ ] 为什么说 DT bindings 是 RTL 的接口契约？

---

## 有哪些开源 AIA RTL 实现？完成度如何？ #hardware #survey

> **速查**
> - `zero-day-labs/riscv-aia`：SystemVerilog，AIA 1.0，APLIC-only / Distributed / Embedded 三种形态，cocotb 验证
> - `OpenXiangShan/ChiselAIA`：Chisel，IMSIC + APLIC，但 APLIC 只支持 MSI mode，direct mode 未实现
> - `OpenXiangShan/OpenAIA`：Verilog，只有 IMSIC，没有 APLIC；XiangShan 以 submodule 使用
> - Minho/CVA6 IP 就是 zero-day-labs 那条线：APLIC 约 2.1k SLoC、IMSIC 约 0.3k SLoC，已在 CVA6 SoC 上跑 OpenSBI/Linux/Bao
> - 共同空白：APLIC direct + MSI 双模、完整虚拟化、嵌入式低延迟路径，仍有实现空间

### 正文

**横向对比**

| 项目 | 语言 | IMSIC | APLIC | 投递模式 | 验证 | 许可/备注 |
|---|---|---|---|---|---|---|
| `zero-day-labs/riscv-aia` | SystemVerilog | ✅ Vanilla/Island/Embedded | ✅ APLIC-only/Distributed/Embedded | direct + MSI | cocotb | Apache-2.0 WITH SHL-2.1；AIA 1.0 |
| `OpenXiangShan/ChiselAIA` | Chisel | ✅ | ✅ | 仅 MSI（DM=1） | `test/*/main.py` | direct mode 未实现 |
| `OpenXiangShan/OpenAIA` | Verilog | ✅ | ❌ | — | XiangShan 集成 | 只做 IMSIC |
| Minho / CVA6 AIA IP | SystemVerilog | ✅ | ✅ | direct + MSI | OpenSBI/Linux/Bao | 即 zero-day-labs 前身；AIA 1.0-RC2 |
| ETH Cheshire 项目 | — | ✅ | ✅ | direct + MSI | 裸机 + Linux | 1S 学生项目，可作集成参考 |

**zero-day-labs/riscv-aia（最完整的开源 RTL）**

- 三种 AIA 形态：
  - **APLIC-only**：只有 APLIC，适合替代 PLIC 的简单 SoC。
  - **Distributed AIA**：APLIC + 每 hart 独立 IMSIC，IMSIC 放在 hart 旁边。
  - **Embedded AIA**：IMSIC 嵌在 APLIC 内，APLIC 直接通过内部接口给 IMSIC 发 pending/enable，适合嵌入式/混合关键性系统。
- 三种 IMSIC 微架构：
  - **Vanilla**：只服务 1 个 hart，放在 hart 旁边。
  - **Island**：多 hart 共用一个系统总线接口，多个配置接口。
  - **Embedded**：Island + 一条给 APLIC 用的额外通信接口。
- 参数集中在 package 文件（`aia_pkg.sv` / `imsic_pkg.sv`）：源数、hart 数、domain 数、优先级位宽等。
- 验证：cocotb testbench，覆盖 APLIC、IMSIC、组合 AIA。
- 集成参考：CVA6 64 位 SoC，替换原 PLIC；OpenSBI/Linux/Bao 验证。

**OpenXiangShan/ChiselAIA**

- Chisel 实现，包含 `IMSIC.scala` 和 `APLIC.scala`。
- APLIC 只支持 MSI 投递（`domaincfg.DM=1`），**direct mode 未实现**。
- 单元测试在 `test/*/main.py`。
- 适合作为「Chisel 生态 / XiangShan 集成」的起点；如果要做完整 APLIC，需要自己补 direct mode、IDC、level/edge gateway 细节。

**OpenXiangShan/OpenAIA**

- Verilog，只实现 IMSIC，不实现 APLIC。
- XiangShan 通过 `.gitmodules` 把它放在 `src/main/resources/aia`。
- XiangShan 用户文档说 AIA 1.0 CSRs 和 IMSIC 开源，但 APLIC 不在开源列表里（需联系团队）。
- 适合研究「高性能核怎么接 IMSIC」，不适合作为完整 APLIC 参考。

**Minho / CVA6 AIA IP**

- 与 zero-day-labs 同一团队（Universidade do Minho）。
- AIA 1.0-RC2 版本：APLIC 6 个模块约 2124 SLoC，IMSIC 2 个模块约 332 SLoC；`NR_INTP_FILES` 参数控制中断文件数。
- 集成方式：CVA6 SoC 原 PLIC 替换为 APLIC，再加 AIA core extensions + IMSIC；MSI 走 CVA6 系统总线。
- 验证：OpenSBI、Linux、Bao hypervisor。
- 提出的优化：APLIC Minimal（一个物理 domain 复用给多个逻辑 M/S domain）、IMSIC Island、IE-AIA（嵌入式/MCS 增强）。

**ETH Cheshire 项目**

- 目标：在 Cheshire RISC-V SoC 里集成 APLIC + 每 hart IMSIC，连接 CVA6；用标准 AIA 接口跑裸机测试和 Linux。
- 可选 ACLIC 低延迟路径，但保留 APLIC/IMSIC 作为标准路径。
- 属于学生项目（1S），适合看「SoC 集成」而不是「RTL 细节」。

**怎么选**

- 要一个**能直接综合、文档齐全、带 cocotb** 的起点：选 `zero-day-labs/riscv-aia`。
- 要在 **Chisel/XiangShan** 里改：选 `ChiselAIA`，但准备好自己补 direct mode。
- 只关心**高性能核的 IMSIC**：看 `OpenAIA`。
- 要**CVA6 SoC 集成案例**：看 Minho 论文 + Cheshire 项目。

**仍未封死的空白**

- APLIC **direct + MSI 双模**且可动态切换的完整 RTL 不多。
- 完整 **VS-file 虚拟化 + 中断优先级传递**（APLIC IPRIO → IMSIC EIID）实现很少。
- **嵌入式/混合关键性**的低延迟路径（ACLIC、Embedded AIA）还在早期。
- **形式化验证**（无丢失中断、claim 原子性、MSI 地址解码）在开源项目里基本空白。

### 自测

- [ ] zero-day-labs 的三种 AIA 形态和三种 IMSIC 微架构分别解决什么问题？
- [ ] ChiselAIA 缺 direct mode，会给「替代 PLIC」带来什么限制？
- [ ] 为什么 APLIC 的 IPRIO 到 IMSIC 的 EIID 映射是虚拟化场景的难点？
- [ ] 如果只做 IMSIC、不做 APLIC，能支持有线设备中断吗？

---

## 自己实现一个 AIA 控制器，应该怎么拆模块和验证？ #arch #hardware

> **速查**
> - 三块硬件：hart 侧 AIA CSR、IMSIC（文件阵列 + MSI 解码 + 本地仲裁）、APLIC（gateway + domain + IDC/MSI 生成）
> - 先定地址几何：hart/guest 位宽同时约束 IMSIC 页号、APLIC `TARGET`、MSI 地址，三者必须一致
> - 优先级模型：APLIC 用 IPRIO（越小越高），IMSIC 用 EIID（越小越高），MSI 模式下要保证映射后顺序
> - 验证阶梯：寄存器/网关单测 → QEMU/Spike 对拍 → OpenSBI+Linux 启动 → KVM guest
> - 最小可用路径：IMSIC + APLIC direct（无虚拟化），再补 MSI mode 和 VS-file

### 正文

**推荐的最小系统边界**

如果你是从零写一个开源 AIA 控制器，建议先明确「做哪一档」：

| 档位 | 包含 | 能跑什么 | 复杂度 |
|---|---|---|---|
| A. PLIC 替代 | APLIC direct only + hart 外部中断输入 | 裸机、RTOS、无 MSI 的 Linux | 低 |
| B. MSI 平台 | APLIC MSI + IMSIC（M/S file） | Linux + OpenSBI + IPI | 中 |
| C. 虚拟化平台 | B + VS-file + `hvictl`/`hviprio` + guest index | KVM guest、Bao | 高 |

多数开源项目从 B 开始，再补 C；A 适合先验证 APLIC gateway 和 IDC。

**模块拆分**

```text
aia_top
├─ aia_csr                    # hart 侧 CSR：mip/mie 扩展、stopi/stopei、mvien/mvip、hvictl/hviprio
├─ imsic_top
│  ├─ imsic_addr_decode       # MSI 地址 → group/hart/guest/file
│  ├─ imsic_file × N          # EIDELIVERY/EITHRESHOLD/EIP/EIE + 本地仲裁
│  ├─ imsic_msi_write         # 0x000/0x004 写端口，数据 → EIID
│  └─ imsic_delivery          # 生成 hart 外部中断信号
├─ aplic_top
│  ├─ aplic_domain × N        # DOMAINCFG/SOURCECFG/TARGET
│  ├─ aplic_gateway × M       # edge/level 检测、pending、enable
│  ├─ aplic_delegate          # root → child 的源委派
│  ├─ aplic_idc × H           # direct mode：IDELIVERY/ITHRESHOLD/TOPI/CLAIMI
│  └─ aplic_msi_gen           # MSI mode：地址/数据拼装、GENMSI
└─ bus_if                     # AXI4-Lite / AHB / 自定义寄存器总线
```

**先定五个参数**

1. **hart 数 H**：决定 IMSIC 文件数、APLIC IDC 数、地址几何位宽。
2. **每个 hart 的 guest 数 G**：决定 VS-file 数和 MSI 地址里的 guest 位宽；G=0 可以先不做虚拟化。
3. **EIID 位宽 k**：`log2(源数) ≤ k ≤ 11`；EIID 0 保留，最大 2047。
4. **APLIC 源数 M**：1–1023；决定 gateway 数组和 `SOURCECFG`/`TARGET` 大小。
5. **IPRIO 位宽**：1–8；如果做成只读 0，就相当于所有源同优先级，软件不能用阈值分级。

**地址几何：一处定义，三处使用**

```text
MSI 地址 = IMSIC_BASE + ((group << group_shift)
                         | (hart  << hart_shift)
                         | (guest << guest_shift)) << 12
```

同一个几何必须同时满足：

- IMSIC 的页号解码（哪个文件被写）。
- APLIC `TARGET[i]` 的字段编码（hart/guest/EIID）。
- 设备树 `riscv,*-index-bits` 的描述。

建议在 RTL 里用一个 package 定义 `HART_BITS`、`GUEST_BITS`、`GROUP_SHIFT`，然后让 IMSIC 解码和 APLIC `TARGET` 生成都引用它。不要在两处各写一遍位域。

**关键设计决策**

| 决策 | 选项 | 影响 |
|---|---|---|
| APLIC 模式 | direct / MSI / 双模 | 双模要两套投递路径和仲裁；direct 简单但虚拟化差 |
| IMSIC 文件组织 | 每 hart 独立 / Island / Embedded | Island 省总线接口；Embedded 省地址空间但耦合高 |
| 优先级 | 固定 EIID=源号 / 可配 IPRIO | 可配更灵活，但 MSI 模式下要处理两套优先级 |
| claim 实现 | `stopei` 写 side effect | 必须原子；读不能有副作用 |
| level 源 | pending 跟随输入 / 锁存 | 跟随输入符合 spec；锁存会丢边沿语义 |
| 端序 | 只支持 LE / LE+BE | `DOMAINCFG.BE`、`seteipnum_be` 都要处理 |

**验证阶梯**

1. **寄存器/单元级**
   - `eidelivery`/`eithreshold` 读写；EIP/EIE 置位/清零；XLEN=64 时奇数选择号非法。
   - `stopei` 的 claim：读返回最高优先级；写清对应 pending；值为 0 时写无效。
   - MSI 写端口：地址解码到正确文件；数据低 11 位截断；越界/保留偏移的行为。
   - APLIC gateway：edge rising/falling、level high/low；enable/disable；pending 语义。
   - APLIC delegation：root → child 的源移动；child 看不到未委派源。
   - APLIC IDC：`TOPI`/`CLAIMI` 的优先级和阈值；level 源 claim 后仍 pending。
2. **参考模型对拍**
   - Spike 的 `imsic_t`/`aplic_t` 做逐指令/逐寄存器对拍。
   - QEMU `aia=aplic-imsic` 做系统级对拍；用同一份 DT 和测试程序。
3. **固件/OS 启动**
   - OpenSBI v1.2+ 启动；确认 M 模式初始化、委派、IPI。
   - Linux 6.10+ 启动；确认 IMSIC/APLIC 驱动 probe、`/proc/interrupts` 里有外部中断。
   - 测试：UART 中断、定时器、`sbi_send_ipi`、PCI MSI（如果有）。
4. **虚拟化**
   - KVM guest + `aia-guests=N`；VS-file 上下文切换；guest IPI；`hvictl`/`hviprio`。
5. **压力/形式化**
   - 1023 个源同时 pending；所有 EIID 边界；阈值动态切换；hart 热插拔。
   - 形式化性质：claim 不丢中断、MSI 地址解码一一对应、level pending 与输入一致。

**建议的仓库结构**

```text
rtl/
  core/aia_csr.sv
  imsic/imsic_top.sv
  imsic/imsic_file.sv
  imsic/imsic_addr_decode.sv
  aplic/aplic_top.sv
  aplic/aplic_gateway.sv
  aplic/aplic_idc.sv
  aplic/aplic_msi_gen.sv
  bus/aia_axi_lite.sv
  aia_pkg.sv
tb/
  cocotb/            # 寄存器、gateway、claim、MSI 解码
  formal/            # claim 原子性、地址解码
sw/
  baremetal/         # 最小 MSI / direct 测试
  linux-dt/          # 设备树片段
docs/
  register-map.md
  integration.md
```

**里程碑**

| 阶段 | 交付 | 验收 |
|---|---|---|
| M1 | AIA CSR + 单文件 IMSIC + MSI 写 | 裸机软件能收到一条 MSI 并 claim |
| M2 | APLIC direct + gateway | 替换 PLIC，Linux 能启动并收 UART 中断 |
| M3 | 多 hart IMSIC + APLIC MSI | OpenSBI/Linux IPI、PCI MSI 正常 |
| M4 | VS-file + 虚拟化 CSR | KVM guest 能收中断、IPI、定时器 |
| M5 | 压力/形式化/性能 | 1023 源、延迟/吞吐数据、形式化通过 |

**最容易写错的六个点**

1. **把 0x80–0xBF 当 MMIO 字节偏移**：它们是间接 CSR 选择号；MMIO 页只有 0x000/0x004 两个 MSI 写端口。
2. **XLEN=64 还实现 64 个 32 位 EIP 寄存器**：应该实现 32 个 64 位寄存器，奇数选择号非法。
3. **优先级方向搞反**：IMSIC 的 EIID 和 APLIC 的 IPRIO 都是**越小越高**。
4. **claim 做成读副作用**：AIA 要求写 `stopei` 才清 pending；读不能清。
5. **level 源 claim 后强行清 pending**：direct mode 下 level pending 跟随输入，claim 清不掉。
6. **MSI 地址几何与 APLIC TARGET 不一致**：中断会静默打到错误文件，最难调。

### 自测

- [ ] 为什么地址几何要在 RTL package 里单点定义，而不能 IMSIC 和 APLIC 各写一套？
- [ ] M1 到 M4 的里程碑，各自的最小验收用例是什么？
- [ ] 形式化验证最值得证明的三条性质是什么？
- [ ] 如果只做 direct mode，哪些 AIA CSR 可以省略？哪些不能？

---

## 参考

- RISC-V AIA 规范（1.0）：`https://github.com/riscv/riscv-aia/releases/download/1.0/riscv-interrupts-1.0.pdf`
- AIA 规范 HTML 参考：`https://docs.riscv.org/reference/hardware/aia/`
- Linux RISC-V AIA 补丁系列（v16）：`https://lore.kernel.org/linux-riscv/20240307140307.646078-1-apatel@ventanamicro.com/`
- Linux 6.10 AIA 支持（queued）：`https://lore.kernel.org/all/20240307140307.646078-1-apatel@ventanamicro.com/`
- OpenSBI v1.2 release：`https://github.com/riscv-software-src/opensbi/releases`
- OpenSBI IMSIC 驱动：`https://github.com/riscv-software-src/opensbi/blob/master/lib/utils/irqchip/imsic.c`
- OpenSBI APLIC 驱动：`https://github.com/riscv-software-src/opensbi/blob/master/lib/utils/irqchip/aplic.c`
- QEMU RISC-V AIA 文档：`https://qemu-stsquad.readthedocs.io/en/docs-next/specs/riscv-aia.html`
- QEMU AIA 补丁（v6/v8）：`https://lore.kernel.org/qemu-devel/`
- zero-day-labs/riscv-aia：`https://github.com/zero-day-labs/riscv-aia`
- OpenXiangShan/ChiselAIA：`https://github.com/OpenXiangShan/ChiselAIA`
- OpenXiangShan/OpenAIA：`https://github.com/OpenXiangShan/OpenAIA`
- Minho AIA IP 海报（RISC-V Summit Europe 2023）：`https://riscv-europe.org/summit/2023/media/proceedings/posters/2023-06-08-Francisco-MARQU%C3%89S-DA-COSTA-abstract.pdf`
- Minho AIA 论文（IEEE Access）：`https://ieeexplore.ieee.org/document/10387321`
- ETH Cheshire AIA 项目：`https://iis-projects.ee.ethz.ch/index.php/A_Standards-Compliant_Advanced_Interrupt_Architecture_for_the_Cheshire_RISC-V_SoC_(1S)`
- Spike IMSIC/APLIC 模型（DeepWiki）：`https://deepwiki.com/riscv-software-src/riscv-isa-sim/2.8-interrupt-controllers-and-imsic`
- KVM Forum 2022 AIA 虚拟化：`https://kvm-forum.qemu.org/2022/AIA_Virtualization_in_KVM_RISCV_final.pdf`
