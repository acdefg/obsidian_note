了。。。。。。。。。。。。。。。。。。。。           ，。。。。。。。。。                                                                                                                                                                                                                                                                                                                                                                             ，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，，*测试平台: thor
测试模型: GR00T-N1.5-3B(这里有其他版本,不确定是否要测试)

### 1222
![[image-13.png]]
基本和官方结果一致
**QPS (Queries Per Second)** 代表的是每秒查询率，也可以理解为模型每秒钟能够处理完成的推理请求数量，它是衡量模型“吞吐量”最核心的指标。计算这个数值的公式其实非常简单，在单次请求（Batch Size = 1）的实时控制场景下，QPS 与延迟（Latency）互为倒数关系，即 $QPS = \frac{1000}{\text{Latency(ms)}}$
我发现你在生成时间线时用的时间是番茄钟循环设置时间，而不是这段 log 的持 log】
】
】
】
】

/、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、、
### latency test
batch size = 1
其中TensorRT的结果: TRT FP16和TRT FP8/nvFP4 都是采用`trtexec`测试
Settings:  Warmup=200, Iterations=500, Metric=Median GPU Compute Time.

Pipeline latency和pytorch的结果是通过官方脚本,使用python脚本得到,这部分结果和官方偏差比较大,推测可能是python编译以及数据转换带来的问题,以及可能是我这里使用的python的计时函数做的,考虑了D2H和H2D的时间(不是很确定)
PyTorch & Pipeline: Measured via Python script ('gr00t_benchmark.py').
Settings: Warmup=5, Steps=20, Metric=Mean Wall Time.
![[thor test-1.png|519x211]]
官方给出的测试数据如下:
![[GR00T_TEST.png|380x329]]

![[image-10.png]]
### ncu test
使用 NVIDIA Nsight Compute (`ncu`) 分析主要 Kernel 的内存使用情况（Memory Throughput, Cache Hit Rate, DRAM Bandwidth）
*   **工具**: `ncu` (NVIDIA Nsight Compute CLI)
*   **指标 (Metrics)**:
    *   `MemoryWorkloadAnalysis`: 详细分析 DRAM、L2 Cache、L1 Cache 的吞吐量和命中率。
    *   `SpeedOfLight`: 查看内存带宽利用率（Memory %）
*   **采样策略**:
    *   抓取**耗时最长**的前几个 Kernel（通常是 GEMM, Attention, Convolution）

![[log-2.png|565x258]]

Metric Explanations:
  Avg(us): Average execution time per kernel launch in microseconds.
  Count:   Number of kernel launches captured.
  %Time:   Percentage of total captured time spent in this category.
  L2(MB):  Total L2 Cache traffic (Bytes) converted to MB.
  L1(MB):  Total L1/Texture Cache traffic (Bytes) converted to MB.
  L2%:     Average L2 Cache throughput utilization (pct of peak).
  L1%:     Average L1/Texture Cache throughput utilization (pct of peak).
  SM%:     Average Streaming Multiprocessor (Compute) throughput utilization (pct of peak).

### nsys test
*   **工具**: `nsys` (NVIDIA Nsight Systems)
*   **指标 (Metrics)**:
    *   `MemoryWorkloadAnalysis`: 分析H2D D2H D2D的op次数以及total time
    *   `SpeedOfLight`: 查看内存带宽利用率（Memory %）
![[log-1.png]]
