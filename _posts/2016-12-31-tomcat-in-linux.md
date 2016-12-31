---
layout: post
title:  "主机无法访问虚拟机中Tomcat服务的问题"
date:   2016-12-31 11:26:43
categories: Linux
tags: Java,Linux
---

在虚拟机的CentOS中安装Tomcat后，可能会出现这样的情况：主机和虚拟机互相可以ping通，但是主机却无法访问虚拟机中的Tomcat服务。原因在于CentOS防火墙没有开放8080端口，解决办法如下：

<h1 id="1">方法一：关闭防火墙</h1>

```
[root@mini ~]# service iptables stop
```

这相当于所有端口全部开放，难免会降低服务器的安全性。

<h1 id="2"> 方法二：开放8080端口</h1>

```
[root@mini ~]# vi /etc/sysconfig/iptables
```

在其中添加一行内容：

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
```

注意允许访问的规则必须在拒绝访问的规则之前，否则不能生效。

重启iptables和Tomcat服务即可：

```
[root@mini ~]# service iptables restart
[root@mini ~]# apps/apache-tomcat-7.0.78/bin/shutdown.sh
[root@mini ~]# apps/apache-tomcat-7.0.78/bin/startup.sh
```

