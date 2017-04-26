---
layout: post
title:  "MyEclipse 创建 Maven 项目及常见问题"
date:   2016-11-04 19:22:15
categories: Java
tags: MyEclipse,Maven
---

# 目录
1. [Maven 简介](#1)
2. [配置 Maven 环境](#2)
    1. [下载](#2_1)
    2. [配置环境变量](#2_2)
    3. [建立 Maven 仓库](#2_3)
    4. [配置 MyEclipse](#2_4)
3. [创建 Maven 项目](#3)
4. [常见的 Error / Warning](#4)
    1. [无尽的 Updating indexes](#4_1)
    2. [不兼容的 jdk](#4_2)
    3. [找不到超类 HttpServlet](#4_3)
    4. [project facet 版本不匹配](#4_4)
    5. [无法建立源文件夹](#4_5)

<h1 id="1">Maven 简介</h1>

Apache Maven，是一个软件（特别是 Java 软件）项目管理及自动构建工具，由 Apache 软件基金会所提供。基于项目对象模型（ Project Object Model , POM ）概念，Maven 利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。

Maven 也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。Maven 曾是 Jakarta 项目的子项目，现为由 Apache 软件基金会主持的独立 Apache 项目。

<h1 id="2">配置 Maven 环境</h1>

<h3 id="2_1">1. 下载</h3>

请移步[下载地址](https://maven.apache.org/download.cgi)，解压至某个文件夹内，如 `D:\apache-maven-3.3.9`。

<h3 id="2_2">2. 配置环境变量</h3>

在 Path 的变量值最后添加 `D:\apache-maven-3.3.9\bin;`，注意和之前的变量值用半角分号隔开。

<h3 id="2_3">3. 建立 Maven 仓库</h3>

新建一个文件夹作为 Maven 仓库目录，如 `D:\maven\repository` 

在 `D:\apache-maven-3.3.9\conf\settings.xml` 文件中添加

```xml
<localRepository>D:\maven\repository</localRepository>
```

将修改后的 `settings.xml` 文件复制到 `D:\maven\` 中。

<h3 id="2_4">4. 配置 MyEclipse</h3>

window -> preferences -> Maven -> Installations -> Add

Installation home 选择 Maven 的解压目录

![配置 MyEclipse](https://s25.postimg.org/onalzz3hr/install_maven_1.png)

至此，MyEclipse 下的 Maven 环境就搭建好了，下载的 jar 包以及对应的源代码和 javadoc，都将存放到我们建立的 Maven 仓库中。

<h1 id="3">创建 Maven 项目</h1>

在 MyEclipse 中，选择 new -> Project... -> Maven Project

![创建 Maven 项目](https://s25.postimg.org/6902vzr73/new_maven_project_1.png)

选择“使用默认的工作空间”。

![创建 Maven 项目](https://s25.postimg.org/6aa0pet0v/new_maven_project_2.png)

建立 java 项目，请选择红框中的 maven-archetype-quickstart ；建立 web 项目，请选择蓝框中的 maven-archetype-webapp。

![创建 Maven 项目](https://s25.postimg.org/4xsbnivlb/new_maven_project_4.png)

Artifact Id 代表项目名称，填写完成后点击 Finish，静待 Maven 工程的创建。

![创建 Maven 项目](https://s25.postimg.org/5opn71ncf/new_maven_project_5.png)

选中新创建的工程，点击 new -> Source Folder。

![创建 Maven 项目](https://s25.postimg.org/om59kar1r/new_maven_project_6.png)

自动创建的两个包 `com.jing.myMavenProject` 看着不爽可以删除。

![创建 Maven 项目](https://s25.postimg.org/4st5rldnz/new_maven_project_7.png)

至此，一个典型的 Maven 项目就建好了，它的目录结构可以是以下两种，分别代表 java 项目和 web 项目。

![创建 Maven 项目](https://s25.postimg.org/hlh9rip9r/new_maven_project_8.png)

![创建 Maven 项目](https://s25.postimg.org/hmvsyoghb/new_maven_project_9.png)

<h1 id="4">常见的 Error / Warning</h1>

使用 MyEclipse 创建 Maven 项目，经常会遇到一些小问题，特别是在创建 web 项目时，特记录如下：

<h3 id="4_1">1. 无尽的 Updating indexes</h3>

配置完 Maven 环境后，以后每次打开 MyEclipse，右下角的进度条就会永无止境地 Updating indexes。

![无尽的 Updating indexes](https://s25.postimg.org/f5fgdo773/problem_1_1.png)

虽然我们可以点击蓝圈里的停止，结束这个过程，然而下次打开 MyEclipse，它还会坚持不懈地出现。

解决方法： window -> preferences -> Maven，取消勾选 `Download repository index updates on startup`，世界清净了。

![结束无尽的 Updating indexes](https://s25.postimg.org/a81vsk57z/problem_1_2.png)

<h3 id="4_2">2. 不兼容的 jdk</h3>

如下图中的Warning

![不兼容的 jdk](https://s25.postimg.org/8ua8x95yn/error_1_1.png)

解决方法： 右键点击项目目录中的 JRE System Library，选择 Properyies。

![Properties](https://s25.postimg.org/4c7y55pwv/warning_1_2.png)

根据本机安装的 jdk 版本，为项目选择合适的 JRE System Library。

![选择合适的 jdk](https://s25.postimg.org/jm7tcclf3/warning_1_3.png)

<h3 id="4_3">3. 找不到超类 HttpServlet</h3>

如下图中的Error

![不兼容的 jdk](https://s25.postimg.org/8ua8x95yn/error_1_1.png)

这是由于缺少 web 项目所需的 jar 包所导致的，导入相应的 jar 包即可。对于 Maven 项目，我们可以通过配置 `pom.xml` 来管理所有的 jar 包。

打开项目中的 `pom.xml`，在 `<dependencies>` 标签中添加：

```xml
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.1</version>
    <scope>provided</scope>
</dependency>
    <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

保存后，MyEclipse 会自动下载配置中的 jar 包。jar 包所对应的 dependency，可以从 [Maven repository](https://mvnrepository.com/) 获取。

<h3 id="4_4">4. project facet 版本不匹配</h3>

![project facet 版本不匹配](https://s25.postimg.org/vlnwxegdr/error_2_1.png)

解决方法：在 Package Explorer 视图中右键单击项目名，选择最后一项 Properties，或者直接选中项目名后 `Alt+Enter` 进入项目属性设置，选择左侧的 Project Facets，根据 jdk 版本，为 Dynamic Web Services 和 Java 选择合适的 Project Facet 版本。

![Project Facets](https://s25.postimg.org/ez6cobnfz/error_2_2.png)

<h3 id="4_5">5. 无法建立源文件夹 src/main/java 和 src/test/java</h3>

对于刚建好的 web 项目，可能看起来和上面说过的目录结构不同，就是少了两个源文件夹 src/main/java 和 src/test/java，如果我们想手动创建这些 Source Folder，却可能会遇到些阻碍。

![无法建立源文件夹](https://s25.postimg.org/6uy8jl10v/problem_2_1.png)

解决方法：同上打开项目属性设置，选择左侧的 Java Build Path，点击 Source 选项卡，选中 missing 的源文件夹，将其移除。

![移除源文件夹](https://s25.postimg.org/qgmpfs3fz/problem_2_2.png)

现在就可以创建这些源文件夹了~

![创建源文件夹](https://s25.postimg.org/jfypn01nz/problem_2_3.png)