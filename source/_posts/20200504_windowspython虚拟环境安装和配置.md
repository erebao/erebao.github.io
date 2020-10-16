---
title: windows python 虚拟环境安装和配置
date: 2020-05-04 15:49:57
categories:
  - python
  - windows
tags:
  - python
  - windows
---

windows 下 Python 虚拟环境安装和配置
<!--more-->

# 安装虚拟环境

```
pip install virtualenvwrapper-win

pip install virtualenv
```

# 创建虚拟环境
```
mkvirtualenv -p C:\Users\Administrator\AppData\Local\Programs\Python\Python36\python.exe spider
```

# 进入虚拟环境
```
workon spider
```

# 退出虚拟环境
```
deactivate
```
