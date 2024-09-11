# UART_protocol

## CODE:
```verilog
`timescale 1ns / 1ps 
 
module top(
    input clk,
    input start,
    input [7:0] txin,
    output reg tx, 
    input rx,
    output [7:0] rxout,
    output rxdone, txdone
    );
    
 parameter clk_value = 100_000;
 parameter baud = 9600;
 
 parameter wait_count = clk_value / baud;
 
 reg bitDone = 0;
 integer count = 0;
 reg bitDone = 0;  //it is trigger pulse
 integer count = 0;  //it will keep the track of required clock pulse to wait
 parameter idle = 0, send = 1, check = 2;
 reg [1:0] state = idle;
 
@@ -64,17 +63,18 @@ module top(
  end
 
 end
 
 ///////////////////////TX Logic
 reg [9:0] txData;///stop bit data start
 integer bitIndex = 0; ///reg [3:0];
 reg [9:0] shifttx = 0;
 reg [9:0] txData;   //{stop bit, Txin[7:0], Start bit}
 integer bitIndex = 0; //it will track the record of bit which transmitted  
 reg [9:0] shifttx = 0;  // it will used during debbuging when we compare recived data
 
 
 always@(posedge clk)
 begin
 case(state)
 idle : 
 idle: 
     begin
           tx       <= 1'b1;
           txData   <= 0;
@@ -83,7 +83,7 @@ module top(
           
            if(start == 1'b1)
              begin
                txData <= {1'b1,txin,1'b0};
                txData <= {1'b1,txin,1'b0}; //{stop bit, Txin[7:0], Start bit}
                state  <= send;
              end
            else
@@ -95,7 +95,7 @@ module top(
  send: begin
           tx       <= txData[bitIndex];
           state    <= check;
           shifttx  <= {txData[bitIndex], shifttx[9:1]};
           shifttx  <= {txData[bitIndex], shifttx[9:1]}; //shifting logic. evert time when txData updated, this shifttx will update and then shift towards right
  end 
  
  check: 
@@ -127,11 +127,11 @@ assign txdone = (bitIndex == 9 && bitDone == 1'b1) ? 1'b1 : 1'b0;
 
 
 ////////////////////////////////RX Logic
 integer rcount = 0;
 integer rindex = 0;
 parameter ridle = 0, rwait = 1, recv = 2, rcheck = 3;
 integer rcount = 0;  //it will give pulse during the middle of the data
 integer rindex = 0;  //it will keep the track of recived data bits (similar like bit_index)
 parameter ridle = 0, rwait = 1, recv = 2;
 reg [1:0] rstate;
 reg [9:0] rxdata;
 reg [9:0] rxdata;  //Similar as txdata, 
 always@(posedge clk)
 begin
 case(rstate)
@@ -162,7 +162,7 @@ begin
       begin
          rcount <= 0;
          rstate <= recv;
          rxdata <= {rx,rxdata[9:1]}; 
          rxdata <= {rx,rxdata[9:1]};  //shift logic
       end
end
 
 
recv : 
begin
     if(rindex <= 9) 
      begin
      if(bitDone == 1'b1) 
        begin
        rindex <= rindex + 1;
        rstate <= rwait;
        end
      end
      else
        begin
        rstate <= ridle;
        rindex <= 0;
        end
end
 
 
default : rstate <= ridle;
 
 
 endcase
 end
 
 
assign rxout = rxdata[8:1]; 
assign rxdone = (rindex == 9 && bitDone == 1'b1) ? 1'b1 : 1'b0;
 
 
 endmodule
 //////////////////////////////////
```
## Testbench
```verilog
module tb;
 
    reg clk = 0;
    reg start = 0;
    reg [7:0] txin;
    wire [7:0] rxout;
    wire rxdone, txdone;
 
   wire txrx;
   
 top dut (clk, start, txin, txrx,txrx, rxout, rxdone, txdone );
 integer i = 0;
 
 initial 
 begin
 start = 1;
 for(i = 0; i < 10; i = i + 1) begin
 txin = $urandom_range(10 , 200);
 @(posedge rxdone);
 @(posedge txdone);
 end
 $stop; 
 end
 
 always #5 clk = ~clk;
 
 endmodule
```
