---
layout: post
title: "Nginx配置 Http Basic Auth 保护目录"
category: "Server"
date: 2014-04-07
---


在使用 nginx 作为前端服务器时，某些资源是需要保护的。可以用 [http auth basic][http auth basic] 来认证简单方便。


它的配置也挺方便的，我们可以沿用由Apache的htpasswd模块生成的.htpasswd文件作为密码文件。
注意，nginx 的 http auth basic 的密码是用 crypt(3) 加密的，而apache是md5加密。所以生成时:


    /your_dir/apache/bin/htpasswd -c -d pass_file user_name

    # 回车输入密码
    # -c 表示生成文件，-d 是以 crypt 加密。

对于没有安装apache的情况下，文章后面会提到其他生成密码文件的方法。

我们将这个 htpasswd文件放到 `nginx/conf` 下，记得修改权限来保护一下。

    chmod 644 htpasswd

然后修改nginx.conf：


    server {

        server_name example.com;

        location / {

         auth_basic            "Password please";

         auth_basic_user_file  htpasswd;     # htpasswd文件放在conf目录下或者采用完整路径

        }

    }

加入了：

    auth_basic            "Password please";
    auth_basic_user_file  htpasswd;

重新载入nginx配置文件：

    service nginxd reload


## 其他方式生成htpasswd文件

Nginx wiki里推荐了很多方式，请见[这里][generate htpasswd]。
其中一种方式是采用python脚本生成，脚本请见[这里][generate htpasswd python]。

下载脚本文件，执行如下命令：

    chmod u+x htpasswd.py

    ./htpasswd.py -c -b htpasswd username password

    #-c为生成文件 htpasswd为文件名 username为用户名  password 为密码




[http auth basic]: http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html
[generate htpasswd]: http://wiki.nginx.org/Faq#How_do_I_generate_an_.htpasswd_file_without_having_Apache_tools_installed.3F
[generate htpasswd python]: http://trac.edgewall.org/browser/trunk/contrib/htpasswd.py?format=txt
