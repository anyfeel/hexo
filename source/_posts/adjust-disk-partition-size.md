---
title: Linux 磁盘调整
date: 2018-09-23 21:19:50
tags: Linux
---

### 磁盘工具
- parted/fdisk/gdisk/ blkid

note:
- 注意区分 PARTUUID 和 UUID
- gdisk sort 可以调整分区序号

### 磁盘格式化
- EFI 分区格式化
```
mkfs.fat -F32 /dev/sdxY -> EFI 分区格式化
```
如果没有 mkfs.fat 则安装 dosfstools

- 分区格式化
```
mkfs.ext4 /dev/sda1
```

### 磁盘启动管理
- 安装 bootctl
```
bootctl —path=/mnt/boot install
```

- 增加磁盘启动选项
```
/boot/loader/entries/arch.conf
```
这里写入的是 root 分区的 PARTUUID


### 磁盘分区大小调整(resize)
- 先删除原有分区，然后再重新建立（启动区块号不能变，否则会丢失数据）
- 使用 resize2fs /dev/sda1 来重新调整分区大小
- 最后需要解决  EFI loader 中 PARTUUID，PARTUUID 可能会有变更
