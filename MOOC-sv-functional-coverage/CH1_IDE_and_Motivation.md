# Chapter 1 IDE and Motivation

[TOC]

## Motivation 1: Knowing how many random values are required to meet Verification

Priority Encoder Design Example:

- Design

```verilog
module penc (
  input [7:0] y,
  output reg [2:0] a
);
  
  always @* begin
    casez(y)
      8'b00000001: a = 0;
      8'b0000001?: a = 1;
      8'b000001??: a = 2;
      8'b00001???: a = 3;
      8'b0001????: a = 4;
      8'b001?????: a = 5;
      8'b01??????: a = 6;
      8'b1???????: a = 7;
      default a = 'bx;
    endcase
    
  end
  
endmodule
```

- Testbench

```verilog
module tb;
  reg [7:0] y;
  wire [2:0] a;
  integer i = 0;
  
  covergroup c;
    option.per_instance = 1;
    coverpoint y {
      bins zero = {8'b00000001};
      wildcard bins one   = {8'b0000001?};
      wildcard bins two   = {8'b000001??};
      wildcard bins three = {8'b00001???};
      wildcard bins four  = {8'b0001????};
      wildcard bins five  = {8'b001?????};
      wildcard bins six   = {8'b01??????};
      wildcard bins seven = {8'b1???????};
    }
    coverpoint a;
  endgroup
  
  c ci;
  
  penc dut(.*);
  
  initial begin
    ci = new();
    for(i=0; i<100; i++) begin
      y = $urandom();
      ci.sample();
      #10;
    end
  end
  
  initial begin
    #1000;
    $finish();
  end
  
endmodule
```



## Motivation 2: Knowing all the transitions are Covered by SPI

Covergroup Example

```verilog
covergroup c @(posedge clk);
    option.per_instance = 1;
    coverpoint dut.state {
        bins start_op = {dut.idle => dut.init};
        bins init_rep = {dut.init[*32]}; // stay in init state for 32 clock cycle
        bins data_send_comp = {dut.init => dut.data_gen => dut.send};
        bins data_send = (dut.send[*32]);
        bins data_send_complete = (dut.send => dut.cont => dut.data_gen => dut.send);
        bins state_start_zero = (dut.send => dut.cont => dut.idle)
    }
endgroup
```



## Motivation 3: All the possible combinations are tested during Verification

Covergroup Example

```verilog
covergroup c @(posedge clk);
    coverpoint rst;
    coverpoint rd;
    coverpoint wr;
    coverpoint din {
        bins lower = {[0:84]};
        bins min = {[85:169]};
        bins high = {[170:255]};
   	}
    coverpoint dout {
        bins lower = {[0:84]};
        bins min = {[85:169]};
        bins high = {[170:255]};
   	}    
 	coverpoint empty;
    coverpoint full;
    cross wr, din {
        ignore_bins unused_wr = binsof(wr) interset {0};
    }
    cross rd, dout {
        ignore_bins unused_rd = binsof(rd) interset {0};
    }
endgroup
```





## How to use IDE

Use EDAplayground for the simulation

- simulator: Aldec Riviera Pro 2020.04

- run.do file

```bash
vsim +acces+r;
run -all;
acdb save
acdb report -db fcover.acdb -txt -o cov.txt -verbose
exec cat cov.txt;
exit
```

