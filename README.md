# de2-115-setup
tracking files used to setup my DE2-115 FPGA board
# setup
```bash
sudo apt-get install iverilog
sudo apt-get install gtkwave
```
# module for tests
```vim
module mini_counter(
input clk,
input rst, //active low as button is normally high
input en_b,
output [8:0] cnt,
output test_out,
output test_out_1
);

//reg [31:0] cntr;

reg [31:0] prescaler;
reg [31:0] cnt_internal;

always @(posedge clk)
begin
if (rst)
  prescaler =0;
else
  prescaler = prescaler +1;
end

assign test_out   = prescaler[24];
assign test_out_1 = prescaler[23];

always @(posedge clk or posedge rst)
begin
if (rst)
begin
  cnt_internal = 32'h00000000;
end
  else if (!en_b)
begin
  cnt_internal = cnt_internal + 1;
end
  end

assign cnt = cnt_internal[31:24];


endmodule
```

# Reference

https://github.com/albertxie/iverilog-tutorial
