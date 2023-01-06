flag_gen
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105151453.png)
fifo
rst = 0
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105152921.png)
rst = 1
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105153313.png)
## slaver_fifo.v
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230106011305.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230106011322.png)

当 valid 为高时，表示要写入数据。如果该时钟周期 ready 为高，则表示已经将数据写入；如果该时钟周期 ready 为低，则需要等到 ready 为高的时钟周期才可以成功将数据写入。
模块结构和功能如下：
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105220612.png)
fifo 设计的是一个同步 fifo，设置了以下几个参数分别代表深度、宽度、地址位数和最大存储量，设置参数以便在调用模块的时候修改，使得模块更具备通用性。

```verilog
    parameter FIFO_DEPTH = 4'b1000;    //d8
    parameter FIFO_WIDE = 6'b10_0000;    //d32
    parameter FIFO_PTR_WIDE = 2'b11;
    parameter MAX_CNT = 4'b1000; 
```

测试代码设置了输入数据递增变化和状态循环变化逻辑，初始选通 `slvx_en_i` 通道选择信号，测试不同的 `rst_i` ,
`a2sx_ack_i`  , `chx_valid_i ` 的信号波形 ：

```verilog
    initial chx_data_i = 0; 
    always @ (posedge clk_i) chx_data_i <= chx_data_i + 1;
        initial count = 0;
        
    always @(posedge clk_i)  count = count + 1;
    always @(count) begin
    if (count == 5'b11111)
        {rst_i,a2sx_ack_i,chx_valid_i}  =  {rst_i,a2sx_ack_i,chx_valid_i} + 1;
    else 
        {rst_i,a2sx_ack_i,chx_valid_i}  =  {rst_i,a2sx_ack_i,chx_valid_i};
    end
```

## arbiter
优先级变化，通道选择改变
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230106015928.png)
初始化阶段和通道 1 优先阶段：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230106020023.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230106024003.png)

```verilog
#20  slv0_val_i = 1;
#10  slv0_req_i = 1;
#10  rst_n = 1;
#10  f2a_id_req_i = 1;
```
设置了 slaver0 的 val 信号早于 rst 产生，可以看到直到 `rst_n` `f2a_id_req_i` , 全部为 1 之后，才开始选择有效通道

## fmt
数据传输过程中，更改数据长度无效
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230106114839.png)
