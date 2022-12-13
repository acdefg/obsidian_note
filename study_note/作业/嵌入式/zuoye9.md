### MAT
[OpenCV学习笔记：Mat类详解（一） - 月夜_1 - 博客园](https://www.cnblogs.com/zb-ml/articles/8856778.html)
**CV_8UC3**，具体含义为“3 通道 8 位无符号数”。

使用举例：

```
Mat src(10,10,CV_32FC3);
```

**.depth 函数**

```
int cv::Mat::depth()    const
```

返回图像深度，即矩阵元素的存储方式

**at 函数**  
at函数的功能是访问矩阵元素，根据不同的使用场景，有多个重载函数可供选择。  
如，访问一个二维的矩阵，可用at函数原型为：

```
_Tp& cv::Mat::at(int i0,int i1)
```

使用方法举例：
```
Mat src = imread("test.jpg");
int elem = src.at<int>(0,0);
```
访问 test.jpg 图像的（0 , 0）元素

### Soble
[sobel算子原理与实现_写代码的胡歌的博客-CSDN博客_sobel算子原理](https://blog.csdn.net/qq_37124237/article/details/82183177)

