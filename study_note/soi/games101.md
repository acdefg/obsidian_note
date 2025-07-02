## Lecture 1
[GAMES101-现代计算机图形学入门-闫令琪\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1X7411F744/?spm_id_from=333.337.search-card.all.click&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202407302129317.png)
1、光栅化
2、几何：曲线和曲面
3、光线追踪
4、动画与模拟

光栅化：
[GAMES101-现代计算机图形学入门-闫令琪\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1X7411F744?t=1763.8)
实时：每秒 30 帧画面

## Lecture 2

## Lecture 9：纹理映射

### 重心坐标
[Lecture 09 Shading 3 (Texture Mapping Cont.)\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1X7411F744?spm_id_from=333.788.videopod.episodes&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7&p=9)
重心坐标系
![](http://cdn.ljc0606.cn/obsidian/202507021627792.png)

![](http://cdn.ljc0606.cn/obsidian/202507021623114.png)
三角形平面上任意一点都可以通过三角形三个顶点表示，如果α、β、γ非负，则点在三角形内，α
+β+γ=1，保证在三角形平面内
重心坐标对应的α、β、γ值可以通过三角形面积求出，Aa 为顶点 A 不相邻的三角形
![](http://cdn.ljc0606.cn/obsidian/202507021626309.png)
重心的坐标
![](http://cdn.ljc0606.cn/obsidian/202507021628108.png)
由于三维到二维的过程坐标值会发生变化，所以先在三维空间做插值，再投影
![](http://cdn.ljc0606.cn/obsidian/202507021633382.png)
简单实现逻辑：
找到纹理坐标，找到对应颜色，应用
![](http://cdn.ljc0606.cn/obsidian/202507021634526.png)
### 问题 1：纹理失真
低分辨率纹理映射高分辨率图片
纹素：纹理上的像素
![](http://cdn.ljc0606.cn/obsidian/202507021636106.png)
