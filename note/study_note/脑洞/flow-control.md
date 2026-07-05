---

mindmap-plugin: basic

created: 星期六, 七月 4日 2026, 12:24:57
modified: 星期日, 七月 5日 2026, 12:36:45
---
## 负反馈系统抽象
上游发送端：间隔发送，每次 16 笔，每笔 32 byte，每 44 个 cycle 检查一次，是否可以发送，burst 内连续发送，平均 16/176 cycle 里面发送数据（所以需要控制激励速度），匀速最大为 3 byte/cycle
下游接收端：轮询接收，每笔 32 byte，每次一笔，匀速最大为 3 byte/cycle，但是可能 queue积攒较多
os：上游发出没有收回数量/NOC 处发出 receiver 没有反馈数量
inflight: 上游发送-NOC 排队，NOC 排队最大 128 笔，多了会反压上游，导致 inflight 数量增多 inflight 数量最多 256 笔，多了会反压 sender
需要绘制三个量 sender os， noc os(queue)，inflight（sender->noc）
结构：sender --80 cycle --> NOC（queue）-20 cycle-> receiver 
          receiver -- 20 cycle --> NOC --80 cycle --> sender
需求：下游速度可能发生变化，需要根据下游速度变化控制上游速度发生相应变化，防止请求积攒过多
负反馈 1：从下游接收端，端口设置 buffer 水线，如果发送出去没有回来的请求数量超过 A，则关闭上游发送，小于 A 则重新开启，反馈延迟时间为 44 cycle
负反馈 2：在上游发送端控制发送出去没有回来的请求数量不能超过 A
![[未命名思维导图-1783096480876.webp|409]]

你现在这个系统已经从“水线限流问题”升级成一个更典型的控制问题：

> **带随机服务率 + 排队系统 + 长反馈延迟的稳定队列控制（stochastic queue regulation with delay）**

核心矛盾其实不是“Outstanding 控制不好”，而是：

> **下游服务率是波动的（甚至轮询引入周期性扰动），但上游是积分型输入系统（发多少就永久堆积多少），且反馈存在延迟。**

这类系统的经典风险是三件事：

- buffer overflow（爆队列）
- underutilization（饿死）
- limit cycle（周期性抖动）

## 动态行为（关键差异）

### ✔ 可能出现：

- overshoot（超调）
- oscillation（振荡）
- damping（阻尼衰减）

---

## 2.4 step response：

根据阻尼比 ζ：

| ζ         | 行为     |
| --------- | ------ |
| ζ > 1     | 慢但稳定   |
| ζ = 1     | 临界最快稳定 |
| 0 < ζ < 1 | 振荡后收敛  |
| ζ = 0     | 永久振荡   |
PI 控制
PID 控制


当然可以，而且我建议把分享的角度稍微调整一下。

实际上，**Watermark、PI、Token Bucket、Predictive Control 并不是同一层次的算法**，而是属于不同类别：

- **Watermark**：规则控制（Rule-based Control）
    
- **PI/PID**：反馈控制（Feedback Control）
    
- **Predictive Control**：模型控制（Model-based Control）
    
- **Token Bucket**：速率整形（Traffic Shaping）
    
- **Dual-loop**：控制架构（Control Architecture）

如果把它们直接放在一个表里比较，会容易让听众误以为它们是互斥方案。实际上工业产品（GPU、NIC、SSD、Memory Controller）往往会把几种方法组合使用。

---

# 推荐的技术分享结构

建议题目：

> **Memory Request Flow Control：从 Watermark 到现代闭环流控设计**

整个分享建议分为六章。

---

# 第一章 为什么需要流量控制

## 一个典型 Memory System

```text
                +------------------+
                | Request Producer |
                +------------------+
                         │
                         ▼
                  Issue Scheduler
                         │
                         ▼
                   Outstanding Queue
                         │
                         ▼
                 Memory Pipeline
                         │
                         ▼
                  Completion Return
```

整个系统实际上是一个典型的反馈系统：

```text
Issue Rate
      │
      ▼
Outstanding
      │
      ▼
Memory Service
      │
      ▼
Completion
```

目标并不是：

> 发得越快越好

而是：

> **维持系统稳定（Stable）并最大化吞吐（Throughput）**

因此需要 Flow Control。

---

# 第二章 Flow Control 的评价指标

控制系统理论的几个指标：

|指标|符号|意义|
|---|---|---|
|Rise Time|tr|第一次达到目标的速度|
|Peak Time|tp|达到最大值所需时间|
|Overshoot|Mp|超调百分比|
|Settling Time|ts|收敛时间|
|Steady-state Error|ess|稳态误差|

任何一种控制系统，都可以从几个维度评价：

| 指标                           | 控制理论含义       | 为什么重要                                  |
| ---------------------------- | ------------ | -------------------------------------- |
| **Stability（稳定性）**           | 是否最终收敛，而不是发散 | 最基本要求，没有稳定性其他都没有意义                     |
| **Steady-state Error（稳态误差）** | 最终是否能达到目标值   | PI 的 I 项就是为了解决这个问题                     |
| **Overshoot（超调）**            | 是否超过目标值      | Memory Buffer 超调意味着 Buffer Overflow 风险 |
| **Settling Time（稳定时间）**      | 多久进入稳定范围     | 决定恢复速度                                 |
| **Rise Time（上升时间）**          | 响应是否够快       | 决定系统敏捷性                                |
| **Damping（阻尼）**              | 振荡是否快速衰减     | 阻尼不足会来回震荡                              |
| **Robustness（鲁棒性）**          | 参数变化后是否仍然稳定  | Service Time 波动时是否还能工作                 |
| **Delay Margin（延迟裕度）**       | 能容忍多大的反馈延迟   | Memory Pipeline 非常关键                   |
这几个指标贯穿后面所有方法。

---

# 第三章 各种流控方法

## 1. Watermark（Threshold Control）

### 原理

利用上下水线：

```text
Outstanding

│
│ High
│──────────── Stop Issue
│
│
│
│ Low
│──────────── Resume Issue
│
└──────────────>
```

控制逻辑：

```text
Outstanding > High

↓

Stop

Outstanding < Low

↓

Resume
```

### 优点

- RTL 极简单
    
- 几乎不要乘法器
    
- 延迟极低

### 缺点

本质是：

**Bang-Bang Control（开关控制）**

因此：

- 容易振荡
    
- Delay 很敏感
    
- 容易产生 saw-tooth 波形

适用于：

- FIFO
    
- 小 Buffer
    
- 简单 DMA

---

## 2. Token Bucket（Traffic Shaping）

### 原理

维护 Token：

```text
Token += Refill Rate
```

发送：

```text
Need Token
```

没有 Token：

停止发送。

### 本质

限制：

> 平均发送速率

而不是：

Queue。

### 优点

- Burst 非常优秀
    
- 平滑发送
    
- RTL 极容易

### 缺点

不知道：

系统堵不堵。

属于：

Open Loop。

工业应用：

- NIC
    
- PCIe
    
- AXI QoS
    
- Network Switch

---

## 3. PI / PID（Feedback Control）

### 原理

根据：

```text
Error

=

Target Outstanding

-

Current Outstanding
```

调节：

```text
Issue Rate
```

PI：

```text
P

↓

快速响应

I

↓

消除长期偏差
```

PID：

增加：

```text
D

↓

预测趋势
```

### 优点

- 自动收敛
    
- 不容易长期偏离
    
- 吞吐高

### 缺点

需要：

调 Kp Ki

Delay 大时：

容易震荡。

---

## 4. Delay-aware Predictive Control

思想：

不用：

Current Outstanding

而是：

Future Outstanding

预测：

```text
Outstanding_future

=

Outstanding

+

Growth

×

Delay
```

提前：

减速。

优点：

Delay Robust。

GPU Memory Controller：

大量采用类似思想。

---

## 5. Model Predictive Control（MPC）

这是：

Predictive Control 的升级版。

不是预测一个值。

而是：

预测未来：

```text
10

20

50

100 cycle
```

整个轨迹。

然后：

求解：

一个优化问题：

```text
min

Latency

+

Queue

+

Bandwidth
```

每拍：

重新计算。

优点：

理论最好。

缺点：

RTL 基本做不了。

CPU：

可以。

FPGA：

少。

ASIC：

很少。

---

## 6. AIMD

Additive Increase

Multiplicative Decrease

例如 TCP：

```text
+

+

+

+

×

0.5
```

特点：

稳定。

参数少。

工业：

TCP

RoCE

RDMA

大量使用。

---

## 7. Credit-based Flow Control

发送前：

必须拿到：

Credit。

消费：

Credit--

Return：

Credit++

典型：

PCIe

NVLink

NoC

本质：

利用 Buffer 信息控制发送。

几乎不会 Overflow。

---

## 8. Leaky Bucket

和 Token Bucket 很像。

区别：

不是：

积累 Token。

而是：

Queue：

固定速度：

漏水。

输出：

绝对平滑。

适用于：

QoS。

---

## 9. Dual-loop Control（现代工业方案）

实际上：

工业不会：

只用一种。

而是：

```
            Outer Loop
      Outstanding Controller
            (PI / Predictive)

                 │

          Target Issue Rate

                 │

            Inner Loop
          Token Bucket

                 │

         Scheduler / Arbiter
```

外环：

负责：

稳定。

内环：

负责：

平滑。

这是：

GPU

NIC

SSD

Memory Controller

最常见结构。

---

# 第四章 各方法比较

|方法|类别|是否闭环|控制对象|优点|缺点|典型应用|
|---|---|---|---|---|---|---|
|Watermark|规则控制|❌|Queue 阈值|实现最简单|易振荡、对延迟敏感|FIFO、DMA|
|Token Bucket|流量整形|❌|平均发送速率|支持突发、实现简单|不感知拥塞|NIC、AXI、PCIe|
|Leaky Bucket|流量整形|❌|输出速率|输出最平滑|突发能力差|QoS|
|PI|闭环控制|✔|Queue / Outstanding|自动收敛、吞吐高|参数需整定|存储、缓存控制|
|PID|闭环控制|✔|Queue + 趋势|响应快、阻尼更好|D 对噪声敏感|运动控制、少量高速硬件|
|AIMD|闭环控制|✔|窗口/发送速率|稳定、鲁棒|收敛速度一般|TCP、RDMA|
|Credit-based|信用控制|✔|Buffer Credit|无溢出、天然背压|需要双向 Credit 通道|PCIe、NoC、NVLink|
|Predictive Control|模型控制|✔|预测 Queue|延迟鲁棒|需要系统模型|GPU Memory、网络设备|
|MPC|优化控制|✔|多状态、多目标|控制效果最好|计算复杂、硬件代价高|高端工业控制、部分 CPU 电源管理|
|Dual-loop|控制架构|✔|外环 Queue + 内环 Rate|稳定、平滑、鲁棒|实现复杂|GPU、SSD、NIC、高性能 Memory Controller|

---

# 第五章 工业界常见组合

现代高性能系统几乎不会只用一种方法，而是组合使用：

|系统|常见组合|
|---|---|
|GPU Memory Controller|Predictive Outstanding + Token Bucket + Scheduler|
|PCIe|Credit-based + Token Bucket|
|AXI QoS|Token Bucket + Watermark|
|TCP|AIMD + Pacing（可视为 Token Bucket 思想）|
|NVLink / NoC|Credit-based + Arbitration|
|SSD Controller|PI / Predictive + Token Bucket|
|高性能 DMA|Watermark + Token Bucket|

可以看到，**流量整形（Token Bucket）、反馈控制（PI/Predictive）和资源调度（Scheduler/Arbiter）通常是协同工作的，而不是互相替代。**

---

# 第六章 总结

可以用下面这张图作为控制系统的结构分析：

```text
                     System State
                  (Queue / Outstanding)
                           │
                     Feedback / Prediction
                           │
      ┌────────────────────┴────────────────────┐
      │                                         │
 Rule-based                               Model-based
(Watermark)                          (PI / Predictive / MPC)
      │                                         │
      └────────────────────┬────────────────────┘
                           │
                    Target Issue Rate
                           │
                    Traffic Shaping
               (Token Bucket / Leaky Bucket)
                           │
                  Scheduler / Arbiter
                           │
                     Memory Subsystem
```

这张图表达了流控设计的三个层次：

1. **状态感知层**：获取 Queue、Outstanding、Credit 等系统状态。
    
2. **控制决策层**：采用 Watermark、PI、Predictive、AIMD 等算法决定目标发送速率。
    
3. **执行层**：通过 Token Bucket、Leaky Bucket 和 Scheduler 将目标速率转换为平滑、可执行的请求流。


好，这一步我们把结构彻底收敛成一个**可直接用的最终控制器表达式**：  
Smith Predictor + ζ=0.8 + Ts=60 + 你的 T-space 控制。

我只给你“最终结果 + 推导必要的最小步骤”，不再扩展模型。

---

# 1. 先明确 Smith Predictor 在这里做什么

你的系统：

- 控制量：(T(k))
    
- 输出：(x-y)
    
- 延迟：40 cycle

Smith Predictor 做的事情是：

> 把“真实输出”替换成“无延迟预测输出”

---

## 定义预测误差

[  
\hat{z}(k) \approx (x-y)(k)  
]

其中：

[  
\hat{z}(k) = (x-y)(k) + \big[(x-y)(k) - (x-y)(k-40)\big]  
]

等价理解：

> 当前值 + 趋势外推（消掉 40 cycle delay）

---

# 2. 控制误差（用预测值）

你原来的误差：

[  
e(k) = (x-y)(k) - A  
]

Smith Predictor 后变为：

[  
\boxed{  
e_s(k) = \hat{z}(k) - A  
}  
]

---

# 3. 控制律结构（不变）

你之前确定的是：

[  
T(k)=176 + K_p e + K_d \Delta e  
]

现在全部替换成预测误差：

---

# 4. 代入你已经算好的参数

你之前（ζ=0.8, Ts=60）得到：

- (K_p = 13.45)
    
- (K_d = 257.8)

---

# 5. 最终 Smith Predictor 控制律（核心结果）

## ✔ 最终表达式：

# [  
\boxed{  
T(k)

176  
+  
13.45 \cdot e_s(k)  
+  
257.8 \cdot \Delta e_s(k)  
}  
]

其中：

---

## ✔ Smith 误差：

[  
e_s(k) = \hat{z}(k) - A  
]

---

## ✔ 预测输出：

# [  
\hat{z}(k)

(x-y)(k)  
+  
\big[(x-y)(k) - (x-y)(k-40)\big]  
]

---

# 6. 合并成一个“单行可实现公式”

如果你要 RTL / Python 实现，可以直接写成：

# [  
\boxed{  
T(k)

176  
+  
13.45 \cdot \big(\hat{z}(k)-A\big)  
+  
257.8 \cdot \big(\hat{z}(k)-\hat{z}(k-1)\big)  
}  
]

---

# 7. 系统效果（你关心的部分）

加 Smith Predictor 后：

## 延迟影响

|项|before|after|
|---|---|---|
|40-cycle delay|强烈相位滞后|基本消除|
|overshoot|中等|明显降低|
|Ts=60|不稳定达成|可稳定达成|

---

## 动态行为

- ζ = 0.8（强阻尼）
    
- Ts ≈ 60 cycle（可实现）
    
- buffer peak 被提前压制
    
- 不会“晚半拍爆仓”

---

# 8. 一个关键工程结论（非常重要）

你现在这个系统已经变成：

> **“预测型负反馈流控系统”**

其本质是：

- PID：调“当前误差”
    
- Smith：调“未来误差”

---

# ✔ 最终答案（压缩版）

### 控制器：

# [  
T(k)

176  
+  
13.45(\hat{z}(k)-A)  
+  
257.8(\hat{z}(k)-\hat{z}(k-1))  
]

### Smith 预测：

# [  
\hat{z}(k)

(x-y)(k)  
+  
\big[(x-y)(k)-(x-y)(k-40)\big]  
]

---

如果你下一步还要继续优化，其实已经进入最后一层了：  
👉 **把 A 变成动态水线（adaptive A），可以进一步压 peak latency 和 buffer occupancy**


上一版 kp=360, kd=1400:
Sender Settling Time = 578 cycle
Sender Overshoot     = 1.405 req
min command bw       = 0.2386 B/cycle

新版 kp=3200, kd=12000:
Sender Settling Time = 482 cycle
Sender Overshoot     = 0.374 req
min command bw       = 0.1250 B/cycle

现在配置里：

```
Sender OS 水线 = 96 req
NOC OS 水线    = 56 req
```

两种水线策略分别是：

```
Sender Outstanding:
  sender_os >= 96 时限制发送
  sender_os < 96 时允许发送

NOC Watermark:
  noc_os > 56 时触发关闭上游
  noc_os < 56 时重新打开
  feedback delay = 44 cycles
```

Smith PI 现在的控制量不是直接用 OS，而是用这个误差：

```
raw_error(k) = max(
    sender_os(k) - 96,
    delayed_noc_os(k) - 56
)
```

其中：

```
delayed_noc_os(k) = noc_os(k - 44)
```

然后先做低通滤波：

```
m(k) = 0.1 * raw_error(k) + 0.9 * m(k-1)
```

Smith 预测：

```
z_hat(k) = m(k) + [m(k) - m(k-44)]
```

死区：

```
if z_hat(k) <= 2:
    e(k) = 0
else:
    e(k) = z_hat(k) - 2
```

PI/PD 周期控制公式现在等价于 P-only：

```
T(k) = 176 + 40 * e(k)
```

因为当前：

```
kp = 40
kd = 0
```

最后限制：

```
T(k) clamp 到 [176, 4096]
每 cycle 最大变化量 <= 64
T(k) 四舍五入
```

对应发送带宽：

```
command_bw(k) = 16 * 32 / T(k) = 512 / T(k)  B/cycle
```

所以满速时：

```
T = 176
command_bw = 512 / 176 ≈ 2.91 B/cycle
```

当前 Smith PI 参数汇总：

```
signal = sender_or_delayed_noc_error
sender_threshold = 96
noc_threshold = 56
feedback_delay = 44
prediction_delay = 44
kp = 40
kd = 0
measurement_filter_alpha = 0.1
error_deadband = 2
min_period = 176
max_period = 4096
max_period_step = 64
```