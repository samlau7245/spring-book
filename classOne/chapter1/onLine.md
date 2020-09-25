
# CentOS7安装JDK

检查是否安装JDK：

```sh
$ java -version
# openjdk version "1.8.0_212"
# OpenJDK Runtime Environment (build 1.8.0_212-b04)
# OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)
```

如果需要卸载JDK：

```sh
# 1. 检查系统安装的OpenJDK
$ rpm -qa|grep openjdk -i

# java-1.8.0-openjdk-devel-debug-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-javadoc-1.8.0.212.b04-0.el7_6.noarch
# java-1.8.0-openjdk-debug-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-demo-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-src-debug-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-javadoc-debug-1.8.0.212.b04-0.el7_6.noarch
# java-1.8.0-openjdk-headless-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-accessibility-debug-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-src-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-javadoc-zip-debug-1.8.0.212.b04-0.el7_6.noarch
# java-1.8.0-openjdk-headless-debug-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-devel-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-demo-debug-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-javadoc-zip-1.8.0.212.b04-0.el7_6.noarch
# java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
# java-1.8.0-openjdk-accessibility-1.8.0.212.b04-0.el7_6.x86_64

# 删除软件包
$ rpm -e --nodeps 需要删除的软件 # eg: rpm -e --nodeps java-1.8.0-openjdk-devel-debug-1.8.0.212.b04-0.el7_6.x86_64
```

重新安装JDK：

1、创建JDK安装目录：

```sh
$ mkdir /usr/java
```

<img src="/assets/images/classOne/cp1/01.png">

2、使用软件把JDK包上传到某目录：

<img src="/assets/images/classOne/cp1/02.png">

3、解压JDK包：

```sh
$ tar -xvf jdk-8u202-linux-x64.tar

# drwxr-xr-x 7   10  143      4096 Dec 16  2018 jdk1.8.0_202
# -rw-r--r-- 1 root root 404162560 Sep 25 18:25 jdk-8u202-linux-x64.tar
```

4、把解压后的JDK文件夹移动到`/usr/java`中：

```sh
$  mv jdk1.8.0_202/ /usr/java/

# [root@VM_0_6_centos /]# cd usr/java/
# [root@VM_0_6_centos java]# ll
# total 4
# drwxr-xr-x 7 10 143 4096 Dec 16  2018 jdk1.8.0_202
```

5、配置环境变量：

```sh
$ vim /etc/profile
```

配置环境变量：

```sh
export JAVA_HOME=/usr/java/jdk1.8.0_202
export CLASSPATH=.:%JAVA_HOME%/lib/dt.jar:%JAVA_HOME%/lib/tools.jar  
export PATH=$PATH:$JAVA_HOME/bin
```

<img src="/assets/images/classOne/cp1/03.png">

6、刷新profile，让配置生效：

```sh
$ source /etc/profile
```

7、检查安装情况：

```sh
$ java -version

# java version "1.8.0_202"
# Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
# Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```

JDK安装成功。

# 部署第一台Tomcat

# 部署第二台Tomcat

# 域名配置方案

# 安全组端口开放















