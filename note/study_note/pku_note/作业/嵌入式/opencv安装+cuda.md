## 下载准备
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


## 编译 cmake 选项
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311216796.png)

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
#### 看看，建个.sh 文件放在目录下
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_C_COMPILER=gcc-10.4 \
      -D CMAKE_CXX_COMPILER=g++-10.4 \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_ENABLE_NONFREE=ON \
      -D WITH_CUDA=ON \
      -D CUDA_ARCH_PTX="" \
      -D CUDA_ARCH_BIN="8.6" \
      -D CUDA_FAST_MATH=1 \
      -D WITH_CUBLAS=ON \
      -D WITH_CUFFT=ON  \
      -D WITH_LAPACK=0 \
      -D WITH_NVCUVID=0 \
      -D WITH_TBB=ON \
      -D WITH_IPP=OFF \
      -D WITH_V4L=ON \
      -D WITH_OPENGL=ON \
      -D WITH_OPENCL=OFF \
      -D WITH_OPENMP=ON  \
      -D WITH_QT=ON \
      -D WITH_GTK=OFF \
      -D FORCE_VTK=ON \
      -D WITH_EIGEN=ON \
      -D EIGEN_INCLUDE_DIR=/usr/include/eigen3 \
      -D WITH_XINE=ON \
      -D WITH_GDAL=ON \
      -D WITH_1394=OFF \
      -D WITH_FFMPEG=ON \
      -D WITH_GSTREAMER=ON \
      -D HAVE_VIDEOIO=1 \
      -D BUILD_PROTOBUF=ON \
      -D BUILD_JAVA=ON \
      -D BUILD_JASPER=ON \
      -D BUILD_opencv_python2=ON \
      -D BUILD_opencv_python3=ON \
      -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib-4.4.0/modules \
      -D INSTALL_PYTHON_EXAMPLES=ON \
      -D INSTALL_C_EXAMPLES=ON \
      -D BUILD_PERF_TESTS=OFF \
      -D BUILD_EXAMPLES=OFF ..
```

#### 问题
有大病问题，这么清楚的 build 看不见瞎嘛，总而言之删了重新解压，重新操作一遍就好了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311213569.png)

之前出现这个问题是要将 gcc，g++降级，我明明降过了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311224803.png)

emmmm，路径带括号，什么路径，就是（cpoy）别用了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311415048.png)

这个说改 cmake -D CUDA_ARCH_BIN="8.6" 这个选项，emmmm，我不这么改了，换成 cmake-gui
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311420494.png)

### cmake-gui
以上设置更加简单的方法
#### 安装

```shell
sudo apt-get install cmake-qt-gui
```

运行：

```shell
cmake-gui
cmake-gui ..(在源码目录下打开)
```

#### 编译 opencv
configure
specify cmplier 找\\usr\\bin 下的 gcc
改选项
#### 问题 1：png

> [!failure]
> 
undefined reference to `png_set_longjmp_fn'

[cmake - undefined reference to `png_set_longjmp_fn' when compiling PCL source file - Stack Overflow](https://stackoverflow.com/questions/36220123/undefined-reference-to-png-set-longjmp-fn-when-compiling-pcl-source-file)

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

#### 问题 2：ade

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

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311525522.png)

## make&install

```shell
make -j8
```
查看最大核数，那就最大可以-16
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311725826.png)
装了一天终于成了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011441400.png)

#### 问题 1：boostdesc_bgm.i
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311704695.png)

download.sh 在 opencv 个根目录下运行这个脚本，操作下面隐藏文件夹 cache
```shell
#!/bin/bash
cd ./.cache/xfeatures2d/
cd boostdesc

curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_lbgm.i > 0ae0675534aa318d9668f2a179c2a052-boostdesc_lbgm.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_binboost_256.i > e6dcfa9f647779eb1ce446a8d759b6ea-boostdesc_binboost_256.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_binboost_128.i > 98ea99d399965c03d555cef3ea502a0b-boostdesc_binboost_128.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_binboost_064.i > 202e1b3e9fec871b04da31f7f016679f-boostdesc_binboost_064.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_bgm_hd.i > 324426a24fa56ad9c5b8e3e0b3e5303e-boostdesc_bgm_hd.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_bgm_bi.i > 232c966b13651bd0e46a1497b0852191-boostdesc_bgm_bi.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/34e4206aef44d50e6bbcd0ab06354b52e7466d26/boostdesc_bgm.i > 0ea90e7a8f3f7876d450e4149c97c74f-boostdesc_bgm.i
cd ../vgg
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_120.i > 151805e03568c9f490a5e3a872777b75-vgg_generated_120.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_64.i > 7126a5d9a8884ebca5aea5d63d677225-vgg_generated_64.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_48.i > e8d0dcd54d1bcfdc29203d011a797179-vgg_generated_48.i
curl https://raw.githubusercontent.com/opencv/opencv_3rdparty/fccf7cd6a4b12079f73bbfb21745f9babcd4eb1d/vgg_generated_80.i > 7cd47228edec52b6d82f46511af325c5-vgg_generated_80.i
```
然后下载了一个补充文件，放在 opencv_contrib/modules/xfeatures2d/src/目录下：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311746943.png)
想安装 tree，查看文件结构，还没好
### 问题 2：nvopticalFlow.h 没得
把 cudaflow 等一些没必要的 module 给关掉，重来就好了
## 配置环境变量
#### 从 cmake option配置 log
```
General configuration for OpenCV 4.7.0-dev =====================================

Version control: unknown

  

Platform:

Timestamp: 2022-12-31T06:50:17Z

Host: Linux 5.15.0-56-generic x86_64

CMake: 3.22.1

CMake generator: Unix Makefiles

CMake build tool: /usr/bin/gmake

Configuration: Release

  

CPU/HW features:

Baseline: SSE SSE2 SSE3

requested: SSE3

Dispatched code generation: SSE4_1 SSE4_2 FP16 AVX AVX2 AVX512_SKX

requested: SSE4_1 SSE4_2 AVX FP16 AVX2 AVX512_SKX

SSE4_1 (15 files): + SSSE3 SSE4_1

SSE4_2 (2 files): + SSSE3 SSE4_1 POPCNT SSE4_2

FP16 (1 files): + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX

AVX (5 files): + SSSE3 SSE4_1 POPCNT SSE4_2 AVX

AVX2 (29 files): + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2

AVX512_SKX (7 files): + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2 AVX_512F AVX512_COMMON AVX512_SKX

  

C/C++:

Built as dynamic libs?: YES

C++ standard: 11

C++ Compiler: /bin/g++-10 (ver 10.4.0)

C++ flags (Release): -fsigned-char -ffast-math -W -Wall -Wreturn-type -Wnon-virtual-dtor -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -fopenmp -O3 -DNDEBUG -DNDEBUG

C++ flags (Debug): -fsigned-char -ffast-math -W -Wall -Wreturn-type -Wnon-virtual-dtor -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -fopenmp -g -O0 -DDEBUG -D_DEBUG

C Compiler: /bin/gcc-10

C flags (Release): -fsigned-char -ffast-math -W -Wall -Wreturn-type -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections -msse -msse2 -msse3 -fvisibility=hidden -fopenmp -O3 -DNDEBUG -DNDEBUG

C flags (Debug): -fsigned-char -ffast-math -W -Wall -Wreturn-type -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections -msse -msse2 -msse3 -fvisibility=hidden -fopenmp -g -O0 -DDEBUG -D_DEBUG

Linker flags (Release): -Wl,--gc-sections -Wl,--as-needed -Wl,--no-undefined

Linker flags (Debug): -Wl,--gc-sections -Wl,--as-needed -Wl,--no-undefined

ccache: YES

Precompiled headers: NO

Extra dependencies: png16 z m pthread cudart_static dl rt nppc nppial nppicc nppidei nppif nppig nppim nppist nppisu nppitc npps cublas cufft -L/usr/local/cuda/lib64 -L/usr/lib/x86_64-linux-gnu

3rdparty dependencies:

  

OpenCV modules:

To be built: alphamat barcode bioinspired core cudaarithm cudabgsegm cudacodec cudafilters cudaimgproc cudalegacy cudawarping cudev datasets dnn dnn_objdetect dnn_superres flann freetype fuzzy hfs highgui img_hash imgcodecs imgproc intensity_transform line_descriptor ml phase_unwrapping photo plot quality reg surface_matching tracking ts video videoio xphoto

Disabled: cudafeatures2d cudaoptflow features2d optflow world xfeatures2d

Disabled by dependency: aruco bgsegm calib3d ccalib cudaobjdetect cudastereo dpm face mcc objdetect rapid rgbd saliency shape stereo stitching structured_light superres text videostab wechat_qrcode ximgproc xobjdetect

Unavailable: cvv gapi hdf java julia matlab ovis python2 python3 sfm viz

Applications: tests perf_tests apps

Documentation: NO

Non-free algorithms: YES

  

GUI: GTK2

GTK+: YES (ver 2.24.33)

GThread : YES (ver 2.72.4)

GtkGlExt: NO

  

Media I/O:

ZLib: /usr/lib/x86_64-linux-gnu/libz.so (ver 1.2.11)

JPEG: /usr/lib/x86_64-linux-gnu/libjpeg.so (ver 80)

WEBP: build (ver encoder: 0x020f)

PNG: /opt/questasim/questasim/linux_x86_64/libpng.so (ver 1.6.37)

TIFF: /usr/lib/x86_64-linux-gnu/libtiff.so (ver 42 / 4.3.0)

JPEG 2000: build (ver 2.4.0)

OpenEXR: /usr/lib/x86_64-linux-gnu/libImath-2_5.so /usr/lib/x86_64-linux-gnu/libIlmImf-2_5.so /usr/lib/x86_64-linux-gnu/libIex-2_5.so /usr/lib/x86_64-linux-gnu/libHalf-2_5.so /usr/lib/x86_64-linux-gnu/libIlmThread-2_5.so (ver 2_5)

HDR: YES

SUNRASTER: YES

PXM: YES

PFM: YES

  

Video I/O:

DC1394: YES (2.2.6)

FFMPEG: YES

avcodec: YES (58.134.100)

avformat: YES (58.76.100)

avutil: YES (56.70.100)

swscale: YES (5.9.100)

avresample: NO

GStreamer: NO

v4l/v4l2: YES (linux/videodev2.h)

  

Parallel framework: OpenMP

  

Trace: YES (with Intel ITT)

  

Other third-party libraries:

VA: NO

Lapack: NO

Eigen: YES (ver 3.4.0)

Custom HAL: NO

Protobuf: build (3.19.1)

  

NVIDIA CUDA: YES (ver 11.6, CUFFT CUBLAS FAST_MATH)

NVIDIA GPU arch: 70 75 80 86

NVIDIA PTX archs:

  

cuDNN: NO

  

OpenCL: YES (no extra features)

Include path: /home/cici/softwares/opencv-4.x/3rdparty/include/opencl/1.2

Link libraries: Dynamic load

  

Python (for build): /usr/bin/python2.7

  

Java:

ant: NO

JNI: /usr/lib/jvm/default-java/include /usr/lib/jvm/default-java/include/linux /usr/lib/jvm/default-java/include

Java wrappers: NO

Java tests: NO

  
Install to: /usr/local
```


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

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011456367.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011526487.png)

#### 问题：opencv.pc
[linux下编译安装opencv生成opencv.pc_浓茶淡酒的博客-CSDN博客](https://blog.csdn.net/s15810751918/article/details/107705387)

## 测试
到这个目录下面执行：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011545157.png)
#### 问题：报错
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202301011546064.png)
解决：
```shell
sudo cmake .. -DCMAKE_CXX_COMPILER=$(which g++) -DCMAKE_C_COMPILER=$(which gcc)
```
参考：
[[已解决] The C++ compiler "/usr/local/bin/c++" is not able to compile a simple test program._HeyMountain的博客-CSDN博客_cmake在编译你的c或c++代码前，会先验证你指定的编译器是否可以正常工作](https://is.gd/VJi6Vm)
[Ubuntu下安装opencv并进行测试_带你去网吧里偷耳机的博客-CSDN博客_检测opencv是否安装成功 ubuntu](https://blog.csdn.net/qq_40123329/article/details/103904087)

## 编译方法
可以设置 cmake，配置 vscode，这里使用最简单的方法：

```shell
nvcc -std=c++11 `pkg-config --cflags opencv4` cuda_image.cu `pkg-config --libs opencv4` -o image
```
cuda_image.cu  --- source code 
image  --- output
### reference
[c++ - Cmake + CUDA + OpenCV - Stack Overflow](https://stackoverflow.com/questions/31881249/cmake-cuda-opencv)  --cmake cuda
[Linux平台CUDA+OpenCV3.4配置 - Brccq - 博客园](https://www.cnblogs.com/br170525/p/8331640.html)  --一篇比较长的模板，可以修改 cuda opencv
[cuda与openCV结合编程（一）_alpc40的博客-CSDN博客_cuda与opencv](https://blog.csdn.net/weixin_39212021/article/details/78884830) --几种编译方法

