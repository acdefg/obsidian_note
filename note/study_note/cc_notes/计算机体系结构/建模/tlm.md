---
tags: [arch]
updated: 2026-10-27
---

# TLM 事务级建模

## TLM-2.0 的 LT 和 AT 有什么区别？ #arch

> **速查**
> - TLM 把引脚级信号换成"函数调用 + 事务对象"：一次事务一次调用，不逐拍翻转
> - **LT（loosely-timed）**：`b_transport` + 时间解耦，快，适合软件 bring-up / 架构探索
> - **AT（approximately-timed）**：`nb_transport_fw/bw` + 四相握手，慢，但能建模流水与争用
> - `tlm_generic_payload` 是统一的事务载体；**DMI** 让 initiator 拿到指针直接访存
> - TLM-2.0 由 OSCI 制定、并入 IEEE 1666-2011；TLM-1.0 的 put/get 语义在各工具间并不统一

### 正文

**① 是什么：从"线"到"函数调用"**

引脚级模型里，一次读要拉地址、等数据、采样；TLM 把这一串压缩成一次函数调用：

```cpp
// initiator 侧
tlm::tlm_generic_payload trans;
trans.set_command(tlm::TLM_READ_COMMAND);
trans.set_address(addr);
trans.set_data_ptr(buf);
trans.set_data_length(128);
sc_time delay = SC_ZERO_TIME;

socket->b_transport(trans, delay);   // 一次调用 = 一次事务
```

`tlm_generic_payload` 里装的是：命令（读/写）、地址、数据指针、长度、字节使能、响应状态、扩展字段。
socket 在 elaboration 阶段绑定，运行时只剩函数调用——**这就是 TLM 比 RTL 快几个数量级的原因**。

**② 横向对比：LT 与 AT**

| | LT（loosely-timed） | AT（approximately-timed） |
|---|---|---|
| 主接口 | `b_transport`（阻塞） | `nb_transport_fw` / `nb_transport_bw`（非阻塞） |
| 时间怎么表达 | 调用返回时把延迟累加到 `delay` | 每个 phase 都带时间标注 |
| 事务粒度 | 一个事务一次调用 | 四相：`BEGIN_REQ → END_REQ → BEGIN_RESP → END_RESP` |
| 能建模什么 | 总延迟、带宽趋势 | 流水、乱序、争用、背靠背 |
| 时间解耦 | 支持（quantum） | 不支持，每事务同步 |
| 速度 | 快 | 慢（通常比 LT 慢一个数量级） |
| 典型用途 | 软件 bring-up、架构探索 | 性能分析、瓶颈定位 |

一句话：**LT 只回答"这次访问多慢"，AT 还能回答"两次访问能不能重叠"**。

**③ 时间解耦（Temporal Decoupling）：quantum 是精度与速度的旋钮**

**是什么**：一种仿真时间协调机制——initiator 在**局部时间**里自己往前走，不必每笔事务都回到
**全局仿真时间**同步。它只用在 LT；AT 每个 phase 都要和全局时间对齐，没有解耦。

| | 局部时间（local time） | 全局时间（global time） |
|---|---|---|
| 谁维护 | initiator 自己 | 仿真内核 |
| 怎么前进 | 每笔事务把 `delay` 累加进来 | 只在调度器第 4 步推进 |
| 约束 | 不能超过 `global + quantum` | 权威值 |

**分工**：target 只负责把延迟写进 `delay` 参数；**要不要 `wait()`、什么时候 sync，由 initiator 决定**。
这就是解耦的契约——把"记时间"和"等时间"拆开。

**什么时候必须同步**（否则会和其他模块对不上）：

1. 局部时间攒够一个 quantum；
2. 下一步要等真实时间（`wait(t)` 等定时事件）；
3. 与另一个模块交换数据、需要对齐时间点之前。

LT 里 initiator 不必每笔事务都 `wait()`，而是把延迟攒在本地，攒够一个 quantum 再同步：

```
1 GHz 模型，每 10 ns 一次事务 → 1e8 次事务/秒
quantum = 1 μs → 每 100 次事务才 sync 一次
上下文切换：1e8 → 1e6 次/秒，少 100 倍
```

代价是**误差上界 = quantum**：initiator 可能已经"跑到"未来 1 μs，而其他模块还停在当前时间。
quantum 调大 → 快、但不准；调小 → 准、但慢。`tlm_utils::tlm_quantumkeeper` 就是干这个的。

最小用法就三行：

```cpp
tlm::tlm_quantumkeeper qk;        // 跟着全局 quantum 走
qk.inc(delay);                    // 记账：局部时间 += 本次事务延迟
if (qk.need_sync()) qk.sync();    // 越过 quantum 才真正 wait()，与全局时间对齐
```

`inc()` 只加局部时间、**不** `wait`；`need_sync()` 判断是否越过边界；`sync()` 才真正消耗仿真时间。
把这三行的调用点挪一挪，就是在精度和速度之间调档。

**④ DMI：绕开 socket 直接访存**

`get_direct_mem_ptr()` 让 target 返回一段内存的**主机指针**，initiator 之后直接读写，不再走 socket。
映射变化时 target 调 `invalidate_direct_mem_ptr()`，initiator 必须丢掉旧指针。

- 适合：DDR / SRAM 这类"大块、无副作用"的存储模型；
- 不适合：有寄存副作用、需要计时的外设——直接访存会把时序和副作用一起绕过。

**⑤ 怎么用：五个坑**

1. `b_transport` 里不 `wait(delay)` → 仿真时间永远不动，模型变成零延迟。
2. LT 的 target 里等外部事件（阻塞等待）→ 时间解耦失效，甚至死锁。
3. AT 只实现 `nb_transport_fw`、不实现 `nb_transport_bw` → 四相走不完，响应回不去。
4. `transport_dbg` 里消耗时间 → 调试路径污染仿真时间（规范要求它零时间）。
5. 不检查 `payload.get_response_status()` → 错误被静默吞掉，后面全对不上。

相关：[[systemc-kernel#delta cycle 是什么？为什么 write() 之后读不到新值？]]

### 自测
- [ ] LT 和 AT 各自能回答什么问题？为什么 AT 更慢？
- [ ] quantum 调大调小分别牺牲什么？误差上界由谁决定？
- [ ] DMI 适合什么模型、不适合什么模型？失效通知由谁发？
- [ ] `b_transport` 里忘了 `wait(delay)` 会发生什么？
