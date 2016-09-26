---
title: 'find mistake deleted file by grep'
date: 2016-09-26 09:38:54
tags: linux
---

在操作 linux 过程中，经常由于错误的操作误删除文件，则可以使用 grep 操作找回

## 命令
```sh
$ grep -a -B 10 -A 100 'vpsee.com' /dev/sda1 > tmp.txt
```
- `-a`表示将磁盘 `/dev/sda1` 当做二进制文件来读
- `-B 10 -A 100` 表示如果匹配到 `vpsee.com`，则打印前 10 行和后 100 行
- 匹配结果重定向到 `tmp.txt`

## note
1. 在发现误删除文件后，尽量减少其他的文件操作避免已删除的文件被覆盖。
2. grep 操作耗费会占用大量的内存空间，如果发现 grep 失败（虚拟机中），可多分配内存重试。
3. 可能删除的文件不在 `/dev/sda1` 磁盘上，可使用 `df` 来查看是否有其他磁盘。
