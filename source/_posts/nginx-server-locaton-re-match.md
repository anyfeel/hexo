---
title: nginx server locaton 匹配顺序
date: 2018-09-24 10:40:52
tags: Nginx
---

## server name
1. 完全匹配
2. 通配符在前面，`*.tesetwb.com`
3. 通配符在后面，`www.testwb.*`
4. 正则表达式匹配 `(.*)?anyfeel.cn`

## location
1. 完全匹配，`=` or `/`
2. 大小写敏感，`~`
3. 大小写不敏感，`~*`
4. 前半部分匹配，`^~`
5. @location, 表示只能用于 nginx 内部跳转
