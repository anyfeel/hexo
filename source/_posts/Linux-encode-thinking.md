---
title: Linux 编码理解
date: 2018-09-23 18:26:37
tags: Linux
---

## 系统的编码方式
在我所使用的 centos6 中使用 echo $LANG 查看编码
系统的编码方式决定了在终端中录入的内容的编码方式

## 编辑器所使用的文件编码方式
在 vim 中，set fileencoding  查看当前编码方式
编辑器的编码方式决定了在代码中录入内容的编码方式，如 `a[‘key’] = value`，此时的 ‘key’ 字段按照 fileencoding 的方式编码

## 代码中使用的编码方式
例如 `if a == b : pass`，此内容的编码方式

## 从文件中读取的内容的编码方式
例如：
```
redis_cli>>
      set  “marco:domains” “basic.b0.upaiyun.com” “basic"
```
此写入redis的内容，根据终端的编码方式决定， 我的系统默认是utf-8 ，所以此处也是utf-8内容

## 解释器的编码方式
python 解释器使用 unicode  编码方式解释执行

note:
在我使用的环境中为 centos + vim 开发环境，处处是 utf-8 编码

## python 编码中文处理
- 总体思路：程序整体内部使用 python 解释器的内建 ascii, 在文件落地本地存储的时候再转存为 utf-8
- 在处理中文字符串的时候，由于系统编码的差异直接使用 string 类型表示中文的时候错误，使用 unicode 表示。
- 使用 str.decode('utf-8')转换为 unicode object
