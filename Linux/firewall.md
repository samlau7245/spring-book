# 安装 iptables-services

```sh
# 关闭防火墙
$ systemctl stop firewalld
$ systemctl mask firewalld

# 安装 iptables-services
$ yum install iptables-services

# 设置开机启动
$ systemctl enable iptables.service

$ systemctl stop iptables
$ systemctl start iptables
$ systemctl restart iptables
$ systemctl reload iptables
```

# 开放8080端口

在 iptables 中添加8080端口：

```sh
$ cd /etc/sysconfig/
$ vi iptables
```

在`iptables`中新增`$ -A IN_public_allow -p tcp -m tcp --dport 8080 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT`。

<img src="/assets/images/useage/09.png"/>

重新启动 iptables:

```sh
$ systemctl stop iptables
$ systemctl start iptables
$ systemctl restart iptables
$ systemctl reload iptables
```

> **[warning] 注意**
>
> 如果云服务关联了`安全组`，当`iptables`开放某个端口时，也需要在安全组中把端口开放出来。

# 安全组端口开放

安全组就类似与防火墙一样。

<img src="/assets/images/classOne/cp1/13.png">

<img src="/assets/images/classOne/cp1/14.png">

<img src="/assets/images/classOne/cp1/15.png">

<img src="/assets/images/classOne/cp1/16.png">

<img src="/assets/images/classOne/cp1/17.png">

