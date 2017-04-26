---
layout: post
title:  "使用 Shadowsocks 正确上网"
date:   2016-11-13 21:29:47
categories: Others
tags: Chrome,Shadowsocks,GFW
---

# 目录
1. [历史渊源](#1)
    1. [很久以前](#1_1)
    2. [不可描述的墙](#1_2)
    3. [ssh tunnel](#1_3)
    4. [Shadowsocks](#1_4)
2. [正确上网](#2)
    1. [安装 Chrome](#2_1)
    2. [安装 SwitchyOmega](#2_2)
    3. [安装 Shodowsocks](#2_3)

对于经常查资料的童鞋们，度娘往往不能满足你们的需求，但是出于某种不可描述的原因，我们还要忍受 google 无法访问带来的阵痛……这个时候，如果有一种免费而又方便配置的正确上网的姿势，那一定是极好的。

<h1 id="1">历史渊源</h1>

Shadowsocks（中文名称：影梭）是使用 Python 、 C++ 、 C# 以及 Go 等语言开发、基于 Apache 许可证的开放源代码软件，用于保护网络流量、加密数据传输以及突破一些不可描述的审查。
Shadowsocks 使用 Socks5 代理方式，Shadowsocks 分为服务器端和客户端。在使用之前，需要先将服务器端部署到服务器上面，然后通过客户端连接并创建本地代理。

<h3 id="1_1">很久以前</h3>

在很久很久以前，我们访问各种网站都是简单而直接的，用户的请求通过互联网发送到服务提供方，服务提供方直接将信息反馈给用户。

![很久以前](https://s25.postimg.org/5q0i37n1r/history01.png)

<h3 id="1_2">不可描述的墙</h3>

突然有一天，GFW 出现了。它就像一堵墙一样，每当用户需要获取信息的时候，就会将它不喜欢的内容统统过滤掉。于是，当我们触发过滤规则的时候，就会收到 `Connection Reset` 这样的响应，而无法接收到正常的内容。

![不可描述的墙](https://s25.postimg.org/51rnk9obz/history02.png)

<h3 id="1_3">ssh tunnel</h3>

人民的智慧是无穷的。人们想到了利用境外服务器代理的方法来绕过墙的过滤，其中包含了各种 HTTP 代理服务、 Socks 服务、 VPN 服务……其中 ssh tunnel 方法最具有代表性。

> 1. 首先用户和境外服务器基于 ssh 建立起一条加密隧道。
> 
> 2. 用户通过建立起来的隧道进行代理，通过 ssh server 向真实的服务发起请求。
> 
> 3. 服务通过 ssh server，经过加密隧道返回给用户。

![ssh tunnel](https://s25.postimg.org/dy2fo7ey7/history03.png)

由于 ssh 基于 RSA 加密技术，所以这堵墙无法对数据传输的过程中的加密数据内容进行关键词分析，避免了被重置链接的问题。但由于创建隧道和传输数据的过程中，ssh 本身的特征是明显的，所以这堵墙 一度通过分析连接的特征进行干扰，导致 ssh 存在被定向进行干扰的问题。

<h3 id="1_4">Shadowsocks</h3>

于是 [clowwindy](https://github.com/clowwindy/shadowsocks) 同学分享并开源了她的[解决方案](https://github.com/shadowsocks/shadowsocks-windows)。

简单理解，Shadowsocks 是将原来 ssh 创建的 Socks5 协议拆开成 server 端和 client 端

> 1. 客户端发出的请求基于 Socks5 协议跟 ss-local 端进行通讯，由于 ss-local 一般是本机或路由器或局域网的其他机器，不需要经过墙，所以解决了通过特征分析进行干扰的问题。
> 
> 2. ss-local 和 ss-server 两端通过多种可选的加密方法进行通讯，经过墙的时候是常规的 TCP 包，没有明显的特征码，而且墙也无法对通讯数据进行解密。
> 
> 3. ss-server 将收到的加密数据进行解密，还原原来的请求，再发送到用户需要访问的服务，获取响应原路返回。

![Shadowsocks](https://s25.postimg.org/421cok967/history04.png)

<h1 id="2">正确上网</h1>

言归正传，以下是 windows 环境下正确上网姿势的配置，所需工具为 [Chrome](https://www.google.com/chrome/) 浏览器 + [SwitchOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) 扩展 + [Shadowsocks](https://github.com/shadowsocks/shadowsocks-windows/releases) 客户端。

<h3 id="2_1">安装 Chrome</h3>

架起梯子之前，Chrome 的链接是打不开的 ^_^ 不过不要灰心，我们可以下载[离线版安装包](http://chromeba.com/help/download.html)。

> 不建议下载某度或者某数字提供的安装包，因为这些都是阉割版，无法自动更新。

<h3 id="2_2">安装 SwitchyOmega</h3>

同样的，上述 SwitchOmega 的应用商店链接也是打不开的 ^_^ 不过还好，我们至少还可以打开  GitHub，请移步[下载地址](https://github.com/FelisCatus/SwitchyOmega/releases)。

下载 `SwitchyOmega.crx`，在 Chrome 地址栏中输入 `chrome://extensions/`，打开扩展程序管理页面，将下载好的 `SwitchyOmega.crx` 拖入浏览器中，安装即可。

安装完成后，浏览器地址栏的右侧会出现一个灰色圈圈的 SwitchyOmega 图标，右键点击选项。

![选项](https://s25.postimg.org/4vdyh6f73/Switch_Omega01.png)

首先新建一个情景模式，不妨将其命名为 `Shadowsocks`。

![新建情景模式](https://s25.postimg.org/8g9u0ejqn/Switch_Omega02.png)

点击新建的情景模式 `Shadowsocks`，填入下面的配置信息。

> 左上角的紫色方框可以点~选一个自己喜欢的颜色吧

![配置情景模式](https://s25.postimg.org/4xxu40iun/Switch_Omega03.png)

点击情景模式 `自动切换`，找到 `规则列表设置`，填入规则列表网站：  `https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`。

![配置自动切换规则列表](https://s25.postimg.org/9xzvppdv3/Switch_Omega04.png)

在 `规则列表` 中填入需要架梯子访问的网址，情景模式选择刚才建立的 `Shadowsocks`，其中网址中的 `*` 为通配符。

![自定义规则](https://s25.postimg.org/3yc4m1t2n/Switch_Omega05.png)

点击左侧的 `应用选项` 保存配置信息。

![应用选项](https://s25.postimg.org/adb5ipzsf/Switch_Omega06.png)

点击 SwitchyOmega 图标，选择自动切换，这样访问规则列表中的网址会使用 Shadowsocks 进行代理，而访问其他网址则会直接连接。

![自动切换](https://s25.postimg.org/5soz3sg33/Switch_Omega07.png)

<h3 id="2_3">安装 Shodowsocks</h3>

请继续移步[下载地址](https://github.com/shadowsocks/shadowsocks-windows/releases)，解压即可无需安装，填入服务器地址，端口号，密码。

![配置Shadowsocks](https://s25.postimg.org/r4rvnqanj/Shodowsocks01.png)

如何获取这些连接信息呢？当然可以直接购买，或者我们可以在一些[网站](http://superssr.tk/)上注册，每天签到就可以获取一定数量的流量，用来查资料绰绰有余~

世界那么大，我想去看看。

![外面的世界](https://s25.postimg.org/5we76aw6n/Shodowsocks02.png)