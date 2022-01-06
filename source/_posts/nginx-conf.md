---
title: nginx配置
date: 2022-1-5 16:58:45
comments: true
toc: true
top_img: /img/sailing.jpg
categories:
  - nginx
tags:
  - nginx
---

## Nginx 介绍

传统的 Web 服务器，每个客户端连接作为一个单独的进程或线程处理，需在切换任务时将 CPU 切换到新的任务并创建一个新的运行时上下文，消耗额外的内存和 CPU 时间，当并发请求增加时，服务器响应变慢，从而对性能产生负面影响。
Nginx 是开源、高性能、高可靠的 Web 和反向代理服务器，而且支持热部署，几乎可以做到 7 \* 24 小时不间断运行，即使运行几个月也不需要重新启动，还能在不间断服务的情况下对软件版本进行热更新。性能是 Nginx 最重要的考量，其占用内存少、并发能力强、能支持高达 5w 个并发连接数，最重要的是，Nginx 是免费的并可以商业化，配置使用也比较简单。

Nginx 的最重要的几个使用场景:

1. 静态资源服务，通过本地文件系统提供服务;
2. 反向代理服务，延伸出包括缓存、负载均衡等;
3. API 服务，OpenResty;

对于前端来说 Node.js 不陌生了，Nginx 和 Node.js 的很多理念类似，HTTP 服务器、事件驱动、异步非阻塞等，且 Nginx 的大部分功能使用 Node.js 也可以实现，但 Nginx 和 Node.js 并不冲突，都有自己擅长的领域。Nginx 擅长于底层服务器端资源的处理（静态资源处理转发、反向代理，负载均衡等），Node.js 更擅长上层具体业务逻辑的处理，两者可以完美组合，共同助力前端开发。

## 相关概念

### 简单请求和非简单请求

首先我们来了解一下简单请求和非简单请求，如果同时满足下面两个条件，就属于简单请求:

1. 请求方法是 HEAD、GET、POST 三种之一；
2. HTTP 头信息不超过右边着几个字段：Accept、Accept-Language、Content-Language、Last-Event-IDContent-Type 只限于三个值 application/x-www-form-urlencoded、multipart/form-data、text/plain;

凡是不同时满足这两个条件的，都属于非简单请求。

浏览器处理简单请求和非简单请求的方式不一样：

#### 简单请求

对于简单请求，浏览器会在头信息中增加 Origin 字段后直接发出，Origin 字段用来说明，本次请求来自的哪个源（协议+域名+端口）。
如果服务器发现 Origin 指定的源不在许可范围内，服务器会返回一个正常的 HTTP 回应，浏览器取到回应之后发现回应的头信息中没有包含 Access-Control-Allow-Origin 字段，就抛出一个错误给 XHR 的 error 事件；
如果服务器发现 Origin 指定的域名在许可范围内，服务器返回的响应会多出几个 Access-Control- 开头的头信息字段。

#### 非简单请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是 PUT 或 DELETE，或 Content-Type 值为 application/json。浏览器会在正式通信之前，发送一次 HTTP 预检 OPTIONS 请求，先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些 HTTP 请求方法和头信息字段。只有得到肯定答复，浏览器才会发出正式的 XHR 请求，否则报错。

### 跨域

在浏览器上当前访问的网站向另一个网站发送请求获取数据的过程就是跨域请求。
跨域是浏览器的同源策略决定的，是一个重要的浏览器安全策略，用于限制一个 origin 的文档或者它加载的脚本与另一个源的资源进行交互，它能够帮助阻隔恶意文档，减少可能被攻击的媒介，可以使用 CORS 配置解除这个限制。如下:

```
# 同源的例子
http://example.com/app1/index.html  # 只是路径不同
http://example.com/app2/index.html

http://Example.com:80  # 只是大小写差异
http://example.com

# 不同源的例子
http://example.com/app1   # 协议不同
https://example.com/app2

http://example.com        # host 不同
http://www.example.com
http://myapp.example.com

http://example.com        # 端口不同
http://example.com:8080
```

### 正向代理和反向代理

反向代理（Reverse Proxy）对应的是正向代理（Forward Proxy），他们的区别：

1. 正向代理： 一般的访问流程是客户端直接向目标服务器发送请求并获取内容，使用正向代理后，客户端改为向代理服务器发送请求，并指定目标服务器（原始服务器），然后由代理服务器和原始服务器通信，转交请求并获得的内容，再返回给客户端。正向代理隐藏了真实的客户端，为客户端收发请求，使真实客户端对服务器不可见；
   举个具体的例子，你的浏览器无法直接访问谷哥，这时候可以通过一个代理服务器来帮助你访问谷哥，那么这个服务器就叫正向代理。
2. 反向代理： 与一般访问流程相比，使用反向代理后，直接收到请求的服务器是代理服务器，然后将请求转发给内部网络上真正进行处理的服务器，得到的结果返回给客户端。反向代理隐藏了真实的服务器，为服务器收发请求，使真实服务器对客户端不可见。一般在处理跨域请求的时候比较常用。现在基本上所有的大型网站都设置了反向代理。
   举个具体的例子，去饭店吃饭，可以点川菜、粤菜、江浙菜，饭店也分别有三个菜系的厨师，但是你作为顾客不用管哪个厨师给你做的菜，只用点菜即可，小二将你菜单中的菜分配给不同的厨师来具体处理，那么这个小二就是反向代理服务器。简单的说，一般给客户端做代理的都是正向代理，给服务器做代理的就是反向代理。

### 负载均衡

一般情况下，客户端发送多个请求到服务器，服务器处理请求，其中一部分可能要操作一些资源比如数据库、静态资源等，服务器处理完毕后，再将结果返回给客户端。
这种模式对于早期的系统来说，功能要求不复杂，且并发请求相对较少的情况下还能胜任，成本也低。随着信息数量不断增长，访问量和数据量飞速增长，以及系统业务复杂度持续增加，这种做法已无法满足要求，并发量特别大时，服务器容易崩。
很明显这是由于服务器性能的瓶颈造成的问题，除了堆机器之外，最重要的做法就是负载均衡。
请求爆发式增长的情况下，单个机器性能再强劲也无法满足要求了，这个时候集群的概念产生了，单个服务器解决不了的问题，可以使用多个服务器，然后将请求分发到各个服务器上，将负载分发到不同的服务器，这就是负载均衡，核心是「分摊压力」。Nginx 实现负载均衡，一般来说指的是将请求转发给服务器集群。
举个具体的例子 ，晚高峰乘坐地铁的时候，入站口经常会有地铁工作人员大喇叭“请走 B 口，B 口人少车空....”，这个工作人员的作用就是负载均衡。

### 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度，降低原来单个服务器的压力。

## Nginx 快速安装

> yum list | grep nginx

> yum install nginx

## Nginx 操作常用命令

Nginx 的命令在控制台中输入 nginx -h 就可以看到完整的命令，这里列举几个常用的命令：

```
nginx -s reload  # 向主进程发送信号，重新加载配置文件，热重启
nginx -s reopen	 # 重启 Nginx
nginx -s stop    # 快速关闭
nginx -s quit    # 等待工作进程处理完成后关闭
nginx -T         # 查看当前 Nginx 最终的配置
nginx -t -c <配置路径>    # 检查配置是否有问题，如果已经在配置目录，则不需要-c
```

systemctl 是 Linux 系统应用管理工具 systemd 的主命令，用于管理系统，我们也可以用它来对 Nginx 进行管理，相关命令如下：

```
systemctl start nginx    # 启动 Nginx
systemctl stop nginx     # 停止 Nginx
systemctl restart nginx  # 重启 Nginx
systemctl reload nginx   # 重新加载 Nginx，用于修改配置后
systemctl enable nginx   # 设置开机启动 Nginx
systemctl disable nginx  # 关闭开机启动 Nginx
systemctl status nginx   # 查看 Nginx 运行状态
```

window 下 Nginx 相关命令如下：
Windows 下 Nginx 的启动、停止等命令在 Windows 下使用 Nginx，我们需要掌握一些基本的操作命令，比如：启动、停止 Nginx 服务，重新载入 Nginx 等，下面我就进行一些简单的介绍。

1. 启动：C:\server\nginx-1.0.2>start nginx 或 C:\server\nginx-1.0.2>nginx.exe
2. 停止：C:\server\nginx-1.0.2>nginx.exe -s stop 或 C:\server\nginx-1.0.2>nginx.exe -s quit 注：stop 是快速停止 nginx，可能并不保存相关信息；quit 是完整有序的停止 nginx，并保存相关信息。
3. 重新载入 Nginx：C:\server\nginx-1.0.2>nginx.exe -s reload 当配置信息修改，需要重新载入这些配置时使用此命令。
4. 重新打开日志文件：C:\server\nginx-1.0.2>nginx.exe -s reopen
5. 查看 Nginx 版本：C:\server\nginx-1.0.2>nginx -v

## Nginx 配置语法

就跟前面文件作用讲解的图所示，Nginx 的主配置文件是 /etc/nginx/nginx.conf，你可以使用 cat -n nginx.conf 来查看配置。
nginx.conf 结构图可以这样概括：

```
main        # 全局配置，对全局生效
├── events  # 配置影响 Nginx 服务器或与用户的网络连接
├── http    # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置
│   ├── upstream # 配置后端服务器具体地址，负载均衡配置不可或缺的部分
│   ├── server   # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块
│   ├── server
│   │   ├── location  # server 块可以包含多个 location 块，location 指令用于匹配 uri
│   │   ├── location
│   │   └── ...
│   └── ...
└── ...
```

一个 Nginx 配置文件的结构就像 nginx.conf 显示的那样，配置文件的语法规则：
配置文件由指令与指令块构成；每条指令以 ;分号结尾，指令与参数间以空格符号分隔；指令块以 {} 大括号将多条指令组织在一起；include 语句允许组合多个配置文件以提升可维护性；使用 #符号添加注释，提高可读性；使用 $ 符号使用变量；部分指令的参数支持正则表达式；

## 典型配置

```conf
#user  nobody;   # 运行用户，默认即是nginx，可以不进行设置
worker_processes  1;  # Nginx 进程数，一般设置为和 CPU 核数一样

#error_log  logs/error.log;    # Nginx 的错误日志存放目录
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;  # Nginx 服务启动时的 pid 存放位置


events {
    # use epoll;  使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
    worker_connections  1024; # 每个进程允许最大并发数
}


http {  # 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
    include       mime.types;  # 文件扩展名与类型映射表
    default_type  application/octet-stream;   # 默认文件类型
    # 设置日志模式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main; # Nginx访问日志存放位置

    sendfile        on;   # 开启高效传输模式
    #tcp_nopush     on;  # 减少网络报文段的数量

    #keepalive_timeout  0;
    keepalive_timeout  65; # 保持连接的时间，也叫超时时间，单位秒

    #gzip  on;
    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 9;
    gzip_types       text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;
    gzip_disable "MSIE [1-6]\.";
    gzip_vary on;

    #必须添加的
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    server {
        listen       80;  # 配置监听的端口
        server_name  localhost;  # 配置的域名

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;  # 网站根目录
            index  index.html index.htm;  # 默认首页文件
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    server {
        listen       1000;
	    server_name  localhost;
	    location / {
            root  D:/Nginx/nginx-1.18.0/html/build;
		    index  index.html index.htm;
            try_files $uri $uri/ /index.html;
	    }

        # location / {
        #     root  D:/Nginx/nginx-1.18.0/html/dist;
        #     try_files $uri $uri/ /index.html;
        # }

        # location / {
        #     proxy_pass http://xxx/;
        #     proxy_set_header Host $host;
        #     proxy_set_header X-Real-IP $remote_addr;
        #     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #     proxy_set_header X-Forwarded-Proto $scheme;
        #     proxy_redirect off;
        # }


        # location ~ .*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$ {
            # expires      7d;
            # add_header Cache-Control no-store;
            # add_header Cache-Control max-age=3600;
            # add_header Cache-Control public;
            # add_header Cache-Control only-if-cached;
            # add_header Cache-Control no-cache;
            # add_header Cache-Control must-revalidate;
        # }

        # location ~ .*\.(?:js|css)$ {
        #     expires      7d;
        # }

        # location ~ .*\.(?:htm|html)$ {
        #     add_header Cache-Control "private";
        # }
    }

    server {
        listen       9523;
        server_name  0.0.0.0;
		#ssl	on;

		#ssl_certificate      F:/ept-prod-jar/ssl/network.crt;
		#ssl_certificate_key  F:/ept-prod-jar/ssl/network.key;

		#ssl_session_cache    shared:SSL:1m;
		#ssl_session_timeout  5m;

		#ssl_ciphers  HIGH:!aNULL:!MD5;
		#ssl_prefer_server_ciphers  on;
		#add_header Access-Control-Allow-Origin *;
        #add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
        #add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
        #if ($request_method = 'OPTIONS') {
        #    return 204;
        #}

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location = /index.html {
            root   D:/Nginx/nginx-1.18.0/html/dist;
            index  index.html;
        }

        location = / {
            root   D:/Nginx/nginx-1.18.0/html/dist;
            index  index.html;
        }

        location /static/ {
            root   D:/Nginx/nginx-1.18.0/html/dist;
			add_header Content-Security-Policy "default-src 'self';";
            add_header X-Content-Type-Options nosniff;
            add_header X-XSS-Protection 1;
        }

        location /js/ {
            root   D:/Nginx/nginx-1.18.0/html/dist;
			add_header Content-Security-Policy "default-src 'self';";
            add_header X-Content-Type-Options nosniff;
            add_header X-XSS-Protection 1;
        }

		# 添加error_page部分，安全相关
        error_page 403 =404 /404.html; # =后不能有空格

		location = /404.html {
    		internal; #return 404
		}

        # location /token {
        #     proxy_pass http://xxx/token;
        #     proxy_set_header Host $host;
        #     proxy_set_header X-Real-IP $remote_addr;
        #     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #     proxy_set_header X-Forwarded-Proto $scheme;
        #     proxy_redirect off;
        # }

        location / {
            proxy_pass https://xxx/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
            # try_files $uri $uri/ /index.html;
        }

		# location /socketNotify/ {
        #     proxy_pass http://xxx/;
        #     proxy_set_header Host $host;
        #     proxy_set_header X-Real-IP $remote_addr;
        #     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #     proxy_set_header X-Forwarded-Proto $scheme;
        #     proxy_http_version 1.1;
        #     proxy_set_header Upgrade $http_upgrade;
        #     proxy_set_header Connection $connection_upgrade;
		# 	proxy_read_timeout 1800s;
        # }
    }

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

server 块可以包含多个 location 块，location 指令用于匹配 uri，语法：

```conf
location [ = | ~ | ~* | ^~] uri {
	...
}
```

指令后面：

`=` 精确匹配路径，用于不含正则表达式的 uri 前，如果匹配成功，不再进行后续的查找；`^~` 用于不含正则表达式的 uri 前，表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找；`~`表示用该符号后面的正则去匹配路径，区分大小写； `~*`表示用该符号后面的正则去匹配路径，不区分大小写。跟`~` 优先级都比较低，如有多个 location 的正则能匹配的话，则使用正则表达式最长的那个；如果 uri 包含正则表达式，则必须要有 `~` 或 `~*` 标志。

### 全局变量

Nginx 有一些常用的全局变量，你可以在配置的任何位置使用它们。

**全局变量名功能**$host请求信息中的Host，如果请求中没有Host行，则等于设置的服务器名，不包含端口$request_method 客户端请求类型，如 GET、POST$remote_addr客户端的IP地址$args 请求中的参数$arg_PARAMETERGET请求中变量名 PARAMETER 参数的值，例如：$http_user_agent(Uaer-Agent 值),$http_referer...$content_length 请求头中的 Content-length 字段$http_user_agent客户端agent信息$http_cookie 客户端 cookie 信息$remote_addr客户端的IP地址$remote_port 客户端的端口$http_user_agent客户端agent信息$server_protocol 请求使用的协议，如 HTTP/1.0、HTTP/1.1$server_addr服务器地址$server_name 服务器名称$server_port服务器的端口号$schemeHTTP 方法（如 http，https）还有更多的内置预定义变量，可以直接搜索关键字「nginx 内置预定义变量」可以看到一堆博客写这个，这些变量都可以在配置文件中直接使用。

## 设置二级域名虚拟主机

在某某云 ☁️ 上购买了域名之后，就可以配置虚拟主机了，一般配置的路径在 域名管理 -> 解析 -> 添加记录 中添加二级域名，配置后某某云会把二级域名也解析到我们配置的服务器 IP 上，然后我们在 Nginx 上配置一下虚拟主机的访问监听，就可以拿到从这个二级域名过来的请求了

现在我自己的服务器上配置了一个 fe 的二级域名，也就是说在外网访问 fe.sherlocked93.club 的时候，也可以访问到我们的服务器了。
由于默认配置文件 /etc/nginx/nginx.conf 的 http 模块中有一句 include /etc/nginx/conf.d/_.conf 也就是说 conf.d 文件夹下的所有 _.conf 文件都会作为子配置项被引入配置文件中。为了维护方便，我在 /etc/nginx/conf.d 文件夹中新建一个 fe.sherlocked93.club.conf：

```conf
# /etc/nginx/conf.d/fe.sherlocked93.club.conf

server {
  listen 80;
	server_name fe.sherlocked93.club;

	location / {
		root  D:/Nginx/nginx-1.18.0/html/fe;
		index index.html;
	}
}
```

然后在 D:/Nginx/nginx-1.18.0/html 文件夹下新建 fe 文件夹，新建文件 index.html，内容随便写点，改完 nginx -s reload 重新加载，浏览器中输入 fe.sherlocked93.club，发现从二级域名就可以访问到我们刚刚新建的 fe 文件夹：

## 配置反向代理

反向代理是工作中最常用的服务器功能，经常被用来解决跨域问题，下面简单介绍一下如何实现反向代理。
首先进入 Nginx 的主配置文件：

> vim /etc/nginx/nginx.conf

去 http 模块的 server 块中的 location /，增加一行将默认网址重定向到最大学习网站 Bilibili 的 proxy_pass 配置 ：

```conf
server {
        listen       8000;
        server_name  localhost;
        location / {
            proxy_pass http://www.bilibili.com;
        }
    }
```

改完保存退出，nginx -s reload 重新加载，进入默认网址，那么现在就直接跳转到 B 站了，实现了一个简单的代理。
实际使用中，可以将请求转发到本机另一个服务器上，也可以根据访问的路径跳转到不同端口的服务中。
比如我们监听 9001 端口，然后把访问不同路径的请求进行反向代理：

把访问 http://127.0.0.1:9001/edu 的请求转发到 http://127.0.0.1:8080把访问http://127.0.0.1:9001/vod 的请求转发到 http://127.0.0.1:8081
这种要怎么配置呢，首先同样打开主配置文件，然后在 http 模块下增加一个 server 块：

```conf
server {
  listen 9001;
  server_name *.sherlocked93.club;

  location ~ /edu/ {
    proxy_pass http://127.0.0.1:8080;
  }

  location ~ /vod/ {
    proxy_pass http://127.0.0.1:8081;
  }
}
```

反向代理还有一些其他的指令，可以了解一下：

1. proxy_set_header 在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息；
2. proxy_connect_timeout 配置 Nginx 与后端代理服务器尝试建立连接的超时时间；
3. proxy_read_timeout 配置 Nginx 向后端服务器组发出 read 请求后，等待相应的超时时间；
4. proxy_send_timeout 配置 Nginx 向后端服务器组发出 write 请求后，等待相应的超时时间；
5. proxy_redirect 用于修改后端服务器返回的响应头中的 Location 和 Refresh。

## 跨域 CORS 配置

关于简单请求、非简单请求、跨域的概念，前面已经介绍过了，还不了解的可以看看前面的讲解。现在前后端分离的项目一统天下，经常本地起了前端服务，需要访问不同的后端地址，不可避免遇到跨域问题。

要解决跨域问题，我们来制造一个跨域问题。首先和前面设置二级域名的方式一样，先设置好 fe.sherlocked93.club 和 be.sherlocked93.club 二级域名，都指向本云服务器地址，虽然对应 IP 是一样的，但是在 fe.sherlocked93.club 域名发出的请求访问 be.sherlocked93.club 域名的请求还是跨域了，因为访问的 host 不一致（如果不知道啥原因参见前面跨域的内容）。

### 使用反向代理解决跨域

在前端服务地址为 fe.sherlocked93.club 的页面请求 be.sherlocked93.club 的后端服务导致的跨域，可以这样配置：

```conf
server {
  listen 9001;
  server_name fe.sherlocked93.club;

  location / {
    proxy_pass be.sherlocked93.club;
  }
}

```

这样就将对前一个域名 fe.sherlocked93.club 的请求全都代理到了 be.sherlocked93.club，前端的请求都被我们用服务器代理到了后端地址下，绕过了跨域。
这里对静态文件的请求和后端服务的请求都以 fe.sherlocked93.club 开始，不易区分，所以为了实现对后端服务请求的统一转发，通常我们会约定对后端服务的请求加上 /apis/ 前缀或者其他的 path 来和对静态资源的请求加以区分，此时我们可以这样配置：

```conf
# 请求跨域，约定代理后端服务请求path以/apis/开头
location ^~/apis/ {
    # 这里重写了请求，将正则匹配中的第一个分组的path拼接到真正的请求后面，并用break停止后续匹配
    rewrite ^/apis/(.*)$ /$1 break;
    proxy_pass be.sherlocked93.club;

    # 两个域名之间cookie的传递与回写
    proxy_cookie_domain be.sherlocked93.club fe.sherlocked93.club;
}

```

这样，静态资源我们使用 fe.sherlocked93.club/xx.html，动态资源我们使用 fe.sherlocked93.club/apis/getAwo，浏览器页面看起来仍然访问的前端服务器，绕过了浏览器的同源策略，毕竟我们看起来并没有跨域。
也可以统一一点，直接把前后端服务器地址直接都转发到另一个 server.sherlocked93.club，只通过在后面添加的 path 来区分请求的是静态资源还是后端服务，看需求了

### 配置 header 解决跨域

当浏览器在访问跨源的服务器时，也可以在跨域的服务器上直接设置 Nginx，从而前端就可以无感地开发，不用把实际上访问后端的地址改成前端服务的地址，这样可适性更高。
比如前端站点是 fe.sherlocked93.club，这个地址下的前端页面请求 be.sherlocked93.club 下的资源，比如前者的 fe.sherlocked93.club/index.html 内容是这样的：

```conf
server {
  listen       80;
  server_name  be.sherlocked93.club;

	add_header 'Access-Control-Allow-Origin' $http_origin;   # 全局变量获得当前请求origin，带cookie的请求不支持*
	add_header 'Access-Control-Allow-Credentials' 'true';    # 为 true 可带上 cookie
	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';  # 允许请求方法
	add_header 'Access-Control-Allow-Headers' $http_access_control_request_headers;  # 允许请求的 header，可以为 *
	add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';

  if ($request_method = 'OPTIONS') {
		add_header 'Access-Control-Max-Age' 1728000;   # OPTIONS 请求的有效期，在有效期内不用发出另一条预检请求
		add_header 'Content-Type' 'text/plain; charset=utf-8';
		add_header 'Content-Length' 0;

		return 204;                  # 200 也可以
	}

	location / {
		root  D:/Nginx/nginx-1.18.0/html/be;
		index index.html;
	}
}
```

然后 nginx -s reload 重新加载配置。这时再访问 fe.sherlocked93.club/index.html 结果如下，请求中出现了我们刚刚配置的 Header：
解决了跨域问题。

## 开启 gzip 压缩

gzip 是一种常用的网页压缩技术，传输的网页经过 gzip 压缩之后大小通常可以变为原来的一半甚至更小（官网原话），更小的网页体积也就意味着带宽的节约和传输速度的提升，特别是对于访问量巨大大型网站来说，每一个静态资源体积的减小，都会带来可观的流量与带宽的节省。

### Nginx 配置 gzip

使用 gzip 不仅需要 Nginx 配置，浏览器端也需要配合，需要在请求消息头中包含 Accept-Encoding: gzip（IE5 之后所有的浏览器都支持了，是现代浏览器的默认设置）。一般在请求 html 和 css 等静态资源的时候，支持的浏览器在 request 请求静态资源的时候，会加上 Accept-Encoding: gzip 这个 header，表示自己支持 gzip 的压缩方式，Nginx 在拿到这个请求的时候，如果有相应配置，就会返回经过 gzip 压缩过的文件给浏览器，并在 response 相应的时候加上 content-encoding: gzip 来告诉浏览器自己采用的压缩方式（因为浏览器在传给服务器的时候一般还告诉服务器自己支持好几种压缩方式），浏览器拿到压缩的文件后，根据自己的解压方式进行解析。
先来看看 Nginx 怎么进行 gzip 配置，和之前的配置一样，为了方便管理，还是在 /etc/nginx/conf.d/ 文件夹中新建配置文件 gzip.conf：

```conf
#gzip  on;
    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 9;
    gzip_types       text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;
    gzip_disable "MSIE [1-6]\.";
    gzip_vary on;
```

稍微解释一下：

1. gzip_types 要采用 gzip 压缩的 MIME 文件类型，其中 text/html 被系统强制启用；
2. gzip_static 默认 off，该模块启用后，Nginx 首先检查是否存在请求静态文件的 gz 结尾的文件，如果有则直接返回该
3. gz 文件内容；
4. gzip_proxied 默认 off，nginx 做为反向代理时启用，用于设置启用或禁用从代理服务器上收到相应内容 gzip 压缩；
5. gzip_vary 用于在响应消息头中添加
6. Vary Accept-Encoding，使代理服务器根据请求头中的
7. Accept-Encoding 识别是否启用 gzip 压缩；
8. gzip_comp_level gzip 压缩比，压缩级别是 1-9，1 压缩级别最低，9 最高，级别越高压缩率越大，压缩时间越长，建议 4-6；
9. gzip_buffers 获取多少内存用于缓存压缩结果，16 8k 表示以 8k\*16 为单位获得；
10. gzip_min_length 允许压缩的页面最小字节数，页面字节数从 header 头中的
11. Content-Length 中进行获取。默认值是 0，不管页面多大都压缩。建议设置成大于 1k 的字节数，小于 1k 可能会越压越大；
12. gzip_http_version 默认 1.1，启用 gzip 所需的 HTTP 最低版本；

这个配置可以插入到 http 模块整个服务器的配置里，也可以插入到需要使用的虚拟主机的 server 或者下面的 location 模块中，当然像上面我们这样写的话就是被 include 到 http 模块中了。

### Webpack 的 gzip 配置

当前端项目使用 Webpack 进行打包的时候，也可以开启 gzip 压缩：

```js
// vue-cli3 的 vue.config.js 文件
const CompressionWebpackPlugin = require('compression-webpack-plugin')

module.exports = {
  // gzip 配置
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      // 生产环境
      return {
        plugins: [new CompressionWebpackPlugin({
          test: /\.js$|\.html$|\.css/,    // 匹配文件名
          threshold: 10240,               // 文件压缩阈值，对超过10k的进行压缩
          deleteOriginalAssets: false     // 是否删除源文件
        })]
      }
    }
  },
  ...
}
```

## 配置负载均衡

负载均衡在之前已经介绍了相关概念了，主要思想就是把负载均匀合理地分发到多个服务器上，实现压力分流的目的。
主要配置如下：

```conf
http {
  upstream myserver {
  	# ip_hash;  # ip_hash 方式
    # fair;   # fair 方式
    server 127.0.0.1:8081;  # 负载均衡目的服务地址
    server 127.0.0.1:8080;
    server 127.0.0.1:8082 weight=10;  # weight 方式，不写默认为 1
  }

  server {
    location / {
    	proxy_pass http://myserver;
      proxy_connect_timeout 10;
    }
  }
}
```

Nginx 提供了好几种分配方式，默认为轮询，就是轮流来。有以下几种分配方式：
轮询，默认方式，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务挂了，能自动剔除；

1. weight，权重分配，指定轮询几率，权重越高，在被访问的概率越大，用于后端服务器性能不均的情况；
2. ip_hash，每个请求按访问 IP 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决动态网页 session 共享问题。负载均衡每次请求都会重新定位到服务器集群中的某一个，那么已经登录某一个服务器的用户再重新定位到另一个服务器，其登录信息将会丢失，这样显然是不妥的；
3. fair（第三方），按后端服务器的响应时间分配，响应时间短的优先分配，依赖第三方插件 nginx-upstream-fair，需要先安装；

## 配置动静分离

动静分离在之前也介绍过了，就是把动态和静态的请求分开。方式主要有两种，一种 是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案。另外一种方法就是动态跟静态文件混合在一起发布， 通过 Nginx 配置来分开。
通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化。则不会从服务器抓取，返回状态码 304，如果有修改，则直接从服务器重新下载，返回状态码 200。

```conf
server {
  location /www/ {
  	root /data/;
    index index.html index.htm;
  }

  location /image/ {
  	root /data/;
    autoindex on;
  }
}
```

## 适配 PC 或移动设备

```conf
server {
    listen       9000;
	server_name  localhost;
	location / {
        root  D:/Nginx/nginx-1.18.0/html/pc;
        if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
            root  D:/Nginx/nginx-1.18.0/html/mobile;
        }
		index  index.html index.htm;
	}
}
```

## 配置 HTTPS

具体配置过程网上挺多的了，也可以使用你购买的某某云，一般都会有免费申请的服务器证书，安装直接看所在云的操作指南即可。
我购买的腾讯云提供的亚洲诚信机构颁发的免费证书只能一个域名使用，二级域名什么的需要另外申请，但是申请审批比较快，一般几分钟就能成功，然后下载证书的压缩文件，里面有个 nginx 文件夹，把 xxx.crt 和 xxx.key 文件拷贝到服务器目录，再配置下：

```
server {
  listen 443 ssl http2 default_server;   # SSL 访问端口号为 443
  server_name sherlocked93.club;         # 填写绑定证书的域名

  ssl_certificate /etc/nginx/https/1_sherlocked93.club_bundle.crt;   # 证书文件地址
  ssl_certificate_key /etc/nginx/https/2_sherlocked93.club.key;      # 私钥文件地址
  ssl_session_timeout 10m;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;      #请按照以下协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
  ssl_prefer_server_ciphers on;

  location / {
    root         D:/Nginx/nginx-1.18.0/html;
    index        index.html index.htm;
  }
}
```

写完 nginx -t -q 校验一下，没问题就 nginx -s reload，现在去访问 https://sherlocked93.club/ 就能访问 HTTPS 版的网站了。
一般还可以加上几个增强安全性的命令：

> add_header X-Frame-Options DENY; # 减少点击劫持

> add_header X-Content-Type-Options nosniff; # 禁止服务器自动解析资源类型

> add_header X-Xss-Protection 1; # 防 XSS 攻击

## 一些常用技巧

### 静态服务

```conf
server {
  listen       80;
  server_name  static.sherlocked93.club;
  charset utf-8;    # 防止中文文件名乱码

  location /download {
    alias	          D:/Nginx/nginx-1.18.0/html/static;  # 静态资源目录

    autoindex               on;    # 开启静态资源列目录
    autoindex_exact_size    off;   # on(默认)显示文件的确切大小，单位是byte；off显示文件大概大小，单位KB、MB、GB
    autoindex_localtime     off;   # off(默认)时显示的文件时间为GMT时间；on显示的文件时间为服务器时间
  }
}
```

### 图片防盗链

```conf
server {
  listen       80;
  server_name  *.sherlocked93.club;

  # 图片防盗链
  location ~* \.(gif|jpg|jpeg|png|bmp|swf)$ {
    valid_referers none blocked 192.168.0.2;  # 只允许本机 IP 外链引用
    if ($invalid_referer){
      return 403;
    }
  }
}
```

### 请求过滤

```conf
# 非指定请求全返回 403
if ( $request_method !~ ^(GET|POST|HEAD)$ ) {
  return 403;
}

location / {
  # IP访问限制（只允许IP是 192.168.0.2 机器访问）
  allow 192.168.0.2;
  deny all;

  root   html;
  index  index.html index.htm;
}
```

### 配置图片、字体等静态文件缓存

由于图片、字体、音频、视频等静态文件在打包的时候通常会增加了 hash，所以缓存可以设置的长一点，先设置强制缓存，再设置协商缓存；如果存在没有 hash 值的静态文件，建议不设置强制缓存，仅通过协商缓存判断是否需要使用缓存。

```conf
# 图片缓存时间设置
location ~ .*\.(css|js|jpg|png|gif|swf|woff|woff2|eot|svg|ttf|otf|mp3|m4a|aac|txt)$ {
	expires 10d;
}

# 如果不希望缓存
expires -1;
```

### 单页面项目 history 路由配置

```conf
server {
  listen       80;
  server_name  fe.sherlocked93.club;

  location / {
    root       D:/Nginx/nginx-1.18.0/html/dist;  # vue 打包后的文件夹
    index      index.html index.htm;
    try_files  $uri $uri/ /index.html @rewrites;

    expires -1;                          # 首页一般没有强制缓存
    add_header Cache-Control no-cache;
  }

  # 接口转发，如果需要的话
  #location ~ ^/api {
  #  proxy_pass http://be.sherlocked93.club;
  #}

  location @rewrites {
    rewrite ^(.+)$ /index.html break;
  }
}
```

### HTTP 请求转发到 HTTPS

配置完 HTTPS 后，浏览器还是可以访问 HTTP 的地址 http://sherlocked93.club/ 的，可以做一个 301 跳转，把对应域名的 HTTP 请求重定向到 HTTPS 上

```conf
server {
    listen      80;
    server_name www.sherlocked93.club;

    # 单域名重定向
    if ($host = 'www.sherlocked93.club'){
        return 301 https://www.sherlocked93.club$request_uri;
    }
    # 全局非 https 协议时重定向
    if ($scheme != 'https') {
        return 301 https://$server_name$request_uri;
    }

    # 或者全部重定向
    return 301 https://$server_name$request_uri;

    # 以上配置选择自己需要的即可，不用全部加
}
```

### 泛域名路径分离

这是一个非常实用的技能，经常有时候我们可能需要配置一些二级或者三级域名，希望通过 Nginx 自动指向对应目录，比如：
test1.doc.sherlocked93.club 自动指向 D:/Nginx/nginx-1.18.0/html/doc/test1 服务器地址；test2.doc.sherlocked93.club 自动指向 D:/Nginx/nginx-1.18.0/html/doc/test2 服务器地址；

```conf
server {
    listen       80;
    server_name  ~^([\w-]+)\.doc\.sherlocked93\.club$;

    root D:/Nginx/nginx-1.18.0/html/doc/$1;
}
```

### 泛域名转发

和之前的功能类似，有时候我们希望把二级或者三级域名链接重写到我们希望的路径，让后端就可以根据路由解析不同的规则：
test1.serv.sherlocked93.club/api?name=a 自动转发到 127.0.0.1:8080/test1/api?name=a；test2.serv.sherlocked93.club/api?name=a 自动转发到 127.0.0.1:8080/test2/api?name=a；

```conf
server {
    listen       80;
    server_name ~^([\w-]+)\.serv\.sherlocked93\.club$;

    location / {
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        Host $http_host;
        proxy_set_header        X-NginX-Proxy true;
        proxy_pass              http://127.0.0.1:8080/$1$request_uri;
    }
}
```

> 参考文档：

- Nginx 中文文档
- Nginx 安装，目录结构与配置文件详解
- Keepalived 安装与配置
- Keepalived+Nginx 实现高可用
- Nginx 与前端开发
- 跨域资源共享 CORS 详解 - 阮一峰的网络日志
- 前端开发者必备的 nginx 知识
- 我也说说 Nginx 解决前端跨域问题，正确的 Nginx 跨域配置
- vue-router history 模式 nginx 配置并配置静态资源缓存 | HolidayPenguin
- nginx 重定向，全局 https，SSL 配置，反代配置参考
- Nginx 入门教程
