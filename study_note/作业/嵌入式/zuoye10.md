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

1.  `__global__ void GPUFunction()：
__global__ 关键字表明该函数将在 GPU 上运行并可全局调用（ 既可以由 CPU ，也可以由 GPU 调用）；
通常，将在 CPU 上执行的代码称为 Host （主机）代码，而将在 GPU 上运行的代码称为 Device （设备）代码；
注意返回类型为 void。使用 __global__ 关键字定义的函数返回值需为 void 类型。
2. `GPUFunction<<<1, 1>>>()：
通常，我们把要运行在 GPU 上的函数称为 kernel （核）函数;
启动核(kernel)函数时，我们必须事先配置 GPU 参数，使用 <<< ... >>> 语法向核函数传递两个必要的参数;
在 <<< ... >>> 中传递的参数用于为核函数设定线程的层次结构，第一个参数定义线程块(Block)的数量，第二个参数定义 Block 中含有的线程(Thread)数量。
CUDA线程的层次结构分为三层：Thread（线程）、Block（块）、Grid（网格），网格由块组成，块由线程组成。
![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221231234731.png)
`someKernel<<<10, 10>>()` 在 GPU 中为该核函数分配 10 个具有 10 个线程的线程块，核函数中的代码将运行 100 次
3. `cudaDeviceSynchronize()：`
与其他并行化的代码类似，核函数启动方式为异步，即 CPU 代码将继续执行而不会等待核函数执行完成；
调用 CUDA 提供的函数 cudaDeviceSynchronize 可以让 Host 代码(CPU) 等待 Device 代码(GPU) 执行完毕，再在 CPU 上继续执行。

### 程序编写
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
对于比较小的图片，可以让线程号对应列数，块对应行：

```c
//定义并行进程数和进程块数
size_t threadsnum = src.cols; // 定义每个block的thread数量
size_t blocksnum = src.rows; // 定义block的数量
//索引号，此时width = blockdim
int index = threadIdx.x + 1 + (blockIdx.x + 1) * blockDim.x;
```
运行结果：
