# TCL Language Introduction

[TOC]

## 概述

### 置换

TCL 解释器运用规则把命令分成一个个独立的单词，同时进行必要的置换。TCL置换分为以下三类。

- 变量置换 $ 
- 命令置换 [] 
- 反斜杠置换\

#### 变量置换

用$表示变量置换。TCL解释器会将认为$后面为变量名，将变量置换成它的值。

```tcl
% set a "snow"
snow
% puts $a
snow
% puts a
a
```

#### 命令置换

用[]表示命令置换。[]内是一个独立的TCL语句

```
% set a [expr 3 + 4]
7
% puts $a
7
```

#### 反斜杠置换

用\表示反斜杠置换。换行符、空格、[、$等被TCL解释器当作特殊符号对待的字符，加上反斜杠后变成普通字符。

```tcl
% puts "[expr $X + $Y]"
2.5
% puts "\[expr $X + $Y\]"
[expr 1.2 + 1.3]
% puts "\[expr \$X + \$Y\]"
[expr $X + $Y]
```

用\t表示TAB。

用\n表示换行符。

```tcl
% puts "a\tb"
a	b
% puts "a\nb"
a
b
```

#### 其他符号

- ”“： TCL解释器对双引号中＄和[]符号会进行变量置换和命令置换。

```tcl
% puts "\t[expr $X + $Y]"
	2.5
```

- {}： 而在花括号中，所有特殊字符都将成为普通字符，TCL解释器不会对 其作特殊处理。

```tcl
% puts {\t[expr $X + $Y]}
\t[expr $X + $Y]
```

- \#： 表示注释



## 变量、数组、列表

###  变量

变量就是某个容器的名称，可以存储一个值。变量的名称在程序运 行期间保持不变，但是变量的值通常会不断改变。

- 定义：`set 变量名 变量值`

- 取值：`$变量名`

```tcl
% set cell "bufx2"
bufx2
% set cell "ivtx2"
ivtx2
% puts $cell
ivtx2
```

例题：假设我们想打印变量varible，后面跟一个"_1"，会发生什么呢？

```tcl
% set a 2
2
% puts $a_1
can't read "a_1": no such variable
```

```tcl
% set a 2
2
% puts ${a}_1		# 用{}来置换出a
2_1
```

### 数组

TCL中数组可以存储很多值，通过元素名来进行检索。类似于某件事物（数组名）几种不同属性（元素名），每一种属性 有其独立的值。

- 定义：`set 数组名（元素名） 值`

```tcl
% set cell_1(ref_name) "bufx2"
bufx2
% set cell_1(full_name) "top/cell_1"
top/cell_1
% set cell_1(pins) "A B C"
A B C
```

- 取值：`$数组名（元素名）`

```tcl
% puts $cell_1(ref_name)
bufx2
```

使用array指令获取数组信息

```tcl
% array size cell_1
3
% array names cell_1
ref_name pins full_name
```

### 列表

列表是标量的有序集合

- 定义：` set 列表名 {元素1 元素2 元素3……}`
- 取值：`$列表名`

```tcl
% set ivt_list {ivtx2 ivtx3 ivtx8}
ivtx2 ivtx3 ivtx8
% puts $ivt_list
ivtx2 ivtx3 ivtx8
```

TCL中有一系列十分方便的列表操作命令

| 命令    | 功能                 |
| ------- | -------------------- |
| concat  | 合并两个列表         |
| lindex  | 选取列表中的某个元素 |
| llength | 列表长度             |
| lappend | 在列表末端追加元素   |
| lsort   | 列表排序             |

#### 列表指令 - concat

- 语法格式 ： `concat 列表1 列表2`

- 功能： 将列表1和列表2合并

```tcl
% set list1 {bufx1 bufx2 bufx4}
bufx1 bufx2 bufx4
% set list2 {ivtx1 ivtx2 ivtx4}
ivtx1 ivtx2 ivtx4
% concat $list1, $list2
bufx1 bufx2 bufx4, ivtx1 ivtx2 ivtx4
```

#### 列表指令 - llength

- 语法格式 ： `llength 列表`

- 功能： 返回列表中的元素个数

```tcl
% set list1 {bufx1 bufx2 bufx4}
bufx1 bufx2 bufx4
% llength $list1
3
```

例题：list1为{bufx1 bufx2 bufx4}，那么 llength [concat $list1 $list1] 会得到多少呢？

```
% llength [concat $list1 $list1]
6
```

#### 列表指令 - lindex

- 语法格式 ： `lindex 列表 n`

- 功能： 返回列表中第n个元素（从0开始计数）

```tcl
% set list1 {bufx1 bufx2 bufx4}
bufx1 bufx2 bufx4
% lindex $list1 0
bufx1
% lindex $list1 1
bufx2
```

例题：如何得到列表list1 {a b c d e f} 的最后一个元素？

```tcl
% set list1 {a b c d e f}
a b c d e f
% llength $list1
6
% expr [llength $list1] - 1
5
% lindex $list1 [expr [llength $list1] - 1]
f
```

#### 列表指令 - lappend

- 语法格式 ：`lappend 列表 新元素`

- 功能： 列表末尾加入新元素

```tcl
% set a {1 2 3}
1 2 3
% lappend a 4
1 2 3 4
% puts $a
1 2 3 4
```

例题：如果我们lappend一个列表会怎么样？

```tcl
% set a {1 2 3}
1 2 3
% set b {4 5}
4 5
% lappend a $b
1 2 3 {4 5}
```

#### 列表指令 - lsort

- 语法格式 ： `lsort 开关 列表`

- 功能： 将列表按照一定规则排序 

- 开关： 缺省时默认按照ASCII码进行排序。 

​			–real 按照浮点数值大小排序 

​			-unique 唯一化，删除重复元素

按照ASCII码排序

```tcl
% set list1 {c d a f b}
c d a f b
% lsort $list1
a b c d f
```

按照数字大小排序

```tcl
% set list2 {-2 3.1 5 0}
-2 3.1 5 0
% lsort -real $list2
-2 0 3.1 5
```

唯一化

```tcl
% set list3 {a c c b a d}
a c c b a d
% lsort -unique $list3
a b c d
```



### 运算

#### 数学运算

a + b

a - b

a * b

a / b

#### 逻辑运算

a <= b 

a >= b

a == b

a != b

#### 数学运算指令 - expr

- 语法格式 ： `expr 运算表达式`

- 功能： 将运算表达式求值

```tcl
% expr 6 + 4
10
% expr 6 - 4
2
```

例题：我们在TCL中经常会遇到下面的现象

```tcl
% expr 5 / 2
2
```

其原因是表达式5/2中5和2都是整数型参数，默认运算结果也是整数型。 如果想要进行浮点运算，只要将其中任意一个数值，写成浮点形式（有小数点）即可

```tcl
% expr 5 / 2.0
2.5
% expr 5.0 / 2
2.5
```



## 控制流

### 控制流-if

- 语法格式：

```
if {判断条件} {
脚本语句
} elseif {判断条件} {
脚本语句
} else {
脚本语句
}
```

注意，上例中脚本语句的'{'一定要写在上一行，因为如果不这样，TCL 解释 器会认为if命令在换行符处已结束，下一行会被当成新的命令，从而导致错误

```tcl
% if {$a > $b} {
puts $a
} else {
puts $b
}
3
```

### 循环指令-foreach

- 语法格式 ：`foreach 变量 列表 循环主体`

- 功能：从第0个元素开始，每次按顺序取得列表的一个元素，将其赋值给变量，然后执行循环主体一次，直到列表最后一个元素

```tcl
% set list1 {3 2 1}
3 2 1
% foreach i $list1 {
puts $i
}
3
2
1
```

### 循环控制指令-break

- 语法格式 ：`break`

- 功能: 结束整个循环过程，并从循环中跳出

```tcl
% set list1 {3 2 1}
3 2 1
% foreach i $list1 {
if {$i == 2} {
break
}
puts $i
}
3
```

### 循环控制指令-continue

- 语法格式 ：`continue`

- 功能: 仅结束本次循环

```tcl
% set list1 {3 2 1}
3 2 1
% foreach i $list1 {
if {$i == 2} {
continue
}
puts $i
}
3
1
```

### 循环控制指令-while

- 语法格式 ： `while 判断语句 循环主体`

- 功能: 如果判断语句成立（返回值非0），就运行脚本，直到不满足判断条件停止循环，此时while命令中断并返回一个空字符串。

```tcl
% set i 3
% while {$i > 0} {
puts $i
incr i -1	# set i [expr $i - 1]
}
3
2
1
```



### 循环控制指令-for

- 语法格式：`for 参数初始化 判断语句 重新初始化参数 循环主体`

- 功能: 如果判断语句返回值非0就进入循环，执行循环主体后，再重新初始化参数。然后再次进行判断，直到判 断语句返回值为0，循环结束。

```tcl
% for {set i 3} {$i > 0} {incr i -1} {
puts $i
}
3
2
1
```

注意{}与{}之间需要有空格



## 过程函数

- 语法格式： `proc 函数名 参数列表 函数主体`
- 功能：类似于C语言中的函数。即用户自定义的功能，方便多次调用

```tcl
% proc add {a b} {
set sum [expr $a + $b]
return $sum
}
% add 3 4
7
```

### 全局变量与局部变量

- 全局变量：在所有过程之外定义的变量。
- 局部变量：对于在过程中定义的变量，因为它们只能在过程中被访问，并且当过程退出时会被自动删除
- 指令`global`，可以在过程内部引用全部变量

```tcl
% set a 1
1
% proc sample {x} {
global a
set a [expr $a+1]
return [expr $a + $x]
}
% sample 3
5
```



## 正则匹配

- 定义：正则表达式是一种特殊的字符串模式，用来去匹配符合规则的字符串
- 正则表达式的`\w`，用来匹配一个数字，字母，下划线
- 正则表达式的`\d`，用来匹配一个数字

### 量词

在TCL中常用一下三种量词

| 符号 | 功能           |
| ---- | -------------- |
| *    | 零次或多次匹配 |
| +    | 一次或多次匹配 |
| ?    | 零次或一次匹配 |

`abc123` -> `\w+\d+` 或者 `\w*\d*`

### 锚位

锚位，用来指示字符串当中的开头和结尾的位置，使得我们能够匹配到正确的字符

| 符号 | 功能       |
| ---- | ---------- |
| ^    | 字符串开头 |
| $    | 字符串结尾 |

### 其他字符

常用的其他字符还有`\s`和`.`

`\s`表示空格

`.`表示任意一个字符

我们不确定具体是什么字符时就可以用`.`表示

### 正则匹配指令 - regexp

- 语法格式：`regexp? switch? exp string? matchVar? ?subMatchVar? subMatchVar ...?`
- 功能：在字符串中使用正则表达式匹配

- switches:
  - -nocase 将字符串中断大小写都当成小写看待
- exp：正则表达式
- string：用来进行匹配的字符串
- matchingstring：表示用正则表达式匹配的所有字符串
- sub1：表示正则表达式中的第一个子表达式匹配的字符串
- sub2：表示正则表达式中的第二个子表达式匹配的字符串

例题：如何匹配字符串"abc456"

```tcl
% regexp {\w+\d+} "abc456"
1
```

例题：如何匹配一个以数字开头并且以数字结尾的字符串？

```tcl
% regexp {^\d.*\d$} "1 dfsal 1"
1
```

### 捕获变量

通过()可以捕获字符串

例如如何将字符串“Snow is 30 years old”中30捕获出来？

```tcl
% regexp {\s(\d\d)\s} "Snow is 30 years old" total age
1
% puts $total
 30
% puts $age
30
```

一次捕获多个字符串

例如如何将字符串“Snow is 30 years old”中Snow 和30一次捕获？

```tcl
% regexp {^(\w+)\s\w+\s(\d+).*} "Snow is 30 years old" total name age
1
% puts $total
Snow is 30 years old
% puts $name
Snow
% puts $age
30
```



## 文本处理

- open
  - 语法格式：`open 文件 打开方式` （打开方式r表示读模式，w表示写模式）
  - 功能：打开文件
- gets
  - 语法格式：`gets field 变量名`
  - 功能：gets读field标识的文件的下一行，并把该行赋给变量，并返回垓行的字符数（文件结尾返回-1）
- close
  - 语法格式：`close field`
  - 功能：关闭文件

例题：整个读入文件过程

```tcl
% set INPUTFILE [open file.txt r]
file17059c73640
% while {[get $INPUTFILE line] >= 0} {
puts "$line"
}
a
b
c
close $INPUTFILE
```

例题：一个完整写入文件过程

```tcl
% set OUTPUTFILE [open file.txt w]
file1705b54a140
% puts $OUTPUTFILE "hello world"
% close $OUTPUTFILE
```

