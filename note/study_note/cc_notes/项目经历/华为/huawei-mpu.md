---
tags: [arch, memory-hierarchy]
updated: 2026-10-18
created: 星期日, 十月 18日 2026, 10:12:06
modified: 星期一, 十月 19日 2026, 10:18:33
---

# 华为 MPU：Message Process 与乱序访存

## Message Process 怎么把「乱序申请」收敛成「顺序发出」？ #arch #memory-hierarchy #performance

> **速查**
> - 输入是带 key 的 message；先分配 lockid，建立 key→lockid 映射
> - 乱序分配：shift queue 记录已发送 key，64 个 key 并行判冲突，只放行不冲突的
> - shift queue 经常满，所以加 dispatch 把源数据打散、拉开间隔，用 `key_conflict` / `full` / `empty` 控制
> - 提前释放：shift queue 每拍记录发送情况，队首判断能否在正常完成前归还 key
> - 功耗：自动时钟门控（模块级 / cell 级）+ 寄存器 D 端门控

### 30 秒口述版 

> 我用 key 标识数据、lockid 标识在途事务。乱序分配靠 shift queue 记录已发送的 key，64 路并行判断冲突，只放行不冲突的 key；因为队列经常 full，所以加 dispatch 把源数据打散、拉开间隔。提前释放也用 shift queue：队尾记录每一拍的发送情况，数据每拍前移，队首判断能否提前释放，并筛查原来的释放信息，避免重复释放。功耗上用自动时钟门控和寄存器 D 端门控。

### 正文

**① 项目定位与数据流**

MPU 是乱序访存系统。Message process 负责三件事：给每个 message 分配 key 和 lockid；允许请求乱序发起查询；在出口按 key 的顺序发出。

```text
上游 message（带 key）
        │
        ▼
┌──────────────────────────────┐
│ lockid 分配 + key→lockid map │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 乱序数据分配                  │
│  shift queue                 │
│  64-key 冲突判断              │
│  dispatch 打散                │
└──────────────┬───────────────┘
               │ 乱序申请
               ▼
            访存查询
               │
               ▼
┌──────────────────────────────┐
│ 顺序发出（按 key 排序）        │
└──────────────┬───────────────┘
               │
               ▼
        提前释放 / 删拍
```

![[huawei-mpu-1792335285476.webp|215]]
**② Message process：key、lockid、乱序申请、顺序发出**

- **key**：数据的身份标识。同一个 key 的多个 message 之间可能存在冲突，不能同时放行。
- **lockid**：一次在途事务/锁的编号。它比 key 窄，查表更快，也把「数据身份」和「事务身份」解耦。
- **乱序申请**：只要两个 message 的 key 不冲突，就可以并行/乱序发起查询，提高访存并行度。
- **顺序发出**：出口按 key 的顺序收敛，保证上层看到的顺序语义。
- **为什么要把两者分开**：乱序是为了吞吐，顺序是为了语义；中间必须有一个重排/排序结构。

> **待确认**：顺序发出是用 reorder buffer、sequence number，还是按 key 排序的队列？谁负责重排？

**③ 乱序数据分配：shift queue + 64-key 冲突判断 + dispatch**

按你的描述，分配流程是：

1. 用 shift queue 记录**已经发送的 key**。
2. 每个周期，把等待中的 key 与已发送 key 做冲突比较；**64 个 key 同时判断**。
3. 不冲突的 key 从等待队列中移出并放行；冲突的继续等待。
4. shift queue 的移位特性让已发送记录和等待队列可以紧凑地向前推进。

问题：这个结构**经常 full**。于是加了 **dispatch**：

- 把源数据打乱、拉开间隔再送入，降低连续 key 冲突的概率。
- 相关控制信号：`key_conflict`（冲突）、`full` / `empty`（队列状态）。

Trade-off：

- 64 路并行比较换来吞吐，代价是比较器面积和时序。
- shift queue 的移位/压缩逻辑有硬件开销。
- dispatch 打散会改变请求到达节奏，可能降低瞬时吞吐，但能减少 full 和冲突。

> **待确认**：这里的「冲突」判定是相同 key、相同地址，还是相同 lock？64 是固定 64 路还是比较器位宽？你补充后我把这段写死。

**④ 提前释放：shift queue 记录每拍发送情况**

- shift queue 的**末尾**记录每一拍的发送情况。
- 每一拍，队列数据往前移。
- 在**队首**判断这个 key 是否可以提前释放。
- 再筛查原来的释放信息：如果 message 还在队列中，说明它没有走「超时释放」路径，此时可以直接释放 key。

目的：不等正常完成/超时，尽早归还 key 和 lockid，提高资源利用率。

风险与检查点：

- 提前释放必须确认该 message 已经不再被下游引用，否则会出现 use-after-free 或重复 key。
- 需要和原来的释放信息做比对，避免重复释放。
- 队列中的位置、发送状态、释放状态三者要能一一对应。

> **待确认**：释放的触发条件是「已发出」还是「已收到响应」？「没有超时释放」的判断依据是什么？

**⑤ Lockid 的分配与 map**

- **分配**：message 进入时从 lockid pool 取一个空闲 id；没有空闲则反压/等待。
- **map**：维护 key → lockid 的映射，供后续查询、释放、重排使用。
- **回收**：正常完成或提前释放时归还 lockid，并从 map 中删除。
- 设计选择：
  - 直接索引 vs 哈希 vs CAM：取决于 lockid 数量、查找延迟、面积。
  - 固定 pool vs 动态分配：固定 pool 简单，动态分配利用率高但控制复杂。
  - 冲突处理：不同 key 映射到同一 lockid 时必须保证互斥。

**⑥ 删拍**

「删拍」指压缩流水线节拍，去掉一级寄存器/一个周期，降低延迟或改善时序。常见手段：

- 合并相邻流水级；
- 把组合逻辑跨寄存器边界重定时（retiming）；
- 用 bypass 跳过不必要的拍。

代价：关键路径可能变长、频率下降；验证和形式检查要重做。

> **待确认**：你删的是哪一拍（lockid 分配、冲突判断还是发出级）？收益是延迟还是面积？

**⑦ 低功耗：自动时钟门控**

- **模块级门控**：模块空闲时关掉它的时钟，省时钟树和触发器翻转功耗。
- **cell 级门控**：更细粒度地 gate 掉没有在使用的 cell。
- **寄存器 D 端门控**：当寄存器不需要更新时，gate 住 D 端，避免输入无意义翻转（也叫数据门控 / operand isolation）。
- 实现上通常用 **ICG**（integrated clock gating，集成时钟门控单元），由使能信号控制；使能要满足 setup/hold，且要 glitch-free。
- 测试模式要 bypass 门控（scan enable），否则扫描链会被切断。

Trade-off：门控逻辑增加面积和使能路径时序；门控太细会带来控制开销和验证复杂度。

### 面试追问

1. 为什么 key 和 lockid 要分开？只用一个行不行？
2. 64 个 key 同时判冲突，冲突矩阵怎么组织？面积和时序怎么权衡？
3. shift queue 和 CAM 相比，为什么选 shift queue？
4. shift queue 经常 full 的根因是什么？dispatch 打散为什么能缓解？
5. dispatch 打乱间隔后，如何保证「顺序发出」不被破坏？
6. 提前释放的判定条件是什么？如何避免释放正在使用的 key？
7. 「筛查原来的释放信息」具体筛什么？重复释放怎么防？
8. lockid 数量怎么定？耗尽时是反压还是丢弃？
9. 删拍删的是哪一级？对频率和延迟的影响怎么量化？
10. 自动时钟门控的使能信号怎么产生？scan/test 模式怎么处理？
11. 这套设计的验证怎么做的？覆盖率、形式验证、随机测试各占多少？
12. 30% 性能提升和 10% 功耗降低的基线、测量方法是什么？

### 自测

- [ ] 不看笔记画出 message process 的数据流，标出 key / lockid / 冲突判断 / 顺序发出
- [ ] 解释 64-key 并行冲突判断的硬件代价，以及为什么需要 dispatch
- [ ] 说清提前释放的三个检查点：已发送、未被引用、未重复释放
- [ ] 解释模块级时钟门控和寄存器 D 端门控分别省的是什么功耗
- [ ] 回答「如果 shift queue 不 full，还需要 dispatch 吗？」

## 需要深挖的内容（按面试命中率排序）

| 优先级 | 主题 | 必须能回答 | 当前缺口 |
|---|---|---|---|
| P0 | 功耗优化 | 门控了哪些模块 / 单元 / 寄存器？使能怎么产生？省的是动态还是漏电？10% 怎么测？ | 具体门控清单、使能条件、测量口径 |
| P0 | 顺序发出 | 排序契约是什么？重排结构在哪？乱序完成、超时、错误怎么处理？ | reorder buffer / sequence number 未确认 |
| P0 | 提前释放 | 触发条件？如何防重复释放和悬空引用？和超时路径怎么配合？ | 触发条件和检查机制未确认 |
| P1 | 64-key 冲突判断 | 冲突怎么定义？比较矩阵怎么组织？一拍完成还是流水？面积 / 时序代价？ | 64 的口径未确认 |
| P1 | Leading-one / priority encoder | 优先链 vs 二分树、全零 / 非 2 的幂、one-hot 生成、项目落点 | 项目里具体用在哪未确认 |
| P1 | shift queue | 为什么不用 FIFO / CAM？满空怎么判？同时读写怎么处理？ | 结构细节 |
| P1 | dispatch | 怎么打散？会不会饿死或破坏顺序？`key_conflict` / `full` / `empty` 怎么用？ | 调度策略 |
| P1 | lockid 分配与 map | 数量怎么定？耗尽怎么办？map 用什么结构？ | 分配策略、map 实现 |
| P2 | 删拍 | 删哪一级？retiming 还是合并？频率 / 延迟收益？ | 具体级数 |
| P2 | 验证 | 用例怎么生成？覆盖率？形式验证？ | 验证方法 |
| P2 | 量化口径 | 30% / 10% 的 baseline、workload、PVT、仿真还是实测 | 数据来源 |

### 功耗优化（示例：深挖到什么程度）

**① 先分账：动态功耗 vs 静态功耗**

- 动态：`P_dyn = α · C · V² · f`。α 是翻转率，C 是电容，V 是电压，f 是频率。
- 静态：`P_static = I_leak · V`。主要来自亚阈值漏电、栅漏电。
- 面试先问「你省的是哪一部分」。只回答「做了时钟门控」不够，要说清省的是时钟树、触发器翻转，还是组合逻辑翻转。

**② 四层门控，省的东西不一样**

| 层级 | 门控对象 | 省什么 | 代价 | 验证点 |
|---|---|---|---|---|
| 模块级 | 整个模块的时钟 | 时钟树 + 模块内所有触发器翻转 | 使能逻辑、唤醒延迟 | 模块空闲判定是否漏判 |
| cell 级 | 未被使用的 cell | 局部时钟 / 翻转 | 控制粒度细，面积和布线开销 | 门控条件是否覆盖所有空闲态 |
| 寄存器级 | 触发器的时钟（ICG） | 触发器时钟功耗 | ICG 面积、使能时序 | 时钟门控检查（STA） |
| 寄存器 D 端 | 触发器的 D 输入 | 组合逻辑翻转（不省时钟） | 多路选择 / 使能逻辑 | 功能是否正确、是否漏掉更新 |

关键区别：

- **时钟门控**：关掉时钟，触发器不翻转，省时钟树和触发器功耗。
- **D 端门控 / operand isolation**：时钟还在，但 D 端不随无意义数据翻转，省组合逻辑和内部节点功耗。
- **power gating**：直接断电，省漏电；需要隔离单元、保持寄存器、电平转换、上电时序。
- 三者不是一回事，面试官经常用这个区分「真做过」和「背过」。

**③ 使能信号怎么来**

- 空闲判定：队列 `empty`、无 valid message、无冲突、map 未被访问、模块处于 reset / idle。
- 使能是组合逻辑还是寄存器输出？组合使能要及时，但要满足 ICG 的 setup / hold，且不能有毛刺。
- scan / test 模式要 bypass：`scan_en | func_en`，否则扫描链被切断。
- 复位和时钟门控的先后顺序要确认，避免复位释放时时钟还没打开。
- 异步时钟域不能直接门控，先同步使能。

**④ 还能往哪深挖（按项目实际选用）**

- **power gating**：电源域划分、header / footer switch、isolation cell、retention flop、wake-up 时序、inrush current。
- **DVFS**：电压频率档位、level shifter、时钟分频、性能监测、切换时的暂停 / 握手。
- **multi-Vt**：关键路径用 LVT，非关键路径用 HVT / SVT 降漏电；和综合约束的关系。
- **存储功耗**：bank 划分、按需使能、retention 模式、减少访问次数。
- **总线 / 编码**：gray code、低翻转编码、减少长线翻转。
- **架构级**：减少无效工作、race-to-idle、并行度与频率的取舍。

**⑤ 怎么量化 10%**

- baseline：优化前的 RTL / 网表，同一 workload、同一 PVT corner、同一频率。
- 功耗拆分：clock tree / flop / combinational / memory / leakage 各占多少。
- 工具：RTL 阶段用仿真活动率 + power estimation；门级用 PrimeTime PX / Voltus 等。
- 口径：动态功耗降了多少、漏电降了多少、总面积 / 时序代价多少。
- 如果没有 silicon 数据，要说清是仿真还是实测，不能把仿真值说成实测值。

**⑥ 面试追问**

1. 你门控了哪些模块？每个模块的空闲条件是什么？
2. 时钟门控和 D 端门控分别省什么？为什么不能只做一种？
3. ICG 的使能信号怎么保证无毛刺？setup / hold 怎么检查？
4. scan / test 模式怎么 bypass？复位时怎么处理？
5. 为什么不用 power gating？如果要用，需要加哪些单元？
6. 门控后时序变差了吗？怎么修的？
7. 10% 是动态、漏电还是总面积归一化后的结果？
8. baseline 是什么？workload 覆盖了哪些场景？
9. 有没有出现过「该门控没门控」或「不该门控被门控」的 bug？怎么发现的？
10. 如果让你再降 5%，你会从哪里入手？

**⑦ 常见陷阱**

- 把 clock gating 说成 power gating：前者省动态，后者省漏电。
- 只讲模块级，不讲寄存器级和 D 端，深度不够。
- 不提 scan bypass 和使能时序，说明没做过实现。
- 10% 没有 baseline 和测量方法，数字不可信。
- 只说「工具自动插的」，说不清工具根据什么条件插、自己做了什么约束。

### 深挖方法（怎么把上面变成自己的东西）

1. **画一张门控分布图**：模块 → 时钟域 → ICG → 使能条件，标出 always-on 的部分。
2. **做一张功耗拆分表**：clock tree / flop / comb / memory / leakage，优化前后各一列。
3. **准备一个反例**：某处不能门控（scan、异步、always-on、复位路径），说明你怎么判断。
4. **准备一个 bug**：门控使能漏判导致功能错误，或门控后时序违例，讲你怎么定位和修。
5. **把数字串成故事**：为什么做 → 怎么做 → 代价是什么 → 怎么验证 → 结果多少。

---

## Leading-one 电路怎么用 RTL 实现？ #arch #performance

> **速查**
> - Leading-one（最高位 1）检测：输出最高有效 1 的位置、one-hot 掩码和 valid
> - 优先链实现最简单，延迟 O(W)；二分树实现延迟 O(log W)，面积更大
> - 64 位常用 6 级二分：32 → 16 → 8 → 4 → 2 → 1
> - 全零时 `valid=0`，`idx` 定义为 0；W 不是 2 的幂时补零或单独处理
> - 在 MPU 里可用于 lockid 空闲位图、冲突位图、仲裁器

### 正文

**① 定义与用途**

给定 `in[W-1:0]`，找**最高位的 1**：

- `valid`：是否存在 1；
- `idx`：最高位 1 的下标；
- `one_hot`：只有该位为 1 的掩码。

用途：

- 优先级仲裁：最高优先级的请求者；
- lockid 分配：从空闲位图里找第一个空闲 id；
- 冲突位图：找第一个冲突 / 第一个可放行的 key；
- 浮点归一化：leading-zero count 的姊妹电路。

**② 实现一：优先链（最简单）**

```verilog
module leading_one #(
    parameter int W = 64,
    localparam int IW = (W <= 1) ? 1 : $clog2(W)
)(
    input  logic [W-1:0] in,
    output logic [W-1:0] one_hot,
    output logic [IW-1:0] idx,
    output logic          valid
);
    always_comb begin
        one_hot = '0;
        idx     = '0;
        valid   = 1'b0;
        for (int i = W-1; i >= 0; i--) begin
            if (!valid && in[i]) begin
                one_hot[i] = 1'b1;
                idx        = i[IW-1:0];
                valid      = 1'b1;
            end
        end
    end
endmodule
```

特点：代码短、易读；综合工具会生成优先 mux 链，延迟随 W 线性增长。W=64 时关键路径偏长。

**③ 实现二：二分树（64 位推荐）**

思路：每次把候选区间劈成两半，先看高半区有没有 1；有就选高半区并置位对应索引位，否则选低半区。

```verilog
module leading_one64 (
    input  logic [63:0] in,
    output logic [5:0]  idx,
    output logic        valid
);
    logic [63:0] m;
    logic [5:0]  i;

    always_comb begin
        m     = in;
        i     = 6'd0;
        valid = |in;

        if (|m[63:32]) begin i = i + 6'd32; m = m[63:32]; end
        if (|m[31:16]) begin i = i + 6'd16; m = m[31:16]; end
        if (|m[15:8])  begin i = i + 6'd8;  m = m[15:8];  end
        if (|m[7:4])   begin i = i + 6'd4;  m = m[7:4];   end
        if (|m[3:2])   begin i = i + 6'd2;  m = m[3:2];   end
        if (m[1])      begin i = i + 6'd1;  end

        idx = i;
    end
endmodule
```

6 级归约，延迟 O(log₂64)=6；面积比优先链大，但高频下更稳。

**④ 几种写法与优劣**

| 写法 | 延迟 | 面积 | 可参数化 | 适用 | 主要问题 |
|---|---|---|---|---|---|
| for 循环优先链 | O(W) | O(W) | 好 | 小位宽、低频、先写对 | 64 位关键路径长 |
| `casez` 优先编码 | O(W) | O(W) | 差（要手写） | 固定 8/16 位 | 64 位代码冗长；`casex` 会把 X 当 don't care |
| 二分树 | O(log W) | O(W) | 好 | 64 位 ASIC、高频 | 代码和连线复杂，索引要逐级拼 |
| 分段 8×8 | O(log W) | 小于全二分树 | 好 | 64/128/256 位 | 组边界控制、组选择 mux |
| 前缀 OR + one-hot | O(log W) | O(W log W)，Brent-Kung 可到 O(W) | 好 | 高速定制、需要 one-hot | 线多；one-hot 转二进制还要编码树 |
| bit-reverse + `x & -x` | O(log W) + 加法器 | O(W) | 好 | FPGA / 软件、找最低位 | 64 位 bit-reverse 连线乱；加法器有延迟 |
| 向下 smear + 边沿检测 | O(log W) | O(W log W) | 好 | 规整流水、one-hot 输出 | 面积大；仍需编码 |
| LUT 级联 | 取决于 LUT 深度 | 小 | 工具相关 | FPGA | ASIC 不适用 |

几个关键取舍：

- **优先链 vs 二分树**：优先链是「从高到低依次看」，综合成优先 mux 链；二分树是「每次砍一半」，延迟从 O(W) 降到 O(log W)，代价是面积和连线。
- **分段 8×8**：先对 8 个 8 位组各做一次「有没有 1」，选出最高非空组，再在组内做 8 位 LOD。面积比 64 位全二分树小，模块可复用，适合 64/128 位。
- **前缀 OR**：`one_hot[i] = in[i] & ~(|in[W-1:i+1])`，即「本位是 1 且更高位没有 1」。天然出 one-hot；如果只要 one-hot grant（仲裁），可以省掉后面的二进制编码。
- **bit-reverse + `x & -x`**：`x & -x` 只隔离**最低位** 1；要找最高位得先反转。硬件里反转是连线置换，加法器是延迟大头。
- **smear**：`m |= m>>1; m |= m>>2; ...` 把最高位 1 以下全部填 1，再 `m & ~(m>>1)` 取边沿。规整但面积大。
- **`casex` 慎用**：它把 X 也当 don't care，仿真可能掩盖 X 传播；优先用 `casez` 或显式比较。

**先确认方向：leading-one 还是 trailing-one**

- 找**最高位** 1（最高优先级仲裁）→ leading-one，需要树或反转。
- 找**最低位** 1（第一个空闲 lockid、最低地址）→ trailing-one，直接 `x & -x`，最省。
- 如果项目里 lockid 分配用的是「第一个空闲」，很可能你真正需要的是 trailing-one，不是 leading-one。先确认这一点，能省掉一整棵反转/树。

W 不是 2 的幂：补零到下一个 2 的幂，输出时屏蔽掉补零位；或写 generate 分支。全零：`valid=0`，`idx=0`（约定即可，但要在文档里写清）。

**分段 8×8 写法（64 位实用版）**

```verilog
module leading_one_seg64 (
    input  logic [63:0] in,
    output logic [5:0]  idx,
    output logic        valid
);
    function automatic [2:0] lod8(input logic [7:0] v);
        integer i;
        logic found;
        begin
            lod8  = 3'd0;
            found = 1'b0;
            for (i = 7; i >= 0; i--) begin
                if (!found && v[i]) begin
                    lod8  = i[2:0];
                    found = 1'b1;
                end
            end
        end
    endfunction

    logic [7:0] grp;
    logic [2:0] grp_idx, bit_idx;
    logic [7:0] sel;

    always_comb begin
        for (int g = 0; g < 8; g++)
            grp[g] = |in[g*8 +: 8];

        grp_idx = lod8(grp);
        valid   = |grp;

        sel = 8'd0;
        for (int g = 0; g < 8; g++)
            if (grp_idx == g[2:0])
                sel = in[g*8 +: 8];

        bit_idx = lod8(sel);
        idx     = {grp_idx, bit_idx};  // 8×8，正好 6 位
    end
endmodule
```

**最低位 1 的写法（trailing-one）**

```verilog
wire [63:0] low_one = in & (~in + 64'd1);  // 等价于 in & -in
```

如果只需要「第一个空闲 lockid」这种最低位，直接用这条，不需要 bit-reverse 和树。要找最高位才先反转、取最低位、再反转。

**选型建议**

| 场景 | 推荐写法 | 理由 |
|---|---|---|
| W ≤ 8 | for 循环 / `casez` | 代码短，延迟可接受 |
| W = 16–32 | 二分树 | O(log W)，面积适中 |
| W = 64 | 二分树或分段 8×8 | 分段面积更小、模块可复用 |
| 只要 one-hot grant | 前缀 OR | 省掉二进制编码树 |
| 找最低位 1 | trailing-one `x & -x` | 不需要反转 / 树 |
| FPGA | 让工具推断 LUT | 手写未必更好 |

**补充：OR-XOR 二叉树（LZD 写法）**

博客 [LZD——前导零检测的高效硬件实现](https://www.cnblogs.com/nvmjom/articles/16955546.html) 给的是前导零检测（LZD，leading-zero detection）的树形实现。它和 leading-one 是同一件事的两种表述：

- W 位输入，LZD 输出 `B`：它既是**前导零个数**，也是**从 MSB 数起的位置码**；`B` 的每一位由 mux 树逐级选出，不是先算计数再换算。
- ⭐ 如果按 LSB 编号，`idx = W - 1 - B`；W=2^n 时 `W-1` 全 1，所以 `idx = ~B`，只是按位取反，不是减法器。
- 全零时 `full=1`，对应我们的 `valid=0`。

结构：

1. **OR 树**：每个节点表示对应区间内是否有 1；父节点 = 左右子节点 OR。
2. **XOR 判断左右**：`xor = Left ^ Right`；1 表示第一个 1 在右区间，0 表示在左区间或不存在。
3. **一般情况修正**：当左=1、右=0 时，朴素 XOR 会误判为右；改成

   ```text
   B[i] = Left ^ (Left | Right)
   ```

   > ⭐ **背诵**：这四个 case 要能默写。

   - 左=1，右=0 → `1 ^ 1 = 0` → 左；
   - 左=0，右=1 → `0 ^ 1 = 1` → 右；
   - 左=1，右=1 → `1 ^ 1 = 0` → 左；
   - 左=0，右=0 → `0 ^ 0 = 0` → 左 / 不存在。
4. **结果传递**：每级的 XOR 结果选择下一级用左子树还是右子树的 XOR 结果，逐级拼出 `B` 的每一位。

延时 / 面积（原文公式，`2^n` 位）：

- 延时：`(n+1)·OR + XOR + MUX`，即 `O(log W)`。
- 面积：`(2^{n+1}-1)(OR+XOR) + (2^{n+1}-2-n)·MUX`，即 **`O(W)`**。
- ⚠️ 原文结尾写「面积和延时都是 `O(log n)` 级别」，**面积这一句是错的**；按它自己的公式，面积是 `O(2^n)=O(W)`。面试时不要跟着说错。

和二分查找写法的区别：

- **二分查找**：每级用 `|` 判断高半区是否有 1，选一半，索引位直接置位；本质是「选区间」。
- ⭐ **OR-XOR 树**：同时建 OR 树和 XOR 选择树；mux 树逐级把选中子树的低位拼上来，`B` 的每一位就是位置位。多一组 XOR 和 MUX，但结构更规整，适合流水。
- 两者延迟都是 `O(log W)`；OR-XOR 树的面积常数更大。

**原文代码结构（Gitee）**

原文的 Verilog 在 [Gitee: verilog-resp/LZD](https://gitee.com/tang_jixuan/verilog-resp/tree/master/LZD)，核心是四个模块：

- `lzd_node_btm`：叶子节点，`data_o = left | right`，`cnt_o = left ^ data_o`，即 `left ^ (left | right)`。
- `lzd_node`：递归节点，实例化左右两个子节点；`data_o` 表示本区间是否有 1，`cnt_o` 的最高位由 `lzd_node_btm` 产生，低位由 mux 在左右子节点的 `cnt` 之间选择。
- `mux`：`sel=1` 选右子节点，`sel=0` 选左子节点。
- `lzd`：顶层，`full = !full_n`，`full_n` 是根节点的 OR。

关键代码：

```verilog
// 叶子：判断第一个 1 在左还是右
assign data_o = left_i || right_i;
assign cnt_o  = left_i ^ data_o;   // left ^ (left | right)

// 递归节点：左右子树 + 本级选择
lzd_node #(W>>1, N-1) node[1:0](data_i, {left, right}, {cnt[1], cnt[0]});
lzd_node_btm node_calc(left, right, data_o, cnt_o[N-1]);
mux #(N-1) cnt_mux(cnt_o[N-1], cnt[1], cnt[0], cnt_o[N-2:0]);
```

注意：原文的 testbench 只 dump 波形，没有自检；复习时建议补一个参考模型（`$clog2` 或逐位循环）做自动比对。

**8 位扁平写法（可直接综合）**

递归版抽象，面试手写更适合扁平展开。8 位只需 3 级：

```verilog
module lzd_or_xor8 (
    input  logic [7:0] data,
    output logic [2:0] zcnt,   // 前导零个数 = 从 MSB 数的位置
    output logic       full    // 全零
);
    // Level 1: 两两 OR
    wire n1_3 = data[7] | data[6];
    wire n1_2 = data[5] | data[4];
    wire n1_1 = data[3] | data[2];
    wire n1_0 = data[1] | data[0];

    // Level 2: 4 位 OR
    wire n2_1 = n1_3 | n1_2;
    wire n2_0 = n1_1 | n1_0;

    // Level 3: 根 OR
    wire any  = n2_1 | n2_0;
    assign full = ~any;

    // 第 2 位：高 4 位还是低 4 位
    wire b2 = n2_1 ^ (n2_1 | n2_0);

    // 第 1 位：选中 4 位半区的两对
    wire b1_hi = n1_3 ^ (n1_3 | n1_2);
    wire b1_lo = n1_1 ^ (n1_1 | n1_0);
    wire b1    = b2 ? b1_lo : b1_hi;

    // 第 0 位：选中 2 位对的左右
    wire b0_76 = data[7] ^ (data[7] | data[6]);
    wire b0_54 = data[5] ^ (data[5] | data[4]);
    wire b0_32 = data[3] ^ (data[3] | data[2]);
    wire b0_10 = data[1] ^ (data[1] | data[0]);

    wire b0_hi = b1 ? b0_54 : b0_76;
    wire b0_lo = b1 ? b0_10 : b0_32;
    wire b0    = b2 ? b0_lo : b0_hi;

    assign zcnt = {b2, b1, b0};
endmodule
```

验证：

| data | 最高位 1 | zcnt | `~zcnt`（LSB 索引） |
|---|---|---|---|
| `8'b1000_0000` | bit7 | `000` | `111` |
| `8'b0010_0000` | bit5 | `010` | `101` |
| `8'b0000_1010` | bit3 | `100` | `011` |
| `8'b0000_0001` | bit0 | `111` | `000` |
| `8'b0000_0000` | 无 | `000` | `full=1`，无效 |

注意：`zcnt` 是从 MSB 数的位置（前导零个数）；要 LSB 索引就 `~zcnt`，W=8 时只是 3 个反相器。

**递归 / 参数化写法（非扁平）**

把「劈半、判断左右、拼低位」写成递归模块，W 只要是 2 的幂就能自动展开：

```verilog
module lzd_node #(
    parameter int W = 8
)(
    input  logic [W-1:0]         data_i,
    output logic                 any_o,    // 本区间是否有 1
    output logic [$clog2(W)-1:0] zcnt_o    // 前导零个数 / MSB-first 索引
);
    localparam int N  = $clog2(W);
    localparam int HW = W / 2;
    localparam int CN = (N > 1) ? (N - 1) : 1;

    generate
        if (W == 2) begin : g_leaf
            // 叶子：2 位
            assign any_o  = data_i[1] | data_i[0];
            assign zcnt_o = data_i[1] ^ any_o;   // bit1=1 → 0；只有 bit0=1 → 1
        end
        else begin : g_node
            logic          left_any, right_any;
            logic [CN-1:0] left_cnt, right_cnt;
            logic          sel;

            // 左 = 高半区，右 = 低半区
            lzd_node #(.W(HW)) u_left (
                .data_i (data_i[W-1:HW]),
                .any_o  (left_any),
                .zcnt_o (left_cnt)
            );
            lzd_node #(.W(HW)) u_right (
                .data_i (data_i[HW-1:0]),
                .any_o  (right_any),
                .zcnt_o (right_cnt)
            );

            assign any_o = left_any | right_any;
            assign sel   = left_any ^ any_o;   // 1: 第一个 1 在右；0: 在左
            assign zcnt_o = {sel, sel ? right_cnt : left_cnt};
        end
    endgenerate
endmodule

module lzd #(
    parameter int W = 8
)(
    input  logic [W-1:0]         data,
    output logic [$clog2(W)-1:0] zcnt,
    output logic                 full
);
    logic any;
    lzd_node #(.W(W)) u_node (
        .data_i (data),
        .any_o  (any),
        .zcnt_o (zcnt)
    );
    assign full = ~any;
endmodule
```

调用关系：

```text
lzd #(8)
└─ lzd_node #(8)          sel = zcnt[2]，选左/右 4 位
   ├─ lzd_node #(4)       sel = zcnt[1]，选左/右 2 位
   │  ├─ lzd_node #(2)    zcnt[0]
   │  └─ lzd_node #(2)
   └─ lzd_node #(4)
      ├─ lzd_node #(2)
      └─ lzd_node #(2)
```

和 Gitee 原码的关系：

- 原码用 `lzd_node node[1:0](...)` 数组实例化 + 独立 `mux` 模块；
- 这里用命名端口 + 三目 mux，逻辑完全等价，读起来更直白；
- 原码的 `cnt_o[N-1]` 和这里的 `sel` 是同一个信号；原码的 `mux` 就是这里的 `sel ? right_cnt : left_cnt`。

**和前面几种写法的对比结论**

| 对比对象 | OR-XOR 树的优势 | 代价 |
|---|---|---|
| 优先链 / `casez` | 延迟 O(log W)，可参数化，64 位不用写 64 行 | 面积更大，代码更复杂 |
| 二分查找 | 同时产出 valid 和位置码；mux 树直接拼索引位；树结构规整 | 每级多一个 XOR 和更宽的 mux；LSB 编号时 `idx = ~B`（只是取反，不是减法器） |
| 分段 8×8 | 单一平衡树，没有组边界控制 | 64 位面积通常大于分段 |
| 前缀 OR + one-hot | 直接出二进制计数，省掉 one-hot→binary 编码树 | 前缀 OR 的布线更规整，面积取决于实现 |
| bit-reverse + `x & -x` | 不需要 bit-reverse 和加法器 | 如果只要 trailing-one，它反而更便宜 |
| smear + 边沿检测 | 直接出计数，面积 O(W) 而非 O(W log W) | smear 更规整、易流水 |
| LUT 级联（FPGA） | ASIC 友好、可参数化 | FPGA 上让工具推断可能更好 |

一句话选型：

- **要前导零计数 / 浮点归一化** → OR-XOR 树（`B` 直接就是位置码）。
- **要最高位 1 的索引 / 仲裁** → 两者都行；二分查找门数更少，OR-XOR 树更规整、易参数化。
- **只要最低位 1** → trailing-one `x & -x`。
- **小位宽** → for 循环 / `casez`。

**⑤ 在 MPU 项目里的落点**

- **lockid 空闲位图**：`free[63:0]`，leading-one 找最高优先级的空闲 lockid；
- **冲突位图**：64 个 key 的冲突结果压成 64 位，leading-one 找第一个冲突；
- **仲裁**：多请求者优先级仲裁，leading-one 直接给出 grant one-hot；
- **和 shift queue 的关系**：shift queue 负责顺序 / 压缩，leading-one 负责在位图里选第一个，两者常配合使用。

> **待确认**：项目里具体用在 lockid 分配还是冲突筛选？位宽是不是 64？这决定上面例子能不能直接复用。

**⑥ 验证与追问**

验证用例：

- 全零 → `valid=0`；
- 每个单 bit → `idx` 等于该 bit；
- 全 1 → `idx=W-1`；
- 随机向量与参考模型比对；
- 形式验证：`valid` 等价于 `|in`；`idx` 指向的位为 1；不存在更高的 1。

追问：

1. leading-one、leading-zero、priority encoder 三者有什么区别？
2. 为什么二分树是 O(log W)？关键路径在哪？
3. W 不是 2 的幂怎么处理？
4. 不用 shifter 怎么产生 one-hot 输出？
5. 全零时 `idx` 怎么定义？X 态怎么处理？
6. 怎么流水化？增加几拍、面积多少？
7. 64 位怎么验证？穷举不可行时用什么？
8. 这个电路在你项目里具体接在哪？

### 自测

- [ ] 不看代码写出 64 位二分树的 6 级判断
- [ ] 说清优先链和二分树的延迟 / 面积取舍
- [ ] 处理全零、非 2 的幂、X 态三个边界
- [ ] 把 leading-one 映射到 lockid 分配或冲突位图

### 背诵项（面试前 5 分钟）

- [ ] **修正公式**：`B = Left ^ (Left | Right)`；四个 case：10→0（左）、01→1（右）、11→0（左）、00→0
- [ ] **递归三行**：`any = left | right`；`sel = left ^ any`；`zcnt = {sel, sel ? right_cnt : left_cnt}`
- [ ] **2 位基准**：`any = d1 | d0`；`zcnt = d1 ^ any`
- [ ] **索引关系**：`zcnt` 是 MSB-first 位置；`idx_lsb = ~zcnt`（W=2^n）
- [ ] **复杂度**：延迟 O(log W)，面积 O(W)；原文「面积 O(log n)」是错的
- [ ] **级数**：8 位 3 级（8→4→2），64 位 6 级
- [ ] **全零**：`full=1`，`zcnt=0`，索引无效
- [ ] **参数**：`N=$clog2(W)`；递归 `W>>1, N-1`；W 必须是 2 的幂
- [ ] **选型**：最高位索引 → 二分 / OR-XOR；最低位 → `x & -x`；小位宽 → for / `casez`
- [ ] **一句话**：OR-XOR 树的 mux 树直接拼索引位，不是「先算计数再换算」
