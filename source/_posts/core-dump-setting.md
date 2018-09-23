---
title: Linux core 设置
date: 2016-09-25 19:32:26
tags: linux
---

linux 程序异常退出时，内核会生成 core 文件，通过 GDB 查看 core 文件可以定位程序异常退出时候的堆栈信息，指示出错的位置。

## 查看 core 文件是否打开
```sh
$ ulimit -c
```
如果结果为0，则表示系统关闭了该功能

## 开启 core
在当前 shell 设置
```sh
$ ulimit -c unlimited
```
为整个用户设置
```sh
$ cat ulimit-c unlimited > ~/.zshrc && source ~/.zshrc
```

## core 设置
### 文件名是否带pid标识
```sh
$ echo "1" > /proc/sys/kernel/core_uses_pid
```
core 文件格式为 `corename_with_format.pid`

```sh
$ echo "0" > /proc/sys/kernel/core_uses_pid
```
core 文件格式为 `corename_with_format`

### 保存位置及格式设置
查看当前设置
```sh
$ cat /proc/sys/kernel/core_pattern
```
位置及格式
```sh
$ echo "/corefile_path/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
```

<!-- more -->

具体参数含义

| 参数 | 描述 |
| :---: | :--- |
| %p | pid |
| %u | current uid |
| %g | current gid |
| %s | signal that caused the coredump |
| %t | UNIX time that the coredump occurred |
| %h | hostname where the coredump happened |
| %e | coredumping executable name |
