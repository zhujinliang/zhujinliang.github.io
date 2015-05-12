---
layout: post
title: "Linux下sed用法"
category: "Shell"
date: 2014-01-20
---


## 1. Sed简介

`sed`全名叫stream editor，流编辑器，用程序的方式来编辑文本，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件，简化对文件的反复操作，编写转换程序等。

本文不会说sed的全部东西，你可以参看[sed的手册][sed_mannual]。

sed的一般有两种形式：

	sed [options] 'sed-command' file(s)

	sed [options] -f scriptfile file(s)

<!-- more -->

## 2. 选项(option)

-e command, --expression=command
一次替换多个模式。

	sed -e '1,3s/my/your/g' -e '3,$s/This/That/g' my.txt
	等价于：
	sed '1,3s/my/your/g; 3,$s/This/That/g' my.txt

-i
修改源文件。

	#-i会修改源文件,但是可以同时使用bak备份
	sed -ibak 's/Ian/IAN/' source.txt
	# or
	sed --in-place=bak 's/Ian/IAN/' source.txt
	# 这样会存在一个文件source.txtbak

-n, --quiet, --silent
取消默认输出。sed处理的时候，一般是会把处理信息输出的，使用`-n`可以取消默认输出。

	sed -n '/fish/p' my.txt


-f, --filer=script-file
使用sed脚本文件名。

-h, --help
打印帮助，并显示bug列表的地址。

-V, --version
打印版本和版权信息。


## 3. Sed命令(sed-command)

sed '[address] [command]' input-file

### 定址(address)

可以通过定址来定位你所希望编辑的行，该地址用数字构成，用逗号分隔的两个行数表示以这两行为起止的行的范围（包括行数表示的那两行）。如1，3表示1，2，3行，美元符号($)表示最后一行。范围可以通过数据，正则表达式或者二者结合的方式确定 。

### sed命令(command)

a\
即append，在当前行后面加入一行文本。

**注意：** `a\` a 后面有\

	sed '[address] a\the-line-to-append' input-file
	# 在最后一行插入字符串"The End."，$ 是定位。
	sed "$ a\ The End." source.txt

c\
替换匹配的行。

	# 替换第2行为字符串 hello world。
	sed "2 c\hello world" source.txt
	# 替换匹配hi的行为字符串 hello world。
	sed "/hi/ c\hello world" source.txt


d
删除匹配的行。

	# 删除第2行。
	sed "2 d" source.txt
	# 删除第3行开始到末尾的行。
	sed "3,$ d" source.txt
	# 删除匹配hi的行。
	sed "/hi/ d" source.txt

i\
即insert，在**当前行上面插入**文本。

	sed '[address] i\the-line-to-insert' input-file
	# 在第1行插入字符串"The Start."，1 是定位。
	sed "1 i\The End." source.txt

p
打印行。配合选项`-n`一起使用，只打印匹配的行。

	# 打印匹配到hello的行
	sed -n '/hello/ p' source.txt
	# 打印第1行到匹配到hello的行
	sed -n '1,/hello/ p' source.txt
	# 打印从匹配hello到world的行
	sed -n '/hello/,/world/ p' source.txt

n
读取下一个输入行，用下一个命令处理新的行而不是用第一个命令。


q
退出Sed。

w file
把行写入到一个文件。

	# 第1行到第4行，写入到out.txt文件
	sed '1,4 w out.txt' source.txt

!
表示后面的命令对所有没有被选定的行发生作用。

s/re/string
用string替换正则表达式re。
g表示行内全面替换。

	# 替换所有的day成night
	sed "s/day/night/g" source.txt


y/ab/cd/
y表示把一个字符翻译为另外的字符（但是不用于正则表达式）。`y/ab/cd/`表示把c替换a，d替换b。

=
打印当前行号码。


## 4. 正则元字符集

双引号中可以通过\来转义。

^
锚定行的开始 如：/^sed/匹配所有以sed开头的行。

$
锚定行的结束 如：/sed$/匹配所有以sed结尾的行。

.
匹配一个非换行符的字符 如：/s.d/匹配s后接一个任意字符，然后是d。

*
匹配零或多个字符 如：/*sed/匹配所有模板是一个或多个空格后紧跟sed的行。

[]
匹配一个指定范围内的字符，如/[Ss]ed/匹配sed和Sed。

[^]
匹配一个不在指定范围内的字符，如：/[^A-RT-Z]ed/匹配不包含A-R和T-Z的一个字母开头，紧跟ed的行。

`\(..\)`
保存匹配的字符，如`s/\(love\)able/\1rs`，loveable被替换成lovers。

&
保存搜索字符用来替换其他字符，如s/love/**&**/，love这成**love**。

`\<`
锚定单词的开始，如:`/\<love/`匹配包含以love开头的单词的行。

`\>`
锚定单词的结束，如`/love\>/`匹配包含以love结尾的单词的行。

`x\{m\}`
重复字符x，m次，如：`/0\{5\}/`匹配包含5个o的行。

`x\{m,\}`
重复字符x,至少m次，如：`/o\{5,\}/`匹配至少有5个o的行。

`x\{m,n\}`
重复字符x，至少m次，不多于n次，如：`/o\{5,10\}/`匹配5--10个o的行。

## 5. sed命令实例

#### 替换：s命令
	sed 's/test/mytest/g' example
	# 在整行范围内把test替换为mytest。如果没有g标记，则只有每行第一个匹配的test被替换成mytest。

	sed -n 's/^test/mytest/p' example
	#(-n)选项和p标志一起使用表示只打印那些发生替换的行。也就是说，如果某一行开头的test被替换成mytest，就打印它。

	sed 's/^192.168.0.1/&localhost/' example
	# &符号表示替换换字符串中被找到的部份。所有以192.168.0.1开头的行都会被替换成它自已加localhost，变成192.168.0.1localhost。

	sed -n 's/\(love\)able/\1rs/p' example
	# love被标记为1，所有loveable会被替换成lovers，而且替换的行会被打印出来。

	sed 's#10#100#g' example
	# 不论什么字符，紧跟着s命令的都被认为是新的分隔符，所以，“#”在这里是分隔符，代替了默认的“/”分隔符。表示把所有10替换成100。

#### 选定行的范围：逗号
	sed -n '/test/,/check/p' example
	# 所有在模板test和check所确定的范围内的行都被打印。

	sed -n '5,/^test/p' example
	# 打印从第五行开始到第一个包含以test开始的行之间的所有行。

	sed '/test/,/check/ s/$/sed test/' example
	# 对于模板test和west之间的行，每行的末尾用字符串sed test替换。

#### 多点编辑：e命令

	sed -e '1,5d' -e 's/test/check/' example
	# (-e)选项允许在同一行里执行多条命令。如例子所示，第一条命令删除1至5行，第二条命令用check替换test。
	# 命令的执行顺序对结果有影响。如果两个命令都是替换命令，那么第一个替换命令将影响第二个替换命令的结果。

	sed --expression='s/test/check/' --expression='/love/d' example
	# 一个比-e更好的命令是--expression。它能给sed表达式赋值。


#### 下一个：n命令

	sed '/test/ { n; s/aa/bb/; }' example
	# 如果test被匹配，则移动到匹配行的下一行，替换这一行的aa，变为bb，并打印该行，然后继续。

#### 翻译：y命令

	sed '1,10y/acd/ACE/' example
	# 把1--10行内所有a转变为A，c转变为C，e转变为E，注意，正则表达式元字符不能使用这个命令。

#### 退出：q命令
	sed '10q' example
	# 打印完第10行后，退出sed。




[sed_mannual]:http://www.gnu.org/software/sed/manual/sed.html
