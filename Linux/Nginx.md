# 简介

反向代理 实现集群、负载均衡

## 正向代理

<img src="/assets/images/useage/19.png"/>

## 反向代理

<img src="/assets/images/useage/20.png"/>

### 路由

Nginx捕获到域名中的路由，然后根据逻辑处理转到不同节点服务器。

<img src="/assets/images/useage/21.png"/>

# 安装使用

安装依赖环境：

```sh
# gcc环境
$ yum install gcc-c++
# PCRE库，用于解析正则表达式
$ yum install -y pcre pcre-devel
# 安装zlib压缩和解压
$ yum install -y zlib zlib-devel
# SSL协议，用于HTTP安全传输，也就是HTTPS
yum install -y openssl openssl-devel
```

nginx 安装：

```sh
$ pwd
# /home/software
# 解压
$ tar -zxvf nginx-1.18.0.tar.gz
# 编译之前，先创建nginx临时目录，如果不创建，在启动nginx的过程中会报错
$ mkdir /var/temp/nginx -p
```

到[官网](http://nginx.org/en/download.html)下载稳定版本的nginx，上传到Linux服务器中。

<img src="/assets/images/classTwo/01.png"/>

配置nginx，目的为了创建makefile文件：

```
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```

参数含义说明：

```sh
./configure
# 指定nginx安装目录
--prefix=/usr/local/nginx
# 指定nginx的PID
--pid-path=/var/run/nginx/nginx.pid
# 锁定安装文件，防止被恶意串改或误操作
--lock-path=/var/lock/nginx.lock
# 错误日志
--error-log-path=/var/log/nginx/error.log
# http日志
--http-log-path=/var/log/nginx/access.log
# 启用gzip模块，在线实时压缩输出数据流
--with-http_gzip_static_module
# 设定客户端请求的临时目录
--http-client-body-temp-path=/var/temp/nginx/client
# 设定http代理临时目录
--http-proxy-temp-path=/var/temp/nginx/proxy
# 设定fastcgi代理临时目录
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi
# 设定uwsgi代理临时目录
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi
# 设定scgi代理临时目录
--http-scgi-temp-path=/var/temp/nginx/scgi
```

<img src="/assets/images/useage/22.png"/>

编译、安装nginx：

```sh
# make编译
$ make
# 安装
$ make install
# 查看安装路径
$ whereis nginx
# nginx: /usr/local/nginx /usr/local/nginx/sbin/nginx.old /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak

# 进入sbin目录启动nginx
$ cd /usr/local/nginx/sbin
$ ./nginx
```

文件目录结构:

```
|-- client_body_temp
|-- conf
|   |-- 1_msnewlifefitness.com_bundle.crt
|   |-- 2_msnewlifefitness.com.key
|   |-- fastcgi.conf
|   |-- fastcgi.conf.default
|   |-- fastcgi_params
|   |-- fastcgi_params.default
|   |-- koi-utf
|   |-- koi-win
|   |-- mime.types
|   |-- mime.types.default
|   |-- nginx.conf // nginx主要配置文件
|   |-- nginx.conf.default
|   |-- scgi_params
|   |-- scgi_params.default
|   |-- uwsgi_params
|   |-- uwsgi_params.default
|   `-- win-utf
|-- fastcgi_temp
|-- html // 静态资源文件夹。
|   |-- 50x.html
|   `-- index.html
|-- logs
|   |-- access.log
|   |-- error.log
|   `-- nginx.pid
|-- proxy_temp
|-- sbin
|   |-- nginx 
|   |-- nginx.bak
|   `-- nginx.old
|-- scgi_temp
`-- uwsgi_temp
```

> **[warning] 注意**
>
> * 云服务安装，防火墙要打开nginx默认端口：80
> * 本地虚拟机，关闭防火墙。
> * 本地WIN、MAC，关闭防火墙。

## nginx其他命令

```sh
# 停止
$ ./nginx -s stop
# 重新加载
$ ./nginx -s reload
```

# 默认显示路径解析

当浏览器访问:`http://123.456.78.910` 的时候，默认`http://123.456.78.910:80/index.html`。

<img src="/assets/images/useage/25.png"/>

# 进程模型

<img src="/assets/images/useage/26.png"/>

```sh
$ ps -ef|grep nginx
# root      3887     1  0 12:29 ?        00:00:00 nginx: master process ./nginx
# nobody    3888  3887  0 12:29 ?        00:00:00 nginx: worker process
```

* `master进程` : 主进程，下发工作给worker并且监控worker。
* `worker进程` : 工作进程。默认只有一个，可以配置worker进程的数量。

<img src="/assets/images/useage/31.png"/>

<img src="/assets/images/useage/27.png"/>

比如：Nginx开启了3个worker进程；3个worker抢占机制：3个worker抢占1个client，会通过互斥锁(accept_muet)保证任务执行唯一，并且防止任务阻塞。

<img src="/assets/images/useage/28.png"/>

当有多个client来请求worker1进程时，因为Nginx是一个异步的，非阻塞的，所以当worker1在处理的时候发现Client1阻塞了以后，会通过`epoll`模型的时间机制来处理北阻塞的Client2、Client3。

<img src="/assets/images/useage/30.png"/>

设置worker限制最大并发数：

<img src="/assets/images/useage/29.png"/>

# nginx.conf

<img src="/assets/images/useage/32.png"/>

* `worker_processes  3;` 这就是一条指令，是在main中的。
* `events{worker_connections  1024;}` 为指令快。
* 指令结尾用 `;` 结束。
* 注释使用`#`。
* 变量、参数使用`$`（例如:`log_format  main  '$remote_addr`）。
* `error_page   500 502 503 504  /50x.html;` : `error_page`为指令，`500 502 503 504  /50x.html`是指令的值，每个值中间用空格或者TAB分隔。


```sh
#user    nobody; # 设置worker进程的用户，指的是Linux中的用户，会涉及到Nginx操作目录或者文件的一些权限，默认 nobody 
#user    root;
# ================= 在Linux中查看 worker user ================= 
# ps -ef|grep nginx
# root     19352     1  0 14:11 ?        00:00:00 nginx: master process ./nginx
# nobody   19353 19352  0 14:11 ?        00:00:00 nginx: worker process
# nobody   19354 19352  0 14:11 ?        00:00:00 nginx: worker process
# nobody   19355 19352  0 14:11 ?        00:00:00 nginx: worker process
# ================= 在Linux中查看 worker user ================= 

worker_processes  3; # worker进程数量。一般是CPU有几个就设置几个，或者 N-1 也行。

# Nginx日志。 日志级别 : debug|info|notice|warn|error|crit|alert|emerg，左->右越来越大。
#error_log  logs/error.log; # 这里的日志路径 `logs/error.log`其实在安装时候设置`./configure`时就已经指定了日志路径`--error-log-path`
#error_log  logs/error.log  notice; # notice 日志级别
#error_log  logs/error.log  info; # info 日志级别

#pid        logs/nginx.pid; # 设置nginx进程PID

# nginx的工作模式
events {
    use epoll; # 默认 epoll
    worker_connections  1024; # 每个worker允许连接的客户端最大连接数，默认 1024。
}

http { # http指令块。针对HTTP网络传输的一些指令设置。
    
    # /usr/local/nginx/conf 两个文件在同级目录中
    # |-- mime.types
    # |-- nginx.conf
    include       mime.types; # include 是引入外部配置，提高可读性，避免配置文件过大。
    default_type  application/octet-stream;

    # 设置日志格式。
    # main 为定义的格式名称。这样 access_log 就可以直接使用 log_format 这个变量。
    # $remote_addr 客户端ip
    # $remote_user 远程客户端用户名，一般为：’-’
    # $time_local 时间和时区
    # $request 请求的url以及method
    # $status 响应状态码
    # $body_bytes_sent 响应客户端内容字节数
    # $http_referer 记录用户从哪个链接跳转过来的
    # $http_user_agent 用户所使用的代理，一般来时都是浏览器
    # $http_x_forwarded_for 通过代理服务器来记录客户端的ip
    # ======================================================
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  logs/access.log  main;

    sendfile        on; # 使用高效文件传输，提升传输性能。
    #tcp_nopush     on; # 指当数据表累积一定大小后才发送，提高了效率，和sendfile一起搭配使用，只有在sendfile启用后才能使用tcp_nopush。[就像外卖员先把所有的外卖拿到然后再送，比拿一个送一个更高效]

    #keepalive_timeout  0;  
    keepalive_timeout  65; # 设置客户端与服务端请求的超时时间，保证客户端多次请求的时候不会重复建立新的连接，节约资源损耗。

    #gzip  on; # 启用压缩，html/js/css压缩后传输会更快。

    server { # 可以在http指令块中设置多个虚拟主机
        listen       80; # 监听端口
        server_name  localhost; # localhost、ip、域名
        location / { # 请求路由映射，匹配拦截
            root   html; # root 请求位置
            index  index.html index.htm; # index 首页设置
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## 多server配置抽离

新配置一个服务，监听端口`89`，注意从防火墙和安全组中把端口开放出来。

```sh
worker_processes  2;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       89;
        server_name  localhost;
        location / {
            root   html;
            index  imooc.html imooc.htm;
        }
    }
}
```

可以把新增的server抽离出去然后通过`include`方式导入。

<img src="/assets/images/useage/33.png"/>

这样重新启动Nginx就直接可以访问89端口服务了。

# 资料

* [CentOS官网](https://www.centos.org/)
* [nginx 官网 ](http://nginx.org/)
* [nginx download](http://nginx.org/en/download.html)




































