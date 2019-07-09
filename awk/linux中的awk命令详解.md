## linux中的awk命令详解

### 1、AWK简介

AWK是一种处理文本文件的语言，是一个强大的文本分析工具。

### 2、AWK语法

`awk [选项参数] 'script' var=value file(s)`

或

`awk [选项参数] -f scriptfile var=value file(s)`

选项参数的说明：

```shell
    -F fs or –field-separator fs
    指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:

    -v var=value or –asign var=value
    赋值一个用户定义变量。

    -f scripfile or –file scriptfile
    从脚本文件中读取awk命令。

    -mf nnn and -mr nnn
    对nnn值设置内在限制，-mf选项限制分配给nnn的最大块数目；-mr选项限制记录的最大数目。这两个功能是Bell实验室版awk的扩展功能，在标准awk中不适用。

    -W compact or –compat, -W traditional or –traditional
    在兼容模式下运行awk。所以gawk的行为和标准的awk完全一样，所有的awk扩展都被忽略。

    -W copyleft or –copyleft, -W copyright or –copyright
    打印简短的版权信息。

    -W help or –help, -W usage or –usage
    打印全部awk选项和每个选项的简短说明。

    -W lint or –lint
    打印不能向传统unix平台移植的结构的警告。

    -W lint-old or –lint-old
    打印关于不能向传统unix平台移植的结构的警告。

    -W posix
    打开兼容模式。但有以下限制，不识别：/x、函数关键字、func、换码序列以及当fs是一个空格时，将新行作为一个域分隔符；操作符和=不能代替^和^=；fflush无效。

    -W re-interval or –re-inerval
    允许间隔正则表达式的使用，参考(grep中的Posix字符类)，如括号表达式[[:alpha:]]。

    -W source program-text or –source program-text
    使用program-text作为源代码，可与-f命令混用。

    -W version or –version
    打印bug报告信息的版本。
```
### 3、基本用法

一段文本：log.txt

```
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

* 用法一：

`awk ‘{[pattern] action}’ {filenames} # 行匹配语句 awk ” 只能用单引号`

每行按空格或TAB分割（默认情况），输出文本中的1、4项

`awk '{print $1,$4}' log.txt`

```
2 a
3 like
This's
10 orange,apple,mongo
```

格式化输出

`awk '{printf "%-8s %-10s\n",$1,$4}' log.txt`

```
2        a
3        like
This's
10       orange,apple,mongo
```

* 用法二：

`awk -F #-F相当于内置变量FS, 指定分割字符`

log1.txt的内容如下：
```
2,this,is,a,test
3 Are you like awk
```   

`awk -F, '{print $1,$2}'   log1.txt`

```
2 this
3 Are you like awk
```

使用多个分隔符.先使用空格分割，然后对分割结果再使用","分割

`awk -F '[ ,]'  '{print $1,$2,$5}'   log1.txt`

```
2 this test
3 Are awk
```

* 用法三：

`awk -v # 设置变量`

`awk -v a=1 '{print $1,$1+a}' log.txt`

```
2 3
3 4
This's 1
10 11
```
`awk -v a=1 '{print $1,$(1+a)}' log.txt`

```
2 this
3 Are
This's a
10 There
```

`awk -v a=1 -v b=s '{print $1,$1+a,$1b}' log.txt`

```
2 3 2s
3 4 3s
This's 1 This'ss
10 11 10s
```

* 用法四：

`awk -f {awk脚本} {文件名}`

### 4、运算符

过滤第一列大于2的行:

`awk '$1>2' log.txt`

```
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

过滤第一列等于2的行:

`awk '$1==2 {print $1,$3}' log.txt`

```
2 is
```

过滤第一列大于2并且第二列等于’Are’的行:

`awk '$1>2 && $2=="Are" {print $1,$2,$3}' log.txt`

```
3 Are you
```

### 5、内建变量

输出顺序号 NR, 匹配文本行号

`awk '{print NR,FNR,$1,$2,$3}' log.txt`
```
1 1 2 this is
2 2 3 Are you
3 3 This's a test
4 4 10 There are
```

指定输出分割符

`awk '{print $1,$2,$5}' OFS=" $ "  log.txt`

```
2 $ this $ test
3 $ Are $ awk
This's $ a $
10 $ There $
```

### 6、使用正则表达式

输出第二列包含 "th"，并打印第二列与第四列

`awk '$2 ~ /th/ {print $2,$4}' log.txt`

```
this a
```
   
~ 表示模式开始。// 中是模式。

输出包含"re" 的行

`awk '/re/ ' log.txt`

```
3 Are you like awk
10 There are orange,apple,mongo
```

忽略大小写:

`awk 'BEGIN{IGNORECASE=1} /this/' log.txt`

```
2 this is a test
This's a test
```

模式取反:

`awk '$2 !~ /th/ {print $2,$4}' log.txt`

```
Are like
a
There orange,apple,mongo
```

`awk '!/th/ {print $2,$4}' log.txt`
```
Are like
a
There orange,apple,mongo
```

### 7、awk脚本

关于awk脚本，我们需要注意两个关键词BEGIN和END。
BEGIN{ 这里面放的是执行前的语句 }
END {这里面放的是处理完所有的行后要执行的语句 }
{这里面放的是处理每一行时要执行的语句}
假设有这么一个文件（学生成绩表）：

cat score.txt
```
Marry   2143 78 84 77
Jack    2321 66 78 45
Tom     2122 48 77 71
Mike    2537 87 97 95
Bob     2415 40 57 62
```
awk脚本如下：

cat cal.awk
```
#!/bin/awk -f
#运行前
BEGIN {
math = 0
english = 0
computer = 0

printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
printf "---------------------------------------------\n"
}
#运行中
{
math+=$3
english+=$4
computer+=$5
printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
#运行后
END {
printf "---------------------------------------------\n"
printf "  TOTAL:%10d %8d %8d \n", math, english, computer
printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}

```

运行结果：

`awk -f cal.sh score.txt`

```
NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL
---------------------------------------------
Marry  2143     78       84       77      239
Jack   2321     66       78       45      189
Tom    2122     48       77       71      196
Mike   2537     87       97       95      279
Bob    2415     40       57       62      159
---------------------------------------------
  TOTAL:       319      393      350
AVERAGE:     63.80    78.60    70.00
```

### 8、一些其他实例：

计算文件大小：

`ls -l *.txt | awk '{sum+=$6} END {print sum}'`

666581

从文件中找出长度大于80的行：

`awk 'length>80' log.txt`


打印九九乘法表：

`seq 9 | sed 'H;g' | awk -v RS='' '{for(i=1;i<=NF;i++)printf("%dx%d=%d%s", i, NR, i*NR, i==NR?"\n":"\t")}'`