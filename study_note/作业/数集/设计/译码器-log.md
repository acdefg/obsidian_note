3-8 译码器，8-3 编码器，格雷码计数器，参数化以上

## 老师给的
<div align=center><img src="https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/3a12c9388992ff1dcedb03fa91cf252.jpg" style="zoom: 50%;"></div>
<img src="https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/e6ccb36aa2e87971e0f038eb4cf4cc7.jpg"style="zoom:50%"/>
<div align=center><img src="https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/a3e7e64c6575429f11ac203f0f656ee.jpg" style="zoom: 67%;"></div>

## CSND
[Verilog设计参数化的译码器与编码器，以及设计4位格雷码计数器_ty_:-)的博客-CSDN博客](https://blog.csdn.net/Zhong_ty/article/details/127614249)
	[独热码_百度百科](https://baike.baidu.com/item/%E7%8B%AC%E7%83%AD%E7%A0%81/1428731)

## 群里发的
[文件路径](D:\Users\Documents\WeChat Files\wxid_1316strf57mi22\FileStorage\File\2022-10\ex_src)

## run_log
### Decoder_short
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104170757.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104211624.png)

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104211553.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104211843.png)

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104211903.png)

```verilog
`timescale 1 ns/ 1 ps
module decoder_coder #(
	parameter n=3,
			  m=1<<n
	)(
	input wire[n-1:0]in,
	output reg[m-1:0]y
	);
	//one-hot
	always@(*)y=1<<in;
endmodule 
```


```verilog                                                                                
// Generated on "11/04/2022 16:59:40"                                                                               
// Verilog Test Bench template for design : decoder_coder
// Simulation tool : ModelSim (Verilog)
// 
`timescale 1 ns/ 1 ps
module decoder_coder_vlg_tst();
parameter N = 2;
parameter M = 1<<N;
// test vector input registers
reg [N-1:0] in;
// wires                                               
wire [M-1:0]  y;

// assign statements (if any)                         
decoder_coder #(.n(N), .m(M)) 
	i1(
// port map - connection between master ports and signals/registers   
	.in(in),
	.y(y)
);

integer i;
initial                                                
begin
	for(i=0; i<M; i=i+1) begin
		in = i;
		#10; 
	end	
	#10 $stop;
end

initial 
	$monitor("in = %b ---> y = %b ", in, y);
                                                 
endmodule

```

### Decoder_long
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104180601.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104213430.png)

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104213454.png)


```verilog
`timescale 1 ns/ 1 ps
module decoder_coder #(
	parameter n=3,
				 m=1<<n
	)(
	input wire[n-1:0]in,
	output reg[m-1:0]y
	);
	//one-hot
	always@(*)
		case(in)
			3'b000: y = 8'b0000_0001;
			3'b001: y = 8'b0000_0010;
			3'b010: y = 8'b0000_0100;
			3'b011: y = 8'b0000_1000;
			3'b100: y = 8'b0001_0000;
			3'b101: y = 8'b0010_0000;
			3'b110: y = 8'b0100_0000;
			3'b111: y = 8'b1000_0000;
		endcase
endmodule 
```

### Coder_short
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104182352.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104183006.png)

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104183031.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104183342.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104183359.png)

```verilog
module decoder_coder #(
	parameter n=3,
			  m=1<<n
)(
	input wire[m-1:0]in,
	output reg[n-1:0]y
);
	integer i;
	always@(*)
	begin:encoder
		for (i=m-1;i>0;i=i-1)
			if(in[i]==1)
			begin
				y = i;
				disable encoder;  //jump out of loop
			end
			else y = 0;
	end
	
endmodule 
```

```verilog
`timescale 1 ns/ 1 ps
module decoder_coder_vlg_tst();
parameter N = 3;
parameter M = 1<<N;
// test vector input registers
reg [M-1:0] in;
// wires                                               
wire [N-1:0]  y;

// assign statements (if any)                          
decoder_coder #(.n(N), .m(M))
	i1 (
// port map - connection between master ports and signals/registers   
	.in(in),
	.y(y)
);

integer i;
initial                                                
begin     
	for(i=0; i<M; i=i+1) begin
		in = 1<<i;
		#10;
	end  
	
	#10 $stop;
end

initial 
	$monitor("in = %b ---> y = %b ", in, y);
endmodule
```

### Coder_long
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104184220.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104192251.png)

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104192306.png)

```verilog
`timescale 1 ns/ 1 ps
module decoder_coder #(
	parameter n=3,
				 m=1<<n
	)(
	input wire[m-1:0]in,
	output reg[n-1:0]y
	);
	//one-hot
	always@(*)
		casez(in)
			8'b1???_????	: y = 3'b111;
			8'b01??_????	: y = 3'b110;
			8'b001?_????	: y = 3'b101;
			8'b0001_????	: y = 3'b100;
			8'b0000_1???	: y = 3'b011;
			8'b0000_01??	: y = 3'b010;
			8'b0000_001?	: y = 3'b001;
			8'b0000_0001    : y = 3'b000;
			default         : y = 3'b000;
		endcase
endmodule 
```

### 格雷码计数器

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104193624.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104193643.png)

![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104193853.png)


```verilog
module gray_counter(
	input clk,
	input rst_n,
	output reg[3:0]gray
);
	always@(posedge clk or negedge rst_n)
	if(!rst_n)	gray <= 4'b0000;
	else
		case(gray)
			4'b0000 : gray <= 4'b0001;
			4'b0001 : gray <= 4'b0011;
			4'b0011 : gray <= 4'b0010;
			4'b0010 : gray <= 4'b0110;
			4'b0110 : gray <= 4'b0111;
			4'b0111 : gray <= 4'b0101;
			4'b0101 : gray <= 4'b0100;
			4'b0100 : gray <= 4'b1100;
			4'b1100 : gray <= 4'b1101;
			4'b1101 : gray <= 4'b1111;
			4'b1111 : gray <= 4'b1110;
			4'b1110 : gray <= 4'b1010;
			4'b1010 : gray <= 4'b1011;
			4'b1011 : gray <= 4'b1001;
			4'b1001 : gray <= 4'b1000;
			4'b1000 : gray <= 4'b0000;
			default 	 : gray <= 4'bx;
		endcase

endmodule 
```

```verilog

`timescale 1 ns/ 1 ps
module gray_counter_vlg_tst();

// test vector input registers
reg clk;
reg rst_n;
// wires                                               
wire [3:0]  gray;

// assign statements (if any)                          
gray_counter i1 (
// port map - connection between master ports and signals/registers   
	.clk(clk),
	.gray(gray),
	.rst_n(rst_n)
);

initial 
begin 
	rst_n = 0;
	clk = 0;
	#5 rst_n = 1;
	#60 rst_n = 0;
	#5 rst_n = 1;
	#100 $stop;
end

always #5 clk = ~clk;

initial $monitor($time,": state: %b",gray);
endmodule
```


