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


#### 问题 3：boostdesc_bgm.i
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311704695.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311703227.png)

download.sh 在 opencv 个根目录下运行这个脚本，操作下面隐藏文件夹cache
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

## make&install

```shell
make -j8
```
查看最大核数
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202212311725826.png)
