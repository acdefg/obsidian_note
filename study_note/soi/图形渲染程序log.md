## 2024.11
### ARGUS(基于 ARMSIM)

### MultiGPUCG
#### issue1
cuda 版本
```
严重性	代码	说明	项目	文件	行	禁止显示状态	详细信息
错误	MSB4019	找不到导入的项目“G:\software\Visual_studio\MSBuild\Microsoft\VC\v170\BuildCustomizations\CUDA 11.1.props”。请确认 Import 声明“G:\software\Visual_studio\MSBuild\Microsoft\VC\v170\\BuildCustomizations\CUDA 11.1.props”中的表达式正确，且文件位于磁盘上。	MultiGPUCG	D:\Users\Downloads\Edge_download\MultiGPUCGSolver-0.1\MultiGPUCGSolver\MultiGPUCG\MultiGPUCG.vcxproj	34		
```

1. 查看电脑 cuda 版本是否一致：v11.2 的位置对应的是自己安装的版本号
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411111009392.png?token=ALRC6IWUG5TJRSR7BQ5BFJ3HGFT2C)

不一致更改 vcxproj 里面的 cuda 版本呢
打开 vcxproj，找到 cuda 版本，直接搜索版本号更改
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411111006779.png?token=ALRC6ISE445U5J5R2BMK6BDHGFTP6)

2. 查看 vs 里面 cuda 路径下面的文件是否齐全
**Microsoft Visual Studio/\2019/\Community/\MSBuild/\Microsoft/\VC/\v160/\BuildCustomizations/\ 
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411111009695.png?token=ALRC6IXL6R4T64QBS2IGTCDHGFTZG)

没有的话从这里拿：版本号根据自己安装的找![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411111009392.png?token=ALRC6IWUG5TJRSR7BQ5BFJ3HGFT2C)

#### issue2
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202411111004577.png?token=ALRC6IWO2WSH5VEWO536QMTHGFTGA)
静态库？没有输入？

## 2024.5
### ARMSIM
[GitHub - kaist-silab/arcsim: ARCSim 0.3.1 Fixes and Installer Script for Linux and MacOS](https://github.com/kaist-silab/arcsim)

### libpng
[[zuoye10#问题 1：png]]
注意在 cmake 在中的顺序

### projects
[Georg SPERL / ARCSim-HYLC · GitLab](https://git.ista.ac.at/gsperl/ARCSim-HYLC)
[Georg SPERL / HYLC · GitLab](https://git.ista.ac.at/gsperl/HYLC/)
[ARCSim: Adaptive Refining and Coarsening Simulator - U.C. Berkeley Computer Graphics Research](http://graphics.berkeley.edu/resources/ARCSim/)

## 2024.2
### MADYPG(基于ARMSIM)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202403071333376.gif)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202403071333378.gif)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202403071333379.gif)
论文链接：
[Mechanics-aware deformation of yarn pattern geometry | ACM Transactions on Graphics](https://dl.acm.org/doi/10.1145/3450626.3459816)
程序链接：
[GitHub - kamleshbhalui/MADYPG: Mechanics-Aware Deformation of Yarn Pattern Geometry](https://github.com/kamleshbhalui/MADYPG)

#### 运行指令 log
在 root 下运行：python exec.py mesh2yarns x(1-13)都可以

```shell
python exec.py mesh2yarns 12
```

-p 可以打开 cpu 不并行
-p 0 不并行

#### 大致问题
vcpkg/CHANGELOG.md 里面有各个库使用的版本
##### tdd 库 中 函数大量 undefined
报错信息如下：
>undefined reference to `tbb::task_group_context::~task_group_context()' /usr/bin/ld: YarnSoup.cpp:(.text.unlikely+0x28b): undefined reference to `tbb::internal::allocate_root_with_context_proxy::free(tbb::task&) const' /usr/bin/ld: YarnSoup.cpp:(.text.unlikely+0x293): undefined reference to `tbb::task_group_context::~task_group_context()' /usr/bin/ld: YarnSoup.cpp:(.text.unlikely+0x2b5): undefined reference to `tbb::internal::allocate_root_with_context_proxy::free(tbb::task&) const' /usr/bin/ld: YarnSoup.cpp:(.text.unlikely+0x2bd): undefined reference to `tbb::task_group_context::~task_group_context()' /usr/bin/ld: YarnSoup.cpp:(.text.unlikely+0x2d4): undefined reference to `tbb::internal::allocate_root_with_context_proxy::free(tbb::task&) const' /usr/bin/ld: YarnSoup.cpp:(.text.unlikely+0x2dc): undefined reference to `tbb::task_group_context::~task_group_context()'，定义了find_package(TBB CONFIG REQUIRED) target_link_libraries(mesh2yarns PRIVATE TBB::tbb)

更改 tdd 库的版本，手动安装新的版本
安装教程：
[linux下安装和使用tbb - 简书](https://www.jianshu.com/p/57b67477ff53)
[ubuntu下安装Intel Threading Building Blocks（TBB）教程\_ubuntu18.04 安装 threading building blocks-CSDN博客](https://blog.csdn.net/wgd852372/article/details/106647200#:~:text=Intel%C2%AE%20Threading%20Building%20Blocks%E5%AE%98%E7%BD%91%E4%B8%8B%E8%BD%BD%E5%90%8E%E8%A7%A3%E5%8E%8B%EF%BC%8C%E5%81%87%E8%AE%BE%E8%A7%A3%E5%8E%8B%E7%9B%AE%E5%BD%95%E4%B8%BAtbb1%E3%80%81%E8%B7%B3%E8%BD%AC%E5%88%B0tbb%E7%9B%AE%E5%BD%95%E4%B8%8B%EF%BC%8C%E6%89%A7%E8%A1%8Cmake%E5%91%BD%E4%BB%A4%E3%80%82%20cd%20tbbmake2%E3%80%81%E6%B7%BB%E5%8A%A0tbb%E5%8F%98%E9%87%8Fcd%20buildchmod%20%2Bx%2A.shsh,%2A.so%20%2Fusr%2Flib64cp%20%2A.so.2%20%2Fusr%2Flib64_ubuntu18.04%20%E5%AE%89%E8%A3%85%20threading%20building%20blocks)

仓库链接：
[Releases · oneapi-src/oneTBB](https://github.com/oneapi-src/oneTBB/releases?page=3)
✔️fixed

##### SDL undefined

error message:
>SDL_waylandvideo.c:(.text+0x688): undefined reference to `wl_proxy_marshal_flags'
/usr/bin/ld: /home/cici/code/physics_simulation/MADYPG/vcpkg/installed/x64-linux/lib/libSDL2.a(SDL_waylandvideo.c.o): in function `display_handle_global':
SDL_waylandvideo.c:(.text+0x8b7): undefined reference to `wl_proxy_marshal_flags'
/usr/bin/ld: SDL_waylandvideo.c:(.text+0x936): undefined reference to `wl_proxy_marshal_flags'
/usr/bin/ld: SDL_waylandvideo.c:(.text+0x9b7): undefined reference to `wl_proxy_marshal_flags'
/usr/bin/ld: SDL_waylandvideo.c:(.text+0x9f4): undefined reference to `wl_proxy_marshal_flags'
/usr/bin/ld: /home/cici/code/physics_simulation/MADYPG/vcpkg/installed/x64-linux/lib/libSDL2.a(SDL_waylandvideo.c.o):SDL_waylandvideo.c:(.text+0xa64): more undefined references to `wl_proxy_marshal_flags' follow

省略中间一堆错误尝试
正确解决办法：
看到 github 的 issue 记录，想修改库的源码
[Fix build against wayland 1.20 by Sodivad · Pull Request #5092 · libsdl-org/SDL · GitHub](https://github.com/libsdl-org/SDL/pull/5092/files)
vcpkg 修改源码方法：
[教程：安装本地修改的依赖项 | Microsoft Learn](https://learn.microsoft.com/zh-cn/vcpkg/consume/install-locally-modified-package?pivots=shell-bash#6---modify-portfilecmake-to-apply-the-patch)
##### libpng undefined
error message:
/usr/bin/ld: /home/cici/code/physics_simulation/MADYPG/vcpkg/installed/x64-linux/lib/libPngImporter.a(PngImporter.cpp.o): in function `Magnum::Trade::PngImporter::doImage2D(unsigned int, unsigned int)::{lambda(png_struct_def*, char const*)#2}::_FUN(png_struct_def*, char const*)':
PngImporter.cpp:(.text+0x3f2): undefined reference to `png_set_longjmp_fn'
/usr/bin/ld: /home/cici/code/physics_simulation/MADYPG/vcpkg/installed/x64-linux/lib/libPngImporter.a(PngImporter.cpp.o): in function `Magnum::Trade::PngImporter::doImage2D(unsigned int, unsigned int)':
PngImporter.cpp:(.text+0x78c): undefined reference to `png_set_longjmp_fn'

修改了 libpng 以源码（不知道有没有用）
重新编译了这两个库
(确认了没用)

查找 libpng 的版本问题，网上普遍流传的问题是这个：有一个更老的版本导致编译链接错误
[undefined reference to \`png\_set\_longjmp\_fn'-CSDN博客](https://blog.csdn.net/wangpanbaoding/article/details/104185058)
我查了我的没有

正确解决方案：
在 cmake 编译 pngimporter 部分加入了 libpng 的链接指令
find_package(libpng CONFIG REQUIRED)
target_link_libraries(main PRIVATE png_static)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202403011800643.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202403011801009.png)



## 2023.11

### 程序整理
[GitHub - wenlongx/Maya-Cloth-Simulation: Generates a shirt model from a CSV file, and simulates it being held from random points](https://github.com/wenlongx/Maya-Cloth-Simulation)

### blender+body_model+neural
[GitHub - acdefg/NeuralClothSim: for models](https://github.com/acdefg/NeuralClothSim)

### 论文 An implementation of Large Steps in Cloth Simulation（1998）✔️
两个都可运行
[cloth.ipynb](https://github.com/MeghaS94/Cloth-simulator/blob/main/cloth.ipynb)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202311040948663.png)

[GitHub - zanesterling/cloth-simulation: An implementation of Large Steps in Cloth Simulation for CSE328](https://github.com/zanesterling/cloth-simulation/tree/master)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202311040944821.png)

### 知识仓库
[[布料模拟]]
[[ZYNQ]]

## 2023.11之前
[GitHub - mmmovania/opencloth: A collection of source codes implementing cloth simulation algorithms in OpenGL](https://github.com/mmmovania/opencloth)
有一些 demo，但是需要 visual studio
或者可以 windows 的打开看看那
status: waiting

[GitHub - dilevin/CSC417-a4-cloth-simulation: Cloth simulation using co-rotational linear elasticity](https://github.com/dilevin/CSC417-a4-cloth-simulation)
status: quit
log: 
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308241317665.png)
按道理来说能检测到路径，但是就是打不开，有毒了
可以试试是不是 ig.. 库安装的问题

[GitHub - MauriceGit/Cloth\_Simulation: Cloth-Visualization via particle-simulation.](https://github.com/MauriceGit/Cloth_Simulation)

### eol-cloth
[GitHub - sueda/eol-cloth: Eulerian-on-Lagrangian Cloth Simulation](https://github.com/sueda/eol-cloth)
那个 cmakelist 个根本不适用 ubuntu，改了一点不想改了，重写一个

### DART
[GitHub - dartsim/dart: DART: Dynamic Animation and Robotics Toolkit](https://github.com/dartsim/dart)
[Biped — DART: Dynamic Animation and Robotics Toolkit 7.0.0-alpha20230101 documentation](https://dart.readthedocs.io/en/latest/user_guide/tutorials/biped.html)
好像是一个完整的库
status：昨天打开 cmake all 在之后就没停下来
#### log
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309010855320.png)
应该是有错误在不断重试

[GitHub - hbertiche/NeuralClothSim](https://github.com/hbertiche/NeuralClothSim)
没看懂具体干啥的，但是可以试试
深度学习做的

[Human Motion Diffusion Model](https://guytevet.github.io/mdm-page/)
深度学习做的

### sheen
[GitHub - sciecode/sheen: GPGPU cloth simulation](https://github.com/sciecode/sheen)
java 写的
效果不错
📍：😒 quit

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311743254.png)

[[vscode_ubuntu_javajs]]
合着我折腾一天配置 java 环境，跑出来就和 github 点进去一样
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011152845.png)
啥也没有，无语死了
不应该阿，代码和好像写了相关的东西 emmmmmm
[• Atomize •](https://sciecode.com/)
另一个网站可以，一下子就运行成功了，哭哭
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011439278.png)

只加载出来了背景，加载方式不太对劲，另外一个用其他在线平台验证的，本地也跑不出来，canvas 的问题？先放弃了。。。。。


[GitHub - JUSTIVE/GPU-Cloth-Simulation: GPU Mass-Spring Simulation Cloth in Unity](https://github.com/JUSTIVE/GPU-Cloth-Simulation)
看起来不错
test on windows
给了一堆测试向量的参考值
📍status：waiting
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311738274.png)

#### ClothSimulation ✔️
[GitHub - johnBuffer/ClothSimulation: Basic cloth simulation using Verlet integration](https://github.com/johnBuffer/ClothSimulation)
代码完整，调用库少
简陋
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311646565.png)
👍 status: success
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308311716756.png)

Left click 	Move view
Wheel 	Zoom
Right click 	Move cloth
Middle click 	Cut cloth

这个差不多，java 新写的
[GitHub - dissimulate/Tearable-Cloth: A tearable cloth simulation using vertlet integration.](https://github.com/dissimulate/Tearable-Cloth)

## 环境
### openGL
[ubuntu配置openGL glut库\_xiadidi的博客-CSDN博客](https://blog.csdn.net/xiadidi/article/details/50867241)
[Ubuntu下搭建OpenGL开发环境（GLFW\_3.3.1 + GLM\_0.9.9 + GLAD）\_RoboticsLearner的博客-CSDN博客](https://blog.csdn.net/l1216766050/article/details/102787618)
[Linux（Ubuntu）使用 sudo apt-get install 命令安装软件的目录在哪？（已解决） - Memory4Young - 博客园](https://www.cnblogs.com/memory4young/p/where-is-sudo-apt-get-install-package-file-path.html)

测试一直报错
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011712246.png)
改了半天搜索路径一直不对，不想在 cmake 在中额外加搜索路径，直接找到这个文件复制进来
```
sudo find / -name libglfw3.a
cp /usr/local/lib/libglfw3.a .  //路径根据具体情况改
```
##### 编译方式
编译 flag
`-lGL`
freeglut:
`-lglut`
GLEW:
`-lglew`
glu:
`-lGLU`

### openmp(待完善)

### SFML
[SFML and Linux (SFML / Learn / 2.5 Tutorials)](https://www.sfml-dev.org/tutorials/2.5/start-linux.php)

**▶️ test pass**

📢 **pay attention**
You must then link the compiled file to the SFML libraries in order to get the final executable. SFML is made of 5 modules (system, window, graphics, network and audio), and there's one library for each of them.
To link an SFML library, you must add "-lsfml-xxx" to your command line, for example "-lsfml-graphics" for the graphics module (the "lib" prefix and the ".so" extension of the library file name must be omitted).
```
g++ main.o -o sfml-app -lsfml-graphics -lsfml-window -lsfml-system
```

### eigen

[ubuntu安装Eigen\_ubuntu22安装eigen\_ClaireQi的博客-CSDN博客](https://blog.csdn.net/wangxiao7474/article/details/103422616)

```
sudo apt-get install libeigen3-dev
```
```
# /usr/include
sudo cp -r /usr/include/eigen3/Eigen /usr/include
# /usr/local/include
sudo cp -r /usr/local/include/eigen3/Eigen /usr/local/include
```
测试：
[Eigen的介绍、安装与入门操作 - 知乎](https://zhuanlan.zhihu.com/p/462494086)

### cmake
[cmake(5)：选择编译器及设置编译器选项\_cmake指定编译器\_翔底的博客-CSDN博客](https://blog.csdn.net/rangfei/article/details/108862896#t3)

### gcc

报错：
```ad-failure
- The C compiler identification is unknown
-- The CXX compiler identification is GNU 11.4.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - failed
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - broken
CMake Error at /usr/share/cmake-3.22/Modules/CMakeTestCCompiler.cmake:69 (message):
  The C compiler

    "/usr/bin/cc"

  is not able to compile a simple test program.

```

```shell
cmake ../CMakeLists.txt -DCMAKE_C_COMPILER=$(which gcc)
```


```shell
-DCMAKE_CXX_COMPILER=$(which g++) -DCMAKE_C_COMPILER=$(which gcc)
```

### gdb 调试

```shell
gdb program-cmd
(gdb) run
(gdb) backtrace
```

[GDB调试入门指南 - 知乎](https://zhuanlan.zhihu.com/p/74897601)

[【Linux】GDB调试教程（新手小白）\_爪可摘星辰的博客-CSDN博客](https://blog.csdn.net/lovely_dzh/article/details/109160337#t12)


### vscode_cmake 配置
[Linux环境下使用VScode调试CMake工程 - 知乎](https://zhuanlan.zhihu.com/p/618043511)

[VScode tasks.json和launch.json的设置 - 知乎](https://zhuanlan.zhihu.com/p/92175757)