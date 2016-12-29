---
layout: post
title:  "CentOS中安装SunJDK"
date:   2016-12-29 20:18:35
categories: Linux
tags: Java,Linux
---

# 目录
1. [卸载自带的OpenJDK](#1)
2. [安装SunJDK](#2)
   1. [下载JDK](#2_1)
   2. [解压](#2_2)
   3. [设置环境变量](#2_3)

<h1 id="1">卸载自带的OpenJDK</h1>

在默认情况下，CentOS会安装OpenOffice之类的软件，而这些软件需要Java的支持，因此系统会默认安装一个OpenJDK环境，如果需要使用Sun的Java环境，最好先卸载这些默认安装的JDK。

查询系统自带的JDK：

```
[root@dogebyte ~]# rpm -qa | grep java
```

此时会列出系统中存在的JDK：

```
tzdata-java-2016j-1.el6.noarch
java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el6_8.x86_64
java-1.6.0-openjdk-1.6.0.41-1.13.13.1.el6_8.x86_64
```

卸载已安装的JDK：

```
[root@dogebyte ~]# rpm -e --nodeps tzdata-java-2016j-1.el6.noarch
[root@dogebyte ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el6_8.x86_64
[root@dogebyte ~]# rpm -e --nodeps java-1.6.0-openjdk-1.6.0.41-1.13.13.1.el6_8.x86_64
```

再次执行`rpm -qa | grep java`没有任何显示，说明Java相关软件已经全部卸载。

<h1 id="2">安装SunJDK</h1>

<h3 id="2_1">下载JDK</h3>

下载jdk-8u131-linux-x64.tar.gz，在SecureCRT中alt+p打开SFTP，执行`put D:\jdk-8u131-linux-x64.tar.gz`，将安装包上传至CentOS的用户目录中。也可以先在SFTP中`cd`至CentOS中的指定目录后再执行`put`上传。

```
sftp> put d:\jdk-8u131-linux-x64.tar.gz
Uploading jdk-8u131-linux-x64.tar.gz to /home/dogebyte/jdk-8u131-linux-x64.tar.gz
  100% 181191KB   5490KB/s 00:00:33     
d:/jdk-8u131-linux-x64.tar.gz: 185540433 bytes transferred in 33 seconds (5490 KB/s)
```



如需下载，则可以先`lcd`至windows的指定目录，然后`get /root/jdk-8u131-linux-x64.tar.gz`进行下载。

<h3 id="2_2">解压</h3>

```
[root@dogebyte ~]# mkdir apps
[root@dogebyte ~]# tar -zxvf jdk-7u45-linux-x64.tar.gz -C apps
```

<h3 id="2_3">设置环境变量</h3>

```
[root@dogebyte ~]# vi /etc/profile
```

`G`跳转至文件尾，`o`在尾部插入新行，添加以下内容：

```
JAVA_HOME=/root/apps/jdk1.8.0_131
JRE_HOME=/root/apps/jdk1.8.0_131/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

`Esc`退出编辑模式，`Shift+zz`或`:wq`保存并退出。

使环境变量的修改生效：

```
[root@dogebyte ~]# source /etc/profile
```

验证安装：

```
[root@dogebyte ~]# java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```