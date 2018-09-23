---
title: Unix 环境变量加载顺序
date: 2018-09-23 10:15:21
tags: Linux
---

## OSX

Mac系统的环境变量加载顺序为

```
/etc/profile -> /etc/paths -> ~/.bash_profile -> ~/.bash_login ->  ~/.profile -> ~/.bashrc
```

特别注意 /etc/paths 中的内容
```
/usr/bin
/bin
/usr/sbin
/sbin
/usr/local/bin
```

Homebrew 安装的软件，其二进制执行文件都放在/usr/local/bin中，
bin 在使用时的查找不是覆盖原则，而是优先查找，所以例如 mac 已经自带了sqlite3，如果 brew 安装后，最新版的 sqlite3 是不会被调用的，因此可以将顺序修改一下以达到目的。

## Linux

Mac系统的环境变量加载顺序为
```
/etc/profile -> (~/.bash_profile | ~/.bash_login | ~/.profile) -> ~/.bashrc -> /etc/bashrc -> ~/.bash_logout
```

/usr/bin:usr/sbin 在 /etc/profile 文件中
