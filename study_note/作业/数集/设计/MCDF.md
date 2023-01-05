flag_gen
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105151453.png)
fifo
rst = 0
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105152921.png)
rst = 1
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105153313.png)
slaver_fifo.v
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105214933.png)
当 valid 为高时，表示要写入数据。如果该时钟周期 ready 为高，则表示已经将数据写入；如果该时钟周期 ready 为低，则需要等到 ready 为高的时钟周期才可以成功将数据写入。
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105220612.png)
fifo 设计的是一个同步 fifo，设置了以下几个参数分别代表深度、宽度、地址位数和最大存储量，设置参数以便在调用模块的时候修改，使得模块更具备通用性。
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230105221255.png)
