---
layout: post
title:  "使用 GitHub Pages 搭建个人博客"
date:   2016-11-03 21:25:23
categories: Jekyll
tags: GitHub,Ruby,Jekyll
---

# 目录
1. [GitHub Pages 简介](#1)
2. [Jekyll 简介](#2)
3. [建立个人 GitHub Pages](#3)
4. [配置 git 环境](#4)
	1. [安装 git for windows](#4_1)
	2. [配置 SSH keys](#4_2)
5. [创建本地项目](#5)
6. [配置 Jekyll 环境](#6)
	1. [安装 Ruby](#6_1)
	2. [安装 DevKit](#6_2)
	3. [安装 Jekyll](#6_3)
	4. [配置代码高亮](#6_4)
7. [扩展阅读](#7)

<h1 id="1">GitHub Pages 简介</h1>

对于一个托管在 [GitHub](https://github.com/) 上的项目，它们看上去往往是这样：

![mybatis 的文件目录](https://s25.postimg.org/590l8sx9r/01_Git_Hub_repository.png)

但是对于一个新手来说，看到一大堆源代码，只会让人头晕脑涨，不知何处入手。他希望看到的是，一个简明易懂的网页，说明每一步应该怎么做。于是 [GitHub Pages](https://pages.github.com/) 诞生了。

[GitHub Pages](https://pages.github.com/) 提供给托管项目的开发者一个更个性化展示自己项目的方法，用来替代默认的源码列表。所以， [GitHub Pages](https://pages.github.com/) 可以被认为是用户编写的、托管在 [GitHub](https://github.com/) 上的静态网页。 [GitHub](https://github.com/) 提供模板，允许站内生成网页，也允许用户自己编写网页后上传。有意思的是，这种上传并不是单纯的上传，而是会经过 [Jekyll](https://jekyllrb.com/) 程序的再处理。

通过第二种方式，我们只需要按照规则建立仓库，将文章上传，就可以生成自己的博客。这样做，既拥有绝对管理权，又享受 [GitHub](https://github.com/) 带来的便利：不管何时何地，只要向主机提交 commit ，就能发布新文章。而且这一切还是免费的， [GitHub](https://github.com/) 提供无限流量，世界各地都有理想的访问速度。

<h1 id="2">Jekyll 简介</h1>

<img alt="Jekyll logo" src="https://jekyllrb.com/img/logo-2x.png" style="width:50%;background-color:#202020;"/>

[Jekyll](https://jekyllrb.com/) 可以将纯文本转换为静态博客网站，官网介绍其三大特点如下：
> 简单：不再需要数据库，不需要开发评论功能，不需要不断的更新版本——只用关心你的博客内容。
> 
> 静态： [Markdown](https://daringfireball.net/projects/markdown/) （或 [Textile](http://redcloth.org/textile) ）、 [Liquid](https://github.com/Shopify/liquid/wiki) 和 HTML & CSS 构建可发布的静态网站。
> 
> 博客支持：支持自定义地址、博客分类、页面、文章以及自定义的布局设计。

[Jekyll](https://jekyllrb.com/) 是一种简单的、适用于博客的、静态网站生成引擎。它使用一个模板目录作为网站布局的基础框架，支持 [Markdown](https://daringfireball.net/projects/markdown/) 、 [Textile](http://redcloth.org/textile) 等标记语言的解析，提供了模板、变量、插件等功能，最终生成一个完整的静态 Web 站点。简言之，只要按照 [Jekyll](https://jekyllrb.com/) 的规范和结构，不用写html，就可以生成网站。

[Jekyll](https://jekyllrb.com/) 使用 [Liquid](https://github.com/Shopify/liquid/wiki) 模板语言，例如： `\{\{ page.title \}\}` 表示文章标题， `\{\{ content \}\}` 表示文章内容。更多模板变量请点 [这里](http://jekyllrb.com/docs/variables/) 。

[Jekyll](https://jekyllrb.com/) 的一般目录结构是酱紫的：
```
.
├ _drafts
    ├ happy-birthday.md
    ┕ hello-world.md
├ _includes
    ├ footer.html
    ┕ header.html
├ _layouts
    ├ default.html
    ┕ post.html
├ _posts
    ├ 2008-08-08-Olympic-Games.md
    ┕ 2012-12-22-end-of-world.textile
├ _site
├ assets
    ├ css
    ├ icon
    ├ image
    ┕ js
├ _config.yml
├ 404.html
├ favicon.ico
┕ index.html
```

> _drafts 目录用来保存草稿，与已经发布的文章不同，这是没有日期的文章，运行 jekyll serve 会自动赋予当前日期。
> 
> _includes 目录中包含的 html 文件可以作为模块来加载。
> 
> _layouts 目录存放文章的模板， default.html 是整个网站的框架， post.html 则是单个文章的模板。
> 
> _posts 目录中保存已发布的文章，文件名必须符合 yyyy-MM-dd-title.md ，否则会出现错误。
> 
> _site 目录存放最终生成的网站文件。
> 
> assets 目录存放网站的布局文件。
> 
> _config.yml 用来保存配置信息。
> 
> 404.html 为 404 错误页面。
> 
> favicon.ico 为网站的小图标。
> 
> index.html 为博客首页。

[Jekyll](https://jekyllrb.com/) 与 [GitHub Pages](https://pages.github.com/) 的关系：[GitHub Pages](https://pages.github.com/)是一个由 [GitHub](https://github.com/) 提供的用于托管项目主页或博客的服务， [Jekyll](https://jekyllrb.com/) 是后台运行的引擎。

<h1 id="3">建立个人 GitHub Pages</h1>

[GitHub Pages](https://pages.github.com/) 分为用户、组织、项目三种，我们的博客要用到的是用户网站。在阅读以下内容之前，请自行[注册 GitHub 账号](https://github.com/join)。

[新建](https://github.com/new)一个 git 仓库，名称为 username.github.io ，用你自己的 [GitHub](https://github.com/) 账户名替换仓库名中的 username 。

![创建 GitHub 仓库](https://s25.postimg.org/723hx4igf/02_new_repository.png)

<h1 id="4">配置 git 环境</h1>

<h3 id="4_1">安装 git for windows</h3>

请移步[下载地址](https://git-for-windows.github.io/) 

本文主要介绍 windows 环境下的安装，其他环境可参考[廖雪峰大神的 git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137396287703354d8c6c01c904c7d9ff056ae23da865a000)。

安装完成后，在开始菜单中找到Git Bash并打开。

![Git Bash](https://s25.postimg.org/49evqu7hr/03_git_bash.png)

在打开的命令行窗口内执行以下命令，设置你的 git 用户名和邮箱：

``` 
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

<h3 id="4_2">配置 SSH keys</h3>

为了和 [GitHub](https://github.com/) 的远程仓库进行传输，需要进行 SSH 加密设置。

打开 windows 用户目录（ win7以上默认用户目录为 C:\Users\username ），查看是否存在 `.ssh` 文件夹，如果有，再看看这个目录下有没有 `id_rsa` 和 `id_rsa.pub` 这两个文件。如果没有，继续在 Git Bash 命令行内执行：

```
$ ssh-keygen -t rsa -C "email@example.com"
```

把邮箱地址替换为自己的邮箱，一直敲回车直到命令执行完成。此时可以在用户目录里找到 `.ssh` 文件夹，里面有 `id_rsa` 和 `id_rsa.pub` 两个文件。其中 `id_rsa` 是私钥，不能泄露； `id_rsa.pub` 是公钥，无需保密。

登录 [GitHub](https://github.com/) ，点击右上角的 [Settings](https://github.com/settings/keys) 。

![个人设置](https://s25.postimg.org/62hsf5sof/04_setting.png)

使用记事本打开 `id_rsa.pub` 文件，复制里面的全部内容，粘贴到下图的 Key 中。

![SSH keys](https://s25.postimg.org/vzbgrrwbz/05_ssh_key.png)

至此， git 环境搭建完成。

<h1 id="5">创建本地项目</h1>

在你的电脑中建立一个文件夹，作为项目的主目录，我们假定其名称为 myblog 。

对该目录进行 git 初始化：在目录空白处右键，点击 `Git Bash Here` ，在当前目录打开Git Bash 命令行，执行 `$ git init` 。

![Git Bash Here](https://s25.postimg.org/i6x1w55kf/06_Git_Bash_Here.png)

此时目录内会生成一个 `.git` 隐藏文件夹，千万不要手动修改里里面的内容，否则将破坏此 git 仓库。

现在有两种方法可以得到喜欢的模板。我们可以使用 [GitHub](https://github.com/) 官方提供的主题：进入[第一步](#1)创建的 [GitHub](https://github.com/) 仓库，点击 `Settings` ，找到 `GitHub Pages` 设置，点击 `Choose a theme` 。

![设置主题](https://s25.postimg.org/9fq11vm9b/07_Choose_a_theme.png)

[GitHub](https://github.com/) 官方提供了大量风格迥异的主题，并且可以预览，现在挑个喜欢的，点击 Select theme 保存到自己的 [GitHub](https://github.com/) 仓库里吧！

![主题列表](https://s25.postimg.org/5ynyywn73/08_themes.png)

另一种获取模板或主题的方法是，使用别人的源码。 [Jekyll 项目](https://github.com/jekyll/jekyll/wiki/sites)给出了许多基于 [Jekyll](https://jekyllrb.com/) 的网站，把这些网站的源码 `fork` 到自己的 [GitHub](https://github.com/) 仓库里即可。

无论是选择 [GitHub](https://github.com/) 官方主题，还是 `fork` 他人的源码，都需要把 [GitHub](https://github.com/) 仓库中的源文件下载到本地的博客目录中。在 [GitHub](https://github.com/) 仓库中点击 `Clone or download` ，复制远端 git 仓库地址。

![克隆](https://s25.postimg.org/jguvb6zcf/09_Clone_or_download.png)

进入本地项目目录，在 Git Bash 命令行中执行： `$ git clone git@github.com:username/username.git` ，注意将 `username` 替换为你自己的 [GitHub](https://github.com/) 用户名，执行完成后，就成功把主题克隆到本地目录中。

<h1 id="6">配置 Jekyll 环境</h1>

一篇博客辣么长，总不能每次都 `push` 到 [GitHub](https://github.com/) 以后再查看效果吧？所以我们需要本地的 [Jekyll](https://jekyllrb.com/) 环境，用来在博文发布前预览。本文主要介绍 windows 环境下的安装。

<h3 id="6_1">安装 Ruby</h3>

请移步[下载地址](http://rubyinstaller.org/downloads/)， x64 代表64位，请根据实际环境选择合适的版本。

![下载 Ruby](https://s25.postimg.org/e6uhxn4hb/10_download_ruby.png)

安装时有两点注意事项：

> 1. 安装目录不能有中文或空格，如允许 C:\Ruby23-x64 ，不允许 D:\Program Files\Ruby
> 
> 2. 勾选 “ 将 Ruby 添加到环境变量 PATH ”

![安装 Ruby](https://s25.postimg.org/b19w7flv3/11_install_ruby.png)

安装完成后，在命令提示符中执行 `ruby -v` ，输出 Ruby 版本号则说明安装成功。

```
C:\Users\username>ruby -v
ruby 2.3.3p222 (2016-10-21 revision 56859) [x64-mingw32]
```

<h3 id="6_2">安装 DevKit</h3>

[DevKit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit) 是一个在 Windows 上帮助简化安装及使用 Ruby C/C++ 扩展如 RDiscount 和 RedCloth 的工具箱，请再次移步[下载地址](http://rubyinstaller.org/downloads/)，下载与操作系统及 Ruby 版本对应的 DevKit 安装包。

![下载 DevKit](https://s25.postimg.org/nh6m16f73/12_download_Dev_Kit.png)

运行安装包，直接解压至某文件夹，如 `C:\DevKit` 。

![安装 DevKit](https://s25.postimg.org/jyum4seb3/13_install_Dev_Kit.png)

在解压目录中打开命令行，执行 `ruby dk.rb init` ，通过初始化来创建 `config.yml` 文件。

```
C:\DevKit>ruby dk.rb init

Initialization complete! Please review and modify the auto-generated
'config.yml' file to ensure it contains the root directories to all
of the installed Rubies you want enhanced by the DevKit.
```

执行完成后，使用记事本打开生成的 `config.yml` 文件，在其末尾添加 `- C:/Ruby23-x64` ，注意短横线后有一个空格。此时 `config.yml` 文件内容如下：

```
# blablabla......
#
# Example:
#
# ---
# - C:/ruby19trunk
# - C:/ruby192dev
#
---
- C:/Ruby23-x64
```

回到命令行，继续执行审查（非必须）和安装。

```
C:\DevKit>ruby dk.rb review
C:\DevKit>ruby dk.rb install
```

<h3 id="6_3">安装 Jekyll</h3>

确保 gem 已经安装：命令行执行 `gem –v` ，输出版本号说明 gem 已经安装。

```
C:\Users\username>gem -v
2.5.2
```

继续执行 `gem install jekyll`

```
C:\Users\username>gem install jekyll
Fetching: liquid-2.5.5.gem (100%)
Successfully installed liquid-2.5.5
Fetching: fast-stemmer-1.0.2.gem (100%)
Temporarily enhancing PATH to include DevKit...
Building native extensions.  This could take a while...
Successfully installed fast-stemmer-1.0.2

blablabla......

Parsing documentation for jekyll-2.0.3
Installing ri documentation for jekyll-2.0.3
27 gems installed
```

可以通过 `jekyll -v` 命令检验是否安装成功

```
C:\Users\username>jekyll -v
jekyll 3.4.3
```

继续安装 `bundle` 、 `jekyll-paginate` 、 `wdm`

```
C:\Users\username>gem install bundle(bundle)
C:\Users\username>gem install jekyll-paginate
C:\Users\username>gem install wdm
```

安装完成后，可以通过 `gem list` 命令查看已安装的 GEMS 及版本号。

```
C:\Users\username>gem list

*** LOCAL GEMS ***

activesupport (4.2.7)
addressable (2.5.1)

blablabla......

wdm (0.1.1)
yajl-ruby (1.1.0)
```

进入本地项目目录，命令行执行 `bundle init` ，使用记事本打开生成的 `Gemfile` 文件，在末尾添加：

```
gem "jekyll"
gem "jekyll-paginate"
gem "wdm"
```

2016年5月1日起， [GitHub Pages](https://pages.github.com/) 将只支持 [kramdown](http://kramdown.gettalong.org/) 作为markdown解析器。在本地项目目录中，使用记事本打开 `_config.yml` 文件，修改或添加：

```
markdown: kramdown
kramdown:
  input: GFM
  extensions:
    - autolink
    - footnotes
    - smart
gems:
  - jekyll-paginate
```

在本地项目目录中，使用命令行执行 `jekyll build` ，项目目录中的文件以及由 `_post` 文件夹中 md 文件生成的 html 文件，都会被复制到 `_site` 目录中。

若使用命令行执行 `jekyll serve` ，项目会先执行 `jekyll build` ，待Jekyll服务开启后，使用浏览器访问 [http://127.0.0.1:4000/](http://127.0.0.1:4000/) 或 [http://localhost:4000/](http://localhost:4000/) ，即可预览查看 `_post` 文件夹中的文章。

<h3 id="6_4">配置代码高亮</h3>

对于代码高亮区， [GitHub Pages](https://pages.github.com/) 将只支持 [Rouge](http://rouge.jneen.net/) 作为代码高亮器（ highlighter ）。 [Rouge](http://rouge.jneen.net/) 是一个纯 [Ruby](https://www.ruby-lang.org/en/) 实现的代码高亮库，它支持高亮 60 多种语言的代码，可以输出 HTML 、 ANSI-256 色文本格式。所以不需要再为了本地预览，而安装 [Python](https://www.python.org/) 和 [Pygments](http://pygments.org/) 了。

在命令行中执行 `gem install rouge` 来安装 [Rouge](http://rouge.jneen.net/) ，可以通过 `rougify -v` 检验是否安装成功。

```
C:\Users\username>rougify -v
1.11.1
```

[Rouge](http://rouge.jneen.net/) 内置了很多代码样式，可以用 `rougify help style` 命令列出。

```
C:\Users\username>rougify help style
usage: rougify style [<theme-name>] [<options>]

Print CSS styles for the given theme.  Extra options are
passed to the theme.  Theme defaults to thankful_eyes.

options:
  --scope       (default: .highlight) a css selector to scope by

available themes:
  base16, base16.dark, base16.monokai, base16.monokai.light, base16.solarized, b
ase16.solarized.dark, colorful, github, gruvbox, gruvbox.light, molokai, monokai
, monokai.sublime, thankful_eyes
```

以 `monokai.sublime` 为例，导出这款样式的css文件，只需执行命令 `rougify style monokai.sublime > rouge_monokai.css` 即可。

将导出的 `rouge_monokai.css` 文件放入本地项目目录的 `\assets\css` 文件夹下，并在 `\_includes\header.html` 文件中引入。

```
<link rel="stylesheet" href="/css/rouge_monokai.css">
```

使用记事本打开本地项目目录中的 `_config.yml` 文件，修改或添加：

```
highlighter: rouge
kramdown:
  syntax_highlighter: rouge
```

此时的 `_config.yml` 文件中应包括

```
highlighter: rouge
markdown: kramdown
kramdown:
  input: GFM
  extensions:
    - autolink
    - footnotes
    - smart
  syntax_highlighter: rouge
gems:
  - jekyll-paginate
```

在写作时，只需要将代码段前添加 `点点点language` ，代码段后添加 `点点点` ，其中点代表键盘上数字1左边的键， language 代表程序语言的类型，如此即可实现代码高亮。 [Rouge](http://rouge.jneen.net/) 支持的全部语言类型请点击[这里](https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers)。

![代码高亮实例](https://s25.postimg.org/hvk6x4ei7/14_highlight.png)

显示效果为：

```java
public class Demo {
	public static void main(String[] args) {
		System.out.println("Hello Rouge!");
	}
}
```

至此，基于 [GitHub Pages](https://pages.github.com/) 和 [Jekyll](https://jekyllrb.com/) 的博客就搭建完成了。以后只需要将博文放入 `_post` 文件夹并 `push` 到远端仓库，就可以实现博客的管理和更新。

<h1 id="7">扩展阅读</h1>

1. [GitHub Pages now faster and simpler with Jekyll 3.0](https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0)
2. [GitHub Help : Configuring Jekyll](https://help.github.com/articles/configuring-jekyll/)
3. [Dependency versions](https://pages.github.com/versions/)
4. [git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)