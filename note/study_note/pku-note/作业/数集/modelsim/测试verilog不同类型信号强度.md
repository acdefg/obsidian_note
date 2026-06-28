---
title: 测试verilog不同类型信号强度
tags: ["note"]
created: 星期日, 六月 28日 2026, 2:58:15 下午
modified: 星期日, 六月 28日 2026, 5:17:00 下午
---

# 仿真说明
仿真环境：modelsim
仿真目的：测试 verilog 不同类型信号强度
仿真程序：
~~~verilog
`timescale 1ns/1ns
module test;
	supply1   vdd;
	supply0   gnd;
	wor	d_wor;
	wand	d_wand;
	trireg	d_trireg;
	tri1	d_tri1;
	tri0	d_tri0;
	
	reg       in0;
	
	initial begin
		in0 = 1'bz;
		#10 in0 = 1;
		#10 in0 = 1'bz;
		#10 in0 = 0;
		#10 in0 = 1'bz;
		#10 in0 = 1'bx;
		#100 $finish;
	end
	
	assign d_wor    = in0;
	assign d_wand  = in0;
	assign d_trireg = in0;
	assign d_tri1    = in0;
	assign d_tri0    = in0;
	assign vdd        = in0;
	assign gnd        = in0;
	initial $monitor($time,, "  d_trireg strenth is %v, vdd strenth is %v, d_tri1 strength is %v", d_trireg, vdd, d_tri1);
endmodule
~~~

# 仿真结果
命令窗口输出：
```txt
#         0    d_trireg strenth is MeX, vdd strenth is Su1, d_tri1 strength is Pu1
#         10   d_trireg strenth is St1, vdd strenth is Su1, d_tri1 strength is St1
#         20   d_trireg strenth is Me1, vdd strenth is Su1, d_tri1 strength is Pu1
#         30   d_trireg strenth is St0, vdd strenth is Su1, d_tri1 strength is St0
#         40   d_trireg strenth is Me0, vdd strenth is Su1, d_tri1 strength is Pu1
#         50   d_trireg strenth is StX, vdd strenth is Su1, d_tri1 strength is StX
```

波形输出：

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921230607.png)

> 紫色实线为高电平，黑色实现为低电平，蓝色实线为高阻态，绿色实线为 St1；红色虚线为 Pu1，黑色虚线为 Pu0，绿色虚线为 MeX

# 仿真分析
使用 $display 函数的%V 输出格式打印信号强度，其结果用用 3 个符号表示，前两个字母表示信号强度，而第三个符号表示信号的逻辑值。
1、 前两位字母含义表：

| 标记符 | 强度名                   | 强度值表示 |
| ------ | ------------------------ | ------- |
| Su     | 电源级驱动 (Supply drive) | 7         |
|St |强驱动 (Strong drive) |6 |
|Pu |上拉级驱动 (Pull drive)| 5 |
|La |大容性 (Large capacitor)| 4 |
|We |弱驱动 (Weak drive) |3  |
|Me|中级容性 (Medium capacitor)| 2 |
|Sm |小容性 (Small capacitor)| 1  |
|Hi |高容性 (High capacitor) |0|

2、最后一位数字含义表：

| 逻辑值 | 表示意义                    |
| ------ | ------------------ |
| 0      | 表示逻辑 0 值                 |
| 1      | 表示逻辑 1 值                 |
| X      | 表示逻辑不定态              |
| Z      | 表示逻辑高阻态              |
| L      | 表示逻辑 0 值，或者逻辑高阻态 |
| H      | 表示逻辑 1 值，或者逻辑高阻态 |

3、trireg、vdd、d_tri1 信号强度变化和 in0 信号值的变化关系：

| in0  | in0 信号值 | trireg | vdd | d_tri1 |
| ---- | ------ | ------ | --- | ------ |
| 1'bz | 高阻   | Mex    | Su1 | Pu1    |
| 1    | 高电平 | St1    | Su1 | St1    |
| 1'bz | 高阻   | Me1    | Su1 | Pu1    |
| 0    | 低电平 | St0    | Su1 | St0    |
| 1'bz | 高阻   | Me0    | Su1 | Pu1    |
| 1'bx | 不定态 | Stx    | Su1 | StX    |

从以上分析可知:
- supply 类型的 vdd 的信号值和信号强度不随 in0 的变化而变化，始终为最高强度；
- trireg 类型和 tri1 类型信号
	- 会在 in0 处于高电平、低电平和不定态的时候跟随其强度和信号值，
	- 而在 in0 处于高阻时，trireg 会以 Me（中等容性）强度保持前一状态数值，tri1 则会处于默认上拉状态（Pu1）
