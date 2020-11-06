# Tomcat 部署方案

<img src="/assets/images/classOne/cp1/04.png">

## 部署第一台Tomcat

<img src="/assets/images/classOne/cp1/05.png">

```sh
# 解压
$ tar -xvf apache-tomcat-9.0.38.tar 
# 重命名
$ mv apache-tomcat-9.0.38 tomcat
# 移动到 /usr/local中
$ mv tomcat/ /usr/local/
# 启动tomcat
$ cd /usr/local/tomcat/bin/
$ ./startup.sh

# Using CATALINA_BASE:   /usr/local/tomcat
# Using CATALINA_HOME:   /usr/local/tomcat
# Using CATALINA_TMPDIR: /usr/local/tomcat/temp
# Using JRE_HOME:        /usr/java/jdk1.8.0_202/jre
# Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
# Using CATALINA_OPTS:   
# Tomcat started.

# 查看端口启动情况
$ lsof -i:8080
# COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# java    30726 root   55u  IPv4 909939      0t0  TCP *:webcache (LISTEN)
```

**现在因为防火墙原因，外网估计访问不到，直接到`iptables`中把`8080`端口加进去！**

启动成功界面：

<img src="/assets/images/classOne/cp1/11.png">

## 同服务器部署第二台Tomcat

把第二台tomcat命名为：`tomcat-api`:

<img src="/assets/images/classOne/cp1/06.png">

```sh
# 移动到 /usr/local中
$ mv tomcat-api/ /usr/local/
```

<img src="/assets/images/classOne/cp1/07.png">

因为已经存在一台tomcat服务，所以第二个tomcat服务就需要进行一些端口修改。

```sh
$ cd /usr/local/tomcat-api/conf/
$ vi server.xml
```

<img src="/assets/images/classOne/cp1/08.png">

修改`profile`全局变量：

```sh
$ vi /etc/profile

# 在profile中添加下面三个环境变量

# The Second Tomcat PATH
CATALINA_API_BASE=/usr/local/tomcat-api
CATALINA_API_HOME=/usr/local/tomcat-api
TOMCAT_API_HOME=/usr/local/tomcat-api

# 重启profile
$ source /etc/profile
```

<img src="/assets/images/classOne/cp1/09.png">

把环境变量添加到`catalina.sh`中：

```sh
$ cd /usr/local/tomcat-api/bin/
$ vi catalina.sh

# 在 OS specific support.  $var _must_ be set to either true or false. 后添加：
export CATALINA_BASE=$CATALINA_API_BASE
export CATALINA_HOME=$CATALINA_API_HOME
```

<img src="/assets/images/classOne/cp1/10.png">

启动tomcat：

```sh
$ cd /usr/local/tomcat-api/bin/
$ ./startup.sh

# Using CATALINA_BASE:   /usr/local/tomcat-api
# Using CATALINA_HOME:   /usr/local/tomcat-api
# Using CATALINA_TMPDIR: /usr/local/tomcat-api/temp
# Using JRE_HOME:        /usr/java/jdk1.8.0_202/jre
# Using CLASSPATH:       /usr/local/tomcat-api/bin/bootstrap.jar:/usr/local/tomcat-api/bin/tomcat-juli.jar
# Using CATALINA_OPTS:   
# Tomcat started.
```

**和第一个tomcat一样，也要把`8088`端口加入到`iptables`中**

<img src="/assets/images/classOne/cp1/10.png">

# 在Mac端启动Tomcat

<img src="/assets/images/Linux/01.png">

```sh
pwd
# /apache-tomcat-9.0.39/
chmod -R 777 bin
cd bin/
./startup.sh 
# Using CATALINA_BASE:   /Users/shanliu/apache-tomcat-9.0.39
# Using CATALINA_HOME:   /Users/shanliu/apache-tomcat-9.0.39
# Using CATALINA_TMPDIR: /Users/shanliu/apache-tomcat-9.0.39/temp
# Using JRE_HOME:        /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
# Using CLASSPATH:       /Users/shanliu/apache-tomcat-9.0.39/bin/bootstrap.jar:/Users/shanliu/apache-tomcat-9.0.39/bin/tomcat-juli.jar
# Using CATALINA_OPTS:   
# Tomcat started.
```