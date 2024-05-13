# 嵌入式第十次作业

### 开发环境

> 操作系统：Ubuntu22.04
> 
> 编译工具：VScode
> 
> 编译环境：c++11，opencv4.7.0，cuda11.6

## 实验内容
请用 GPU 解决练习 9 的问题，并与上次作业中的方法比较计算时间。

## 实验原理
cuda 是 nivida 显卡配套的 toolkit，加速系统在运行程序时首先会运行 CPU 程序，在运行到需要 GPU 进行大规模并行计算的函数时，再将对应函数载入 GPU 执行。由 GPU 加速的依然还是纯 CPU 的应用程序，只是某些模块在运行时调入了 GPU 中，该模块在同步完毕后将会重新回到 CPU 中执行主程序的后续代码。
### cuda 编程框架

```c
//create varible pointer
unsigned char* datain;
//malloc gpu memory
cudaMalloc((void**)&datain, m_img.rows * m_img.cols * sizeof(unsigned char));
//将cpu数据传到gpu
cudaMemcpy(datain, src.data, src.rows * src.cols * sizeof(unsigned char), cudaMemcpyHostToDevice);
//定义线程数和模块数
size_t threadsnum = src.cols; // 定义每个block的thread数量

size_t blocksnum = src.rows; // 定义block的数量
//运行gpu函数
cannygpu<<<blocksnum, threadsnum>>>(datain, dataout);
//同步，gpu和cpu频率不同
cudaDeviceSynchronize();
//将gpu数据传输到cpu
cudaMemcpy(imgopmp2.data, dataout, src.rows * src.cols * sizeof(unsigned char), cudaMemcpyDeviceToHost);
//释放指针分配的内存
cudaFree(datain);

//定义gpu函数
__global__ void cannygpu(unsigned char* datain)

{

int index = threadIdx.x + 1 + (blockIdx.x + 1) * blockDim.x;

//printf("index:%d\n",index);

}
```

1. ` __global__ void GPUFunction()：`
__global__ 关键字表明该函数将在 GPU 上运行并可全局调用（ 既可以由 CPU ，也可以由 GPU 调用）；
通常，将在 CPU 上执行的代码称为 Host （主机）代码，而将在 GPU 上运行的代码称为 Device （设备）代码；
注意返回类型为 void。使用 __global__ 关键字定义的函数返回值需为 void 类型。
2. `GPUFunction<<<1, 1>>>()：`
通常，我们把要运行在 GPU 上的函数称为 kernel （核）函数;
启动核(kernel)函数时，我们必须事先配置 GPU 参数，使用 <<< ... >>> 语法向核函数传递两个必要的参数;
在 <<< ... >>> 中传递的参数用于为核函数设定线程的层次结构
第一个参数定义线程块(Block)的数量，第二个参数定义 Block 中含有的线程(Thread)数量。
CUDA线程的层次结构分为三层：Thread（线程）、Block（块）、Grid（网格），网格由块组成，块由线程组成。
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221231234731.png)
`someKernel<<<10, 10>>()` 在 GPU 中为该核函数分配 10 个具有 10 个线程的线程块，核函数中的代码将运行 100 次
3. `cudaDeviceSynchronize()：`
与其他并行化的代码类似，核函数启动方式为异步，即 CPU 代码将继续执行而不会等待核函数执行完成；
调用 CUDA 提供的函数 cudaDeviceSynchronize 可以让 Host 代码(CPU) 等待 Device 代码(GPU) 执行完毕，再在 CPU 上继续执行。

## 程序编写
首先，gpu 在中不能计算整数 为为参数的 `sqrt() 函数，先定义两个浮点数进行计算，在将结果放到数组中。
原来的处理程序：
```c
for (int i = 1; i < src.rows - 1; i++)
{
	for (int j = 1; j < src.cols - 1; j++)
	{
		imgopmp.data[i*imgopmp.step + j] = sqrt((src.data[(i - 1)*src.step + j + 1]
			+ 2 * src.data[i*src.step + j + 1]
			+ src.data[(i + 1)*src.step + j + 1]
			- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
			- src.data[(i + 1)*src.step + j - 1])    //gx
			*(src.data[(i - 1)*src.step + j + 1]
			+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
			- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
			- src.data[(i + 1)*src.step + j - 1])    //gx
			+ (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
			+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
			- 2 * src.data[(i + 1)*src.step + j]
			- src.data[(i + 1)*src.step + j + 1])    //gy
			* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
			+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
			- 2 * src.data[(i + 1)*src.step + j]
			- src.data[(i + 1)*src.step + j + 1]));  //gy
	}
}
```
cuda 将加速使用程序：
```c
int index = threadIdx.x + 1 + (blockIdx.x + 1) * blockDim.x;
float colorx;
float colory;
colorx = datain[i + width + 1]
		+ 2 * datain[i + 1]
		+ datain[i + width + 1]
		- datain[i - width - 1] - 2 * datain[i - 1]
		- datain[i + width - 1];
colory = datain[i - width - 1] + 2 * datain[i - width]
		+ datain[i + width + 1] - datain[i + width - 1]
		- 2 * datain[i + width]
		- datain[i + width + 1];  //gy
dataout[i] = sqrt(colorx*colorx+colory*colory);
```
threadIdx.x 线程号 
blockIdx.x 线程块号 
blockDim.x 块中 x 为维度线程的个数（这里表示块中所有线程数）

其次，gpu 的的线程数量和块数量有限制，可以在 [CUDA - Wikipedia](https://en.wikipedia.org/wiki/CUDA) 这里查询到：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022222652.png)
gpu 算力为 8.6 的对应创建单个块中线程上限为 1024，x 为维块个数上限较大
gpu 算力可以通过编程得到，程序在附录中，运行结果如下，也可以在上面链接中搜索得到
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022340476.png)

### 较小的图片
对于比较小的图片，可以让线程号对应列数，块对应行：

```c
//定义并行进程数和进程块数
size_t threadsnum = src.cols; // 定义每个block的thread数量
size_t blocksnum = src.rows; // 定义block的数量
//索引号，此时width = blockdim
int index = threadIdx.x + 1 + (blockIdx.x + 1) * blockDim.x;
```
运行结果：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022345848.png)

```c
origin use time: 171862
openmp use time: 34064
gpu cuda use time: 41
image size : 512 * 510
```

### 较大的图片
对于较大的图片，线程数量有限制不能等于图片列宽，对于块的数量，一般情况下都会够用，但是也可以采取下面的方法。
当进程数量小于处理数组数据量时，一种方法是可以让同一个进程执行多次，另一种是继续扩大进程数量。

**让同一个进程执行多次：**
```c
int stride = gridDim.x * blockDim.x;  //一个网格中线程块数量*块中线程数量= 一个网格中线程数量
for(int i = index; i < N; i += stride)
{
	colorx = datain[i + width + 1]
			+ 2 * datain[i + 1]
			+ datain[i + width + 1]
			- datain[i - width - 1] - 2 * datain[i - 1]
			- datain[i + width - 1];
	colory = datain[i - width - 1] + 2 * datain[i - width]
			+ datain[i + width + 1] - datain[i + width - 1]
			- 2 * datain[i + width]
			- datain[i + width + 1];  //gy
	dataout[i] = sqrt(colorx*colorx+colory*colory);
	if(i>N) printf("!!!");
}
```
运行结果如下，完整代码在附录中：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022316707.png)

**增加进程数量：**
增加进程数量可以增加维度，这里做图片的处理，块的数量足够，递增变量维度也仅需要二维，增加块数量即可
```c
int N = m_img.rows * m_img.cols;
size_t threadsnum = 1024; // 定义thread数量为最大值
size_t blocksnum = (N + threadsnum - 1)/threadsnum; // block的数量，使得浪费的进程数量最小

//gpu函数中：
if(index < N)   //在数组索引范围内的进程才执行，可以加快运行速度
{
	colorx = datain[index + width + 1]
			+ 2 * datain[i + 1]
			+ datain[index + width + 1]
			- datain[index - width - 1] - 2 * datain[index - 1]
			- datain[index + width - 1];
	colory = datain[index - width - 1] + 2 * datain[index - width]
			+ datain[index + width + 1] - datain[index + width - 1]
			- 2 * datain[index + width]
			- datain[index + width + 1];  //gy
	dataout[index] = sqrt(colorx*colorx+colory*colory);
	//if(i>N) printf("!!!");
}
```
运行结果如下，完整代码在附录中：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301022319208.png)

### 实验结论
可以从运行结果中看出，使用 gpu 对于程序速度有巨大的提升，是成百倍的，且在处理数据量范围内，进程数量越多，运行速度越快 。 

## 实验过程记录
> [!info]
> 
以下是环境安装和测试记录

### cuda 安装
参考：[Ubuntu 22.04 安装cuda，适用20.04_AIhub的博客-CSDN博客_ubuntu22.04安装cuda](https://is.gd/H3L2qQ)
[kali linux 安装CUDA 11.6问题总结 - FreeBuf网络安全行业门户](https://www.freebuf.com/sectool/328870.html)
#### 安装驱动 
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291206968.png)
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291221320.png)
查看是否安装成功，以及最大 cuda 不版本支持
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212291452085.png)
2. 安装 cuda toolkit，参考 [kali linux 安装CUDA 11.6问题总结 - FreeBuf网络安全行业门户](https://www.freebuf.com/sectool/328870.html)，过程省略

###  问题总结
#### 问题 1：缺少 liburcu6
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292226234.png)

```
sudo vim /etc/apt/sources.list
```
加上
```
deb http://ftp.de.debian.org/debian bullseye main
```
update 一下
#### 问题 2：没有 key
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292233359.png)
选了其中的一个 key，添加一下，会有 warning，正确解决如链接：[apt key - Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead - Stack Overflow](https://is.gd/hoVl1w) 事实上，不用管 warning，直接再 update 阿安装就可以了
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292234809.png)
按顺序安装下面两个
```zsh
sudo apt-get install liburcu6 
sudo apt-get -y install cuda 
```
至此安装完成，查看一下下
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292241297.png)
添加环境变量，这条我改过
```zsh
zshconfig
```
添加这两句
```txt
export  PATH=/usr/local/cuda/bin:$PATH  
export  LD_LIBRARY_PATH=/usr/local/cuda/lib64$LD_LIBRARY_PATH
```
输入 `nvcc -V`，查看更改成功
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212292244738.png)
#### 问题 3：gcc,g++降级编译
运行出现下面问题：
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212301822118.png)
根据这篇把 gcc/g++降级了，还是不行 [error: parameter packs not expanded with ‘...’ · Issue #119 · NVlabs/instant-ngp · GitHub](https://github.com/NVlabs/instant-ngp/issues/119)
降级新办法：[[ubuntu][原创]ubuntu gcc g++降级方法_FL1623863129 的博客-CSDN 博客_ubuntu 22 gcc12 降到 11](https://blog.csdn.net/FL1623863129/article/details/115192387)

```shell
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 48
```

### opencv 的 cuda 在支持安装
上次安装的时候忘记加上 cuda 支持了，这次重新安装：
#### 下载准备
```shell
git clone https://github.com/opencv/opencv_contrib.git
git clone https://github.com/opencv/opencv.git
```
下载后，将 opencv_contrib 文件夹移动到 opencv 中去。

```shell
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```
会少 python2-dev，python-numpy，lib...，lib 年那个加了个 source list [[zuoye9#error messages]]

video.h
```
cp ./modules/videoio/include/opencv2/videoio/videoio_c.h /usr/include/sys/videoio.h
cp ./modules/videoio/include/opencv2/videoio/videoio_c.h /usr/include/video/videoio.h
```

#### 编译 cmake 选项
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311216796.png)

使用的：
1. 在 opencv 名目录下，新建 build，进去

```shell
mkdir build
cd build
```

```shell
cmake -D CMAKE_INSTALL_PREFIX=/usr/local -D CMAKE_BUILD_TYPE=Release -D OPENCV_GENERATE_PKGCONFIG=ON -D ENABLE_CXX11=1 -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules -D OPENCV_ENABLE_NONFREE=True -D INSTALL_PYTHON_EXAMPLES=ON -D INSTALL_C_EXAMPLES=ON -D WITH_CUDA=ON -D WITH_TBB=ON -D ENABLE_FAST_MATH=1 -D WITH_OPENMP=ON -D WITH_CUFFT=ON -D WITH_CUBLAS=ON ..
```

-D BUILD_opencv_world=ON   编译生成 libopencv_world.so
-D CUDA_NVCC_FLAGS=–expt-relaxed-constexpr  使用 abs
建个.sh 文件放在目录下
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_C_COMPILER=gcc-10.4 \
      -D CMAKE_CXX_COMPILER=g++-10.4 \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_ENABLE_NONFREE=ON \
      -D WITH_CUDA=ON \
....//文件很长不列出了
```

#### cmake-gui
以上设置比较麻烦，使用gui更加简单
安装：
```shell
sudo apt-get install cmake-qt-gui
```

运行：
```shell
cmake-gui
cmake-gui ..(在源码目录下打开)
```

将 cuda 新选中，删掉不必要的模块，需要制定 gcc，g++编译器
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301030018681.png)
configure
specify cmplier 找\\usr\\bin 下的 gcc

##### 问题
###### 问题 1：png

> [!failure]
> 
undefined reference to `png_set_longjmp_fn'

[cmake - undefined reference to png_set_longjmp_fn when compiling PCL source file - Stack Overflow](https://stackoverflow.com/questions/36220123/undefined-reference-to-png-set-longjmp-fn-when-compiling-pcl-source-file)

cmake list 了里面加上这个
```
# LibPNG
option(WITH_PNG "PNG file support" TRUE)
if(WITH_PNG)
    # search for pkg-config
    include (FindPkgConfig)
    if (NOT PKG_CONFIG_FOUND)
        message (FATAL_ERROR "pkg-config not found")
    endif ()

    # check for libpng
    pkg_check_modules (LIBPNG libpng16 REQUIRED)
    if (NOT LIBPNG_FOUND)
        message(FATAL_ERROR "You don't seem to have libpng16 development libraries installed")
    else ()
        include_directories (${LIBPNG_INCLUDE_DIRS})
        link_directories (${LIBPNG_LIBRARY_DIRS})
        link_libraries (${LIBPNG_LIBRARIES})
    endif ()
endif(WITH_PNG)
```

###### 问题 2：ade

> [!failure]
> 
CMake Error at modules/gapi/cmake/DownloadADE.cmake:23 (add_library):  
No SOURCES given to target: ade  
Call Stack (most recent call first):  
modules/gapi/cmake/init.cmake:20 (include)  
cmake/OpenCVModule.cmake:298 (include)  
cmake/OpenCVModule.cmake:361 (_add_modules_1)  
cmake/OpenCVModule.cmake:385 (ocv_glob_modules)  
CMakeLists.txt:971 (ocv_register_modules)

![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311525522.png)

#### make&install
```shell
make -j8
```
查看最大核数，那就最大可以-16
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311725826.png)
装了一天终于成了
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011441400.png)

##### 问题
###### 问题 1：boostdesc_bgm.i 其缺失
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311704695.png)

download.sh 在 opencv 个根目录下运行这个脚本，操作下面隐藏文件夹 cache
```shell
#!/bin/bash
cd ./.cache/xfeatures2d/
cd boostdesc

curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_lbgm.i > 0ae0675534aa318d9668f2a179c2a052-boostdesc_lbgm.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_binboost_256.i > e6dcfa9f647779eb1ce446a8d759b6ea-boostdesc_binboost_256.i
...//不列出了
```
然后下载了一个补充文件，放在 opencv_contrib/modules/xfeatures2d/src/目录下：
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311746943.png)
想安装 tree，查看文件结构，还没好
###### 问题 2：nvopticalFlow.h 没有
把 cudaflow 等一些没必要的 module 给关掉，重来就好了

#### 配置环境变量
##### 从 cmake option 配置 log

```shell
zshconfig
# bash用：
sudo gedit /etc/bash.bashrc

# 加入
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
export PKG_CONFIG_PATH

# update
update
# or
sudo updatedb
```

```shell
#打开下列文件
sudo gedit /etc/ld.so.conf.d/opencv.conf 
 
# 添加lib路經
/usr/local/lib
 
# 更新
sudo ldconfig
```

查看版本号
```shell
pkg-config --modversion opencv4
```
更改前：
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011456367.png)
更改后：
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011526487.png)

##### 问题：opencv.pc
[linux下编译安装opencv生成opencv.pc_浓茶淡酒的博客-CSDN博客](https://blog.csdn.net/s15810751918/article/details/107705387)

#### 测试
到这个目录下面执行：
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011545157.png)
##### 问题：报错
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011546064.png)
解决：
```shell
sudo cmake .. -DCMAKE_CXX_COMPILER=$(which g++) -DCMAKE_C_COMPILER=$(which gcc)
```
参考：
[[已解决] The C++ compiler "/usr/local/bin/c++" is not able to compile a simple test program._HeyMountain 的博客-CSDN 博客_cmake 在编译你的 c 或 c++代码前，会先验证你指定的编译器是否可以正常工作](https://is.gd/VJi6Vm)
[Ubuntu下安装opencv并进行测试_带你去网吧里偷耳机的博客-CSDN博客_检测opencv是否安装成功 ubuntu](https://blog.csdn.net/qq_40123329/article/details/103904087)

#### 编译方法
可以设置 cmake，配置 vscode，这里使用最简单的方法：

```shell
nvcc -std=c++11 `pkg-config --cflags opencv4` cuda_image.cu `pkg-config --libs opencv4` -o image
```
cuda_image.cu  --- source code 
image  --- output
以上编译方法要检查好配置了 pkg-config cmake 选项、配好了环境变量、以及生成了 opencv4. pc 文件

## 附录

### 较小图片
```c
#include<opencv2/opencv.hpp>
#include<iostream>
#include "omp.h"
#include <time.h>

using namespace std;
using namespace cv;

__global__ void cannygpu(unsigned char* datain, unsigned char* dataout)
{
	int index = threadIdx.x + 1 + (blockIdx.x + 1) * blockDim.x;
	float colorx;
	float colory;
	colorx = datain[index + blockDim.x + 1]
			+ 2 * datain[index + 1]
			+ datain[index + blockDim.x + 1]
			- datain[index - blockDim.x - 1] - 2 * datain[index - 1]
			- datain[index + blockDim.x - 1];
	colory = datain[index - blockDim.x - 1] + 2 * datain[index - blockDim.x]
			+ datain[index + blockDim.x + 1] - datain[index + blockDim.x - 1]
			- 2 * datain[index + blockDim.x]
			- datain[index + blockDim.x + 1];  //gy
	dataout[index] = sqrt(colorx*colorx+colory*colory);
	//printf("[index]: %d datain: %d  dataout: %d\n", index, datain[index], dataout[index]);
	//printf("index:%d\n",index);
}

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
				- src.data[(i + 1)*src.step + j - 1])    //gx
				*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])    //gx
				+ (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])    //gy
				* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));  //gy
		}
	}
	clock_t end2 = clock();

//gpu
    Mat imgopmp2(src.rows, src.cols, CV_8UC1, Scalar(0));
	unsigned char* datain;
    unsigned char* dataout;

    cudaMalloc((void**)&datain, m_img.rows * m_img.cols * sizeof(unsigned char));
    cudaMalloc((void**)&dataout, m_img.rows * m_img.cols * sizeof(unsigned char));

	cudaMemcpy(datain, src.data, src.rows * src.cols * sizeof(unsigned char), cudaMemcpyHostToDevice);

	size_t threadsnum = src.cols; // 定义每个block的thread数量
  	size_t blocksnum = src.rows; // 定义block的数量

	clock_t start3 = clock();
	cannygpu<<<blocksnum, threadsnum>>>(datain, dataout);
  	cudaDeviceSynchronize();
	
	clock_t end3 = clock();

	cudaMemcpy(imgopmp2.data, dataout, src.rows * src.cols * sizeof(unsigned char), cudaMemcpyDeviceToHost);

	cudaFree(datain);
	cudaFree(dataout);

	printf("origin use time: %d\n",end1-start1); 
	printf("openmp use time: %d\n",end2-start2);
	printf("gpu cuda use time: %d\n",end3-start3);
	printf("image size : %d * %d\n",src.rows,src.cols);

	//printf("accelarate ratio: %f\n",double((double)(end2-start2)/(double)(end1-start1)));
	imshow("原图", src);
	imshow("gradient", imgopmp);
	imshow("cuda", imgopmp2);

	waitKey(0);
	return 0;
}
```

### 较大图片
#### 程序 1

```c
#include<opencv2/opencv.hpp>
#include<iostream>
#include "omp.h"
#include <time.h>

using namespace std;
using namespace cv;

//当图片边长大于1024时，使用
__global__ void cannygpu(unsigned char* datain, unsigned char* dataout, int width, int N)
{
	int index = threadIdx.x + 1 + (blockIdx.x + 1) * blockDim.x;
	int stride = gridDim.x * blockDim.x;  //一个网格中线程块数量*块中线程数量
	float colorx;
	float colory;

	for(int i = index; i < N; i += stride)
	{
		colorx = datain[i + width + 1]
				+ 2 * datain[i + 1]
				+ datain[i + width + 1]
				- datain[i - width - 1] - 2 * datain[i - 1]
				- datain[i + width - 1];
		colory = datain[i - width - 1] + 2 * datain[i - width]
				+ datain[i + width + 1] - datain[i + width - 1]
				- 2 * datain[i + width]
				- datain[i + width + 1];  //gy
		dataout[i] = sqrt(colorx*colorx+colory*colory);
		if(i>N) printf("!!!");
	}
	//printf("[index]: %d datain: %d  dataout: %d\n", index, datain[index], dataout[index]);
	//printf("index:%d\n",index);
}

int main()
{
	Mat m_img = imread("2.jpg");
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
				- src.data[(i + 1)*src.step + j - 1])    //gx
				*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])    //gx
				+ (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])    //gy
				* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));  //gy
 
 
		}
	}
	clock_t end2 = clock();

//gpu
    Mat imgopmp2(src.rows, src.cols, CV_8UC1, Scalar(0));
	unsigned char* datain;
    unsigned char* dataout;
	int N = m_img.rows * m_img.cols;

    cudaMalloc((void**)&datain, N * sizeof(unsigned char));
    cudaMalloc((void**)&dataout, N * sizeof(unsigned char));

	cudaMemcpy(datain, src.data, N * sizeof(unsigned char), cudaMemcpyHostToDevice);

	size_t threadsnum = 1024; // 定义每个block的thread数量
  	size_t blocksnum = src.rows; // 定义block的数量

	clock_t start3 = clock();
	cannygpu<<<blocksnum, threadsnum>>>(datain, dataout,src.cols,N);
  	cudaDeviceSynchronize();
	
	clock_t end3 = clock();

	cudaMemcpy(imgopmp2.data, dataout, src.rows * src.cols * sizeof(unsigned char), cudaMemcpyDeviceToHost);

	cudaFree(datain);
	cudaFree(dataout);

	printf("origin use time: %d\n",end1-start1); 
	printf("openmp use time: %d\n",end2-start2);
	printf("gpu cuda use time: %d\n",end3-start3);
	printf("image size : %d * %d\n",src.rows,src.cols);

	printf("accelarate ratio: %f\n",double((double)(end3-start3)/(double)(end1-start1)));

	imshow("原图", src);
	imshow("gradient", imgopmp);
	imshow("cuda", imgopmp2);

	waitKey(0);
	return 0;
}
```
#### 程序 2

```c
#include<opencv2/opencv.hpp>
#include<iostream>
#include "omp.h"
#include <time.h>

using namespace std;
using namespace cv;

//当图片边长大于1024时，使用
__global__ void cannygpu(unsigned char* datain, unsigned char* dataout, int width, int N)
{
	int index = threadIdx.x + 1 + (blockIdx.x + 1) * blockDim.x;
	//int stride = gridDim.x * blockDim.x;  //一个网格中线程块数量*块中线程数量
	float colorx;
	float colory;

	if(index < N)
	{
		int i = index;
	//for(int i = index; i < N; i += stride)
	{
		colorx = datain[i + width + 1]
				+ 2 * datain[i + 1]
				+ datain[i + width + 1]
				- datain[i - width - 1] - 2 * datain[i - 1]
				- datain[i + width - 1];
		colory = datain[i - width - 1] + 2 * datain[i - width]
				+ datain[i + width + 1] - datain[i + width - 1]
				- 2 * datain[i + width]
				- datain[i + width + 1];  //gy
		dataout[i] = sqrt(colorx*colorx+colory*colory);
		//if(i>N) printf("!!!");
	}

	}
	//printf("[index]: %d datain: %d  dataout: %d\n", index, datain[index], dataout[index]);
	//printf("index:%d\n",index);
}

int main()
{
	Mat m_img = imread("2.jpg");
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
    
	Mat imgopmp(src.rows, src.cols, CV_8UC1, Scalar(0));
	clock_t start2 = clock();
	#pragma omp parallel for
	for (int i = 1; i < src.rows - 1; i++)
	{
		//#pragma omp parallel for
		for (int j = 1; j < src.cols - 1; j++)
		{
		    imgopmp.data[i*imgopmp.step + j] = sqrt((src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1]
				+ src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])    //gx
				*(src.data[(i - 1)*src.step + j + 1]
				+ 2 * src.data[i*src.step + j + 1] + src.data[(i + 1)*src.step + j + 1]
				- src.data[(i - 1)*src.step + j - 1] - 2 * src.data[i*src.step + j - 1]
				- src.data[(i + 1)*src.step + j - 1])    //gx
				+ (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1])    //gy
				* (src.data[(i - 1)*src.step + j - 1] + 2 * src.data[(i - 1)*src.step + j]
				+ src.data[(i - 1)*src.step + j + 1] - src.data[(i + 1)*src.step + j - 1]
				- 2 * src.data[(i + 1)*src.step + j]
				- src.data[(i + 1)*src.step + j + 1]));  //gy
		}
	}
	clock_t end2 = clock();

//gpu

    Mat imgopmp2(src.rows, src.cols, CV_8UC1, Scalar(0));
	unsigned char* datain;
    unsigned char* dataout;
	int N = m_img.rows * m_img.cols;

    cudaMalloc((void**)&datain, N * sizeof(unsigned char));
    cudaMalloc((void**)&dataout, N * sizeof(unsigned char));

	cudaMemcpy(datain, src.data, N * sizeof(unsigned char), cudaMemcpyHostToDevice);

	size_t threadsnum = 1024; // 定义每个block的thread数量
  	size_t blocksnum = (N + threadsnum - 1)/threadsnum; // 定义block的数量

	clock_t start3 = clock();
	cannygpu<<<blocksnum, threadsnum>>>(datain, dataout,src.cols,N);
  	cudaDeviceSynchronize();
	
	clock_t end3 = clock();

	cudaMemcpy(imgopmp2.data, dataout, src.rows * src.cols * sizeof(unsigned char), cudaMemcpyDeviceToHost);

	cudaFree(datain);
	cudaFree(dataout);

	printf("origin  use time: %d\n",end1-start1); 
	printf("openmp  use time: %d\n",end2-start2);
	printf("gpucuda use time: %d\n",end3-start3);
	printf("image size : %d * %d\n",src.rows,src.cols);

	printf("accelarate ratio: %f\n",double((double)(end3-start3)/(double)(end1-start1)));

	//测试处理正确性
	//imshow("原图", src);
	//imshow("origin", imgorig);
	//imshow("openmp", imgopmp);
	//imshow("cuda", imgopmp2);

	waitKey(0);
	return 0;
}
```
