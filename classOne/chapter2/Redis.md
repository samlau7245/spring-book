# Redis 基础

<img src="/assets/images/classOne/cp1/97.png"/>

* `Redis`
* `NoSql` Not Only Sql，高性能读取、高可用、存数据、做缓存。

<img src="/assets/images/classOne/cp1/91.png"/>

## 分布式缓存

`NoSql`就是分布式缓存的中间件。

### 分布式缓存方案对比

* `EHCache`： 基于JAVA开发，基于JVM缓存；缺点是不支持缓存共享，一般用于但应用。
* `MemCache`：优点：多线程。缺点：无法缓存，无法持久化。
* `Redis` ： 优点：可持久化。缺点：单线程。

## 安装Redis

[Redis Download](https://redis.io/download)

* 解压

```sh
cd /home/software
tar -xvf redis-6.0.9.tar
cd redis-6.0.9
```

* 安装GCC编译环境：`yum install gcc-c++`
* 编译、安装： `make && make install`

> **[warning] 如果make过程中遇到警告或者报错，有可能是GCC版本比较低，可以升级下版本**
>
```sh
gcc -v
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
```

* 配置Redis开机自启动

<img src="/assets/images/classOne/cp1/92.png"/>

* 配置Redis文件

```sh
# 创建Redis文件夹，用于存放Redis配置文件
mkdir /usr/local/redis
# 拷贝 redis.conf 到 /usr/local/redis 文件夹中
cp /home/software/redis-6.0.9/redis.conf /usr/local/redis/
```

* 修改Redis核心配置文件`/usr/local/redis/redis.conf`

```
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes # yes 为了让redis启动在Linux后台运行

# Note that you must specify a directory here, not a file name.
dir /usr/local/redis/working # 修改Redis的工作目录，数据库将会被写在这个工作目录中

# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT OUT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# bind 127.0.0.1
bind 0.0.0.0 # 0.0.0.0 表示可以让远程连接，不受IP限制

# IMPORTANT NOTE: starting with Redis 6 "requirepass" is just a compatibility
# layer on top of the new ACL system. The option effect will be just setting
# the password for the default user. Clients will still authenticate using
# AUTH <password> as usually, or more explicitly with AUTH default <password>
# if they follow the new protocol: both will work.
#
# requirepass foobared
requirepass imooc # redis用户密码，默认是没有设置的，这个在线上一定要设置一个一定复杂度的密码
```

* 修改`/etc/init.d/redis_init_script`redis启动脚本

<img src="/assets/images/classOne/cp1/93.png"/>

修改`/usr/local/redis/redis.conf`文件名保持和配置一致`/usr/local/redis/6379.conf`

<img src="/assets/images/classOne/cp1/96.png"/>

* 修改`/etc/init.d/redis_init_script`redis启动脚本的执行权限、启动redis

```sh
pwd
# /etc/init.d
chmod 777 redis_init_script
./redis_init_script start
```

<img src="/assets/images/classOne/cp1/94.png"/>

* 设置开机启动,修改`/etc/init.d/redis_init_script`redis启动脚本

```sh
#chkconfig: 22345 10 90
#description: Start and Stop redis
```

<img src="/assets/images/classOne/cp1/95.png"/>

执行: `chkconfig redis_init_script on`

## Redis命令行客户端基本使用

## Redis数据类型

### string
### hash
### list
### set
### zset

# SpringBoot整合Redis实战

# Redis进阶提升与主从复制

# Redis 哨兵机制与实现

# Redis集群

# Redis缓存雪崩方案

# Redis批量查询的优化设计




































