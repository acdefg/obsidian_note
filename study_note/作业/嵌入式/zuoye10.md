# 嵌入式第十次作业

### 开发环境

> 操作系统：Ubuntu22.04
> 
> 编译环境：VScode
> 
> 版本：c++11，opencv4.6.0

## 实验内容
请用 GPU 解决练习 9 的问题，并与上次作业中的方法比较计算时间。

## 实验原理
cuda 是 nivida 显卡配套的 toolkit，加速系统在运行程序时首先会运行 CPU 程序，在运行到需要 GPU 进行大规模并行计算的函数时，再将对应函数载入 GPU 执行。由 GPU 加速的依然还是纯 CPU 的应用程序，只是某些模块在运行时调入了 GPU 中，该模块在同步完毕后将会重新回到 CPU 中执行主程序的后续代码。
### cuda 编程框架

1.  `__global__ void GPUFunction()：
__global__ 关键字表明该函数将在 GPU 上运行并可全局调用（ 既可以由 CPU ，也可以由 GPU 调用）；
通常，将在 CPU 上执行的代码称为 Host （主机）代码，而将在 GPU 上运行的代码称为 Device （设备）代码；
注意返回类型为 void。使用 __global__ 关键字定义的函数返回值需为 void 类型。

2. `GPUFunction<<<1, 1>>>()：
通常，我们把要运行在 GPU 上的函数称为 kernel （核）函数;
启动核(kernel)函数时，我们必须事先配置 GPU 参数，使用 <<< ... >>> 语法向核函数传递两个必要的参数;
在 <<< ... >>> 中传递的参数用于为核函数设定线程的层次结构，第一个参数定义线程块(Block)的数量，第二个参数定义 Block 中含有的线程(Thread)数量。
![10](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221231234731.png)


3. `cudaDeviceSynchronize()：`
与其他并行化的代码类似，核函数启动方式为异步，即 CPU 代码将继续执行而不会等待核函数执行完成；
调用 CUDA 提供的函数 cudaDeviceSynchronize 可以让 Host 代码(CPU) 等待 Device 代码(GPU) 执行完毕，再在 CPU 上继续执行。


