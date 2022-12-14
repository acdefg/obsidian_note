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

### 安装 opencv
```
sudo apt-get install libopencv-dev
cd /usr/include 
sudo ln -s /usr/include/opencv4/opencv2 /usr/include/
```
[c++ 中——fatal error: opencv2/opencv.hpp: No such file or directory #include ＜opencv2/opencv.hpp＞_程序猿的探索之路的博客-CSDN博客](https://blog.csdn.net/nyist_yangguang/article/details/120442569)

### error messages
[ubuntu换源更新失败：The following signatures couldn‘t be verified because the public key is not available_sxiaocaicai的博客-CSDN博客](https://is.gd/phn0oR)
[Linux/Debian/Ubuntu报错解决：W: Target Packages (main/binary-amd64/Packages) is configured multiple times_zhangpeterx的博客-CSDN博客_w: target packages](https://is.gd/DHLCRc)
[Ubuntu 20 仓库 http://security.ubuntu.com/ubuntu xenial-security InRelease 没有数字签名 解决_CyrusZhou的博客-CSDN博客](https://is.gd/LBwf0y)
[E: Unable to locate package libjasper-dev的解决办法（亲测可以解决）_爱跑步的mango的博客-CSDN博客](https://is.gd/P6V8s3)
```
E: Package 'python-dev' has no installation candidate
E: Unable to locate package python-numpy
E: Unable to locate package libdc1394-22-dev
```
[linux/videodev.h : no such file or directory_Alex-xt的博客-CSDN博客_linux/videodev.h: no such file or directory](https://is.gd/QSi0mE)

直接安 4.x
[cmake - undefined reference to `png_set_longjmp_fn' when compiling PCL source file - Stack Overflow](https://is.gd/jkhxkT)

这个错误花了很长时间，在 open4.x 的文件夹里面的 cmakelist 加上这里面的话，我加在开头了

除了上面那个错误，其余可以参考这篇
[Ubuntu22.04安装opencv4并配置VsCode_南叔先生的博客-CSDN博客_vscode配置opencv4](https://is.gd/giATjh)
[Ubuntu22.04安装OpenCV4.5.1_Larry酷睿的博客-CSDN博客_ubuntu安装opencv4.5](https://is.gd/ARt86O)