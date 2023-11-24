[【精选】verilog流水线和乘法器的设计（需要请自取）\_verilog乘法运算优化\_松花江路2600号的博客-CSDN博客](https://blog.csdn.net/weixin_47032674/article/details/115032405)

普通乘法器
```verilog
module    mult_low
    #(parameter N=4,
      parameter M=4)
     (
      input                     clk,
      input                     rstn,
      input                     data_rdy ,  //数据输入使能
      input [N-1:0]             mult1,      //被乘数
      input [M-1:0]             mult2,      //乘数

      output                    res_rdy ,   //数据输出使能
      output [N+M-1:0]          res         //乘法结果
      );

    //calculate counter
    reg [31:0]           cnt ;
    //乘法周期计数器
    wire [31:0]          cnt_temp = (cnt == M)? 'b0 : cnt + 1'b1 ;
    always @(posedge clk or negedge rstn) begin
        if (!rstn) begin
            cnt    <= 'b0 ;
        end
        else if (data_rdy) begin    //数据使能时开始计数
            cnt    <= cnt_temp ;
        end
        else if (cnt != 0 ) begin  //防止输入使能端持续时间过短
            cnt    <= cnt_temp ;
        end
        else begin
            cnt    <= 'b0 ;
        end
    end

    //multiply
    reg [M-1:0]          mult2_shift ;
    reg [M+N-1:0]        mult1_shift ;
    reg [M+N-1:0]        mult1_acc ;
    always @(posedge clk or negedge rstn) begin
        if (!rstn) begin
            mult2_shift    <= 'b0 ;
            mult2_shift    <= 'b0 ;
            mult1_acc      <= 'b0 ;
        end
        else if (data_rdy && cnt=='b0) begin  //初始化
            mult1_shift    <= {{(N){1'b0}}, mult1} << 1 ;  
            mult2_shift    <= mult2 >> 1 ;  
            mult1_acc      <= mult2[0] ? {{(N){1'b0}}, mult1} : 'b0 ;
        end
        else if (cnt != M) begin
            mult1_shift    <= mult1_shift << 1 ;  //被乘数乘2
            mult2_shift    <= mult2_shift >> 1 ;  //乘数右移，方便判断
            //判断乘数对应为是否为1，为1则累加
            mult1_acc      <= mult2_shift[0] ? mult1_acc + mult1_shift : mult1_acc ;
        end
        else begin
            mult2_shift    <= 'b0 ;
            mult2_shift    <= 'b0 ;
            mult1_acc      <= 'b0 ;
        end
    end

    //results
    reg [M+N-1:0]        res_r ;
    reg                  res_rdy_r ;
    always @(posedge clk or negedge rstn) begin
        if (!rstn) begin
            res_r          <= 'b0 ;
            res_rdy_r      <= 'b0 ;
        end  
        else if (cnt == M) begin
            res_r          <= mult1_acc ;  //乘法周期结束时输出结果
            res_rdy_r      <= 1'b1 ;
        end
        else begin
            res_r          <= 'b0 ;
            res_rdy_r      <= 'b0 ;
        end
    end

    assign res_rdy       = res_rdy_r;
    assign res           = res_r;

endmodule

```

流水线乘法器
```verilog
module    mult_cell
    #(parameter N=4,
      parameter M=4)
    (
      input                     clk,
      input                     rstn,
      input                     en,
      input [M+N-1:0]           mult1,      //被乘数
      input [M-1:0]             mult2,      //乘数
      input [M+N-1:0]           mult1_acci, //上次累加结果

      output reg [M+N-1:0]      mult1_o,     //被乘数移位后保存值
      output reg [M-1:0]        mult2_shift, //乘数移位后保存值
      output reg [N+M-1:0]      mult1_acco,  //当前累加结果
      output reg                rdy );

    always @(posedge clk or negedge rstn) begin
        if (!rstn) begin
            rdy            <= 'b0 ;
            mult1_o        <= 'b0 ;
            mult1_acco     <= 'b0 ;
            mult2_shift    <= 'b0 ;
        end
        else if (en) begin
            rdy            <= 1'b1 ;
            mult2_shift    <= mult2 >> 1 ;
            mult1_o        <= mult1 << 1 ;
            if (mult2[0]) begin
                //乘数对应位为1则累加
                mult1_acco  <= mult1_acci + mult1 ;  
            end
            else begin
                mult1_acco  <= mult1_acci ; //乘数对应位为1则保持
            end
        end
        else begin
            rdy            <= 'b0 ;
            mult1_o        <= 'b0 ;
            mult1_acco     <= 'b0 ;
            mult2_shift    <= 'b0 ;
        end
    end 

endmodule

```

