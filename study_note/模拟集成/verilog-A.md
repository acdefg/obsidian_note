简单教程
[Verilog-A 语言简单入门教程 – Analog-Life](https://www.analog-life.com/2022/04/veriloga-quick-learning/)

官方但无索引的手册
[https://www.siue.edu/\~gengel/ece585WebStuff/OVI\_VerilogA.pdf](https://www.siue.edu/~gengel/ece585WebStuff/OVI_VerilogA.pdf)
有索引的手册
[edadownload.software.keysight.com/eedl/ads/2011\_01/pdf/verilogaref.pdf](http://edadownload.software.keysight.com/eedl/ads/2011_01/pdf/verilogaref.pdf)
官方但有索引的手册文件
[veriloga使用手册. - Analog/RF IC 资料共享 - EETOP 创芯网论坛 (原名：电子顶级开发网) -](https://bbs.eetop.cn/thread-250118-1-1.html)

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
## functions
### timer
The timer function schedules an event that occurs at an absolute time (as specified by start_time). The analog simulator places a time point at, or just beyond, the time of the event. If period is specified, then the timer function schedules subsequent events at multiples of the period.

> [!note] explain
> 
 timer_function ::=
timer ( start_time [ , period ] )
start_time ::=
expression
period ::=
expression


> [!NOTE] usage
	module bitStream (out) ;
	output out ;
	electrical out ;
	parameter period = 1.0 ;
	integer x ;
	analog begin
		@(timer(0, period))
		x = $random + 0.5 ;
		V(out) <+ transition( x, 0.0, period/100.0 ) ;
	end
	endmodule

### cross

> [!NOTE] explain
cross_function ::=
cross ( expression [ , opt_args ] )
opt_args ::=
direction [ , time_tol [ , expression_tol ] ]
direction ::=
+1 | -1
time_tol ::=
expression
expression_tol ::=
expression

cross 在 expression 与 0 的交点
direction： 0/不设 上下沿都检测
				+1 上升沿
				-1 下降沿
时间和表达容差都必须为正。如果表达式容差设置了，交叉点必须满足时间和表达容差	

```verilog
@(cross(V(smpl) - 2.5, +1))
```




