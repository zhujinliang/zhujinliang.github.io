---
layout: post
title: "Django项目中优化前端开发"
category: "Front-End"
date: 2014-09-11
---

## 1 前言

随着互联网的飞速发展，越来越重视页面交互和体验，随之而来前端开发也越来越复杂。随着我所在公司业务的不断扩展，Pingjia项目中页面也越来越多，
而每个页面面临着需要加载的CSS和JavaScript文件越来越多，越来越大的问题。

JavaScript中为了重用代码，显而易见，我们会提取出一些共用模块，而模块与模块之前会出现很多依赖，当一个页面所需要的模块越来越多时，
开发过程中，需要通过`script`标签引入需要的JS文件，其复杂程度可想而知，加载的模块文件越多对页面加载速度的影响也会比较大。（浏览器对一个域名同时发起请求数量是有限制的，当然有人会说，那用多个域名来代理静态文件请求就可以解决了呀，对的，这是其他可以优化的地方。）

当然可以对css和js文件进行合并压缩，对css文件合并压缩相对来说简单些，但js文件在合并之前需要处理依赖关系，若由
人工来处理，其工作又比较枯燥乏味，而且人不能避免100%不犯错误。

当然行业中已经有比较成熟的前端构建工具——[Grunt JS][gruntjs english](中文文档请移歩[这里][gruntjs chinese])，可以帮助我们解决这些问题。

具体grunt的安装，配置文件等，请参照[grunt文档][gruntjs chinese]，这里不再赘述。

<!-- more -->

## 2 处理CSS
利用Grunt的`grunt-contrib-cssmin`插件来压缩CSS文件。

具体方法如下：

当前目录为`pingjia/static/`，package.json和Gruntfile.js文件都在当前目录下。

1. `package.json`中`devDependencies`需要增加`grunt-contrib-cssmin`插件依赖，如下：

        "devDependencies": {
            "grunt": "~0.4.5",
            "grunt-contrib-cssmin": "~0.10.0"
        }

2. 运行`npm install`安装需要的node依赖包。

3. 配置`Gruntfile.js`，主要配置如下：

        module.exports = function(grunt) {
            grunt.initConfig({
                pkg: grunt.file.readJSON('package.json'),
                cssmin: {
                    options: {
                        report: 'gzip'
                    },
                    libs: {
                        // Minify libs css files.
                        files: [
                            {
                                expand: true,
                                cwd: 'libs/src/',
                                src: '*.css',
                                dest: 'libs/dist/'
                            }
                        ]
                    },
                    web_app: {
                        src: 'web_app/src/web_app.css',
                        dest: 'web_app/dist/web_app.css'
                    }
                },
            }
            grunt.loadNpmTasks('grunt-contrib-cssmin');
            grunt.registerTask('css', ['cssmin']);
        };

`cssmin`是任务task，里面有两个目标target，分别为`libs`和`web_app`。
具体的配置语法，请参见[Grunt 任务配置][gruntjs chinese]。

4. 运行`grunt css`，执行css压缩任务，可以看到如下信息：

        Running "cssmin:libs" (cssmin) task
        File libs/dist/public.css created: 1.06 kB → 809 B → 516 B (gzip)

        Running "cssmin:web_app" (cssmin) task
        File web_app/dist/web_app.css created: 6.66 kB → 5.2 kB → 1.46 kB (gzip)

    即可以在`libs/dist/`和`web_app/dist/`目录下看到压缩好的css文件。

5. 将HTML中，加载该CSS文件的link表签中的路径修改为压缩好的CSS文件即可。
该步骤，可以待所有静态文件一起处理完之后，通过shell脚本统一替换，利用`sed`。



## 3 处理JavaScript

前端模块化开发可以采用`sea.js`，构建工具采用`grunt`。


* 利用sea.js可以解决JavaScript中命名冲突和依赖管理的问题，进行模块化开发还可以带来很多好处：

    * 模块的版本管理。通过别名等配置，配合构建工具，可以比较轻松地实现模块的版本管理。

    * 提高可维护性。模块化可以让每个文件的职责单一，非常有利于代码的维护。
    Sea.js 还提供了 nocache、debug 等插件，拥有在线调试等功能，能比较明显地提升效率。

    * 前端性能优化。Sea.js 通过异步加载模块，这对页面性能非常有益。Sea.js 还提供了 combo、flush 等插件，
    配合服务端，可以很好地对页面性能进行调优。

    * 跨环境共享模块。CMD 模块定义规范与 Node.js 的模块规范非常相近。通过 Sea.js 的 Node.js 版本，
    可以很方便实现模块的跨服务器和浏览器共享。

    sea.js使用教程，请移歩这里[sea.js 官网][sea.js site]。

* 利用grunt来解决，JavaScript模块化之后，文件过多，以及文件压缩的问题。

    主要任务是，检查JavaScript的语法和规范问题，解决JavaScript各模块依赖问题，合并，压缩JavaScript文件。


具体方法如下：

使用到的grunt插件：

* `grunt-contrib-jshint`插件来检查JavaScript的语法和规范问题
* `grunt-cmd-transport`插件来提取sea.js的依赖
* `grunt-cmd-concat`插件来合并JavaScript文件
* `grunt-contrib-uglify`插件压缩JavaScript文件
* `grunt-contrib-clean`插件，清理无用的文件和目录

当前目录为`pingjia/static/`，package.json和Gruntfile.js文件都在当前目录下。


1. `package.json`中`devDependencies`需要增加grunt插件依赖，如下：

        "devDependencies": {
            "grunt": "~0.4.5",
            "grunt-cmd-transport": "~0.2.0",
            "grunt-contrib-clean": "~0.4.0",
            "grunt-contrib-concat": "0.5.0",
            "grunt-contrib-jshint": "^0.11.2",
            "grunt-contrib-uglify": "~0.2.2",
        }

2. 运行`npm install`安装需要的node依赖包。

3. 配置`Gruntfile.js`，主要配置如下：

        module.exports = function(grunt) {
            grunt.initConfig({
                pkg: grunt.file.readJSON('package.json'),
                transport: {
                    options: {
                        paths: ['.'],      // sea模块路径
                        idleading: '',     // 生成的模块id的前缀
                        alias: '<%= pkg.conf.alias %>'   // html文件中seajs.config中指定的alias，这里为方便起见，把所有的alias都放在pkg.config.alias配置中
                    },
                    web_app: {
                        options: {
                            idleading: 'web_app/dist/'
                        },
                        files: [
                            {
                                expand: true,
                                cwd: 'web_app/src/',   // 所有src指定的匹配都将相对于此处指定的路径
                                src: ['web_app.js', 'public.js'],    // 相对于cwd路径的匹配模式
                                dest: 'web_app/.build'    // 目标文件路径前缀
                            }
                        ]
                    }
                },
                concat: {
                    options: {
                        separator: ";"
                    },
                    web_app: {
                        src: ['web_app/.build/*.js', '!web_app/.build/*-debug.js'],
                        dest: 'web_app/dist/web_app.js'
                    }
                },
                uglify: {
                    libs: {
                        // Minify libs js files.
                        files: [
                            {
                                expand: true,
                                cwd: 'libs/src/',
                                src: '*.js',
                                dest: 'libs/dist'
                            }
                        ]
                    },
                    web_app: {
                        src: 'web_app/dist/web_app.js',
                        dest: 'web_app/dist/web_app.js'
                    }
                },
                clean: {
                    build: ['*/.build']
                },
                jshint: {
                    files: {
                        src: ['web_app/src/web_app.js']
                    },
                    options: {
                        asi:false,//检测行尾是否加分号
                        indent: true,//代码缩进
                        latedef: "nofunc",//禁止定义之前使用变量，忽略 function 函数声明
                        quotmark: false,//为 true 时，禁止单引号和双引号混用
                        unused: false,//变量未使用
                        lastsemic: true,//检查一行代码最后声明后面的分号是否遗漏
                        noempty:true,//如果为真，JSHint会禁止出现空的代码块
                        curly: true, //大括号包裹
                        eqeqeq: false,//如果为真，JSHint会看你在代码中是否都用了===或者是!==，而不是使用==和!=。建议在比较0，''(空字符)，undefined，null，false和true的时候使用===和!===。
                        newcap: true,//对于首字母大写的函数（声明的类），强制使用new
                        noarg: true, //禁用arguments.caller和arguments.callee
                        sub: true, //对于属性使用aaa.bbb而不是aaa['bbb']
                        undef: false, //查找所有未定义变量
                        maxerr:30,//设定错误的阈值，超过这个阈值jshint不再向下检查，提示错误太多
                        boss: true,
                        jquery: true,
                        browser: true,
                        devel: true,
                        predef: [
                            "define",
                            "seajs"
                        ]
                    }
                }
            });

            grunt.loadNpmTasks('grunt-contrib-uglify');
            grunt.loadNpmTasks('grunt-cmd-transport');
            grunt.loadNpmTasks('grunt-cmd-concat');
            grunt.loadNpmTasks('grunt-contrib-uglify');
            grunt.loadNpmTasks('grunt-contrib-clean');

            grunt.registerTask('checkjs', ['jshint']);
            grunt.registerTask('js', ['jshint', 'transport', 'concat', 'uglify', 'clean']);
        };

4. 运行`grunt js`，执行检查JS语法，提取依赖，合并，压缩任务，可以看到如下信息：

        Running "jshint:files" (jshint) task
        >> 1 files lint free.

        Done, without errors.

        Running "transport:web_app" (transport) task
        transport 2 files

        Running "concat:web_app" (concat) task
        Concated 1 files

        Running "uglify:libs" (uglify) task
        File "libs/dist/jquery.js" created.
        File "libs/dist/public.js" created.
        File "libs/dist/sea.js" created.

        Running "uglify:web_app" (uglify) task
        File "web_app/dist/web_app.js" created.

        Running "clean:build" (clean) task
        Cleaning "web_app/.build"...OK

        Done, without errors.

    执行了jshint任务，检查了JS语法，以及规范。

    执行了transport任务的web_app目标，提取了两个文件的依赖。

    执行了concat任务的web_app目标，合并成了一个文件。

    执行了uglify任务的libs目标和web_app目标，压缩之后，得到`libs/dist/jquery.js`，`libs/dist/public.js`，
    `libs/dist/sea.js`文件和`web_app/dist/web_app.js`文件。

    执行clean任务，删除了web_app/.build文件夹。


5. 将HTML中，加载JavaScript文件的link表签中的路径修改为压缩好的JavaScript文件即可。

该步骤，可以待所有静态文件一起处理完之后，通过shell脚本统一替换，利用`sed`。


## 3 压缩图片

利用Grunt的`grunt-contrib-imagemin`插件来压缩CSS文件。

具体方法如下：

当前目录为`pingjia/static/`，package.json和Gruntfile.js文件都在当前目录下。

1. `package.json`中`devDependencies`需要增加`grunt-contrib-imagemin`插件依赖，如下：

        "devDependencies": {
            "grunt": "~0.4.5",
            "grunt-contrib-imagemin": "^0.9.4"
        }

2. 运行`npm install`安装需要的node依赖包。

3. 配置`Gruntfile.js`，主要配置如下：

        module.exports = function(grunt) {
            grunt.initConfig({
                pkg: grunt.file.readJSON('package.json'),
                //压缩图片
                imagemin: {
                    prod: {
                        options: {
                            optimizationLevel: 7,
                            pngquant: true
                        },
                        files: [
                            {expand: true, cwd:'libs/src/', src: ['images/*.{png,jpg,jpeg,gif}'], dest: 'libs/dist/'},
                            {expand: true, cwd:'web_app/src/', src: ['images/*.{png,jpg,jpeg,gif}'], dest: 'web_app/dist/'},
                        ]
                    }
                },
            }
            grunt.loadNpmTasks('grunt-contrib-imagemin');

            grunt.registerTask('image', ['imagemin']);
        };

    在`imagemin`任务的files中，配置要压缩的图片路径，格式，以及压缩之后，目标路径。

4. 运行`grunt image`，压缩图片，会在设置的目标路径下得到压缩之后的图片，可以看到如下信息：

        Running "imagemin:prod" (imagemin) task
        Minified 32 images (saved 560.86 kB)

        Done, without errors.

    执行了imagemin任务，压缩了32个图片文件，节省了560.86KB。

5. 将HTML中，加载JavaScript文件的link表签中的路径修改为压缩好的图片文件路径即可，在CSS中设置的图片路径由于是相对路径，所以不用修改。
该步骤，可以待所有静态文件一起处理完之后，通过shell脚本统一替换，利用`sed`。



## 4 前端开发流程

采用前面提到的方法处理前端静态文件之后，前端的开发流程，有相应的调整，具体请见下图：

1. 当要开发一个新的页面app1时的开发流程。

    ![开发一个新页面][develop new page]

2. 修改一个页面app2的css或js文件时的开发流程。

    ![修改原有页面][modify page]


## 5 自动化处理静态文件

前端开发人员只需要配置好Gruntfile.js中的需要处理的文件，以及目标目录等，剩下的工作可以交给自动化脚本来处理。

脚本文件 check_static_files，使用：

1. 需要指定工程名`PROJECT_NAME`，和`HTML_FILE_PATH` HTML模板文件目录。

2. 执行 `./check_static_files` 查看使用方法，简单易懂，具体如下：

        ******************** Check static files of pingjia **********
        Project directory: /Users/jzhu/Projects/ping

        Usage: check_static_files <command> [dir]

        commands are as follows:

        install     Install all the packages.
        checkjs     Check js files.
        css         Compact css files.
        js          Check js and compact js files.
        replace     Replace '/src/' to '/dist/' in html files.
        static      Check js, compact js, compact css and min images.
        deploy      Install all the packages, check js, compact css and compact js.

        [dir] is optional.

运维人员可以在部署项目的时候，可以非常方便的直接使用该脚本来完成静态文件的处理，当然啦，前提是前端开发人员已经把配置配好了。
所以，在项目上线之前，最好能够在测试环境下，完全模拟压缩之后的情况，进行完整系统测试，避免压缩之后出现问题。


check_static_files 脚本具体如下，


    #!/bin/bash

    # Script to check static files when in development before commit and push code.

    PROJECT_NAME="pingjia"    # 指定工程名
    echo "******************** Check static files of ${PROJECT_NAME} **********"

    cd `dirname $0`
    project_dir=`pwd`
    echo "Project directory:" ${project_dir}


    # 所有需要压缩的目录 以项目目录为根目录
    GRUNT_FILE_PATH=$(find . -name "Gruntfile.js" ! -path '*node_modules*' | sed 's/Gruntfile\.js//g')
    HTML_FILE_PATH="pingjia/templates/"   # 需要指定HTML模板文件路径


    # function for reminder
    NORMAL=$(tput sgr0)
    GREEN=$(tput setaf 2; tput bold)
    RED=$(tput setaf 1)
    function red() {
        echo -e "$RED$*$NORMAL"
    }
    function green() {
        echo -e "$GREEN$*$NORMAL"
    }


    function check_error() {
        if [ $1 -gt 0 ]; then
            red "$2 error!!!"
            exit 1
        fi
    }


    function install_package() {
        echo "Install packages."
        paths=${GRUNT_FILE_PATH}
        if [ -n "$1" ]; then
            paths=$1
        fi
        for path in ${paths}; do
            cd ${path}
            echo "Now in:" `pwd`
            # Install required package
            echo "Install required package."
            npm install
            check_error $? "Install packages."

            echo "Return to project directory."
            cd ${project_dir}
        done

        green "Congratulations! Install all the packages successfully!"
    }

    function replace_src_to_dist() {
        echo "Replace the src to dist in html files."
        for path in ${HTML_FILE_PATH}; do
            echo "Now in:" `pwd`
            files=$(grep -r "/src/" ${path} | sed "s/:.*//g")
            for file in ${files}; do
                echo "Replace ${file}."
                sed -i "s/\/src\//\/dist\//g" ${file}
            done
        done
        green "Congratulations! Replace the '/src/' to '/dist/' successfully!"
    }

    function check_js() {
        echo "Check JavaScript."
        paths=${GRUNT_FILE_PATH}
        if [ -n "$1" ]; then
            paths=$1
        fi
        for path in ${paths}; do
            cd ${path}
            echo "Now in:" `pwd`
            echo "Check JavaScript."
            grunt checkjs
            check_error $? "Grunt checkjs"

            echo "Return to project directory."
            cd ${project_dir}
        done

        green "Congratulations! Check all the JavaScript successfully!"
    }

    function min_images() {
        echo "Min image files."
        paths=${GRUNT_FILE_PATH}
        for path in ${paths}; do
            cd ${path}
            echo "Now in:" `pwd`
            echo "Min images."
            grunt images
            check_error $? "Grunt min images"

            echo "Return to project directory."
            cd ${project_dir}
        done

        green "Congratulations! Compact all the image files successfully!"
    }

    function compact_css() {
        echo "Compact CSS files."
        paths=${GRUNT_FILE_PATH}
        if [ -n "$1" ]; then
            paths=$1
        fi
        for path in ${paths}; do
            cd ${path}
            echo "Now in:" `pwd`
            echo "Compact CSS."
            grunt css
            check_error $? "Grunt compact css"

            echo "Return to project directory."
            cd ${project_dir}
        done

        green "Congratulations! Compact all the CSS files successfully!"
    }

    function compact_js() {
        echo "Compact JS files."
        paths=${GRUNT_FILE_PATH}
        if [ -n "$1" ]; then
            paths=$1
        fi
        for path in ${paths}; do
            cd ${path}
            echo "Now in:" `pwd`
            echo "Compact JS."
            grunt js
            check_error $? "Grunt compact js"

            echo "Return to project directory."
            cd ${project_dir}
        done

        green "Congratulations! Compact all the JS files successfully!"
    }

    commands="install checkjs css js replace all help"

    function useage() {
        echo "
    Usage: check_static_files <command> [dir]

    commands are as follows:

    install     Install all the packages.
    checkjs     Check js files.
    css         Compact css files.
    js          Check js and compact js files.
    replace     Replace '/src/' to '/dist/' in html files.
    static      Check js, compact js, compact css and min images.
    deploy      Install all the packages, check js, compact css and compact js.

    [dir] is optional.
    "
    }

    case $1 in
        install)
            install_package $2
            ;;
        checkjs)
            check_js $2
            ;;
        css)
            compact_css $2
            ;;
        js)
            check_js $2
            compact_js $2
            ;;
        replace)
            replace_src_to_dist
            ;;
        static)
            check_js $2
            compact_js $2
            compact_css $2
            min_images
            ;;
        deploy):
            install_package
            check_js
            compact_css
            compact_js
            replace_src_to_dist
            ;;
        help)
            useage;;
        *)
            useage;;
esac



若有疑问，请联系：[我][contact me]

[sea.js site]:http://seajs.org
[gruntjs english]:http://gruntjs.com/
[gruntjs chinese]:http://gruntjs.org/
[contact me]: /page/about.html

[develop new page]: http://zhujinliang.qiniudn.com/img/blog/pingjia/develop_new_page.png
[modify page]: http://zhujinliang.qiniudn.com/img/blog/pingjia/modify_page.png

