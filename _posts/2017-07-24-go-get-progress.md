---
layout: post
title:  "显示go get的进度"
date:   2017-07-24 23:36:34
categories: GO
tags: Go
---

# 目录

1. [修改源代码](#1)
2. [重新编译](#2)

<h1 id="1">修改源代码</h1>

打开`$GOROOT/src/cmd/go/vcs.go`，修改以下两处

1. 增加`--progress`

```go
// vcsGit describes how to use Git.
var vcsGit = &vcsCmd{
    name: "Git",
    cmd:  "git",

    createCmd:   []string{"clone --progress {repo} {dir}", "--git-dir={dir}/.git submodule update --init --recursive"},
    downloadCmd: []string{"pull --ff-only", "submodule update --init --recursive"},
    ...
}
```

2. 重定向输出

```go
// run1 is the generalized implementation of run and runOutput.
func (v *vcsCmd) run1(dir string, cmdline string, keyval []string, verbose bool) ([]byte, error) {
    ...
    var buf bytes.Buffer
    cmd.Stdout = &buf
    cmd.Stderr = &buf
    cmd.Stdout = os.Stdout // 新增
    cmd.Stderr = os.Stderr // 新增
    err = cmd.Run()
    out := buf.Bytes()
    ...
}
```

<h1 id="2">重新编译</h1>

1. [下载go1.4](https://golang.org/dl/)，解压到`C:\Users\username\go1.4`
2. [下载gcc](https://sourceforge.net/projects/mingw/files/Installer/)，配置环境变量
   1. LIBRARY_PATH = C:\MinGW\lib

   2. C_INCLUDE_PATH = C:\MinGW\include

   3. Path中添加C:\MinGW\bin

      ![安装gcc](https://s25.postimg.org/se6tiutin/Min_Gw.png)

运行`$GOROOT/src/all.bash`重新编译即可