---
layout: post
title: "Linux下find查找文件用法"
category: "Shell"
date: 2013-07-21
---

# Linux下find查找命令用法


## find 简介

find可以递归的在指定目录下查找满足条件的文件。

**find命令的一般形式为:**

    find pathname -options [-print -exec -ok ...]

**find命令的参数:**

    pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
    -print： find命令将匹配的文件输出到标准输出。
    -exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {} \;，注意{}和\；之间的空格。
    -ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

**find命令选项option**

    -name 
    按照文件名查找文件。 我们可以用`?`和`*`来扩大我们的搜索范围：（`?`代表一个单个字母，而`*`指的是任意的字母数量。）

    -path
    按照文件路径查找文件。

    -perm 
    按照文件权限来查找文件。

    -prune 
    使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用-depth选项，那么-prune将被find命令忽略。

    -user 
    按照文件属主来查找文件。

    -group 
    按照文件所属的组来查找文件。

    -mtime -n +n 
    按照文件的更改时间来查找文件， -n表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前。

    类似的还有以下:

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

    -size n：[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计。（kMGTP）
    -depth：在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找。
    -fstype：查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件/etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息。

    -mount：在查找文件时不跨越文件系统mount点。
    -follow：如果find命令遇到符号链接文件，就跟踪至链接所指向的文件。
    -cpio：对匹配的文件使用cpio命令，将这些文件备份到磁带设备中。


## find 命令的实用操作

下面是find一些常用参数的例子，其他的参数用到的时候可以用man再查。


1. 用文件名查找文件

    可以使用某种文件名模式来匹配文件，记住要用引号将文件名模式引起来。

        $ find ~ -name "*.txt"
    
    想要在当前目录及子目录中查找所有的`*.txt`文件，可以用：

        $ find . -name "*.txt"
    
    想要的当前目录及子目录中查找文件名以一个大写字母开头的文件，可以用：

        $ find . -name "[A-Z]*"
    
    想要在/etc目录中查找文件名以host开头的文件，可以用：

        $ find /etc -name "host*"

    如果想在当前目录查找文件名以两个小写字母开头，跟着是两个数字，最后是.txt的文件，下面的命令就能够返回名为ax37.txt的文件：

        $find . -name "[a-z][a-z][0-9][0-9].txt"

    若要忽略大小写，查找文件，可以用：

        $ find /etc -iname "host*"

2. 用路径名查找

    可以使用某种路径名模式来匹配文件和文件夹。查找文件的路径中带有`*/dist/*`的文件和文件夹。

        $ find . -path "*/dist/*"

3. 指定find搜索时的搜索的深度（若不指定，find默认会遍历所有）

    使用mindepth和maxdepth限定搜索指定目录的深度。

    在root目录及其1层深的子目录下查找passwd文件。

        $ find -maxdepth 2 -name "passwd"

    在第二层子目录和第四层子目录之间查找passwd文件。

        $ find -mindepth 3 -maxdepth 5 -name "passwd"

4. 相反匹配，匹配不满足该条件的文件（使用`-not`或`!`）

    显示所有的名字不是test的文件或者目录。

        $ find . -not -name "test"
        # 或
        $ find . \! -name "test"

5. 查找满足条件1，或条件2的文件

    使用 expression1 -or expression2。

    找到最后访问时间在5分钟以内的，或者大小在5M以内的文件。

        $ find . -amin 5 -or -size 5M

6. 在find命令查找到的文件上执行命令（{}将会被当前文件名取代）

    找到所有`*.pyc`文件，并执行删除操作。

        find . -iname '*.pyc' -exec rm {} \;


7. 查找5个最大的文件

    列出当前目录及子目录下的5个最大的文件。

        $ find . -type f -exec ls -s {} \; | sort -n -r | head -5

8. 给常用find操作取别名

    若你发现有些东西很有用，你可以给他取别名。并且在任何你希望的地方执行。
    
    找到所有`*.pyc`文件，并删除。

        alias rmpyc="find . -iname '*.pyc' -exec rm {} \;"

    找到大于100M的tar文件，并删除。

        alias rm100m="find . -type f -name '*.tar' -size 100M -exec rm -i {} \;"
