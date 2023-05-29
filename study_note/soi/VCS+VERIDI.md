![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527112358.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527112423.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527112525.png)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527112545.png)

### filelist 自动生成指令
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527115954.png)

```bash
find -name "*.v" >filelist.f
```
hhhh，这指令可真好使，下级目录的也能生成，爱了爱了
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230527120317.png)

[VCS+VERDI+Makefile 初体验 - 知乎](https://zhuanlan.zhihu.com/p/563041890)

### vcd

```verilog
   initial

    begin

        $dumpfile("./zoom_out.vcd");

        $dumpvars(0,my_zoomout_tb);

    end
    
```

```verilog
  initial begin
    $fsdbDumpfile("novas.fsdb");
    $fsdbDumpvars("+all");
  end  
```

```verilog
    +vcs+flush+all                        \
    +vcs+dumpvars                         \
    -vcd ./1.vdc                          \


```