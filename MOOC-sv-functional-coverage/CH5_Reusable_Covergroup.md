# Chapter 5 Reusable Covergroup

[TOC]

## Pass by Value

```verilog
covergroup c (logic [1:0] value);
    option.per_instance = 1;
    coverpoint a {
        bins f[] = {[0:value]};
    }
endgroup
```



## Pass by Reference

```verilog
logic [1:0] a, b;

covergroup c (ref logic [1:0] var1, input string name);
    option.name = name;
    option.per_instance = 1;
    coverpoint var1;
endgroup

initial begin
    c ca = new(a, "Coverage of a");
    c cb = new(b, "Coverage of b");
end
```



## Combination of pass by value and pass by reference

```verilog
covergroup check_var (ref logic [3:0] var1, input int value);
    option.per_instance = 1;
    coverpoint var1 {
        bins f[] = {[0:value]};
    }
endgroup
```

