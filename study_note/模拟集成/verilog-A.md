简单教程
[Verilog-A 语言简单入门教程 – Analog-Life](https://www.analog-life.com/2022/04/veriloga-quick-learning/)

手册
[https://www.siue.edu/\~gengel/ece585WebStuff/OVI\_VerilogA.pdf](https://www.siue.edu/~gengel/ece585WebStuff/OVI_VerilogA.pdf)

## 事件
### 初始事件 initial step
在 Verilog HDL 中，有一种叫做 initial 的语句，这个语句内的内容会在仿真起始阶段执行一次。Verilog-A 中也有类似的事件控制语句 @(initial_step)，这个控制事件中的语句在仿真迭代开始时会执行一次。一般都用它来赋初值或者计算一些变量的初始值。它的用法是：
```
analog begin // 事件控制语句必须放在 analog 语句内部
    @(initial_step) begin
       bits = 0;
       error = 0;
    end
end
```

