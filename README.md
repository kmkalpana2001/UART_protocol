# UART_protocol

# Contents 
 <div class="toc">
  <ul>
    <li><a href="#header-1">UART Interface</a></li>
	</ul>
</div>

 <div class="toc">
  <ul>
    <li><a href="#header-2">How does it works?</a></li>
</ul>
</div>

<div class="toc">
  <ul>
    <li><a href="#header-3">Clock for desired Baud</a></li>
	</ul>
</div>

<div class="toc">
  <ul>
    <li><a href="#header-4">URAT Transmitter(TxD)</a></li>
</ul>
</div>

<div class="toc">
  <ul>
    <li><a href="#header-5">UART Reciever(RxD)</a></li>
	</ul>
</div>

<div class="toc">
  <ul>
    <li><a href="#header-6">Design Code for UART</a></li>
	</ul>
</div

 <div class="toc">
  <ul>
    <li><a href="#header-7">Test Bench Code</a></li>
	</ul>
</div>



## <h1 id="header-1">UART Interface</h1>

 UART stands for Universal Asynchronous Receiver/Transmitter.
 
• It uses two lines viz. TxD and RxD for transmit and receive functions.

• It is asynchronous communication. Hence data rate should be matched between devices wanting to communicate.

• It supports data rate of about 230 to 460 Kbps (maximum).

• It supports distance of about 50 feet.

• No common clock is being used. Moreover devices use their own independent clock signals.

• UART protocol consists of start bit, 8 bits of data and stop bit.

• It is also known as RS232 interface.

![image](https://github.com/user-attachments/assets/cf777a7b-7367-4a73-bec2-f38f30aa1805)

VCC and GND pins are not shown here. We assume that for both the devices VCC and GND are at same level, then only will able to achieve error free communication.


## <h2 id="header-1">How does it works?</h2>

![image](https://github.com/user-attachments/assets/28f2d950-cc69-4bdb-8694-e5b5dc1900c4)

When the interface is in an idle state, we wil lbe getting a logic high on its pin.

whenever any one device wish to start a communication, it will first send zero on the line. Data length could be varied between 5 to 9 and common data length is 8.

Whenever we want to send data from device-1 to device-2 , we will makee Tx zero that will be sense by device-2 and understand that device want to send the data to it, then will serially send the data from Tx to Rx.

Now, if we consider device-2 we need to sample the values which are sent by device-1.

We need to focus on bid duration, we need to find out the middle and we will be sampling that value and storing it into a shift register . We will be sampling all the 8-bits of a frame.

Then we will be collecting the parity bit, checking whether parity is matching to the parity that we get from the data bits and finally we sample and stop.


## <h3 id="header-3">Clock for desired Baud</h3>

We need to find out the wait count, wait count for an half clk period and wait count for an entire bid duration.

Once we complete the duration of a bit we will be generating a trigger. Similarly we generate the trigger on completing tsecond bit also.

So, we just need to find out the wait count for this entire period.

Wait count = Fclk/ Fbaud = Input clk frequency / desured baud rate

**How to generate trigger**:- We will keep on counting the clk tick. Once we reach to a wait count, we will be generating a trigger.


## <h4 id="header-4">UART Transmitter(TxD)</h4>

![image](https://github.com/user-attachments/assets/805b82b8-bb27-4ba4-aba7-ab181701b971)


## <h5 id="header-5">UART Transmitter(TxD)</h5>

![image](https://github.com/user-attachments/assets/a9b95b1f-d19b-4b35-8a95-d092b75fac04)

![image](https://github.com/user-attachments/assets/3d7fa84f-ef0d-4ec2-bf5e-eac24186ed55)



##  <h6 id="header-6">Design Code for UART</h6>
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
## <h7 id="header-7">Test Bench Code</h7>
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
