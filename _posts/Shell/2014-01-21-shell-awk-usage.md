---
layout: post
title: "Linux下awk用法"
category: "Shell"
date: 2014-01-21
---

# Linux下awk用法

## awk 简介
awk是一种用于处理文本的编程语言工具。它不仅是 Linux 中也是任何环境中现有的功能最强大的数据处理引擎之一。其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。awk提供了极其强大的功能：可以进行样式装入、流控制、数学运算符、进程控制语句甚至于内置的变量和函数。

awk 用法：awk 'pattern {action}'

执行顺序：awk 一行行读入输入文件，顺序执行‘’内内容，按模式匹配来采取动作。

其他调用：awk 可用内部变量和函数，条件与循环语句，也可执行数学运算和字符串操作。
此外，可以使用 `BEGIN`和 `END`来执行处理前预操作和处理后后继操作。

### awk中常用的变量及其含义
见下表：

|变量名|含义|
|-----|----|
|$0|当前整行|
|$1|第一个字段，$2,$3以此类推|
|ARGC|命令行变元个数 argument count|
|ARGV|命令行变元数组|
|FILENAME|当前输入文件名|
|FNR|当前文件中的记录号|
|FS|输入字段分隔符，默认为一个空格 field seperator|
|RS|输入记录分隔符 record seperator|
|NF|当前记录里字段总数 ordinal number of field|
|NR|到目前为止记录数，即行数 ordinal number of record|
|OFS|输出域分隔符 output field seperator|
|ORS|输出记录分隔符 output record seperator|

## awk常用例子

### 匹配行
    awk '/101/'               file # 显示文件file中包含101的匹配行。
    awk '/101/,/105/'         file # 显示文件file中从包含101的匹配行到包含105的匹配行。
    awk '$1 == 5'             file # 显示文件file中第一个域等于5的匹配行。
    awk '$1 == "CT"'          file # 显示文件file中分隔后第一个域的值等于字符串CT的，注意必须带双引号
    awk '$1 * $2 >100 '       file # 显示文件file中分隔后第一个域的值与第二个域的值乘积大于100的行
    awk '$2 >5 && $2<=15'     file
    awk '$1 ~ /101/ {print $1}' file # 显示文件中第一个域匹配101的行（记录）
    awk '$1 * $2 >100 {print $1}' file # 显示文件中第一个域和第二个域乘积大于100的第一个域的值
    awk '$1 == 'Chi' {$3 = 'China'; print}' file # 找到匹配行后先将第3个域替换后再显示该行（记录）
    awk '{$7 %= 3; print $7}' file # 将第7域被3除，并将余数赋给第7域再打印。

### awk常用变量使用
    awk '{print NR,NF,$1,$NF,}' file # 显示文件file的当前记录号、域数量和每一行的第一个和最后一个域。
    awk '/101/ {print $1,$2 + 10}' file # 显示文件file的匹配行的第一、二个域加10。
    awk '/101/ {print $1 $2}' file 显示文件file的匹配行的第一、二个域，但显示时域中间没有分隔符，即拼接字符串。
    awk '{wage=$2+$3; print wage}' file # 为变量wage赋值并打印该变量。

### 设定新的分隔符

    df | awk '$4>1000000 '         # 通过管道符获得输入，如：显示第4个域满足条件的行。
    awk -F "|" '{print $1}'   file # 按照新的分隔符“|”进行操作。
    awk  'BEGIN { FS="[: \t|]" }
    {print $1,$2,$3}'         file # 通过设置输入分隔符（FS="[: \t|]"）正则表达式，修改输入分隔符。

    Sep="|"
    awk -F $Sep '{print $1}'  file # 按照环境变量Sep的值做为分隔符。
    awk -F '[ :\t|]' '{print $1}' file # 按照正则表达式的值做为分隔符，这里代表空格、:、TAB、|同时做为分隔符。
    awk -F '[][]'    '{print $1}' file # 按照正则表达式的值做为分隔符，这里代表[或]

### 从文件读取执行操作

    awk -f awkfile            file # 通过文件awkfile的内容依次进行控制。
    cat awkfile
    /101/{print "\047 Hello! \047"} # 遇到匹配行以后打印 ' Hello! '.\047代表单引号。
    {print $1,$2}                   # 因为没有模式控制，打印每一行的前两个域。

### 通过BEGIN进行一些预处理或设置

    awk   'BEGIN { OFS="%"}
    {print $1,$2}'           file # 通过设置输出分隔符（OFS="%"）修改输出格式。

    awk   'BEGIN { max=100 ;print "max=" max}   # BEGIN 表示在处理任意行之前进行的操作。
    {max=($1 >max ?$1:max); print $1,"Now max is "max}' file # 取得文件第一个域的最大值。

    awk '{print ($1>4 ? "high "$1: "low "$1)}' file # 当文件file，第一个域的值大于4时显示high

    表达式1?表达式2:表达式3 相当于：
    if (表达式1)
        表达式2
    else
        表达式3

### 通过END进行处理完后的操作

    awk '/tom/ {count++;}
        END {print "tom was found "count" times"}' file # END表示在所有输入行处理完后进行处理。

    awk '{cost+=$4}
         END {print "The total is $" cost > "filename"}'    file # 统计第四域的总和，再将结果输出到filename中。

    awk '{if ($4>1000&&$4<2000) c1+=$4;
    else if ($4>2000&&$4<3000) c2+=$4;
    else if ($4>3000&&$4<4000) c3+=$4;
    else c4+=$4; }
    END {printf  "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4}"' file  # 通过if和else if完成条件语句

    awk '{if ($4>3000&&$4<4000) exit;
    else c4+=$4; }
    END {printf  "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4}"' file # 通过exit在某条件时退出，但是仍执行END操作。

    awk '{if ($4>3000) next;
    else c4+=$4; }
    END {printf  "c4=[%d]\n",c4}"' file # 通过next在某条件时跳过该行，对下一行执行操作。


### awk中使用循环

    awk '{ i=1;while(i<NF) {print NF,$i;i++}}' file # 通过while语句实现循环。

    awk '{ for(i=1;i<NF;i++) {print NF,$i}}'   file # 通过for语句实现循环。

    type file|awk -F "/" '
    { for(i=1;i<NF;i++)
    { if(i==NF-1) { printf "%s",$i }
    else { printf "%s/",$i } }}'               # 显示一个文件的全路径。


### awk中调用系统变量

在awk中调用系统变量必须用单引号，如果是双引号，则表示字符串

    Flag=abcd
    awk '{print '$Flag'}'   结果为abcd
    awk '{print  "$Flag"}'   结果为$Flag
