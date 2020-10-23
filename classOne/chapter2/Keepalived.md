
# Keepalived

> `Keepalived`双机主备可以解决 Nginx宕机导致节点用不了的情况；注意：主备关系不是集群关系，只有在主节点挂了以后备用机才会被启动。

<img src="/assets/images/classOne/cp1/56.png"/>

**keepalived组件特点：**

* 解决单点故障。
* 实现高可用HA(High Ability)机制。
* 组件遵循VRRP协议。

>* `VRRP协议` : 虚拟路由冗余协议(`Virtual Router Redundancy Protocol`),用于解决局域网中配置静态网关出现单点失效现象的路由协议;VRRP是一种容错协议，它通过把几台路由设备联合组成一台虚拟的路由设备，并通过一定的机制来保证当主机的下一跳设备出现故障时，可以及时将业务切换到其它设备，从而保持通讯的连续性和可靠性；也可实现当主机出现问题时发送邮件给管理员的功能。
>* `VRRP的工作原理`: RRP将局域网的一组路由器构成一个备份组，相当于一台虚拟路由器。局域网内的主机只需要知道这个`虚拟路由器的IP地址(VIP Virtual IP Address)`，并不需知道具体某台设备的IP地址，将网络内主机的缺省网关设置为该虚拟路由器的IP地址，主机就可以利用该虚拟网关与外部网络进行通信。

**双机主备原理**

<img src="/assets/images/classOne/cp1/57.png"/>

## 安装

* [Keepalived 安装官网](https://www.keepalived.org/download.html)
* 用FTP工具把`keepalived-2.1.5.tar`上传到云服务`/home/software/`中。
* 解压`tar -xvf keepalived-2.1.5.tar`
* 配置`configure`：

```sh
./configure \
--prefix=/usr/local/keepalived \
--sysconf=/etc

# --prefix ： keepalived 安装位置
# --sysconf： keepalived 核心配置文件所在的位置，固定位置，改成其他位置的话 keepalived 会启动不了。在 /var/log/message中会有报错。
```

* 如果遇到下面的警告就安装`libnl/libnl-3` : `yum -y install libnl libnl-devel`

```
*** WARNING - this build will not support IPVS with IPv6. Please install libnl/libnl-3 dev libraries to support IPv6 with IPVS.
```

* 编译、安装: `make && make install`
* 查看安装的位置: 

```sh
whereis keepalived
# keepalived: /etc/keepalived /usr/local/keepalived
```

其中`/etc/keepalived`目录下有一个`keepalived.conf`文件，这就是keepalived的核心配置文件。

如果安装过程中遇到如下问题：

```
Making all in lib
make[1]: Entering directory `/home/software/keepalived-2.1.5/lib'
make  all-am
make[2]: Entering directory `/home/software/keepalived-2.1.5/lib'
make[2]: Leaving directory `/home/software/keepalived-2.1.5/lib'
make[1]: Leaving directory `/home/software/keepalived-2.1.5/lib'
Making all in keepalived
make[1]: Entering directory `/home/software/keepalived-2.1.5/keepalived'
Making all in core
make[2]: Entering directory `/home/software/keepalived-2.1.5/keepalived/core'
  CC       namespaces.o
In file included from /usr/include/netlink/handlers.h:19:0,
                 from /usr/include/netlink/socket.h:16,
                 from namespaces.c:171:
/usr/include/netlink/netlink-kernel.h:193:2: error: unknown type name ‘__u32’
  __u32 group;
  ^
make[2]: *** [namespaces.o] Error 1
make[2]: Leaving directory `/home/software/keepalived-2.1.5/keepalived/core'
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory `/home/software/keepalived-2.1.5/keepalived'
make: *** [all-recursive] Error 1
```

在`netlink-kernel.h`中报错`unknown type name ‘__u32’`，那么就在文件头部添加： `#include <asm/types.h>`，再进行`./configure`往下的步骤。

# 配置

`/etc/keepalived/keepalived.conf` 配置文件说明：

```
! Configuration File for keepalived

# 全局配置
global_defs {
   # 通知Email，VIP是和主机、备用用绑定在一起的，如果主备机发生了切换，这里就会发邮件进行通知。
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30

   # 路由id：当前安装keepalived的节点主机标识符，保证全局唯一，相当于主键
   router_id LVS_DEVEL

   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

# 基于VRRP协议的实例，可以看出一个服务器节点
vrrp_instance VI_1 {
    # 表示状态是'MASTER'主机还是备用机'BACKUP'
    state MASTER
    # 该实例绑定的网卡， 可以通过 `ip addr` 命令去查看网卡信息
    interface eth0
    # 虚拟路由ID，只要保证每台主备节点一致即可
    virtual_router_id 51
    # 权重，master权重一般高于backup，如果有多个，那就是选举，谁的权重高，谁就当选，master最高，其他堕胎backup之间谁的priority高时，当master挂了就启动谁。
    priority 100
    # 心跳，主备之间同步检查时间间隔，单位秒
    advert_int 1 # 1秒
    # 认证权限密码，防止非法节点进入，每台主备之间的用户名、密码相同
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟出来的ip，可以有多个（vip）
    virtual_ipaddress {
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}
```

主备服务配置说明：

<img src="/assets/images/classOne/cp1/61.png"/>

## 主配置

* `/etc/keepalived/keepalived.conf` 配置`MASTER`主机：

```
! Configuration File for keepalived

global_defs {
   router_id keep_133
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1 # 1秒
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.55.140
    }
}
```

* 查看网卡信息： `ip addr`

<img src="/assets/images/classOne/cp1/62.png"/>

* 启动 keepalived，启动前 keepalived 要在停止状态

```sh
cd /usr/local/keepalived/sbin
./keepalived
```

<img src="/assets/images/classOne/cp1/60.png"/>

* 查看keepalived进程: `ps -ef|grep keepalived`

<img src="/assets/images/classOne/cp1/59.png"/>

* 测试

<img src="/assets/images/classOne/cp1/63.png"/>

### 将keepalived注册为系统服务

将`keepalived`作为服务注册到Linux系统中。

```sh
cd /home/software/keepalived-2.1.5/keepalived/etc
cp init.d/keepalived /etc/init.d/
cp sysconfig/keepalived /etc/sysconfig/
#cp：是否覆盖"/etc/sysconfig/keepalived"？ y
# 让服务生效
systemctl daemon-reload
# 启动 keepalived 服务
systemctl start keepalived.service
# 关闭 keepalived 服务
systemctl stop keepalived.service
# 重启 keepalived 服务
systemctl restart keepalived.service
```

## 备用机配置

* 配置备用机

```
! Configuration File for keepalived

global_defs {
   router_id keep_134
}

vrrp_instance VI_1 {
    # 备用机设置为BACKUP
    state BACKUP
    interface ens33
    virtual_router_id 51
    # 权重低于MASTER
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    	# 注意：主备两台的vip都是一样的，绑定到同一个vip
        192.168.55.140
    }
}
```

<img src="/assets/images/classOne/cp1/64.png"/>

可以看到`192.168.55.140`并没有加到网卡下，是因为`MASTER`现在还在正常运行，现在把`133`机器停掉，再次试一下。

<img src="/assets/images/classOne/cp1/65.png"/>

这次浏览`www.keep.com`时，因为`MASTER`宕机了，所以就直接访问到了`BACKUP`备用机。

<img src="/assets/images/classOne/cp1/66.png"/>

## Nginx自启动

> `Nginx`挂掉而服务器不挂的情况下，`keepalived`不会切换到备用机；`keepalived`是在服务节点挂掉的情况下才起作用。

现在起一个需求： 在`Nginx`挂掉的情况下自动自重启，如果重启不成功的情况下自动切换到备用机。

* 编辑自重启逻辑脚本`vim /etc/keepalived/check_nginx_alive_or_not.sh`

```sh
#!/bin/bash

A=`ps -C nginx --no-header |wc -l`
# 判断nginx是否宕机，如果宕机了，尝试重启
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    # 等待一小会再次检查nginx，如果没有启动成功，则停止keepalived，使其启动备用机
    sleep 3
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi
```

* 修改脚本权限 `chmod +x /etc/keepalived/check_nginx_alive_or_not.sh`
* 配置keepalived监听脚本

```
vrrp_script check_nginx_alive {
    script "/etc/keepalived/check_nginx_alive_or_not.sh"
    interval 2 # 每隔两秒运行上一行脚本
    weight 10 # 如果脚本运行成功，则升级权重+10
    # weight -10 # 如果脚本运行失败，则升级权重-10
}
```

* 在`vrrp_instance`中新增监控脚本

```
track_script {
    check_nginx_alive   # 追踪 nginx 脚本
}
```

* 重启keepalived，让脚本生效

```sh
systemctl restart keepalived
```

这样`/etc/keepalived/keepalived.conf`的整体内容如下：

```
! Configuration File for keepalived

global_defs {
   router_id keep_133
}

vrrp_script check_nginx_alive {
    script "/etc/keepalived/check_nginx_alive_or_not.sh"
    interval 2
    weight 10
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        check_nginx_alive
    }
    virtual_ipaddress {
        192.168.55.140
    }
}
```

## 双主热备

双机主备的缺点：需要两台服务器资源消耗非常高，且如果`MASTER`很稳定宕机的几率很小的情况下，`BACKUP`主机基本就是在浪费资源。

<img src="/assets/images/classOne/cp1/67.png"/>

### DNS解析

<img src="/assets/images/classOne/cp1/68.png"/>



### 双机热备配置

<img src="/assets/images/classOne/cp1/69.png"/>

* 主节点配置

```
! Configuration File for keepalived

global_defs {
   router_id keep_133
}

# 第一组路由-主
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.55.140
    }
}

# 第二组路由-备
vrrp_instance VI_2 {
    state BACKUP
    interface ens33
    virtual_router_id 52
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.55.141
    }
}
```

* 备用节点配置

```
! Configuration File for keepalived

global_defs {
   router_id keep_134
}

# 第一组路由-备
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.55.140
    }
}

# 第二组路由-主
vrrp_instance VI_2 {
    state MASTER
    interface ens33
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.55.141
    }
}
```

* 重启主备两台机器： `systemctl restart keepalived.service`
* 通过`ip addr`查看两台机器的网卡情况，可以发现各种有一个主VIP

<img src="/assets/images/classOne/cp1/70.png"/>


































