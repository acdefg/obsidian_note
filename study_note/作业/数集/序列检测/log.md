<img src="https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221029195353.png" style="zoom: 67%;" />     <img src="https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221029194813.png" style="zoom: 43%;" />

FSM1
```verilog
`timescale 1ns/1ns 
module fsm1 ( 
input wire clk , 
			reset_n , 
			X , 
output wire Z1 , 
			Z2 
); 
	reg [1: 0] cnt; 
	always @(posedge clk, negedge reset_n) 
		if( ! reset_n ) 
			cnt <= 2'b00; 
		else if( X ) begin 
			if(cnt != 2'b11) 
				cnt <= cnt + 1'b1; 
		end 
	assign Z1 = ^ cnt ; 
	assign Z2 = & cnt ; 
endmodule
```

FSM2
```verilog
module fsm2 ( 
input wire clk , 
			reset , 
			x , 
output wire Z1 , 
			Z2 
); 
	reg [1: 0] state; 
	parameter ST0 = 2'b00, ST1 = 2'b01, 
				ST2 = 2'b11, ST3 = 2'b10; 
	always @(posedge clk, negedge reset) 
		if(!reset) 
			state <= ST0; 
		else if( x ) 
			case(state) 
				ST0 : state <= ST1; 
				ST1 : state <= ST2; 
				ST2 : state <= ST3; 
				default: state <= ST3; 
		endcase 
	assign Z1 = state == ST1 || state == ST2 ; 
	assign Z2 = state == ST3; 
endmodule
```

FSM3
```verilog
module fsm3 ( 
input wire clk , 
			reset , 
			x , 
output wire Z1 , 
			Z2 
			); 
	reg [2: 0] x_dly;
	always @(posedge clk, negedge reset) 
		if(!reset) 
			x_dly <= 0; 
		else begin 
			x_dly[0] <= x; 
			x_dly[1] <= x_dly[0]; 
			x_dly[2] <= x_dly[1] | x_dly[2]; 
		end 
	assign Z1 = x_dly[0] & !x_dly[2]; 
	assign Z2 = x_dly[2] ; 
endmodule
```

```verilog
module FSM_EXP(clk,rst,sin,cout,cstout);
input clk;                                  //状态机时钟
input rst;                                  //复位信号  
input [0:1] sin;                            //来自外部的状态机控制信号
output [4:0] cstout;                        //状态机对外部发出的控制信号信号输出
output [3:0] cout;
reg [3:0] cout;
assign cstout=cst;
parameter s0=0,s1=1,s2=2,s3=3,s4=4;         //定义状态参数
reg[4:0] cst,nst;                           //当前状态、下一状态
always@(posedge clk or negedge rst)         //主控时序进程
begin
	if(!rst) cst<=s0;                       //复位有效时，下一状态进入s0
	else cst<=nst; 
end
always@(cst or sin)begin                    //主控组合进程 
	case(cst)
		s0:begin cout<=5;
			if(sin==2'b00) nst<=s0;
			else nst<=s1;
			end			
		s1:begin cout<=8;
			if(sin==2'b01) nst<=s1;
			else nst<=s2;
			end 			
		s2:begin cout<=12;
			if(sin==2'b10) nst<=s2;
			else nst<=s3;
			end		
		s3:begin cout<=14;
			if(sin==2'b11) nst<=s3;
			else nst<=s4;
			end		
		s4:begin cout<=9;
			nst<=s0;
			end
		default:nst<=s0;	//现太若未出现以上各太，返回初态s0
		endcase
	end
endmodule
```



