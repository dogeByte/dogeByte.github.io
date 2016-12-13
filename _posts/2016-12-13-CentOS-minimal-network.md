---
layout: post
title:  "最小化安装CentOS后的网络配置"
date:   2016-12-13 17:13:58
categories: Linux
tags: Linux
---

# 目录
1. [配置VMware虚拟网络](#1)
2. [配置CentOS网络](#2)
3. [无法ping通主机的解决办法](#3)

<h1 id="1">配置VMware虚拟网络</h1>

进入VMware的虚拟网络编辑器（virtual network editor），选择VMnet8的NAT配置：

![虚拟网络编辑器](https://s25.postimg.org/3pagpakbz/Cent_OS_minimal_network_1.png)

![虚拟网络编辑器](https://s25.postimg.org/m64vg40a7/Cent_OS_minimal_network_2.png)

<h1 id="2">配置CentOS网络</h1>

```
[root@mini ~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

将内容修改为：

```
DEVICE=eth0
HWADDR=00:0C:29:8C:E2:96
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.85.5
NETMASK=255.255.255.0
GATEWAY=192.168.85.1
DNS1=192.168.85.1
DNS2=8.8.8.8
BROADCAST=192.168.85.255
```

重启网络服务：

```
[root@mini ~]# service network restart
```

<h1 id="3">无法ping通主机的解决办法</h1>

虚拟机的网络设置好以后，会出现这样的情况：主机可以ping通虚拟机，但虚拟机无法ping通主机。这是由于Windows7的防火墙默认禁用了ICMPv4-In规则。

![Windows防火墙高级配置](https://s25.postimg.org/bkl043tyn/Cent_OS_minimal_network_3.png)

打开Windows防火墙高级配置，在入站规则中启用“文件和打印机共享（回显请求 - ICMPv4-In）”即可。