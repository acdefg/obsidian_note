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

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104173909.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104174410.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104175901.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104175915.png)

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
	for(i=0; i<M; i=i+1)
		#10 
		in = i;
	#10 $stop;
end

initial 
	$monitor("in = %b ---> y = %b ", in, y);
                                                 
endmodule

```

### Decoder_long
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221104180601.png)

