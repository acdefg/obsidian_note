## 论文
这篇论文的第一部分主要讲了以下内容：

- **基于GPU的MAS预处理器用于布料和可变形物体模拟。** 作者提出了一种新颖的多级加性Schwarz（MAS）预处理器，可以加速GPU上的各种线性和非线性求解器。预处理器使用小的、不重叠的域，可以高效地并行求解，并使用粗略空间校正来提高收敛速度。该预处理器有效、快速、预计算成本低且对刚度和问题大小具有可扩展性。
- **使用Morton码进行多级域构造。** 作者使用Morton码排序来实现系统矩阵节点之间的空间局部性，并将它们分割成不同级别的超节点。他们还使用节点之间的连接来避免不同体之间的虚假耦合伪像。他们提出了一种跳过方法，通过重用前几个时间步骤中排序过的节点来减少域构造的开销。
- **通过单向消元进行矩阵预计算。** 作者提出了一种快速算法，通过直接对系统矩阵应用高斯-约旦消元来计算每个域的子矩阵逆。他们还开发了一种选择性更新方案，以处理非线性求解器或动态接触引起的轻微矩阵修改。
- **通过对称矩阵向量乘法进行运行时预处理。** 作者通过使用预计算的子矩阵逆进行简单的矩阵向量乘法来执行运行时预处理。他们发明了一种具有平衡工作负载和零写冲突的对称矩阵向量乘法方法，可以进一步降低GPU上预处理的运行时成本。
- **实验结果和比较。** 作者在许多布料和可变形物体模拟示例中展示了他们预处理器的性能，包括实时模拟。他们表明，他们的预处理器优于其他竞争者，如 GPU 上的多网格 AmgX、CPU 上的 ichol 和 ILUT 等。他们还验证了他们预处理器与各种求解器（如 PCG、加速梯度下降和 L-BFGS）之间的兼容性。

## 环境
### openGL
[ubuntu配置openGL glut库\_xiadidi的博客-CSDN博客](https://blog.csdn.net/xiadidi/article/details/50867241)
### openmp


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
