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

>* [Redis Download](https://redis.io/download)
>* “Redis Desktop Manager” Redis数据库的界面客户端

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

* 查看redis是否存活：`redis-cli -a password ping`

<img src="/assets/images/classOne/cp1/98.png"/>

* 关闭redis： `redis-cli -a password shutdown`

* 通过脚本关闭redis： `./redis_init_script stop`

<img src="/assets/images/classOne/cp1/99.png"/>

* 进入到redis客户端 `redis-cli`
* 输入用户密码： `auth pwd`
* 设置缓存： `set key value`
* 获取缓存： `get key`
* 删除缓存： `del key`
* 查看所有的key (不建议在生产上使用，有性能影响)： `keys *`
* key的类型：`type key`
* 切换数据库，总共默认16个： `select index`
* 删除当前下边db中的数据： `flushdb`
* 删除所有db中的数据： `flushall`

## Redis数据类型

### string

* `get、set、del` ：查询、设置、删除, `set name imooc`
* `set rekey data` ：设置已经存在的key，会覆盖 `set name imooc1`
* `setnx rekey data` ：设置已经存在的key，不会覆盖 `setnx name imooc2`
* `set key value ex time` ：设置带过期时间的数据 `set name imooc ex 20` 过期时间 20秒
* `expire key time` ：设置过期时间 `expire name 30` 过期时间30秒
* `ttl key` ：(time to leave)查看剩余时间，-1永不过期，-2过期 `ttl name` 
* `append key` ：合并字符串
* `strlen key` ：字符串长度
* `incr key` ：累加1
* `decr key` ：类减1
* `incrby key num` ：累加给定数值
* `decrby key num` ：累减给定数值
* `getrange key start end` ：截取数据，end=-1 代表到最后
* `setrange key start newdata` ：从start位置开始替换数据
* `mset` ：连续设值 
* `mget` ：连续取值
* `msetnx` ：连续设置，如果存在则不设置

```
127.0.0.1:6379> ttl name
(integer) -1
127.0.0.1:6379> set name imooc ex 20
OK
127.0.0.1:6379> ttl name
(integer) 18
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> keys *
(empty array)

127.0.0.1:6379> get name
"imooc"
127.0.0.1:6379> append name 123
(integer) 8
127.0.0.1:6379> get name
"imooc123"

127.0.0.1:6379> get name
"imooc"
127.0.0.1:6379> strlen name
(integer) 5

127.0.0.1:6379> set age 18
OK
127.0.0.1:6379> get age
"18"
127.0.0.1:6379> incr age
(integer) 19
127.0.0.1:6379> get age
"19"
127.0.0.1:6379> decr age
(integer) 18
127.0.0.1:6379> get age
"18"

127.0.0.1:6379> get age
"18"
127.0.0.1:6379> incrby age 10
(integer) 28
127.0.0.1:6379> decrby age 10
(integer) 18

127.0.0.1:6379> get name
"imooc"
127.0.0.1:6379> getrange name 0 -1
"imooc"
127.0.0.1:6379> getrange name 1 2
"mo"

127.0.0.1:6379> get name
"imooc"
127.0.0.1:6379> setrange name 1 AAA
(integer) 5
127.0.0.1:6379> get name
"iAAAc"

127.0.0.1:6379> mset k1 a1 k2 a2 k3 a3
OK
127.0.0.1:6379> mget k1 k2 k3
1) "a1"
2) "a2"
3) "a3"
127.0.0.1:6379> keys *
1) "age"
2) "k3"
3) "k1"
4) "k2"
5) "name"
```

### hash

`hash` ：类似map，存储结构化数据结构，比如存储一个对象（不能有嵌套对象）。

* `hset user name imooc` : 创建一个user对象，这个对象中包含name属性，name值为imooc
* `hget user name`：获得user对象中name的值
* `hmset`：设置对象中的多个键值对。`hmset user age 18 phone 139123123`
* `hmget`：获得对象中的多个属性。`hmget user age phone`
* `hgetall user`：获得整个user对象的内容
* `hincrby user age 2`：累加属性
* `hincrbyfloat user age 2.2`：累加属性
* `hlen user`：获得user对象有多少个属性
* `hexists user age`：判断user对象中属性age是否存在
* `hkeys user`：获得user对象中的所有属性
* `hvals user`：获得user对象中的所有值
* `hdel user`：删除对象

```
127.0.0.1:6379> hget user name
"imooc"
127.0.0.1:6379> hmset user age 18 phone 139123123
OK
127.0.0.1:6379> hmsetnx user age 18
(error) ERR unknown command `hmsetnx`, with args beginning with: `user`, `age`, `18`, 
127.0.0.1:6379> hmget user age phone
1) "18"
2) "139123123"
127.0.0.1:6379> hgetall user
1) "name"
2) "imooc"
3) "age"
4) "18"
5) "phone"
6) "139123123"
127.0.0.1:6379> hincrby user age 2
(integer) 20
127.0.0.1:6379> hincrbyfloat user age 2.2
"22.2"
127.0.0.1:6379> hlen user
(integer) 3
127.0.0.1:6379> hexists user age
(integer) 1
127.0.0.1:6379> hkeys user
1) "name"
2) "age"
3) "phone"
127.0.0.1:6379> hvals user
1) "imooc"
2) "22.2"
3) "139123123"
```

### list

`list` : 列表，[a, b, c, d, …]。

* `lpush userList 1 2 3 4 5`：构建一个list，从左边开始存入数据
* `rpush userList 1 2 3 4 5`：构建一个list，从右边开始存入数据
* `lrange list start end`：获得数据
* `lpop userList`：从左侧开始拿出一个数据,拿出值以后list中的值就没了
* `rpop userList`：从右侧开始拿出一个数据,拿出值以后list中的值就没了
* `llen userList`：list长度
* `lindex list index`：获取list下标的值，取出的值在list中还存在
* `lset list index value`：把某个下标的值替换
* `linsert list before/after value`：插入一个新的值
* `lrem list num value`：删除几个相同数据
* `ltrim list start end`：截取值，替换原来的list
* `del list`：删除list

```
127.0.0.1:6379> lpush user 1 2 3 4 5
(integer) 5
127.0.0.1:6379> lrange user 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"

127.0.0.1:6379> lrange user1 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"

127.0.0.1:6379> lpop user 
"5"
127.0.0.1:6379> rpop user
"1"
127.0.0.1:6379> lrange user 0 -1
1) "4"
2) "3"
3) "2"
127.0.0.1:6379> lindex user 1
"3"
127.0.0.1:6379> lrange user 0 -1
1) "4"
2) "3"
3) "2"
127.0.0.1:6379> lset user 1 7
OK
127.0.0.1:6379> lrange user 0 -1
1) "4"
2) "7"
3) "2"
127.0.0.1:6379> linsert user BEFORE 7 9
(integer) 4
127.0.0.1:6379> lrange user 0 -1
1) "4"
2) "9"
3) "7"
4) "2"
```

### set

```
127.0.0.1:6379> sadd set 1 2 3 4 5 2 # 新增set元素
(integer) 5
127.0.0.1:6379> smembers set
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
127.0.0.1:6379> scard set 
(integer) 5
127.0.0.1:6379> sismember set 5
(integer) 1

127.0.0.1:6379> clear
127.0.0.1:6379> srem set 4
(integer) 1
127.0.0.1:6379> smembers set
1) "1"
2) "2"
3) "3"
4) "5"
127.0.0.1:6379> spop set 
"2"
127.0.0.1:6379> smembers set
1) "1"
2) "3"
3) "5"
127.0.0.1:6379> spop set 2
1) "1"
2) "5"
127.0.0.1:6379> smembers set
1) "3"

127.0.0.1:6379> SADD set1 1 2 3 4 5 6 7 8 9 10
(integer) 10
127.0.0.1:6379> SMEMBERS set1
 1) "1"
 2) "2"
 3) "3"
 4) "4"
 5) "5"
 6) "6"
 7) "7"
 8) "8"
 9) "9"
10) "10"
127.0.0.1:6379> SRANDMEMBER set1 3
1) "2"
2) "10"
3) "6"
127.0.0.1:6379> SRANDMEMBER set1 3
1) "5"
2) "3"
3) "6"

127.0.0.1:6379> SMOVE set1 set 8 移动元素
(integer) 1
127.0.0.1:6379> SMEMBERS set
1) "3"
2) "8"
127.0.0.1:6379> SMEMBERS set1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "9"
9) "10"

127.0.0.1:6379> SDIFF set set1 # set set1 差集
1) "8"
127.0.0.1:6379> SDIFF set1 set # set1 set 差集
1) "1"
2) "2"
3) "4"
4) "5"
5) "6"
6) "7"
7) "9"
8) "10"
127.0.0.1:6379> SMEMBERS set
1) "3"
2) "8"
127.0.0.1:6379> SMEMBERS set1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "9"
9) "10"

127.0.0.1:6379> SINTER set set1  # set set1 交集
1) "3"

127.0.0.1:6379> SUNION set set1 # set set1 并集
 1) "1"
 2) "2"
 3) "3"
 4) "4"
 5) "5"
 6) "6"
 7) "7"
 8) "8"
 9) "9"
10) "10"
```

### zset

`sorted set`：排序的set，可以去重可以排序，比如可以根据用户积分做排名，积分作为set的一个数值，根据数值可以做排序。set中的每一个memeber都带有一个分数

* `zadd zset 10 value1 20 value2 30 value3`：设置member和对应的分数
* `zrange zset 0 -1`：查看所有zset中的内容
* `zrange zset 0 -1 withscores`：带有分数
* `zrank zset value`：获得对应的下标
* `zscore zset value`：获得对应的分数
* `zcard zset`：统计个数
* `zcount zset 分数1 分数2`：统计个数
* `zrangebyscore zset 分数1 分数2`：查询分数之间的member(包含分数1 分数2)
* `zrangebyscore zset (分数1 (分数2`：查询分数之间的member（不包含分数1 和 分数2）
* `zrangebyscore zset 分数1 分数2 limit start end`：查询分数之间的member(包含分数1 分数2)，获得的结果集再次根据下标区间做查询
* `zrem zset value`：删除member

# SpringBoot整合Redis实战

## Redis线程模型

* `Redis`使用的是单线程模型。

<img src="/assets/images/classOne/cp1/100.png"/>

简单实例

<img src="/assets/images/classOne/cp1/101.png"/>

## SpringBoot接入Redis

* 添加依赖：

```xml
<!-- 引入 redis 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

* 添加配置：

```
spring:
  redis:
    database: 0 # 可以指定自己的数据库下标
    host: 192.168.1.191
    port: 6379
    password: imooc
```

* controller测试：

```java
@Autowired
private RedisTemplate redisTemplate;

redisTemplate.opsForValue().set(key, value);

(String)redisTemplate.opsForValue().get(key);

redisTemplate.delete(key);
```

# Redis进阶提升与主从复制

# Redis 哨兵机制与实现

# Redis集群

# Redis缓存雪崩方案

# Redis批量查询的优化设计




































