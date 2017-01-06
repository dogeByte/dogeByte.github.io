---
layout: post
title:  "MySQL的安装与配置"
date:   2016-12-31 11:26:43
categories: Linux
tags: MySQL,Linux
---

# 目录

1. [卸载默认的mysql-libs](#1)
2. [使用yum安装MySQL](#2)
3. [初始化及相关配置](#3)
4. [rpm安装](#4)

<h1 id="1">卸载默认的mysql-libs</h1>

查看已安装的MySQL：

```
[root@mini yum.repos.d]# yum list installed | grep mysql
mysql-libs.x86_64      5.1.73-8.el6_8   @anaconda-CentOS-201703281317.x86_64/6.9
```

或使用命令`rpm -qa | grep mysql`。卸载已安装的mysql-libs：

```
[root@mini ~]# rpm -e --nodeps mysql-libs
```

清空数据库缓存：

```
[root@mini ~]# yum clean dbcache
```

<h1 id="2">安装MySQL</h1>

查看yum上提供的可下载的MySQL数据库版本：

```
[root@mini ~]# yum list | grep mysql
mysql.x86_64                               5.1.73-8.el6_8                @base  
mysql-devel.x86_64                         5.1.73-8.el6_8                @base  
mysql-libs.x86_64                          5.1.73-8.el6_8                @base  
mysql-server.x86_64                        5.1.73-8.el6_8                @base  
apr-util-mysql.x86_64                      1.3.9-3.el6_0.1               base   
bacula-director-mysql.x86_64               5.0.0-13.el6                  base   
bacula-storage-mysql.x86_64                5.0.0-13.el6                  base   
dovecot-mysql.x86_64                       1:2.0.9-22.el6                base   
freeradius-mysql.x86_64                    2.2.6-6.el6_7                 base   
libdbi-dbd-mysql.x86_64                    0.8.3-5.1.el6                 base   
mod_auth_mysql.x86_64                      1:3.0.0-11.el6_0.1            base   
mysql-bench.x86_64                         5.1.73-8.el6_8                base   
mysql-connector-java.noarch                1:5.1.17-6.el6                base   
mysql-connector-odbc.x86_64                5.1.5r1144-7.el6              base   
mysql-devel.i686                           5.1.73-8.el6_8                base   
mysql-embedded.i686                        5.1.73-8.el6_8                base   
mysql-embedded.x86_64                      5.1.73-8.el6_8                base   
mysql-embedded-devel.i686                  5.1.73-8.el6_8                base   
mysql-embedded-devel.x86_64                5.1.73-8.el6_8                base   
mysql-libs.i686                            5.1.73-8.el6_8                base   
mysql-test.x86_64                          5.1.73-8.el6_8                base   
pcp-pmda-mysql.x86_64                      3.10.9-9.el6                  base   
php-mysql.x86_64                           5.3.3-49.el6                  base   
qt-mysql.i686                              1:4.6.2-28.el6_5              base   
qt-mysql.x86_64                            1:4.6.2-28.el6_5              base   
rsyslog-mysql.x86_64                       5.8.10-10.el6_6               base   
rsyslog7-mysql.x86_64                      7.4.10-7.el6                  base   
```

可见只有5.1.73版本的MySQL，如果需要更高的版本，可以移步[MySQL官网](https://dev.mysql.com/downloads/mysql/)下载所需版本的rpm包进行安装。

通过yum安装mysql、mysql-server和mysql-devel：

```
yum install -y mysql-server mysql mysql-devel
```

<h1 id="3">初始化及相关配置</h1>

在安装完MySQL数据库以后，系统会多出一个mysqld服务，通过`service mysqld start`命令就可以启动数据库服务：

```
[root@mini ~]# service mysqld start
```

如果是第一次启动，MySQL服务器首先会进行初始化的配置：

```
初始化 MySQL 数据库： WARNING: The host 'mini' could not be looked up with resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MySQL version. The MySQL daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MySQL privileges !
Installing MySQL system tables...
OK
Filling help tables...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/bin/mysqladmin -u root password 'new-password'
/usr/bin/mysqladmin -u root -h mini password 'new-password'

Alternatively you can run:
/usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr ; /usr/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/mysql-test ; perl mysql-test-run.pl

Please report any problems with the /usr/bin/mysqlbug script!

                                                           [确定]
正在启动 mysqld：                                           [确定]
```

注意到其中有一行信息`/usr/bin/mysqladmin -u root password 'new-password'`，提示需要为MySQL的root用户设置密码：

```
[root@mini ~]# mysqladmin -u root password 'abcdefg'
```

此时就可以使用root账户登录MySQL：

```
[root@mini ~]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| test               |
+--------------------+
3 rows in set (0.00 sec)
```

使用MySQL数据库时，必须先启动mysqld服务。可以查看mysqld服务是否开机自启动，如果没有自启动，则将其开启：

```
[root@mini ~]# chkconfig --list | grep mysqld
mysqld          0:关闭  1:关闭  2:关闭  3:关闭  4:关闭  5:关闭  6:关闭
[root@mini ~]# chkconfig mysqld on
[root@mini ~]# chkconfig --list | grep mysqld
mysqld          0:关闭  1:关闭  2:启用  3:启用  4:启用  5:启用  6:关闭
```

<h1 id="4">rpm安装</h1>

如果yum中没有需要的版本，可以移步[MySQL官网](https://dev.mysql.com/downloads/mysql/)下载所需版本的rpm包进行安装。

```
[root@mini ~]# rpm -ivh MySQL-server-5.5.56-1.el6.x86_64.rpm MySQL-client-5.5.56-1.el6.x86_64.rpm 
warning: MySQL-server-5.5.56-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                ########################################### [100%]
   1:MySQL-client           ########################################### [ 50%]
   2:MySQL-server           ########################################### [100%]
[root@mini ~]# rpm -qa | grep MySQL
perl-DBD-MySQL-4.013-3.el6.x86_64
MySQL-server-5.5.56-1.el6.x86_64
MySQL-client-5.5.56-1.el6.x86_64
```

启动mysql服务：

```
[root@mini ~]# service mysql start
Starting MySQL.Logging to '/var/lib/mysql/mini.err'.
. SUCCESS! 
```

