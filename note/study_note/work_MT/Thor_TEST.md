
## 1222
### VGGT
- GPU：**NVIDIA Thor**
- Batch size：**1**
- 测试方式：端到端 pipeline + 各子模块拆分
- model：vggt v1.5-1B

| Component         | Torch (ms) | FP16 (ms) | FP16 QPS | Quant (ms) | Quant QPS |
| ----------------- | ---------- | --------- | -------- | ---------- | --------- |
| Backbone          | 2.30298    | 4.27      | 427.728  | (Shared)   | —         |
| State Encoder     | 0.10       | 0.0390625 | 20518.1  | (Shared)   | —         |
| Action Encoder    | 0.34       | 0.109     | 8432.58  | (Shared）   | —         |
| Action Decoder    | 0.10       | 0.0341797 | 22669.4  | (Shared)   | —         |
| DiT               | 9.31       | 5.50934   | 179.354  | 3.25653    | 302.488   |
| ViT               | 11.82      | 5.13354   | 193.011  | 3.94464    | 251.931   |
| LLM               | 20.45      | 7.48969   | 133.053  | 6.02655    | 164.787   |
| **FULL PIPELINE** | 87.83      | 49.65     | —        | 37.24      | —         |

基本和官方结果一致

**QPS (Queries Per Second)** 代表的是每秒查询率，也可以理解为模型每秒钟能够处理完成的推理请求数量，它是衡量模型“吞吐量”最核心的指标。计算这个数值的公式其实非常简单，在单次请求（Batch Size = 1）的实时控制场景下，QPS 与延迟（Latency）互为倒数关系，即 $QPS = \frac{1000}{\text{Latency(ms)}}$

$$Effective TFLOPS=FLOPs per inference × QPS$$
