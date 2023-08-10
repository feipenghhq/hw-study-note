# Chapter 2 Getting Started: Covergroup and Coverpoint

[TOC]

## Fundamentals

### Coverage

Measure of all the intended values of the signals being exercised onto design during verification.

### Covergroup

User defined datatype covering all the coverage points / transition point into single block.

Capture values of the Coverage point collectively at same event.

Cross between Coverage point within same Covergroup

```verilog
covergroup c @(Event);
	coverpoint a;
	coverpoint b;
	coverpoint c;
	coverpoint state {
        bins trans_0_1 = {0 => 1};
	}
	cross a, b;
endgroup

initial begin
    c1 = new();
	// some code
    c1.sample();
end
```

### Coverpoint

Variable / expression that are used in the RTL implementation

Allows us to measure whether all possible combinations for the signals are achieved during verification

Each possible value lead to creation of the bins (container to hold hits)

Bins could be implicit/explicit

```verilog
covergroup c @(Event);
    coverpoint a;	// automatic bins
    coverpoint b {	// explicit bins
        bins b0 = {0};
        bins b1 = {1};
    }
endgroup
```

### Coverpoint with iff

iff construct disable the coverage on occurrence of specified condition

```verilog
covergroup c @(Event);
    coverpoint a iff (!rst);
endgroup
```



## Understanding Covergroup and Event

Different ways of sampling covergroup

1. Synchronizing signal
2. Manually specify when to sample the value

```verilog
module tb;
    reg [1:0] a;
    reg clk = 0;
    
    always #5 clk = ~clk;
    
    initial begin
       	#200;
        $finish();
    end
 
   	// Method 1: Synchronizing signal
    `ifdef METHOD_1
    covergroup c @(posedge clk);	// Add event on covergroup defination
        coverpoint a;
    endgroup
    
    initial begin
        c ci = new();
      for (int i = 0; i < 20; i++) begin
            @(posedge clk);
            a = $urandom();
          	$info("value of a :%0d", a);
        end
    end
    `endif
    
    // Method 2: Manually specify when to sample the value
    `ifdef METHOD_2
    covergroup c;
        coverpoint a;
    endgroup
    
    initial begin
        c ci = new();
      	for (int i = 0; i < 20; i++) begin
            @(posedge clk);
            a = $urandom();
          	$info("value of a :%0d", a);
            ci.sample();	// Manually sampling
        end
    end
    `endif
    
endmodule
```



## Understanding `option.per_instance`

`option.per_instance = 1` should the coverage information for each instance in the covergroup in the coverage report

```verilog
module tb;
    reg [1:0] a;
    reg       b;
    reg clk = 0;
    
    always #5 clk = ~clk;
    
    initial begin
       	#200;
        $finish();
    end
 
  	covergroup c @(posedge clk);
        option.per_instance = 1;
        coverpoint a;
        coverpoint b;
    endgroup
    
    initial begin
        c ci = new();
      	for (int i = 0; i < 20; i++) begin
            @(posedge clk);
            a = $urandom();
          	b = $urandom();
          	$info("value of a :%0d", a);
        end
    end
endmodule
```



## Turning on/off Coverage with specific conditions

Use `iff` to specify condition on turning on coverage

```verilog
module tb;
    reg [1:0] a;
    reg       b;
    reg rst = 0;
    
    initial begin
       	rst = 1;
       	#30;
        rst = 0;
    end
    
    initial begin
       	#200;
        $finish();
    end
 
  	covergroup c;
        option.per_instance = 1;
        coverpoint a iff (!rst);	// only check the coverpoint when !rst
    endgroup
    
    initial begin
        c ci = new();
      	for (int i = 0; i < 20; i++) begin
            a = $urandom();
            ci.sample();
            #10;
        end
    end
endmodule
```