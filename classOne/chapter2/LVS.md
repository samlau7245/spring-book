
# LVS 负载均衡

* [LVS(Linux Virtual Server,虚拟服务器)](http://linux-vs.org) 是一个虚拟的服务器集群系统。
* LVS(`ipvs`)已经被集成到了`Linux`内核中，它是`负载均衡调度器`。
* 负载能力： LVS > Nginx。
* LVS工作在4层，在接受到请求报文后可以直接转发而不会处理请求报文，工作效率高；Nginx接收到请求报文需要对报文进行处理然后转发，会有一定的性能损耗。

<img src="/assets/images/classOne/cp1/75.png"/>

**LVS和Nginx工作区别：**

<img src="/assets/images/classOne/cp1/71.png"/>

# LVS三种模式

* DIP: `Director Server IP(转发者IP/内网IP)`，主要用于和内部主机通讯的IP地址。
* RIP: `Real Server IP(真实IP/内网IP)`，后端服务器的IP地址。
* VIP: `Virtual IP(虚拟IP)`，向外部直接面向用户请求，作为用户请求的目标的IP地址。

## NAT模式：基于网络地址转换

<img src="/assets/images/classOne/cp1/72.png"/>

可以看出来，基本和Nginx是类似的。

## TUN模式：IP隧道模式

**前提：所有计算机节点必须要有网卡，用于建立隧道，所有的通讯都会通过隧道形成连接，这样每台Server可以直接响应报文发送给用户。**

<img src="/assets/images/classOne/cp1/73.png"/>

所有的Server都暴露给了用户，不太安全。

## DR模式：直接路由模式

<img src="/assets/images/classOne/cp1/74.png"/>

# 搭建DR模式

<img src="/assets/images/classOne/cp1/77.png"/>

## 配置LVS节点与ipvsadm

### 创建子接口

* 所有计算机节点关闭网络配置管理器，因为有可能会和网络接口冲突

```sh
systemctl stop NetworkManager 
systemctl disable NetworkManager
```

* 进入网卡配置目录，找到自己的网卡:`cd /etc/sysconfig/network-scripts/`

<img src="/assets/images/classOne/cp1/76.png"/>

* 拷贝`ifcfg-ens33`创建一个新的子接口：

```sh
# 数字 1 为别名，可以取任意数字
cp ifcfg-ens33 ifcfg-ens33:1
```

* 修改`ifcfg-ens33:1`配置： `vim ifcfg-ens33:1`

```
BOOTPROTO=static
DEVICE=ens33:1
ONBOOT=yes
IPADDR=192.168.55.150
NETMASK=255.255.255.0
```

> `192.168.55.150` 就是VIP，是提供给外网用户访问的ip地址，道理和`nginx+keepalived`那时讲的VIP是一样的。

* 重启网络服务

```sh
service network restart
```

<img src="/assets/images/classOne/cp1/78.png"/>

### 安装ipvsadm

```sh
yum install ipvsadm
```

* 检查是否安装成功

```sh
ipvsadm -Ln
```

<img src="/assets/images/classOne/cp1/79.png"/>

> **[info] 关于云服务的虚拟IP**
>
> * 阿里云不支持虚拟IP，需要购买他的负载均衡服务。
> * 腾讯云支持虚拟IP，但是需要额外购买，一台节点最大支持10个虚拟IP。

## 两台RS配置虚拟IP

## 两台RS配置arp

## 使用ipvsadm配置集群规则










