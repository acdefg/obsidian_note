## python 测试

### 方法
tb 目录下
```python
python test_axi.py > test.txt
```
python 运行并且重点向到 test.txt 文档内
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307270839111.png)


### 内容
1. **基准测试（测试 1）**：
    - 此测试读取整个 AXI RAM 的内容，并以十六进制格式打印出来。
2. **直接写入测试（测试2）**：
    - 此测试将数据“test”写入到AXI RAM的地址0x00。
    - 写入后，它从同一地址读取内容，并验证数据是否成功写入。
3. **通过端口0写入测试（测试3）**：
    - 此测试启动了来自AXI Master（端口0）到AXI RAM的写操作。
    - 它将数据（0x11 0x22 0x33 0x44）写入到AXI RAM的地址0x04。
    - 写入后，它从地址0x04读取内容，并验证数据是否成功写入。
4. **通过端口0读取测试（测试4）**：
    - 此测试将数据（0x11 0x22 0x33 0x44）写入到AXI RAM的地址0x04。
    - 它启动了来自AXI Master（端口0）到AXI RAM的读操作，以从地址0x04读取数据。
    - 读取后，它检查读取的数据是否与先前写入的数据相匹配。
5. **不同写入测试（测试5）**：
    - 此测试使用不同的参数执行对AXI RAM的多种写入操作，例如长度、偏移量和传输大小。
    - 它通过更改要写入的数据长度、内存中的偏移量和传输大小来测试不同的场景。
    - 它执行正常写入、带主控暂停的写入以及带从设备（RAM）暂停的写入。
6. **不同读取测试（测试6）**：
    - 此测试使用不同的参数执行对AXI RAM的多种读取操作，例如长度、偏移量和传输大小。
    - 它通过更改要读取的数据长度、内存中的偏移量和传输大小来测试不同的场景。
    - 它执行正常读取、带主控暂停的读取以及带从设备（RAM）暂停的读取。

测试 5,6 从测试的具体内容
- 数据长度（length）：测试数据写入的字节数，范围从 1 字节到 1024 字节。
- 内存偏移量（offset）：在 AXI RAM 中写入数据的起始偏移量，范围从 4 字节到 8 字节，以及 4096-4 在字节。
- 传输大小（size）：用于写入操作的传输大小，包括 3 种情况：2（4 字节传输），1（2 字节传输）和 0（1 字节传输）。

## pytest + tox
最外层
```python
pytest
```

内容：
所有模块的 py 测试内容
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307240020939.png)

## cocotb
tb 名目录下 make
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307240022756.png)


## 环境配置
[Install](http://hnikolov.github.io/pihdf_doc/install.html)
编译不成功，无法生成 vip 为文件
找了一下 issue
[Failed to make myhdl.vpi based on Makefile of Icarus Verilog · Issue #273 · myhdl/myhdl · GitHub](https://github.com/myhdl/myhdl/issues/273)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308062009447.png)
缺少.h 文件：
[vpi\_user.h](https://github.com/cocotb/cocotb/blob/v1.3.0/cocotb/share/include/vpi_user.h)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308062133196.png)
画框的语句执行有 bug
直接执行下面的就可以用了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202308062133354.png)

## 仿真设置


```shell
iverilog -o my_design_tb.vvp my_design.v my_testbench.v
vvp -M. -mmy_design my_design_tb.vvp +sdf_file=my_design.sdf
```

### 后仿
- 首先，它使用三个嵌套循环来遍历不同的 `length`，`offset` 和 `size`，`cache` 值。
- 然后，它使用另一个循环来遍历不同的等待状态。
- 在每次迭代中，它会打印出当前的`length`，`offset`，`size`和`cache`值。
- 然后，它会计算地址并生成测试数据。
- 接下来，它会使用`axi_ram_inst.write_mem()`方法将测试数据写入内存。
- 然后，它会使用`axi_master_inst.init_write()`方法初始化写操作。
- 在等待时钟上升沿之后，它会使用`axi_ram_inst.read_mem()`方法读取内存中的数据并打印出来。
- 最后，它会使用断言来检查读取到的数据是否与预期相符。

python: 3.10.12
pytest: 7.4.0
myhdl: 0.11
Icarus Verilog: 11.0
gtkwave: 3.3

## 模块
### `axi_adapter` module

AXI width adapter module with parametrizable data and address interface widths. Supports INCR burst types and narrow bursts. Wrapper for `axi_adapter_rd` and `axi_adapter_wr`.

### [](https://github.com/alexforencich/verilog-axi#axi_adapter_rd-module)

### `axi_adapter_rd` module

AXI width adapter module with parametrizable data and address interface widths. Supports INCR burst types and narrow bursts.

### [](https://github.com/alexforencich/verilog-axi#axi_adapter_wr-module)

### `axi_adapter_wr` module

AXI width adapter module with parametrizable data and address interface widths. Supports INCR burst types and narrow bursts.

### `axi_cdma` module

AXI to AXI DMA engine with parametrizable data and address interface widths. Generates full-width INCR bursts only, with parametrizable maximum burst length. Supports unaligned transfers, which can be disabled via parameter to save on resource consumption.

### [](https://github.com/alexforencich/verilog-axi#axi_cdma_desc_mux-module)

### `axi_cdma_desc_mux` module

Descriptor multiplexer/demultiplexer for AXI CDMA module. Enables sharing the AXI CDMA module between multiple request sources, interleaving requests and distributing responses.

### [](https://github.com/alexforencich/verilog-axi#axi_crossbar-module)

### `axi_crossbar` module

AXI nonblocking crossbar interconnect with parametrizable data and address interface widths and master and slave interface counts. Supports all burst types. Fully nonblocking with completely separate read and write paths; ID-based transaction ordering protection logic; and per-port address decode, admission control, and decode error handling. Wrapper for `axi_crossbar_rd` and `axi_crossbar_wr`.

Wrappers can generated with `axi_crossbar_wrap.py`.

### [](https://github.com/alexforencich/verilog-axi#axi_crossbar_addr-module)

### `axi_crossbar_addr` module

Address decode and admission control module for AXI nonblocking crossbar interconnect.

### [](https://github.com/alexforencich/verilog-axi#axi_crossbar_rd-module)

### `axi_crossbar_rd` module

AXI nonblocking crossbar interconnect with parametrizable data and address interface widths and master and slave interface counts. Read interface only. Supports all burst types. Fully nonblocking with completely separate read and write paths; ID-based transaction ordering protection logic; and per-port address decode, admission control, and decode error handling.

### [](https://github.com/alexforencich/verilog-axi#axi_crossbar_wr-module)

### `axi_crossbar_wr` module

AXI nonblocking crossbar interconnect with parametrizable data and address interface widths and master and slave interface counts. Write interface only. Supports all burst types. Fully nonblocking with completely separate read and write paths; ID-based transaction ordering protection logic; and per-port address decode, admission control, and decode error handling.

### [](https://github.com/alexforencich/verilog-axi#axi_dma-module)

### `axi_dma` module

AXI to AXI stream DMA engine with parametrizable data and address interface widths. Generates full-width INCR bursts only, with parametrizable maximum burst length. Supports unaligned transfers, which can be disabled via parameter to save on resource consumption. Wrapper for `axi_dma_rd` and `axi_dma_wr`.

### [](https://github.com/alexforencich/verilog-axi#axi_dma_desc_mux-module)

### `axi_dma_desc_mux` module

Descriptor multiplexer/demultiplexer for AXI DMA module. Enables sharing the AXI DMA module between multiple request sources, interleaving requests and distributing responses.

### [](https://github.com/alexforencich/verilog-axi#axi_dma_rd-module)

### `axi_dma_rd` module

AXI to AXI stream DMA engine with parametrizable data and address interface widths. Generates full-width INCR bursts only, with parametrizable maximum burst length. Supports unaligned transfers, which can be disabled via parameter to save on resource consumption.

### [](https://github.com/alexforencich/verilog-axi#axi_dma_wr-module)

### `axi_dma_wr` module

AXI stream to AXI DMA engine with parametrizable data and address interface widths. Generates full-width INCR bursts only, with parametrizable maximum burst length. Supports unaligned transfers, which can be disabled via parameter to save on resource consumption.

### [](https://github.com/alexforencich/verilog-axi#axi_dp_ram-module)

### `axi_dp_ram` module

AXI dual-port RAM with parametrizable data and address interface widths. Supports FIXED and INCR burst types as well as narrow bursts.

### [](https://github.com/alexforencich/verilog-axi#axi_fifo-module)

### `axi_fifo` module

AXI FIFO with parametrizable data and address interface widths. Supports all burst types. Optionally can delay the address channel until either the write data is completely shifted into the FIFO or the read data FIFO has enough capacity to fit the whole burst. Wrapper for `axi_fifo_rd` and `axi_fifo_wr`.

### [](https://github.com/alexforencich/verilog-axi#axi_fifo_rd-module)

### `axi_fifo_rd` module

AXI FIFO with parametrizable data and address interface widths. AR and R channels only. Supports all burst types. Optionally can delay the address channel until either the read data FIFO is empty or has enough capacity to fit the whole burst.

### [](https://github.com/alexforencich/verilog-axi#axi_fifo_wr-module)

### `axi_fifo_wr` module

AXI FIFO with parametrizable data and address interface widths. WR, W, and B channels only. Supports all burst types. Optionally can delay the address channel until the write data is shifted completely into the write data FIFO, or the current burst completely fills the write data FIFO.

### [](https://github.com/alexforencich/verilog-axi#axi_interconnect-module)

### `axi_interconnect` module

AXI shared interconnect with parametrizable data and address interface widths and master and slave interface counts. Supports all burst types. Small in area, but does not support concurrent operations.

Wrappers can generated with `axi_interconnect_wrap.py`.

### [](https://github.com/alexforencich/verilog-axi#axi_ram-module)

### `axi_ram` module

AXI RAM with parametrizable data and address interface widths. Supports FIXED and INCR burst types as well as narrow bursts.

### [](https://github.com/alexforencich/verilog-axi#axi_ram_rd_if-module)

### `axi_ram_rd_if` module

AXI RAM read interface with parametrizable data and address interface widths. Handles bursts and presents a simplified internal memory interface. Supports FIXED and INCR burst types as well as narrow bursts.

### [](https://github.com/alexforencich/verilog-axi#axi_ram_wr_if-module)

### `axi_ram_wr_if` module

AXI RAM write interface with parametrizable data and address interface widths. Handles bursts and presents a simplified internal memory interface. Supports FIXED and INCR burst types as well as narrow bursts.

### [](https://github.com/alexforencich/verilog-axi#axi_ram_wr_rd_if-module)

### `axi_ram_wr_rd_if` module

AXI RAM read/write interface with parametrizable data and address interface widths. Handles bursts and presents a simplified internal memory interface. Supports FIXED and INCR burst types as well as narrow bursts. Wrapper for `axi_ram_rd_if` and `axi_ram_wr_if`.

### [](https://github.com/alexforencich/verilog-axi#axi_register-module)

### `axi_register` module

AXI register with parametrizable data and address interface widths. Supports all burst types. Inserts simple buffers or skid buffers into all channels. Channel register types can be individually changed or bypassed. Wrapper for `axi_register_rd` and `axi_register_wr`.

### [](https://github.com/alexforencich/verilog-axi#axi_register_rd-module)

### `axi_register_rd` module

AXI register with parametrizable data and address interface widths. AR and R channels only. Supports all burst types. Inserts simple buffers or skid buffers into all channels. Channel register types can be individually changed or bypassed.

### [](https://github.com/alexforencich/verilog-axi#axi_register_wr-module)

### `axi_register_wr` module

AXI register with parametrizable data and address interface widths. WR, W, and B channels only. Supports all burst types. Inserts simple buffers or skid buffers into all channels. Channel register types can be individually changed or bypassed.



### 波形

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202307270902686.png)

### 后仿
[Verilog PLI Tutorial Part-VI](https://www.asic-world.com/verilog/pli6.html)