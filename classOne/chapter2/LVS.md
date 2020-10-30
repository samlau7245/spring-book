
# LVS 负载均衡

* [LVS(Linux Virtual Server,Linux虚拟服务器)](http://linux-vs.org) 是一个虚拟的服务器集群系统。
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
# service network restart
systemctl restart NetworkManager
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

* 所有计算机节点关闭网络配置管理器，因为有可能会和网络接口冲突

```sh
systemctl stop NetworkManager 
systemctl disable NetworkManager
```

```sh
cd /etc/sysconfig/network-scripts/
cp ifcfg-lo ifcfg-lo:1
vim ifcfg-lo:1
```

`ifcfg-lo:1`文件修改内容：

```
DEVICE=lo:1
IPADDR=192.168.55.150
NETMASK=255.255.255.255
```

重启网络服务：

```sh
#service network restart
systemctl restart NetworkManager
```

* 这是`133`服务节点：

<img src="/assets/images/classOne/cp1/80.png"/>

* 这是`134`服务节点：

<img src="/assets/images/classOne/cp1/81.png"/>

## 两台RS配置ARP

### ARP

arp响应级别与通告行为-的概念:

* arp-ignore：ARP响应级别（处理请求）
	* 0：只要本机配置了ip，就能响应请求
	* 1：请求的目标地址到达对应的网络接口，才会响应请求
* arp-announce：ARP通告行为（返回响应）
	* 0：本机上任何网络接口都向外通告，所有的网卡都能接受到通告
	* 1：尽可能避免本网卡与不匹配的目标进行通告
	* 2：只在本网卡通告

### 配置流程

* 添加网卡ARP配置：

```sh
vim /etc/sysctl.conf
```

`sysctl.conf`文件：配置`所有网卡`、`默认网卡`和`虚拟网卡`的ARP响应级别和通告行为，分别为:`all`、`default`、`lo`。

```
# configration for lvs
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.lo.arp_ignore = 1

net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_announce = 2
```

* 刷新配置文件：

```sh
sysctl -p
```

* 增加一个网关，用于接收数据报文，当有请求到本机后，会交给lo去处理：

```sh
route add -host 192.168.55.150 dev lo:1
route -n
```

<img src="/assets/images/classOne/cp1/82.png"/>

设置开机自启动：

```sh
echo "route add -host 192.168.55.150 dev lo:1" >> /etc/rc.local
```

<img src="/assets/images/classOne/cp1/83.png"/>

## 使用ipvsadm配置集群规则

```sh
ipvsadm -A -t 192.168.55.150:80 -s rr

ipvsadm -a -t 192.168.55.150:80 -r 192.168.55.133:80 -g
# ipvsadm -a -t 192.168.55.150:80 -r 192.168.55.134:80 -g
ipvsadm -a -t 192.168.55.150:80 -r 192.168.55.137:80 -g
```

<img src="/assets/images/classOne/cp1/84.png"/>

> **[info] 节点地址更改提示**
>
> 因为虚拟机重启，IP被更改，`192.168.55.134` 更改为 `192.168.55.137`

* 保存规则库，否则重启会失效: `ipvsadm -S`
* 查看集群列表： `ipvsadm -Ln`
* 查看集群状态： `ipvsadm -Ln --stats`

<img src="/assets/images/classOne/cp1/86.png"/>

### ipvsadm 命令

```
Commands:
--add-service     -A        add virtual service with options 【添加集群】
--edit-service    -E        edit virtual service with options
--delete-service  -D        delete virtual service
--clear           -C        clear the whole table 【清除掉已经设置的负载均衡的规则】
--restore         -R        restore rules from stdin
--save            -S        save rules to stdout 【保存规则库】
--add-server      -a        add real server with options 【添加一个真实的服务器】
--edit-server     -e        edit real server with options
--delete-server   -d        delete real server
--list            -L|-l     list the table 【集群列表】
--zero            -Z        zero counters in a service or all services
--set tcp tcpfin udp        set connection timeout values 【设置tcp、udp超时时间】
--start-daemon              start connection sync daemon
--stop-daemon               stop connection sync daemon
--help            -h        display this help message

Options:
  --tcp-service  -t service-address   service-address is host[:port] 【TCP协议,host 就是LVS的VIP】
  --udp-service  -u service-address   service-address is host[:port]
  --fwmark-service  -f fwmark         fwmark is an integer greater than zero
  --ipv6         -6                   fwmark entry uses IPv6
  --scheduler    -s scheduler         one of rr(轮询)|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,the default scheduler is wlc. 【设置负载均衡算法】
  --pe            engine              alternate persistence engine may be sip,not set by default.
  --persistent   -p [timeout]         persistent service 【连接持久化的时间，连接超时时间，默认300s】
  --netmask      -M netmask           persistent granularity mask
  --real-server  -r server-address    server-address is host (and port) 【真实服务器的IP地址】
  --gatewaying   -g                   gatewaying (direct routing) (default) 【设置DR模式】
  --ipip         -i                   ipip encapsulation (tunneling)
  --masquerading -m                   masquerading (NAT)
  --weight       -w weight            capacity of real server
  --u-threshold  -x uthreshold        upper threshold of connections
  --l-threshold  -y lthreshold        lower threshold of connections
  --mcast-interface interface         multicast interface for connection sync
  --syncid sid                        syncid for connection sync (default=255)
  --connection   -c                   output of current IPVS connections
  --timeout                           output of timeout (tcp tcpfin udp)
  --daemon                            output of daemon information
  --stats                             output of statistics information
  --rate                              output of rate information
  --exact                             expand numbers (display exact values)
  --thresholds                        output of thresholds information
  --persistent-conn                   output of persistent connection info
  --nosort                            disable sorting output of service/server entries
  --sort                              does nothing, for backwards compatibility
  --ops          -o                   one-packet scheduling
  --numeric      -n                   numeric output of addresses and ports
  --sched-flags  -b flags             scheduler flags (comma-separated)
```

### 其他ipvsadm命令

* 重启ipvsadm，重启后需要重新配置 ： `service ipvsadm restart`

# 验证DR模式

<img src="/assets/images/classOne/cp1/85.png"/>

当访问`150`这个VIP时，通过查看集群状态`ipvsadm -Ln --stats`可以看到`InBytes(上游)`服务有数据，`OutBytes(下游)`是没有数据的，这个和我们的DR模式一致。

## 验证节点超时时间

现在当重复访问`150`这个VIP时,访问的节点始终是`133`这台服务器，是因为集群与默认的连接超时时间，现在我们更改下配置：

* `ipvsadm -E -t 192.168.55.150:80 -s rr -p 5` : 修改集群连接超时时间为5秒。
* 查看持久化连接 ： `ipvsadm -Ln --persistent-conn`

<img src="/assets/images/classOne/cp1/87.png"/>

* 设置tcp tcpfin udp 的过期时间（一般保持默认）： `ipvsadm --set 1 1 1`
* 查看连接请求过期时间以及请求源ip和目标ip ： `ipvsadm -Lnc`
* 查看过期时间 ： `ipvsadm -Ln --timeout`

<img src="/assets/images/classOne/cp1/88.png"/>

当设置集群的`expire`超时时间过了就会更换访问节点。

# LVS + KeepAlived

LVS现在是单节点服务，和Nginx相同，如果当前节点宕机了就没有了备用机使用，可以通过`LVS + KeepAlived`实现LVS的高可用。

<img src="/assets/images/classOne/cp1/89.png"/>

* 配置`192.168.55.135`节点的`keepalived`配置文件：

```
! Configuration File for keepalived

global_defs {
   router_id keep_135
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 41
    priority 100
    advert_int 1 # 1秒
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.55.150
    }
}

# 配置集群地址访问的IP+端口，端口和Nginx保持一致，80(因为现在的RS是Nginx服务)
virtual_server 192.168.55.150 80 {
    # 健康检查的时间(秒)
    delay_loop 6
    # 负载均衡算法，默认rr。
    lb_algo rr
    # LVS模式，默认NAT，lb=Load Balance
    lb_kind DR
    # 会话时间(秒)，默认50
    persistence_timeout 10
    # 协议
    protocol TCP

    # 真实服务器
    real_server 192.168.55.139 80 {
        weight 1
        # 设置健康检查
        TCP_CHECK {
          # 检查的端口
          connect_port 80
          # 超时时间秒
          connect_timeout 5
          # 重试次数
          nb_get_retry 2
          # 间隔时间，秒
          delay_before_retry 3
        }
    }

    real_server 192.168.55.137 80 {
        weight 1
        # 设置健康检查
        TCP_CHECK {
          # 检查的端口
          connect_port 80
          # 超时时间秒
          connect_timeout 5
          # 重试次数
          nb_get_retry 2
          # 间隔时间，秒
          delay_before_retry 3
        }
    }
}
```

* 启动 LVS + keepalived 

```sh
# 清除负载均衡的规则
ipvsadm -C
# 看看集群列表中是否所有节点配置已经被清除
ipvsadm -Ln
# 重启keepalived服务
systemctl restart keepalived
```

<img src="/assets/images/classOne/cp1/90.png"/>


> **[warning] For warning**
>
> 公网没有VIP这一说，云服务器直接购买云负载均衡器服务即可，他的底层就是LVS




















































