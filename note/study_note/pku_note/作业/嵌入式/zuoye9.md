### MAT
[OpenCV学习笔记：Mat类详解（一） - 月夜_1 - 博客园](https://www.cnblogs.com/zb-ml/articles/8856778.html)
**CV_8UC3**，具体含义为“3 通道 8 位无符号数”。

使用举例：

```
Mat src(10,10,CV_32FC3);
```

**.depth 函数**

```
int cv::Mat::depth()  
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


### log


2.jpg
origin use time: 12806
openmp use time: 8295
openmp2 use time: 8247
accelarate ratio: 0.643995

1.jpg
origin use time: 22214
openmp use time: 8468
openmp2 use time: 8581

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212142111632.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212142111601.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212142112108.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212142112202.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212142112099.png)
openmp full use time: 74714
openmp gradienty use time: 18683
openmp gradientx use time: 37674


2.jpg
origin full use time: 33763
openmp full use time: 8234
origin y use time: 2297
openmp y use time: 2308
origin x use time: 4538
openmp x use time: 4514

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212142122123.png)

#### summary
x 方向代码有平方运算
step 含义
加上一些配置过程
介绍 soble 

```c++
//unused

#include<opencv2/opencv.hpp>
#include<iostream>
#include "omp.h"
#include <time.h>

using namespace std;
using namespace cv;

int main()
{
	Mat m_img = imread("1.jpg");
	Mat src(m_img.rows, m_img.cols, CV_8UC1, Scalar(0));
	cvtColor(m_img, src, COLOR_RGB2GRAY);

	clock_t start = clock();
	Mat dstImage(src.rows, src.cols, CV_8UC1, Scalar(0));
	for (int i = 1; i < src.rows - 1; i++)
	{
		for (int j = 1; j < src.cols - 1; j++)
		{
			dstImage.data[i*dstImage.step + j] = sqrt((src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1]
				+ src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1]) + (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));
 
		}
 
	}
	clock_t end = clock();
	printf("origin full use time: %d\n",end-start);

	clock_t start1 = clock();
	Mat dstImage1(src.rows, src.cols, CV_8UC1, Scalar(0));
	#pragma omp parallel for
	for (int i = 1; i < src.rows - 1; i++)
	{
		//#pragma omp parallel for
		for (int j = 1; j < src.cols - 1; j++)
		{
			dstImage1.data[i*dstImage1.step + j] = sqrt((src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1]
				+ src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1]) + (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));
 
		}
 
	}
	clock_t end1 = clock();
	printf("openmp full use time: %d\n",end1-start1);

	clock_t start2 = clock();
	Mat grad_y(src.rows, src.cols, CV_8UC1, Scalar(0));
	{
		for (int i = 1; i < src.rows - 1; i++)
		{
			for (int j = 1; j < src.cols - 1; j++)
			{
				grad_y.data[i*grad_y.step + j] = abs((src.data[(i - 1)*src.step + j + 1]
					+ 2 * src.data[i*src.step + j + 1]
					+ src.data[(i + 1)*src.step + j + 1]
					- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
					- src.data[(i + 1)*src.step + j - 1]));
			}
		}
	}
	clock_t end2 = clock();
	printf("origin y use time: %d\n",end2-start2);

	clock_t start3 = clock();
	Mat grad_y1(src.rows, src.cols, CV_8UC1, Scalar(0));
	{
		#pragma omp parallel for
		for (int i = 1; i < src.rows - 1; i++)
		{
			for (int j = 1; j < src.cols - 1; j++)
			{
				grad_y1.data[i*grad_y1.step + j] = abs((src.data[(i - 1)*src.step + j + 1]
					+ 2 * src.data[i*src.step + j + 1]
					+ src.data[(i + 1)*src.step + j + 1]
					- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
					- src.data[(i + 1)*src.step + j - 1]));
			}
		}
	}
	clock_t end3 = clock();
	printf("openmp y use time: %d\n",end3-start3);

	clock_t start4 = clock();
	Mat grad_x(src.rows, src.cols, CV_8UC1, Scalar(0));
	{
		for (int i = 1; i < src.rows - 1; i++)
		{
			for (int j = 1; j < src.cols - 1; j++)
			{
				grad_x.data[i*grad_x.step + j] = sqrt((src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
					+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
					- 2 * src.data[(i + 1)*src.step + j]
					- src.data[(i + 1)*src.step + j + 1])* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
					+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
					- 2 * src.data[(i + 1)*src.step + j]
					- src.data[(i + 1)*src.step + j + 1]));
			}
		}
	}
	clock_t end4 = clock();
	printf("origin x use time: %d\n",end4-start4);

	clock_t start5 = clock();
	Mat grad_x1(src.rows, src.cols, CV_8UC1, Scalar(0));
	{
		#pragma omp parallel for
		for (int i = 1; i < src.rows - 1; i++)
		{
			for (int j = 1; j < src.cols - 1; j++)
			{
				grad_x1.data[i*grad_x1.step + j] = sqrt((src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
					+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
					- 2 * src.data[(i + 1)*src.step + j]
					- src.data[(i + 1)*src.step + j + 1])* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
					+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
					- 2 * src.data[(i + 1)*src.step + j]
					- src.data[(i + 1)*src.step + j + 1]));
			}
		}
	}
	clock_t end5 = clock();
	printf("openmp x use time: %d\n",end5-start5);

	// imshow("原图", src);
	// imshow("gradient", dstImage);
	// imshow("Vertical gradient", grad_y);
	// imshow("Horizontal gradient", grad_x);
 
	waitKey(0);
	return 0;
}
```

```c++

#include<opencv2/opencv.hpp>
#include<iostream>
#include "omp.h"
#include <time.h>

using namespace std;
using namespace cv;

int main()
{
	Mat m_img = imread("1.jpg");
	Mat src(m_img.rows, m_img.cols, CV_8UC1, Scalar(0));
	cvtColor(m_img, src, COLOR_RGB2GRAY);

	Mat imgorig(src.rows, src.cols, CV_8UC1, Scalar(0));
	clock_t start1 = clock();
	for (int i = 1; i < src.rows - 1; i++)
	{
		for (int j = 1; j < src.cols - 1; j++)
		{
		    imgorig.data[i*imgorig.step + j] = sqrt((src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1]
				+ src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1]) + (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));
 
		}
	}
	clock_t end1 = clock();
	printf("origin use time: %d\n",end1-start1); 
    
	Mat imgopmp(src.rows, src.cols, CV_8UC1, Scalar(0));
	clock_t start2 = clock();
	#pragma omp parallel for
	for (int i = 1; i < src.rows - 1; i++)
	{
		#pragma omp parallel for
		for (int j = 1; j < src.cols - 1; j++)
		{
		    imgopmp.data[i*imgopmp.step + j] = sqrt((src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1]
				+ src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1]) + (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));
 
		}
	}
	clock_t end2 = clock();
	printf("openmp use time: %d\n",end2-start2);

    Mat imgopmp2(src.rows, src.cols, CV_8UC1, Scalar(0));
	clock_t start3 = clock();
	#pragma omp parallel for
	for (int i = 1; i < src.rows - 1; i++)
	{
		//#pragma omp parallel for
		for (int j = 1; j < src.cols - 1; j++)
		{
		    imgopmp2.data[i*imgopmp2.step + j] = sqrt((src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1]
				+ src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1]) + (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));
 
		}
	}
	clock_t end3 = clock();
	printf("openmp2 use time: %d\n",end3-start3);
	
	printf("accelarate ratio: %f\n",double((double)(end3-start3)/(double)(end1-start1)));
	// imshow("原图", src);
	// imshow("gradient", imgopmp);
 
	waitKey(0);
	return 0;
}
```