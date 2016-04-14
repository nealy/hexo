title: 在Cygwin中使用msysgit
date: 2016-04-14 15:45:53
updated: 2016-04-14 15:45:53
tags: [git, cygwin]
categories: git
---

`Cygwin` 是一个在 windows 平台上运行的优秀的类 UNIX 模拟环境，是 cygnus solutions 公司开发的自由软件。Cygwin 内嵌支持 git，对于使用 git 的开发者来讲，是不可多得的优秀工具。但由于内嵌的 git 版本过低，而且对于文件系统庞大的支持不是非常友好，所以我们使用 msysgit 来替换内嵌的 git 程序来提高 git 性能。这样既可以保留 Cygwin 的优雅界面，又拥有强大的性能，一举两得。
<!-- more -->

首先，当然需要安装 Cygwin 和 msysgit，很简单，不多说。
Cygwin 官网： https://www.cygwin.com
msysgit 官网： https://git-scm.com

接下来就来看看如何替换 Cygwin 中的 git：
假设 Cygwin 安装目录是 `C:\cygwin64`，msysgit 安装目录是 `C:\Program Files (x86)\Git`。
* Rename C:\cygwin64\bin\git.exe to C:\cygwin64\bin\git.cygwin64.exe (to back it up)
* Copy C:\Program Files (x86)\Git\bin\git.exe to C:\cygwin64\bin
* Copy the folder C:\Program Files (x86)\Git\libexec to C:\cygwin64\ (root path)
* From within cygwin environment, run git config –global color.ui always

需要注意的是，`C:\Program Files (x86)\Git\bin` 需要添加到环境变量 `path` 中。

Enjoy!
