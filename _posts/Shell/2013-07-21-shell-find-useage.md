---
layout: post
title: "Linux下find查找文件用法"
category: "Shell"
date: 2013-07-21
---

# Linux下find查找命令用法


find ./ -name hello     #在当前目录下查找文件名为hello的文件

在-name参数中我们可以用`?`和`*`来扩大我们的搜索范围：（`?`代表一个单个字母，而`*`指的是任意的字母数量。）

jzhu@fedora$find . -name ?File
./SecondDir/aFile
$ find . -name *File
./SecondDir/aFile
./SecondDir/AnotherFile
./SecondDir/ThirdFile

csaba@csaba-pc ~/tmp/NetTuts $ find . -name *File 2> /dev/null 1>./SecondDir
错误信息和标准输出被传送到了不同的文件中。
find . -name *File 1>./SecondDir/ThirdFile 2>&1
　　重定向被解释执行是从右到左的。首先开始执行的是 2>&1,这里的意思是重定向标准错误输出到标准输出。然后是1>./SecondDir/ThirdFile，这里的意思是重定向标准输出（此时已经有错误信息在文件里面了）到指定的文件。


## find 简介

find可以递归的在指定目录下查找。

**find命令的一般形式为:**

    find pathname -options [-print -exec -ok ...]

**find命令的参数:**

    pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
    -print： find命令将匹配的文件输出到标准输出。
    -exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格。
    -ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

**find命令选项option**

    -name 
    按照文件名查找文件。

    -perm 
    按照文件权限来查找文件。

    -prune 
    使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用-depth选项，那么-prune将被find命令忽略。

    -user 
    按照文件属主来查找文件。

    -group 
    按照文件所属的组来查找文件。

    -mtime -n +n 
    按照文件的更改时间来查找文件， - n表示文件更改时间距现在n天以内，+ n表示文件更改时间距现在n天以前。find命令还有-atime和-ctime 选项，但它们都和-m time选项。

    -nogroup 
    查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在。

    -nouser 
    查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在。
    
    -newer file1 ! file2 
    查找更改时间比文件file1新但比文件file2旧的文件。
    
    -type 
    查找某一类型的文件，诸如：

        b - 块设备文件。
        d - 目录。
        c - 字符设备文件。
        p - 管道文件。
        l - 符号链接文件。
        f - 普通文件。

    -size n：[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计。
    -depth：在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找。
    -fstype：查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件/etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息。

    -mount：在查找文件时不跨越文件系统mount点。
    -follow：如果find命令遇到符号链接文件，就跟踪至链接所指向的文件。
    -cpio：对匹配的文件使用cpio命令，将这些文件备份到磁带设备中。

    另外,下面三个的区别:

       -amin n
    　　查找系统中最后N分钟访问的文件

    　　-atime n
    　　查找系统中最后n*24小时访问的文件

    　　-cmin n
    　　查找系统中最后N分钟被改变文件状态的文件

    　　-ctime n
    　　查找系统中最后n*24小时被改变文件状态的文件

       -mmin n
    　　查找系统中最后N分钟被改变文件数据的文件

    　　-mtime n
    　　查找系统中最后n*24小时被改变文件数据的文件

## find 命令的实用操作

下面是find一些常用参数的例子，其他的参数用到的时候可以用man再查。


* 

    可以使用某种文件名模式来匹配文件，记住要用引号将文件名模式引起来。

        $ find ~ -name "*.txt"
    
    想要在当前目录及子目录中查找所有的‘ *.txt’文件，可以用：

        $ find . -name "*.txt"
    
    想要的当前目录及子目录中查找文件名以一个大写字母开头的文件，可以用：

        $ find . -name "[A-Z]*"
    
    想要在/etc目录中查找文件名以host开头的文件，可以用：

        $ find /etc -name "host*"

    如果想在当前目录查找文件名以两个小写字母开头，跟着是两个数字，最后是.txt的文件，下面的命令就能够返回名为ax37.txt的文件：

        $find . -name "[a-z][a-z][0-9][0-9].txt"

* -path：




