
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
* 安装`libnl/libnl-3`依赖：`yum -y install libnl libnl-devel`
* 配置`configure`：

```sh
./configure \
--prefix=/usr/local/keepalived \
--sysconf=/etc

# --prefix ： keepalived 安装位置
# --sysconf： keepalived 核心配置文件所在的位置，固定位置，改成其他位置的话 keepalived 会启动不了。在 /var/log/message中会有报错。
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

## 主配置

## 备用机配置

## Nginx自启动

## 双主热备















































