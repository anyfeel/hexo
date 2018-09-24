---
title: TCP 连接状态整理
date: 2018-09-24 09:41:42
tags: TCP/IP
---

## 要点
1. 五层架构与七层架构
2. 链路层作用 ARP，以太网包格式
3. IP/TCP 格式
4. TCP 状态机及常用的错误码，time_wait 相关系，SO_RESUSEADDR 作用。
5. NAGEL 算法(TCP_NODELAY关闭NAGEL)滑动窗口拥塞避免（慢启动），快速重传与快速恢复，选择重传
6. 四种定时器的作用 ACK 定时器，persist 定时器，keep-alive 定时器，2MSL 定时器
7. 使用 TCPDUMP 和应用程序分析 TCP 状态机

> note:so_linger 作用： 发送 RST 包来断开连接，而不是发送 FIN.

## connect resty by peer
### 含义
本端向对端发送数据，但对端无法识别该连接，返回一个 RST 强制关闭连接

### 原因
* 当尝试和未开放的服务器端口建立tcp连接时，服务器tcp将会直接向客户端发送reset报文，表现状态是连接决绝。
* 双方之前已经正常建立了通信通道，也可能进行过了交互，当某一方在交互的过程中发生了异常，如崩溃等，异常的一方会向对端发送reset报文，通知对方将连接关闭
* 当收到TCP报文，但是发现该报文不是已建立的TCP连接列表可处理的，则其直接向对端发送reset报文
* ack报文丢失，并且超出一定的重传次数或时间后，会主动向对端发送reset报文释放该TCP连接
* 一端设置了 SO_LINGER 选项来关闭连接，则对端会收到 connection by peer 错误来关闭连接

## TCP 状态机
![tcp 状态机](http://onepiece.nos-eastchina1.126.net/images/tcp-status.jpeg)
