---
title: 使用 pyenv 构建线上 python 环境
date: 2018-09-23 18:34:44
tags: Linux
---

## 使用 pyenv 部署 python 线上环境
当生产服务器因权限问题或者系统版本阉割出现 python 依赖问题时，可以使用 [pyenv](https://github.com/yyuu/pyenv)

## 安装方法
1.  下载 pyenv.
```
$ git clone https://github.com/yyuu/pyenv.git ~/.pyenv
```

2. 设置`PYENV_ROOT` 为 `pyenv` 安装路径, 添加 `$PYENV_ROOT/bin` 至环境变量
```
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
```

3. 开启 `pyenv` `shims` 和自动补全
```
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```

4. 重启 shell 让 `pyenv` 生效
```
$ exec $SHELL
```

5. 安装需要的 python 版本 至 $PYENV_ROOT/versions 目录
```
$ pyenv install 2.7.8
```

## pyenv 常用命令
```
pyenv install 2.7.5
pyenv versions
pyenv version
pyenv local 2.7.5
pyenv shell 2.7.5
```
