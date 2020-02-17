---
layout: post
title: 博客迁移到云服务器的踩坑记录
---

## 前言

最近买了个香港的VPS用来搭梯子。

心想着：不是刚好有个服务器，顺便把博客搬过去。

还能加快访问速度，一举两得。


## 环境搭建

在Centos7上安装**ruby**环境和**Jekyll**。

我是看的这篇博客 [地址](https://cloud.tencent.com/developer/article/1453573)。

搭建完以后，如果你用**jekyll serve -H 0.0.0.0 -P 80 --detach --trace**这条命令来运行，会收到如下错误。

```
rom /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/webrick/httpserver.rb:47:in `initialize'
	 8: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/webrick/server.rb:108:in `initialize'
	 7: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/webrick/server.rb:127:in `listen'
	 6: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/webrick/utils.rb:65:in `create_listeners'
	 5: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/socket.rb:762:in `tcp_server_sockets'
	 4: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/socket.rb:227:in `foreach'
	 3: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/socket.rb:227:in `each'
	 2: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/socket.rb:764:in `block in tcp_server_sockets'
	 1: from /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/socket.rb:201:in `listen'
/usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/2.6.0/socket.rb:201:in `bind': Permission denied - bind(2) for 0.0.0.0:80 (Errno::EACCES)
```
权限拒绝，那么就上网查一下呗。

查阅了资料才发现，Linux只有root用户才能使用1024以下的端口。

说白了就是80端口不给用呗。

重新运行命令**jekyll serve -H 0.0.0.0 -P 3000 --detach --trace**。

没想到还是不行。

我于是去问了一下SugerHosts的客服。

他是这么回答的。

>>> DECADE云服务器默认开放22、80和443端口，如果需要使用其他端口需要在防火墙功能中添加允许的IN类型端口

于是去防火墙控制板把端口开了。

然而每次访问需要加端口号，感觉就很蠢。

既然这样，我们可以把80端口转发到3000端口上去。

这不就是端口转发么？


## Nginx端口转发

首先安装Nginx。

```
sudo yum -y install nginx
```

找到配置文件的位置。

```
sudo nginx -t
```
设置转发。

```
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  127.0.0.0.1;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_redirect default;
        }


        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

## 域名

接下来就是用域名绑定服务器了。

我原来使用CloudFlare做域名解析。

现在可以直接把CLoudFlare上的域名删了，直接从域名购买商处添加对应的A记录和地址。

这样就大功告成了。

测试中开了梯子网页是秒开的，国内的话稍微慢一点。

还有个小问题，网易云由于版权的原因，国外的话音乐外链可能听不了。

## SSL证书

获取免费的Let's Encrypt证书 [地址](https://freessl.cn/)。

这个证书的最大好处就在于可靠性高，不过三个月得换一次。

把生成的证书用scp命令传到服务器上来。

再用nginx配置ssl证书。

nginx默认用户是nobody，和启动用户不一致，可能会导致403错误。

把nginx的用户改成root就行了。

这样，网站就可以同时用http和https访问了。






