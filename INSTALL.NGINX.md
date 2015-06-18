title: 技能丨为你的朋友们提供谷歌搜索引擎／又名简易谷歌搜索镜像制作
date: June 17, 2015 8:40 PM
categories: 技能
tags: [谷歌镜像,nginx-module]


---
![](http://i.imgur.com/CB2PTfD.jpg)
教程中我会用飘准的白话文介绍的，快的话，７/８分钟就好了。我们的目标是，用最小的代价：给自己和身边的朋友提供实时的谷歌搜索。
<!--more-->
#####free-search-project丨付费自由搜索项目，由你维护的谷歌搜索镜像。
![](http://i.imgur.com/6euaDOm.png)
####Demo：https://s.limboshop.info
####必备工具：
1.一台可用的VPS（[虚拟专用服务器](http://baike.baidu.com/view/309631.htm)）；
2.伺服器系统选择：ubuntu 14.04（[介绍](http://www.ubuntu.org.cn/global)）

>本教程只介绍在Ubuntu 14.04的特定操作系统下简易做谷歌镜像服务的操作方法，初次尝试你可能需要一个纯净的系统来支持。<font><font color="red">注：本教程是以root用户通过ssh登入系统进行操作。（VPS一般都默认为此用户）</font> <font><font color="gr">//购买[搬瓦工](https://bandwagonhost.com/cart.php?a=confproduct&i=1)//[linode.com](https://www.linode.com/?r=e52ffb7bb6f974b0ef841740e547d87cf0742d69)//[conoha.jp](https://www.conoha.jp/referral/?token=koGIOJhl5cKdz7NH.MsG.Wtv6BURaFQfctkqzR9cT5HZ5P6AP4I-I06)//前两者均支持万事达卡和[paypal](https://www.paypal.com)，conoha支持支付宝。</font>

---

####步骤0：登入系统后执行一次源更新

    $ apt-get update
####步骤1：下载＝Nginx Module for Google
1.1 下载nginx-1.7.8

    $ wget http://nginx.org/download/nginx-1.7.8.tar.gz
1.2 解压nginx-1.7.8

    $ tar -xf nginx-1.7.8.tar.gz
1.3 安装git
     
     $ apt-get install git
1.4 git clone　相关nginx模块：[ngx_http_google_filter_module](https://github.com/cuber/ngx_http_google_filter_module)

    $ git clone https://github.com/cuber/ngx_http_google_filter_module
1.5 git clone 相关nginx模块：[ngx_http_substitutions_filter_module](https://github.com/yaoweibin/ngx_http_substitutions_filter_module)

    $ git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module
1.6 安装 [PCRE Library](http://www.pcre.org/free-search-project.md)

    $ apt-get install libpcre3 libpcre3-dev
1.8 安装　[openssl-lib](http://stackoverflow.com/questions/3016956/how-do-i-install-the-openssl-c-library-on-ubuntu)

    $ apt-get install openssl libssl-devlibssl-dev
####步骤２：编译之前下载的nginx-1.7.8　（参阅1.1-1.2步）

    $ cd /root/nginx-1.7.8

2.1 编译 ./configure

    $ ./configure --prefix=/usr/local/nginx-1.7.8 --add-module=/root/ngx_http_google_filter_module --add-module=/root/ngx_http_substitutions_filter_module --with-http_ssl_module  

![](http://i.imgur.com/VAKcwOh.png)
2.2 如上一条操作无误，则继续：make
    
    $ make && make install
2.3 启动 nginx或关闭nginx ()

    $ /usr/local/nginx-1.7.8/nginx
    $ killall nginx
2.4 查看　nginx 版本(是不是version=1.7.8)

    $ nginx -v 
2.5 查看nginx 配置(是不是＝步骤2.1设置相关)

    $ nginx -V
2.6 配置nginx.conf文件

     $ nginx -t #会显示配置是否0k? 
>**e.g.**
>nginx: the configuration file /usr/local/nginx-1.7.8/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx-1.7.8/conf/nginx.conf test is successful

2.7 如步骤2.6默认配置文件在 /usr/local/nginx-1.7.8/conf/nginx.conf

    $ vi /usr/local/nginx-1.7.8/conf/nginx.conf
    
白话文解释：下面＃1为需要你找到并手动注释掉的代码行（如下操作即可，在相应代码行前加＃号），其他未加＃1的需要你手动添加，这段话不用加上去，谢谢。／／自己对比下源文件和这里的差别就可以找出来了。大约从34行开始注释吧。
    #1    server {
    #1       listen       80;
    #1     server_name  localhost;
    #1
    #1      #charset koi8-r;
    #1
    #access_log  logs/host.access.log  main;

    #1       location / {
    #1         root   html;
    #1       index  index.html index.htm;
    #1 }
    server {
    # ... part of server configuration
    resolver 8.8.8.8 8.8.4.4;
    location / {
    google on;
    google_scholar on;
    # set language to German
    google_language zh-CN;
    }
    # ...
    }

    server {
    # ...
    location / {
    google on;
    google_ssl_off "www.google.com";
    }
    # ...
    }

    upstream www.google.com {
    server 173.194.38.1:443;
    server 173.194.38.2:443;
    server 173.194.38.3:443;
    server 173.194.38.4:443;
    }

    #
    # configuration of vps(us)
    #
    server {
    listen 80;
    server_name www.google.com;
    # ...
    location / {
    proxy_pass https://www.google.com;
    # ...
    }
    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
#####附录：
3.1　项目地址：https://github.com/cuber/ngx_http_google_filter_module
3.2　如何安装nginx第三方模块：http://www.ttlsa.com/nginx/how-to-install-nginx-third-modules/
3.3　SSL+Nginxの配置相关知识：https://github.com/cuber/ngx_http_google_filter_module/issues/27
3.4　原作者相关答疑：https://github.com/cuber/ngx_http_google_filter_module/issues (我建议在有疑问／建议可在此pull issues)
#####其他反向代理文章
3.5　[openresty+lua在反向代理服务中的玩法](http://drops.wooyun.org/tips/6403)
3.6　[反向代理的有趣用法及如何禁止被反代](http://drops.wooyun.org/tips/509)
#####Demo
4.0　Demo：[https:s.limboshop.info](https:s.limboshop.info)
####步骤５：给浏览器自定义搜索引擎
![](http://i.imgur.com/SLj05Q6.png)
**e.g.**
设置默认搜索为：https://s.limboshop.info
![](http://i.imgur.com/o0mUtNx.png)
####步骤6：将此文分享给你信任的圈子吧
文章地址：http://story.limboshop.info/2015/06/17/free-search-project/
####笔者后言
尽管可以翻墙去使用 Google，但是有时连翻墙手段都不稳定的情况下，此刻又有搜索国外技术内容的需求时怎么办？如果你有一个VPS，提供相关fq服务，若没有搭建网站相关，可以试试这个？为周围的同学朋友提供自由的谷歌搜索引擎。
####联系我吧或在文章下方评论留言
邮箱：limboshop.info@gmail.com
主站：http://limboshop.info
花园：http://story.limboshop.info
维护：http://note.limboshop.info
Google+：http://plus.google.com/u/0/+MarkWongUnique/posts
／／
---
文章地址：http://story.limboshop.info/2015/06/17/free-search-project/
本文遵循：　[姓名標示-非商業性-相同方式分享 3.0 中國大陸 (CC BY-NC-SA 3.0 CN) ](姓名標示-非商業性-相同方式分享 3.0 中國大陸 (CC BY-NC-SA 3.0 CN)