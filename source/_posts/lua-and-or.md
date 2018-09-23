---
title: lua and 和 or 踩坑
date: 2016-09-25 23:59:14
tags: lua
---

## lua and or 操作
* `a and b`
如果a为false，则返回 a；否则返回 b

* `a or b`
如果 a 为 true，则返回 a；否则返回 b

C语言中的语句：`x = a? b : c`，在Lua中可以写成：`x = a and b or c`

## 存在的问题
```
debug = 0(or 1)
step = debug and 0 or 80
```

则无论 debug 取 0 还是 1，step 输出都是 0
* 当 debug 为0
```
0 and 0 = 0
0 or 80 = 0
```
* 当 debug 为 1
```
0 and 0 = 0
0 or 80 = 0
```

## 错误原因
lua将 0 认为是 true

## 解决方法
定义 `debug = true or false`
```
step = debug and 0 or 80
```
则当 debug 为 false 时候，step 为 80
