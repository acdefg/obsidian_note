## packed unpacked
在 SystemVerilog 中，packed 数组和 unpacked 数组主要有以下区别：
**packed array:** 维度声明在标识符名字之前。
**unpacked array**: 维度声明在标识符名字之后。

```verilog
bit [7:0] c1; // packed array of scalar bit types
real u [7:0]; // unpacked array of real types
```


**一、存储方式**
1.  **packed 数组**：
    -   以紧凑的方式存储，类似于连续的位序列。
    -   例如，一个 8 位的 packed 数组可以被视为一个连续的 8 位二进制序列存储在内存中。
2.  **unpacked 数组**：
    -   每个元素都独立地存储在内存中，类似于传统编程语言中的数组存储方式。
    -   对于一个由多个字节组成的 unpacked 数组，每个元素都有自己独立的存储空间。
**二、声明方式**
1.  **packed 数组**：在变量前定义大小
    -   使用 `[bit_vector_size]` 的语法进行声明，其中 `bit_vector_size` 表示位宽。
    -   例如：`logic [7:0] packed_array;` 声明了一个 8 位宽的 packed 数组。
2.  **unpacked 数组**：在变量名后定义大小
    -   使用传统的数组声明方式，例如 `logic [array_size] unpacked_array;`，其中 `array_size` 表示数组的大小。
    -   例如：`logic [7] unpacked_array;` 声明了一个包含 8 个元素的 unpacked 数组。
**三、访问方式**
1.  **packed 数组**：
    -   可以作为一个整体进行赋值和操作，也可以通过位索引的方式访问特定的位。
    -   例如，可以使用 `packed_array = 8'b10101010;` 进行整体赋值，或者使用 `packed_array[3]` 访问第 4 位。
2.  **unpacked 数组**：
    -   通过索引访问单个元素，类似于传统数组的访问方式。
    -   例如：`unpacked_array[3] = 1'b1;` 表示将 unpacked_array 数组的第 4 个元素赋值为 1。
## always @* 和 always comb 的区别
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202409162231506.png)

### ref
[https://zhuanlan.zhihu.com/p/404777602](https://zhuanlan.zhihu.com/p/404777602)
[SystemVerilog Class Constructor](https://www.chipverify.com/systemverilog/systemverilog-class-constructor)
[Base Classes](https://www.chipverify.com/uvm/base-classes)