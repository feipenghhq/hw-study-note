# Chapter 3 Getting Started: Bins

[TOC]

## Fundamentals - bins

Bins keep track/record of number of times user hits specific value of coverpoint expression

## Automatic/implicit bins 

if user do not specify bins, simulator automatically creates bins for user depending on the possible values of coverpoint and auto_bin_max.

### Setting auto_bin_max

Can be applied to individual coverpoint / entire covergroup

```verilog
// default auto_bin_max = 64
reg [7:0] a, b;

covergroup c1;
    option.auto_bin_max = <user_defined_value>;
    coverpoint a;
    coverpoint b;
endgroup

covergroup c2;
    coverpoint a {
        option.auto_bin_max = <user_defined_value>;
    }
    coverpoint b {
        option.auto_bin_max = <user_defined_value>;
    }
endgroup
```

### Size of automatic bins

auto_bin_max define the size of the bin. There are 3 possible cases:

1. possible value < number of bin: 

   - each bin will be a value

2. possible value  ==  number of bin: 

   - each bin will be a value

3. possible value  >  number of bin: 

   - Each bin will represent a group of values. # of Value in each bins = possible / number of bin. The last bin will cover all the remaining values if it can't be divided equally.
   - For example, if data is 4 bit having 16 different value and auto_bin_max = 4. Then 16 / 4 = 4 so there will be 4 bins. bin 0 will cover [0:3], bin 1 will cover [4:7], bin 2 will cover [8:11], bin 3 will cover [8:11]

   - For example, if data is 4 bit having 16 different value and auto_bin_max = 3. Then 16 / 3 = 5.33. Rounding down so each bin will have 5 values. bin 0 will cover [0:4], bin 1 will cover [5:9], bin 2 will cover [10:15]. 

   

## Explicit bins

User defined bins. 

Coverage calculated only for the specified bins

```verilog
reg [1:0] a;

covergroup c;
    coverpoint a {
        bins a0 = {0}; 
        bins a1 = {1}; 
        bins a2 = {2};
        bins a3 = {3};
        // Array can be used to simplify the above statement
        bins aV[] = {[0:3]};
        // multiple value is also possible such as
        bins aV = {0, 1, 2};
        bins aV = [0:2];
    }
    coverpoint b {
        bins bv = {0};
    }
endgroup
```

### Default bins

Create bin which will include the values of the coverpoint not covered by any of the explicit bins

```verilog
reg [1:0] a;

covergroup c;
    coverpoint a {
        bins a0 = {0};
        bins a1 = {1};
        bins unused = default; // 2, 3
    }
endgroup
```

### Different ways of specifying bins

```verilog
a = {0};			// single bin to cover 0
a = {0, 1};			// single bin to cover 0 and 1 hits
a = {[0:3]};		// single bin to cover 0 1 2 3
a[] = {[0:3]};		// array of bins = values
a[] = {0, [2:3]};	// different range
a[] = {[0:$]};		// 0 to last possible value
a[] = {[$:7]};		// possible starting value till 7
a[2] = {[0:3]};		// fixed array - 2 element array - a[0] = [0:1], a[1] = [2:3]
```

### Using Arrays in bins

```verilog
reg [3:0] a;

covergroup c;
    coverpoint a {
        // 1. Array without range:
        bins aV[] = {[0:10]}; // create 11 bins (0 ~ 10)
        // 2. Duplicated element: 
        // When size is not specified, duplicated value will be ignored.
        bins aV1[] = {[0:8], 1, 2, 3}; // create 9 bins (0 ~ 9)
        // 3. Array with range:
        // when the # of value is greater then the bin size, simulator groups the 
        // elements. Duplicated value will be considered
        bins aV2[4] = {[0:8], 1, 2, 3}; // create 4 bins:
        								// bin0 - 0,1,2
        								// bin1 - 3,4,5
        								// bin2 - 6,7,8
        								// bin3 - 1,2,3
    }
    coverpoint b {
        bins bv = {0};
    }
endgroup
```

### Bins with expression

```verilog
covergroup c;
    // Returns the number of bits which is 1
    coverpoint $countbits(a, 1'b1) {
        bins aValue[] = {[0:4]};
    }
endgroup
```







