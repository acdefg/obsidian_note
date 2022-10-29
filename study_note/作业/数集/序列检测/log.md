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
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20221029194813.png)

FSM3
