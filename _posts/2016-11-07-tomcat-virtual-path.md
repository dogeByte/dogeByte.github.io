---
layout: post
title:  "Tomcat 虚拟路径配置与主目录配置"
date:   2016-11-07 07:44:27
categories: Java
tags: Tomcat
---

# 目录
1. [虚拟路径配置](#1)
	1. [方法一](#1_1)
	2. [方法二](#1_2)
2. [主目录配置](#2)

<h1 id="1">虚拟路径配置</h1>

假设 Tomcat 服务器的安装目录为 `C:/tomcat`，一般情况下，我们都是直接引用 `C:/tomcat/webapps` 目录下的 web 项目，如果我们要部署一个保存在其它路径的 web 项目，就需要在 tomcat 中设置虚拟路径。

Tomcat 的加载 web 项目的顺序是先加载 `C:/tomcat/conf/Catalina/localhost` 目录下的 xml 文件(文件中配置了 web 项目所在路径)，然后再加载 `C:/tomcat/webapps` 目录下的 web 项目。

假设现在需要部署 `D:/project` 目录下的项目 `myProject`。

<h3 id="1_1">方法一</h3>

 `C:/tomcat/Catalina/localhost` 目录下新建 xml 文件，文件名与项目名相同，即 `myProject.xml`，文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context path="/myProject" docBase="D:/project/myProject" debug="0" reloadable="true" />
```

其中属性 `path` 的值实际上是由文件名决定的，因此可以将其省略，即：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context docBase="D:/project/myProject" debug="0" reloadable="true" />
```

<h3 id="1_2">方法二</h3>

打开 `C:/tomcat/conf/server.xml` 文件，在 `<Host>` 标签中添加：

```xml
<Context path="/myProject" docBase="D:/project/myProject" debug="0" reloadable="true" />
```

注意，此时的 `path` 属性不可省略。

通过上述两种配置方式，我们就可以通过 `http://localhost:8080/myProject` 对此 web 项目进行访问。如果将 `C:/tomcat/conf/server.xml` 中 `<Connector>` 标签的 `port` 属性值修改为 `80`，则可以省略访问路径中的端口号，即 `http://localhost/myProject`。

<h1 id="2">主目录配置</h1>

Tomcat 服务器的默认主目录是 `C:/tomcat/webapps/ROOT`。要想修改主目录，只需在 `C:/tomcat/conf/server.xml` 的 `<Host>` 标签中添加：

```xml
<Context path="" docBase="D:/myProject" debug="0" reloadable="true"  crossContext="true" />
```

重启 Tomcat，主目录就被成功修改为 myProject 这个项目，通过 `http://localhost:8080/` 即可对其直接访问。

与此同时，`C:/tomcat/Catalina/localhost` 目录中自动生成了 `ROOT.xml` 文件，用来配置原主目录对应项目 ROOT 的访问路径。