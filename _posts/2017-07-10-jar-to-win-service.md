---
layout: post
title:  "将jar注册为Windows服务"
date:   2017-07-10 23:23:41
categories: Java
tags: Java
---

# 目录

1. [打包jar](#1)
2. [安装JSW](#2)
3. [配置wrapper服务](#3)
4. [启动服务](#4)

<h1 id="1">打包jar</h1>

对于在Itellij Idea下创建的Maven项目，可以直接使用idea进行打包

![打包设置](https://s25.postimg.org/f67ntnaen/image.png)

![打包设置](https://s25.postimg.org/hc1yo5dv3/image.png)

![打包设置](https://s25.postimg.org/9kpu3bz3j/image.png)

设置完成后，build项目即可将jar包输出至`output directory`。注意应将配置文件排除，避免修改配置造成再次打包的麻烦。这样的话，代码中对配置文件的读取应采用相对路径(相对jar的位置)，如: 

```java
DOMConfigurator.configure("log4j.xml");
Ini ini = new Ini(new File("conf.ini"));
```

<h1 id="2">安装JSW</h1>

java service wrapper可以将jar注册为Windows服务，请移步[下载地址](https://wrapper.tanukisoftware.com/doc/english/download.jsp)下载对应的版本，并解压文件。

<h1 id="3">配置wrapper服务</h1>

在任意位置新建文件夹`service`及其子目录`bin` `conf` `classes` `lib` `log` ，将下表中wrapper文件夹中的文件复制到对应的service目录中。

<table border="1">

<tr align="center">

<td>wrapper</td>

<td>service</td>

<td>备注</td>

</tr>

<tr>

<td>/bin/wrapper.exe</td>

<td>/bin/wrapper.exe</td>

<td></td>

</tr>

<tr>

<td>/src/bin/App.bat.in</td>

<td>/bin/App.bat</td>

<td>直接执行jar</td>

</tr>

<tr>

<td>/src/bin/InstallApp-NT.bat.in</td>

<td>/bin/InstallApp-NT.bat</td>

<td>注册Windows服务</td>

</tr>

<tr>

<td>/src/bin/UninstallApp-NT.bat.in</td>

<td>/bin/UninstallApp-NT.bat</td>

<td>注销Windows服务</td>

</tr>

<tr>

<td>/lib/Wrapper.dll</td>

<td>/lib/Wrapper.dll</td>

<td></td>

</tr>

<tr>

<td>/lib/Wrapper.jar</td>

<td>/lib/Wrapper.jar</td>

<td></td>

</tr>

<tr>

<td>src/conf/wrapper.conf.in</td>

<td>conf/wrapper.conf</td>

<td>服务配置文件</td>

</tr>

<tr>

<td>src/conf/wrapper-license.conf.in</td>

<td>conf/wrapper-license.conf</td>

<td></td>

</tr>

</table>

将自己的jar包放入`service/classes`目录中，配置文件放入`service/bin`目录中。如果通过`java -jar x.jar`来运行x.jar，则需要将配置文件和jar文件放在同一目录中。

外部依赖包放入`service/classes`或`service/lib`。

修改`service/conf/wrapper.conf`:

```shell
wrapper.java.command=%JAVA_HOME%/bin/java (java目录)
wrapper.java.classpath.2=../classes/monitor.jar (自己的jar包)
wrapper.app.parameter.1=com.jing.thread.FileListener (包含main方法的类)
wrapper.name=file listener (服务名称)
wrapper.displayname=file listener (服务显示名称)
```

<h1 id="4">启动服务</h1>

运行`/bin/InstallApp-NT.bat`即可注册服务，`services.msc`打开服务，启动相应的服务即可。