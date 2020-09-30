# 离线安装

## 资源下载

<img src="/assets/images/useage/04.png"/>

[mariadb download](https://downloads.mariadb.org/mariadb/10.5.5/) --> [Repository Configuration Tool](https://downloads.mariadb.org/mariadb/repositories/)

<img src="/assets/images/useage/05.png"/>

<img src="/assets/images/useage/06.png"/>

在`rpms`文件夹中有很多文件，可以通过[MariaDB Installation (Version 10.1.21) via RPMs on CentOS 7](https://mariadb.com/kb/en/mariadb-installation-version-10121-via-rpms-on-centos-7/)安装教程来进行安装。

<img src="/assets/images/useage/07.png"/>

## 安装

<img src="/assets/images/useage/08.png"/>

安装步骤:

1) First install all of the dependencies needed. Its easy to do this via YUM packages: yum install rsync nmap lsof perl-DBI nc

```sh
$ yum install rsync nmap lsof perl-DBI nc
```

2) rpm -ivh jemalloc-3.6.0-1.el7.x86_64.rpm ==> `rpm -ivh jemalloc-3.6.0-1.el7.x86_64.rpm`

3) rpm -ivh jemalloc-devel-3.6.0-1.el7.x86_64.rpm ==> `rpm -ivh  jemalloc-devel-3.6.0-1.el7.x86_64.rpm`

```sh
$ rpm -ivh jemalloc-3.6.0-1.el7.x86_64.rpm
$ rpm -ivh  jemalloc-devel-3.6.0-1.el7.x86_64.rpm
```

4) rpm -ivh MariaDB-10.1.21-centos7-x86_64-common.rpm MariaDB-10.1.21-centos7-x86_64-compat.rpm MariaDB-10.1.21-centos7-x86_64-client.rpm galera-25.3.19-1.rhel7.el7.centos.x86_64.rpm MariaDB-10.1.21-centos7-x86_64-server.rpm

```sh
rpm -ivh MariaDB-common-10.5.5-1.el7.centos.x86_64.rpm MariaDB-compat-10.5.5-1.el7.centos.x86_64.rpm galera-4-26.4.5-1.el7.centos.x86_64.rpm MariaDB-client-10.5.5-1.el7.centos.x86_64.rpm MariaDB-server-10.5.5-1.el7.centos.x86_64.rpm
```

```sh
$ rpm -ivh MariaDB-common-10.5.5-1.el7.centos.x86_64.rpm MariaDB-compat-10.5.5-1.el7.centos.x86_64.rpm galera-4-26.4.5-1.el7.centos.x86_64.rpm MariaDB-client-10.5.5-1.el7.centos.x86_64.rpm MariaDB-server-10.5.5-1.el7.centos.x86_64.rpm
# warning: MariaDB-common-10.5.5-1.el7.centos.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 1bb943db: NOKEY
# error: Failed dependencies:
#         libboost_program_options.so.1.53.0()(64bit) is needed by galera-4-26.4.5-1.el7.centos.x86_64
#         socat is needed by galera-4-26.4.5-1.el7.centos.x86_64
#         libpcre2-8.so.0()(64bit) is needed by MariaDB-client-10.5.5-1.el7.centos.x86_64
#         libpcre2-8.so.0()(64bit) is needed by MariaDB-server-10.5.5-1.el7.centos.x86_64
```

问题：

```sh
# libboost_program_options.so.1.53.0()(64bit) is needed by galera-4-26.4.5-1.el7.centos.x86_64
$ yum install boost-devel.x86_64

# ibpcre2-8.so.0()(64bit) is needed by MariaDB-client-10.5.5-1.el7.centos.x86_64
# ibpcre2-8.so.0()(64bit) is needed by MariaDB-server-10.5.5-1.el7.centos.x86_64
$ yum install pcre2

# socat is needed by galera-4-26.4.5-1.el7.centos.x86_64
$ yum install socat
```

新问题：

```sh
$ rpm -ivh MariaDB-common-10.5.5-1.el7.centos.x86_64.rpm MariaDB-compat-10.5.5-1.el7.centos.x86_64.rpm MariaDB-client-10.5.5-1.el7.centos.x86_64.rpm galera-4-26.4.5-1.el7.centos.x86_64.rpm MariaDB-server-10.5.5-1.el7.centos.x86_64.rpm
# Preparing...                          ################################# [100%]
#         file /usr/share/mysql/charsets/Index.xml from install of MariaDB-common-10.5.5-1.el7.centos.x86_64 conflicts with file from package mysql-community-common-5.7.26-1.el7.x86_64
#         file /usr/share/mysql/charsets/armscii8.xml from install of MariaDB-common-10.5.5-1.el7.centos.x86_64 conflicts with file from package mysql-community-common-5.7.26-1.el7.x86_64
#         file /usr/share/mysql/charsets/ascii.xml from install of MariaDB-common-10.5.5-1.el7.centos.x86_64 conflicts with file from package mysql-community-common-5.7.26-1.el7.x86_64
#         file /usr/share/mysql/charsets/cp1250.xml from install of MariaDB-common-10.5.5-1.el7.centos.x86_64 conflicts with file from package mysql-community-common-5.7.26-1.el7.x86_64
#         file /usr/share/mysql/charsets/cp1251.xml from install of MariaDB-common-10.5.5-1.el7.centos.x86_64 conflicts with file from package mysql-community-common-5.7.26-1.el7.x86_64
#         file /usr/share/mysql/charsets/cp1256.xml from install of MariaDB-common-10.5.5-1.el7.centos.x86_64 conflicts with file from package mysql-community-common-5.7.26-1.el7.x86_64
#         file /usr/share/mysql/charsets/cp1257.xml from install of MariaDB-common-10.5.5-1.el7.centos.x86_64 conflicts with file from package mysql-community-common-5.7.26-1.el7.x86_64
# .....

# 解决：
$ yum -y remove mysql-community-common-5.*
```

问题都解决后：

```sh
$ rpm -ivh MariaDB-common-10.5.5-1.el7.centos.x86_64.rpm MariaDB-compat-10.5.5-1.el7.centos.x86_64.rpm MariaDB-client-10.5.5-1.el7.centos.x86_64.rpm galera-4-26.4.5-1.el7.centos.x86_64.rpm MariaDB-server-10.5.5-1.el7.centos.x86_64.rpm
# Preparing...                          ################################# [100%]
# Updating / installing...
#    1:MariaDB-compat-10.5.5-1.el7.cento################################# [ 20%]
#    2:MariaDB-common-10.5.5-1.el7.cento################################# [ 40%]
#    3:MariaDB-client-10.5.5-1.el7.cento################################# [ 60%]
#    4:galera-4-26.4.5-1.el7.centos     ################################# [ 80%]
#    5:MariaDB-server-10.5.5-1.el7.cento################################# [100%]
```

# 配置

```sh
# 设置开机自启动
# systemctl enable mariadb

# 启动数据库
$ service mysql start
# 启动数据库配置: @see (https://mariadb.com/kb/en/mysql_secure_installation/)
$ mysql_secure_installation
```

```sh
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): #密码是空，回车即可。
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y #是否设置root密码，这里进行设置
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y  #删除匿名用户
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n #允许root远程登陆数据库
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] n #不删除test数据库，后面测试可以使用
 ... skipping.

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y #重载授权表
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

# 使用

```sh
$ mysql -u root -p

# Enter password: 
# Welcome to the MariaDB monitor.  Commands end with ; or \g.
# Your MariaDB connection id is 23
# Server version: 10.5.5-MariaDB MariaDB Server

# Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

# Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
# MariaDB [(none)]> 
```

## 远程连接

查看端口使用情况：

```sh
$ lsof -i:3306
# COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# mariadbd 3547 mysql   15u  IPv6  26085      0t0  TCP *:mysql (LISTEN)
```

开放`3306`端口：`-A IN_public_allow -p tcp -m tcp --dport 3306 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT`，然后重新启动防火墙。

赋予root用户远程连接权限：

```sh
$ mysql -u root -p

# 下面操作时在数据库里
# grant all privileges on 库名.表名 to '用户名'@'IP地址' identified by '密码' with grant option;
# 库名:要远程访问的数据库名称,所有的数据库使用“*” 
# 表名:要远程访问的数据库下的表的名称，所有的表使用“*” 
# 用户名:要赋给远程访问权限的用户名称 
# IP地址:可以远程访问的电脑的IP地址，所有的地址使用“%” 
# 密码:要赋给远程访问权限的用户对应使用的密码
$ grant all privileges on *.* to 'root'@'%' identified by 'root密码'; 
$ flush privileges;

# 查看数据库使用用户：
$ use mysql;
$ select host,user from user where user='root';
```

<img src="/assets/images/useage/10.png"/>

## 手动删除匿名用户

```sh
$ mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 10.5.5-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> select user,host from user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
4 rows in set (0.000 sec)

MariaDB [mysql]> delete from user where user='';
Query OK, 0 rows affected (0.000 sec)
```

# 资料

* [https://mariadb.org](https://mariadb.org)
