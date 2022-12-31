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
-D CUDA_NVCC_FLAGS=–expt-relaxed-constexpr  使用abs
2. make&install

```shell
make -j8

```

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
