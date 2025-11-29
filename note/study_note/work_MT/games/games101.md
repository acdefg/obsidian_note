
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

## Lecture 3： Transformation
### scale
![image.png|351x251](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251110220244344.png) ![|254x80](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251110220628906.png)

### shear matrix
![image.png|413x298](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251110221709296.png)

### rotate
![image.png|414x294](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251110221923862.png)
用 [1,0] 和 [0,1] 变换后得到的位置，就可以得到上述矩阵

### 线性变化
![image-1|400x302](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104745.png)

### 齐次坐标的引入
![image-2|395x202](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104746.png) 平移变换没有办法用线性变换的方式表示

![image-3|391x270](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104747.png) 使用 3 维切边实现 2 维平移变换 

![image-4|387x220](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104748.png) 通过 w 是 1 还是 0 来表示是点还是向量

![image-5|384x127](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104749.png) 一个点+一个点会等于一个点的中点

### 仿射变换
![image-6|377x295](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104750.png)
仿射变换 = 线性变换 + 平移

### 2d transformation summary
![image-7|368x318](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104751.png)

### 逆变换
复杂的变换可以通过一系列简单的操作得到
变换的顺序会影响变换的结果（矩阵不满足交换律）
![image-8|364x260](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104752.png)

## Lecture 4
![image-12|379x217](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602440.png)

![image-9|381x269](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602441.png)
投影，将近平面和远平面变换成一样的大小
定义透视投影的视锥：
1. 长宽比 
2. 可视角度：相机视角到上下边界的夹角
![image-10|380x176](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602442.png) n 是 near，代表相机到近平面的距离


## Lecture 5： rasterization
raster： 表示 draw onto the screen
pixel： 屏幕上有着独立颜色的小块（short for picture element）
screen space：屏幕空间，像素坐标为 (x,y), x,y 都是整数
![image-13|332x233](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602443.png)

视口变化：将[-1,1]^2 的立方体映射到屏幕空间
![image-14|335x198](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602444.png)


framebuffer： memory for a raster display
image：2D arrary
### 为什么用三角形
![image-15|403x283](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602445.png)

光栅化需要知道：每个像素和三角形的关系
### 一种光栅化的方法：采样
![image-16|349x237](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602446.png)

![image-17|353x130](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602447.png)

![image-19|298x236](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602448.png)

对于三角形 abc， 对某个点 Q， 若 Q 叉乘， ab，bc，ca 的结果都为正/负，则点表示在三角形内(右手螺旋定理)
如果一个点处在两个三角形的边界上，那么自行选择处于哪个三角形上

## Lecture 6：反走样
![image-20|473x333](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602449.png)

### MSAA
![image-21|372x260](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602450.png)

![image-22|369x268](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602451.png)

msaa：通过增加采样点，获得更加合理的覆盖率（一个像素 4 个采样点，有一个在三角形内，则颜色覆盖率为 25%）
fxaa：先得到有锯齿的图，然后进行边缘检测和滤波
taa：temperol aa，复用了上一帧的结果
![image-23|377x304](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251122205602452.png)

## Lecture 7： Z-test and shading
### z-buffering
![image-9|333x264](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104753.png)
对每个像素表示的最浅的深度（离屏幕最近的）
frame buffer：只存最后的结果
depth buffer：对每个像素，离屏幕最近的距离
![image-10|346x217](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104754.png)
### shading
对不同物质，应用不同材质的过程
光线如何和材质进行作用，如何去反射
![image-12|391x270](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104755.png)
着色具有局部性，这里并不会生成阴影
![image-13|424x314](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104756.png)

漫反射
![image-14|342x264](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104757.png)
光线打在同样材质但是不同角度的物体表面上，生成的明暗是不一样的
![image-15|455x313](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104758.png)
考虑单位面积接收到的光线的能量，这个值和光线方向和物体表面的法线方向的夹角的余弦值成正比，l，n 都是单位向量
![image-17|322x242](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104759.png)
![image-19|320x221](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104760.png)

能量守恒，但是能量密度和传播距离成平方反比
有多少能量到达乘以有多少能量被接收，如果夹角余弦为负，则表示光线没有经过物体表面的该点
漫反射物体表面反射的光线会被反射到各个不同的方向，所以不需要考虑观测方向

## Lecture 8： shading，pipeline
### blinn-phong 着色模型
 ![image-20|503x370](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104761.png)
 高光出现的原理是：镜面反射方向和观测方向和接近，也就相等于法线方向和半程向量方向接近
 为什么用半程向量：h 其实等于观测方向 v+光线方向 h，这比反射方向更好算
 简化了多少能量被吸收
p 为什么控制夹角的衰减系数
![[image-21.png|292x159]]
![image-22|209x159](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104762.png)

环境光：用一个常数来假设物体获得的环境光
![image-24|309x227](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104763.png)

![image-25|504x314](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104764.png)
将三个部分加起来
### shading pipeline
对每个三角形/面进行颜色计算
![image-26|395x248](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104765.png)
对每个顶点计算法线，得到颜色，然后对于三角形内部的颜色计算插值
![[image-27.png|391x256]]
模型足够复杂的情况下，三种 shading 得到的效果差不多
![image-28|389x287](https://imag060625.oss-cn-beijing.aliyuncs.com/img/20251129223104766.png)




## Lecture 9：纹理映射

### 重心坐标
[Lecture 09 Shading 3 (Texture Mapping Cont.)\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1X7411F744?spm_id_from=333.788.videopod.episodes&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7&p=9)
重心坐标系
![|654x18](http://cdn.ljc0606.cn/obsidian/202507021627792.png)

![](http://cdn.ljc0606.cn/obsidian/202507021623114.png)
三角形平面上任意一点都可以通过三角形三个顶点表示，如果α、β、γ非负，则点在三角形内，α
+β+γ=1，保证在三角形平面内
重心坐标对应的α、β、γ值可以通过三角形面积求出，Aa 为顶点 A 不相邻的三角形
![](http://cdn.ljc0606.cn/obsidian/202507021626309.png)
重心的坐标
![|654x18](http://cdn.ljc0606.cn/obsidian/202507021628108.png)
由于三维到二维的过程坐标值会发生变化，所以先在三维空间做插值，再投影
![](http://cdn.ljc0606.cn/obsidian/202507021633382.png)
简单实现逻辑：
找到纹理坐标，找到对应颜色，应用
![](http://cdn.ljc0606.cn/obsidian/202507021634526.png)
### 问题 1：纹理失真
低分辨率纹理映射高分辨率图片
纹素：纹理上的像素
![](http://cdn.ljc0606.cn/obsidian/202507021636106.png)
#### 双线性插值
水平和竖直方向针对不同中心点的不同颜色进行插值
![](http://cdn.ljc0606.cn/obsidian/202507021639109.png)

### 纹理走样
纹理分辨率大于像素分辨率，形成了摩尔纹
远处的像素覆盖的纹理更多
![](http://cdn.ljc0606.cn/obsidian/202507021644939.png)

#### 超采样解决
采样频率高、代价大
![](http://cdn.ljc0606.cn/obsidian/202507021645080.png)

#### 避免采样解决（更优）
点查询、范围查询
Mipmap：快速近似正方形范围查询
![|647x18](http://cdn.ljc0606.cn/obsidian/202507021654174.png)
提前计算不同层级的 mipmap，引入额外存储是原图的三分之一
![](http://cdn.ljc0606.cn/obsidian/202507021655608.png)

如何计算 mipmap 层级
![](http://cdn.ljc0606.cn/obsidian/202507021658046.png)
将红色中心点以及周围中心点映射到纹理上，并且求出这些点在纹理上的距离
![](http://cdn.ljc0606.cn/obsidian/202507021658713.png)
#### 离散的若干层直接的连续
层与层之间的线性插值，三线性插值（非常非常广泛的应用！）
![](http://cdn.ljc0606.cn/obsidian/202507021703131.png)

#### mipmap 的局限性
只能查询一个方块内的插值，远处模糊
![](http://cdn.ljc0606.cn/obsidian/202507021706961.png)
#### 各向异性过滤
部分解决三线性插值，只能查询正方形区域，各向异性过滤可以查询一个长方形区域
![](http://cdn.ljc0606.cn/obsidian/202507021908960.png)

#### EWA 过滤
各向异性过滤对比 mipmap，各向异性过滤生成的图是原本的三倍
![|0x0](http://cdn.ljc0606.cn/obsidian/202507021909916.png)

[Learning/GAMES101笔记及作业/笔记/GAMES101 计算机图形学入门--闫令琪.md at master · zeroo0o0/Learning · GitHub](https://github.com/zeroo0o0/Learning/blob/master/GAMES101%E7%AC%94%E8%AE%B0%E5%8F%8A%E4%BD%9C%E4%B8%9A/%E7%AC%94%E8%AE%B0/GAMES101%20%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9B%BE%E5%BD%A2%E5%AD%A6%E5%85%A5%E9%97%A8--%E9%97%AB%E4%BB%A4%E7%90%AA.md)
[Games101作业1~8(含提高项)](https://zhuanlan.zhihu.com/p/1912986768225047690)

[OpenGL - LearnOpenGL CN](https://learnopengl-cn.github.io/01%20Getting%20started/01%20OpenGL/)

