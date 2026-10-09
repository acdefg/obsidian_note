---
title: AXI
tags: ["note"]
created: 星期日, 六月 28日 2026, 2:58:15 下午
modified: 星期日, 六月 28日 2026, 5:16:58 下午
---

五个通道
busrt 概念
trans 概念
burst 类型：fixed、incr、warp
非对齐访问
信号依赖关系

# outstanding 问题
**ostd >= (soc_latency*use_w) / (bus_w*tp_burst)**
[https://mp.weixin.qq.com/s/T6YCQ-RZTupiPjC82bUrJw](https://mp.weixin.qq.com/s/T6YCQ-RZTupiPjC82bUrJw)

# pipeline
[AHB总线笔记（一） - 哔哩哔哩](https://www.bilibili.com/read/cv15736206/)