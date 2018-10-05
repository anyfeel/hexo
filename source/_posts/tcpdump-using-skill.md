---
title: Unix tcpdump 使用技巧
date: 2018-03-18 21:46:18
tags: Linux
---

## 常用 tcpdump 参数解析
- -i 指定网卡，一般不清楚网卡设置直接使用 "any" 表示抓取所有网卡
- -A 使用 ASCII 码打印收到的每个包
- -X 同时以十六进制和 ASCII 打印包

## 常用抓包实例
```
# host 和 port 过滤
tcpdump -i any -Ans 0 "src host 1.1.1.1 && dst host 2.2.2.2 && dst port 3100"

# GET
tcpdump -i eth1 'tcp[(tcp[12]>>2):4] = 0x47455420'

# POST
tcpdump -i eth1 'tcp[(tcp[12]>>2):4] = 0x504f5354'

# TCP 标志位
tcpdump -i any -Ans 0 'host 183.214.154.4 && tcp[tcpflags]=tcp-syn'

```

<!-- more --> >

## tcpdump 过滤器
### 目标语法 `dst` 和 `src` 表示来源和目的地
eg: src host and dst port

### 逻辑语法 `and`、`or` and `not`
eg: src host1 or src host2

### 包内容过滤
proto[x:y] 过滤从x字节开始的y字节数。比如ip[2:2]过滤出3、4字节（第一字节从0开始排

eg:
```
tcp[(tcp[12]>>2):4] = 0x47455420
tcp[12]>>4<<2 == tcp[12]>>22
tcp[12]>>4 拿到 13 字节的 高 4 位，
>>2相当于除以 4(32/8)，即偏移的 8 bits 位数
```

### 标志位过滤
tcp[tcpflags]=tcp-syn

## TCP Header Format

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

 Data Offset:  4 bits

    The number of ```32 bit words``` in the TCP Header.  This indicates where
    the data begins.  The TCP header (even one including options) is an
    integral number of 32 bits long.
```


## IP format

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
