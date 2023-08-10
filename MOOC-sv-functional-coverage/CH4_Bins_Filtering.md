# Chapter 4 Bins Filtering

[TOC]

## `with` clause

Allows creation of the bins if condition specified with clause matches

### Filtering values from the Variable

```verilog
coverpoint a {
    // 0 2 4 6 8 10 12 14 ...
    bins used_a[] = a with (item %2 == 0);
}

```

### Filtering values from the range of Values

```verilog
coverpoint a {
    // 2 4 6 8 10
    bins = used_a[] = {[1:10]} with (item %2 == 0);
}
```



## `illegal_bins`

Set of values of variable excluded from coverage if marked illegal

Throw error if illegal values are applied

```verilog
coverpoint a {
    illegal_bins unused_a = {2, 3};
}
```

### Creation of empty bins

If we put a value in both the regular bin and illegal bin, then it will be removed from the regular bin hence creating an empty bin

```verilog
coverpoint a {
    // 2 and 3 will be removed from used_a
    bins used_a = {0, 1, 2, 3};
    illegal_bins unused_a = {2, 3};
}
```



## `ignore_bins`

Specified values with ignore bins are excluded from the code coverage

Do not throw error if send ignore bins to the variable

```verilog
coverpoint a {
    ignore_bins unused_a = {2, 3};
}
```

### Advantage of ignore_bins over illegal_bins

Assume a is 8 bit ranging from 0 ~ 255. We want to cover a from [1-100] while ignore 23, 45, 67, 89, 93

If we use illegal_bins, we need to remove them from bin definition

```verilog
coverpoint a {
    bins aValue[] = {[1:22], [24:44],[46:66],[68:88],[90:92],[94:100]};
}
```

If we use ignore bins, we can add these ignore value to ignore_bins

```verilog
coverpoint a {
    bins aValue[] = {[1:100]};
    ignore_bins unused_a[] = {23, 45, 67, 89, 93};
}
```

### Ignore range of values from Coverage

```verilog
coverpoint a {
	ignore_bins unused_range1[] = {[3:7], [32:36], [47:50],[61:64]};
}
```



## `wildcard bins`

Wildcard bins treat 'x', 'z', and '?' as 0 or 1 during coverage calculation. Values of few bits from the vector do not affect functionality.

```verilog
coverpoint a {
    wilecard bins low = {2'b0?}; 
}
```



