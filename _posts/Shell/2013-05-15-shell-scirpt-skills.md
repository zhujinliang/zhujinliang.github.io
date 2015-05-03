---
layout: post
title: "Shell小技巧"
category: "Shell"
tags: [Shell, Linux]
---



整理了一些shell小技巧，在写shell脚本的时候，会用到。

### 1. 让你的echo丰富多彩
很多时候，你会想让echo能以多种颜色区分不同输出。比如，绿色表示成功，红色告知失败，黄色提示警告。

    NORMAL=$(tput sgr0)
    GREEN=$(tput setaf 2; tput bold)
    YELLOW=$(tput setaf 3)
    RED=$(tput setaf 1)
    function red() {
        echo -e "$RED$*$NORMAL"
    }
    function green() {
        echo -e "$GREEN$*$NORMAL"
    }
    function yellow() {
        echo -e "$YELLOW$*$NORMAL"
    }
    # To print success
    green "Task has been completed"
    # To print error
    red "The configuration file does not exist"
    # To print warning
    yellow "You have to use higher version."

这里使用 `tput` 来配置输出颜色，输出文本，最后再恢复默认输出颜色。如果想对 `tput` 了解更多，参看 prompt-color-using-tput 。

### 2. 输出debug信息
仅当设置DEBUG标志时才打印调试信息。

    function debug() {
    if [[ $DEBUG ]]
    then
        echo ">>> $*"
    fi
    }
    # For any debug message
    debug "Trying to find config file"

还有来自于一些很酷的Geeks的单行debug函数：

    function debug() { ((DEBUG)) && echo ">>> $*"; }
    function debug() { [ "$DEBUG" ] && echo ">>> $*"; }

<!-- more -->

### 3. 检查特定的可执行文件是否存在

    OK=0
    FAIL=1
    function require_curl()
    {
        which curl &>/dev/null
        if [ $? -eq 0 ]
        then
            return $OK
        fi
        return $FAIL
    }

这里使用 which 命令来查找可执行文件 curl 的路径。如果成功找到，则可执行文件文件是存在的，否则就不存在。 &>/dev/null 将标准输出和标准错误重定向到 /dv/null （也就是不显示在终端上了）。
一些朋友建议可以直接使用 which 返回的状态码。

    # From cool geeks at hacker news
    function require_curl() { which "curl" &>/dev/null; }
    function require_curl() { which -s "curl"; }

### 4. 显示脚本的使用说明
在我开始写Shell脚本的初期，常会使用 echo 命令显示脚本的使用说明。 但当说明的文字较多时，echo 语句就会变得一团糟。随后我发现，可以使用 cat命令来显示使用说明。

    cat << EOF
    Usage: myscript <command> <arguments>
    VERSION: 1.0
    Available Commands
    install - Install package
    uninstall - Uninstall package
    update - Update package
    list - List packages
    EOF

这里的 << 称为 here document，它可以将字符串放置在两个 EOF 之间。

### 5. 用户设置 vs. 默认配置
我们有时会希望在用户没有提供设置参数时能够使用默认值。

    URL=${URL:-http://localhost:8080}

这一语句检查环境变量 URL ，如果不存在，就将其设置为 localhost。
### 检查字符串的长度

    if [ ${#authy_api_key} != 32 ]
    then
        red "you have entered a wrong API key"
        return $FAIL
    fi

`${#VARIABLE_NAME}` 可以给出字符串的长度。

### 6. 为读取输入设置时限

    READ_TIMEOUT=60
    read -t "$READ_TIMEOUT" input
    # if you do not want quotes, then escape it
    input=$(sed "s/[;\`\"\$\' ]//g" <<< $input)
    # For reading number, then you can escape other characters
    input=$(sed 's/[^0-9]*//g' <<< $input)

### 7. 获取目录名和文件名

这个写shell脚本中用的很多，一般都要了解当前脚本文件的目录，以备后用。

    # To find base directory of current file.
    APP_ROOT=`dirname "$0"`
    # To find the file name
    filename=`basename "$filepath"`
    # To find the file name without extension
    filename=`basename "$filepath" .html`


