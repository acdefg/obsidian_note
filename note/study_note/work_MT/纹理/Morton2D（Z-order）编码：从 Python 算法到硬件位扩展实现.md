## 什么是 Morton2D（Z-order）

Morton2D（也称 Z-order Curve）是一种将 **二维坐标 (x, y)** 映射为 **一维地址** 的方法，其核心目标是：

> **在一维线性地址中，尽量保持二维空间的局部性**

它被广泛应用在：
- GPU 纹理寻址
- Cache / Tiling
- BVH / QuadTree
- 空间数据索引

