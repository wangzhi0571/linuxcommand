## linux中的awk命令详解

### 1、AWK简介

AWK是一种处理文本文件的语言，是一个强大的文本分析工具。

### 2、AWK语法

```
awk -F|-f|-v 'BEGIN{ } / / {comand1;comand2} END{ }' file
-F 定义列分隔符
-f 指定调用脚本
-v 定义变量
' '引用代码块，awk执行语句必须包含在内
BEGIN{ } 初始化代码块，在对每一行进行处理之前，初始化代码，主要是引用全局变量，设置FS分隔符
{ } 命令代码块，包含一条或多条命令
// 用来定义需要匹配的模式（字符串或者正则表达式），对满足匹配模式的行进行上条代码块的操作
END{ }  结尾代码块，在对每一行进行处理之后再执行的代码块，主要是进行最终计算或输出结尾摘要信息
```

### 特殊要点

```
$0          表示整个当前行
$1          每行第一个字段
NF          字段数量变量
NR          每行的记录号，多文件记录递增
FNR         与NR类似，不过多文件记录不递增，每个文件都从1开始
\t          制表符
\n          换行符
FS          BEGIN时定义分隔符
RS          输入的记录分隔符， 默认为换行符(即文本是按一行一行输入)
~           匹配，与==相比不是精确比较
!~          不匹配，不精确比较
==          等于，必须全部相等，精确比较
!=          不等于，精确比较
&&          逻辑与
||          逻辑或
+           匹配时表示1个或1个以上
/[0-9][0-9]+/    两个或两个以上数字
/[0-9][0-9]*/    一个或一个以上数字
FILENAME    文件名
OFS         输出字段分隔符， 默认也是空格，可以改为制表符等
ORS         输出的记录分隔符，默认为换行符,即处理结果也是一行一行输出到屏幕
-F'[:#/]'   定义三个分隔符
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

cat cal.sh
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
printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3, $4, $5, $3 + $4 + $5
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