---
tags: [index, taxonomy]
updated: 2026-10-09
---

# 主题分类表

`learn-note` 每次归类和写标签前都会读这张表。**这张表是知识库的唯一分类权威**——
改表 = 改分类法。新建主题文件前若发现表里没有对应领域，说明该改表了，而不是即兴造个文件名。

## 分类表

| 领域 | 文件名 | frontmatter tags | 覆盖范围 |
|---|---|---|---|
| CUDA | `cuda-memory.md` | `[cuda, memory]` | 内存模型、pinned、UVA、异步拷贝、stream |
| CUDA | `cuda-execution.md` | `[cuda, execution]` | SIMT、warp、occupancy、bank conflict |
| 计算机体系结构 | `computer-arch.md` | `[arch, gpu]` | cache、流水线、分支预测、乱序执行；GPU 微架构（SM/TMA/L2）、memory hierarchy、roofline |
| 图形学 | `graphics-rendering.md` | `[graphics, rendering]` | 光栅化、光追、shading、GAMES101 |
| 图形学 | `geometry-processing.md` | `[graphics, geometry]` | 网格、细分、参数化 |
| 图形学 | `graphics-gaussian-splatting.md` | `[graphics, 3dgs]` | 3DGS / 4DGS、可逆渲染、splat 表示与压缩 |
| 深度学习系统 | `dl-systems.md` | `[dl, systems]` | 训练/推理系统、算子、并行策略 |
| 具身智能 | `embodied-ai.md` | `[embodied, robotics]` | 机器人、仿真、感知–决策–控制 |
| 数字电路 | `digital-design.md` | `[digital, verilog]` | Verilog、时序、综合、Zynq |
| C++ | `cpp.md` | `[cpp]` | 语言特性、模板、内存模型、并发 |
| 工具链 | `toolchain.md` | `[toolchain]` | 构建、调试、profiler、环境 |

## 归类歧义怎么办

一个概念同时沾两个领域时（例：pinned memory 既是 CUDA 也是 OS 的虚拟内存话题）：

1. **先查覆盖范围**列的措辞，哪边写了就归哪边。
2. 两边都没写 → 归到**你更可能从哪个角度回查**的那个领域。
3. 仍然模棱两可 → 归主领域，在另一个领域开一行 `[[链接]]` 占位。这是唯一该开占位的情形。
4. **把决定补进覆盖范围列**，下次就不用再想。

不做的事：**不要为了一次归类新建一个领域**。领域是慢慢长出来的，不是每次提问都长一个。

## 怎么改这张表

直接改，不用问谁：

- 加领域 → 加一行，`文件名` 用 kebab-case，`tags` 给 1–2 个英文词
- 合领域 → 删行、把已有文件改名、更新 `INDEX.md` 里的链接
- 改文件名 → 记得同时改 INDEX 和所有 `[[指向它的双链]]`

## 变更记录

- 2026-10-09 — 初版。由 10 行草稿定稿：GPU 架构并入 `computer-arch.md`；图形学拆三片（4DGS 单列）；补 `embodied-ai.md`。
- 2026-10-09 — 从 `D:\code\notes\` 迁到 vault 内 `note\study_note\cc_notes\`，双链与标签面板自此生效。
