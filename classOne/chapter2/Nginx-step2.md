# 跨域配置支持

<img src="/assets/images/classOne/cp1/34.png"/>

<img src="/assets/images/classOne/cp1/35.png"/>

<img src="/assets/images/classOne/cp1/36.png"/>

```
#允许跨域请求的域，*代表所有
add_header 'Access-Control-Allow-Origin' *;
#允许带上cookie请求
add_header 'Access-Control-Allow-Credentials' 'true';
#允许请求的方法，比如 GET/POST/PUT/DELETE
add_header 'Access-Control-Allow-Methods' *;
#允许请求的header
add_header 'Access-Control-Allow-Headers' *;
```

# 静态资源防盗链

```
#对源站点验证
valid_referers *.imooc.com; 
#非法引入会进入下方判断
if ($invalid_referer) {
    return 404;
} 
```

<img src="/assets/images/classOne/cp1/37.png"/>

# 模块化设计

<img src="/assets/images/classOne/cp1/38.png"/>

* `event module` : 事件处理器。
* `phase handler` : 处理客户端请求，处理响应。
* `output filter` : 过滤器，过滤一些数据。
* `upstream` : 反响代理模块。
* `load balcancer` :  负载均衡器。
* `extend module` : extend 继承模块，实现第三方模块。


```
源码包目录解析：
|-- auto 自动检测识别操作系统
|-- CHANGES 历史版本号
|-- CHANGES.ru 俄罗斯的版本
|-- conf 配置
|-- configure 编译配置
|-- contrib 工具包
|-- html 默认存放的静态文件
|-- LICENSE 
|-- Makefile
|-- man 
|-- objs 第三方模块存放地点
|-- README
`-- src 源码
    |-- core
    |-- event
    |-- http http模块
    |-- mail mail模块
    |-- misc 辅助代码
    |-- os 操作系统
    `-- stream 
```

# 集群负载均衡

<img src="/assets/images/classOne/cp1/39.png"/>

# 搭建3台Tomcat集群

<img src="/assets/images/classOne/cp1/40.png"/>

编辑`imooc.conf`:

```
server {
    listen       89;
    server_name  localhost;
    location /imooc {
        root   /home;
    }
}

# 配置上游服务器
upstream tomcats {
    server 192.168.55.129:8080;
    server 192.168.55.130:8080;
    server 192.168.55.131:8080;
}

server {
    listen       88;
    server_name  www.test.com;

    location / {
        proxy_pass http://tomcats; # 代理
    }
}
```

本地测试域名可以修改本地`hosts`：

<img src="/assets/images/classOne/cp1/41.png"/>

访问网页：`http://www.test.com:88/`

<img src="/assets/images/classOne/cp1/42.png"/>


# 用JMeter测试单节点与集群的并发异常率

[JMeter](https://jmeter.apache.org/)可以用来测试网站的性能。

<img src="/assets/images/classOne/cp1/44.png"/>

<img src="/assets/images/classOne/cp1/43.png"/>

> 1. 看`聚合报告`中，如果异常率超过公司限定的百分比，就应该考虑扩容了。

# 负载均衡

## 轮询

当每个云服务器的配置一样的情况下，负载均衡可以采用轮询(默认就是轮询)的形式。

## 权重

当每个云服务器的配置不一样的情况下，负载均衡可以根据不同的配置进行权重配置；配置好的服务器进行加权，反之减少权重。

编辑`imooc.conf`:

```
server {
    listen       89;
    server_name  localhost;
    location /imooc {
        root   /home;
    }
}

# 配置上游服务器
upstream tomcats {
    # weight 就是权重配置，默认值就是1。这样如果有6个请求的话其中3个请求会在131服务器，2个请求会在130服务器，最后1个会在129服务器。
    server 192.168.55.129:8080 weight=1; # 129服务器配置最差
    server 192.168.55.130:8080 weight=2; # 130服务器配置较好
    server 192.168.55.131:8080 weight=3; # 131服务器配置最好
}

server {
    listen       88;
    server_name  www.test.com;

    location / {
        proxy_pass http://tomcats; # 代理
    }
}
```

## ip_hash

`ip_hash` 可以保证用户访问可以请求到上游服务中的固定的服务器，前提是用户IP没有发生更改。**不能把后台服务器直接移除，只能标记。**

```
upstream tomcats {
    ip_hash;
    server 192.168.55.129:8080;
    server 192.168.55.130:8080;
    server 192.168.55.131:8080 down; 
}
```

## url_hash

## least_conn


# upstream

## max_conns 限制最大连接数

`max_conns` 限制每台server的连接数，用于保护避免过载，可起到限流作用。

```
upstream tomcats {
    server 192.168.55.129:8080 max_conns=2; 
    server 192.168.55.130:8080 max_conns=2; 
    server 192.168.55.131:8080 max_conns=2; 
}
```

## slow_start 

* `slow_start` 缓慢启动，让服务器慢慢的加入到集群中。
* `slow_start` 不能使用在`hash`和`random load balancing`中。
* 如果在`upstream`中只有一台server，则`slow_start`参数无效。

```
upstream tomcats {
    server 192.168.55.129:8080 weight=6 slow_start=60s; 
    server 192.168.55.130:8080 weight=2; 
    server 192.168.55.131:8080 weight=2; 
}
```

## down、backup

* `down` 用于标记服务节点不可用。

```
upstream tomcats {
    server 192.168.55.129:8080 down; 
    server 192.168.55.130:8080 weight=1; 
    server 192.168.55.131:8080 weight=1; 
}
```

* `backup` 表示当前服务器节点为备用机。只有在其他服务器都宕机之后才会加入到集群中。
* `backup` 不能使用在`hash`和`random load balancing`中。

```
upstream tomcats {
    server 192.168.55.129:8080 backup; 
    server 192.168.55.130:8080 weight=1; 
    server 192.168.55.131:8080 weight=1; 
}
```

## max_fails、fail_timeout

* `max_fails` 表示失败几次，则标记server已宕机，踢出上游服务。
* `fail_timeout` 表示失败重试的时间。

```
upstream tomcats {
    server 192.168.55.129:8080 max_fails=2 fail_timeout=15s;
    server 192.168.55.130:8080 weight=1; 
    server 192.168.55.131:8080 weight=1; 
}
```

`max_fails=2 fail_timeout=15s;` 代表在15秒内请求某一server失败达到2次后，则认为该server已经挂了或者宕机了，随后再过15秒，这15秒内不会有新的请求到达刚刚挂掉的节点上，而是会请求到正常运作的server，15秒后会再有新请求尝试连接挂掉的server，如果还是失败，重复上一过程，直到恢复。


# Keepalived 提高吞吐量

* [keepalive 官网说明](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive)
* `keepalive 32;` 设置连接处理的数量。
* `proxy_http_version 1.1;` 设置长连接HTTP版本号为1.1。
* `proxy_set_header Connection "";` 清除 Connect Header 信息。

```
server {
    listen       89;
    server_name  localhost;
    location /imooc {
        root   /home;
    }
}

# 配置上游服务器
upstream tomcats {
    server 192.168.55.129:8080;
    server 192.168.55.130:8080;
    server 192.168.55.131:8080;

    keepalive 32;
}

server {
    listen       88;
    server_name  www.test.com;

    location / {
        proxy_pass http://tomcats; # 代理
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

# Nginx缓存

<img src="/assets/images/classOne/cp1/45.png"/>

## 浏览器缓存

`浏览器缓存` : 加速用户访问。提升单个用户体验。

```
server {
    listen       89;
    server_name  localhost;
    location /imooc {
        root   /home;

        # expires 10s; 缓存时间为10秒,
        # expires @22h30m;
        # expires -1h;
        # expires epoch;
        # expires off;
        # expires max;
    }
}
```

* 格式： `expires [time]`，示例： `expires 10s`，含义：允许浏览器缓存该资源10秒。

<img src="/assets/images/classOne/cp1/46.png"/>

* 格式： `expires @[time]`，示例：`expires @22h30m`，含义：缓存到晚上22:30就到期了。

<img src="/assets/images/classOne/cp1/46.png"/>

* 格式： `expires -[time]`，示例：`expires -1h;`，含义：缓存提前过期，距离现在的1H之前就过期了。

* `expires epoch;` 不设置cache。

<img src="/assets/images/classOne/cp1/48.png"/>

* `expires off;` 默认缓存时间。

<img src="/assets/images/classOne/cp1/49.png"/>

* `expires max;` 最大缓存时间。

<img src="/assets/images/classOne/cp1/50.png"/>



## Nginx缓存

`Nginx缓存` : 缓存在Nginx端，提升所有访问到该Nginx的用户体验；提示访问`上游(upstream)`服务器的速度；用户访问仍然会产生请求流量。

```
# 配置上游服务器
upstream tomcats {
    server 192.168.55.129:8080;
    server 192.168.55.130:8080;
    server 192.168.55.131:8080;
}

# 设置缓存目录
#     keys_zone 设置共享内存以及占用空间大小； `mycache:5m` 内存空间的名字为 mycache，内存空间可以使用大小为 5M。
#     max_size 设置缓存大小 
#     inactive 超过此时间则被清理；`inactive=1m` 超过1分钟，缓存会被自动清理。
#     use_temp_path 临时目录，使用后会影响nginx性能； `use_temp_path=off` 关闭临时目录。
proxy_cache_path /usr/local/nginx/upstream_cache keys_zone=mycache:5m max_size=1g inactive=1m use_temp_path=off;

server {
    listen       88;
    server_name  www.test.com;

    location / {
        proxy_pass http://tomcats; # 代理

        proxy_cache mycache; # 启用缓存，和keys_zone一致
        proxy_cache_valid   200 304 8h; # 针对200和304状态码缓存时间为8小时
    }
}
```

# Nginx配置HTTPS域名证书

## 腾讯云

[腾讯云- Nginx 服务器证书安装](https://cloud.tencent.com/document/product/400/35244)

### 安装SSL模块

<img src="/assets/images/classOne/cp1/51.png"/>

<img src="/assets/images/classOne/cp1/52.png"/>

安装SSL模块(`http_ssl_module`)。

```sh
# 安装前先停掉Nginx服务
/usr/local/nginx/sbin/nginx -s stop
# cd到源项目
cd /home/software/nginx-1.18.0/
# 新增SSL模块
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
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-http_ssl_module
# 编译
make
# 安装
make install
```

把下载的SSL证书上传到云服务器`/usr/local/nginx/conf`: 

<img src="/assets/images/classOne/cp1/53.png"/>

### 配置HTTPS

`.conf`配置文件新增一个server:

```
server {
   listen 443 ssl; #SSL 访问端口号为 443
   server_name  imoocdsp.com; #绑定证书的域名

   # 配置ssl证书
   ssl_certificate      1_www.imoocdsp.com_bundle.crt;
   # 配置证书秘钥
   ssl_certificate_key  2_www.imoocdsp.com.key;

   # ssl会话cache
   ssl_session_cache    shared:SSL:1m;
   # ssl会话超时时间
   ssl_session_timeout  5m;

   # 配置加密套件，写法遵循 openssl 标准,固定写法
   ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
   ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
   ssl_prefer_server_ciphers on;
   
   location / {
       proxy_pass http://tomcats/;
       index  index.html index.htm;
   }
}
```

> Nginx 版本为 nginx/1.15.0 以上请使用 listen 443 ssl 代替 listen 443 和 ssl on。

重新启动Nginx服务：

```sh
./nginx -s reload
# 查看启动状态
./nginx -V
```

访问：`https://imoocdsp.com`，如果访问不了注意看下防火墙和安全组是否把端口开放出来。

如果遇到如下报错：

```
the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf:122
```

基本就是重新`make`的代码没有覆盖旧的`nginx`，下面是解决：

* 备份`nginx.conf`。(必要时可删除`/usr/local/nginx`文件夹)
* 关闭Nginx : `/usr/local/nginx/sbin/nginx -s stop` 
* 重新make：

```sh
cd /home/software/nginx-1.18.0/

# 配置 configure
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
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-http_ssl_module

# 将刚刚编译好的nginx覆盖掉原有的nginx （这个时候nginx要停止状态）
/usr/local/nginx/sbin/nginx -s stop
cp ./objs/nginx /usr/local/nginx/sbin/

# 启动nginx
/usr/local/nginx/sbin/nginx
# 查看安装
/usr/local/nginx/sbin/nginx -V
```

# 部署Nginx到云端(动静分离)

* 静态数据：`css、js、html、images、audios、videos...`。
* 动态数据：API接口。

动静分离特点：

* 分布式：就是把动态API接口、静态资源给分开，基本就是一种分布式的解决方案。
* 前后端解耦：
* 静态归Nginx管理
* 接口服务化。可以为各平台提供服务。

动静分离实现方式：

* CDN： 域名 -> DNS解析出IP -> 访问IP指定服务器中的资源。

<img src="/assets/images/classOne/cp1/54.png"/>

* Nginx：

<img src="/assets/images/classOne/cp1/55.png"/>

动静分离实现问题：

* 跨域问题： 可以通过`SpringBoot`、`Nginx`、`jsonp`去解决。
* 分布式会话：在集群中的每一次请求都要保证用户的会话都是他去发起的，可以通过`Redis`去解决。

# 测试与日志调试

```sh
# cd到Tomcat-api 目录
cd cd /usr/local/tomcat-api/
# tail 从最后一行往上看，相当于打开文件不会持续监听。
tail logs/catalina.out

# 持续监听日志，相当于控制台
tail logs/catalina.out -f
```

# Nginx增加简单的用户鉴权

```sh
# 安装http密码工具
yum install httpd-tools
# 创建密码库目录
mkdir /usr/local/nginx/db
# 创建用户及密码
htpasswd -c /usr/local/nginx/db/passwd.db usrname
```

```
location / {
    ...    
    auth_basic "secret";
    auth_basic_user_file /usr/local/nginx/db/passwd.db;
    ...
}
```

```sh
# 添加新的用户名、密码到 hotel 数据库
htpasswd -b  hotel.db root admin
```































































