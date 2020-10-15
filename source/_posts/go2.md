---
title: 《Golang语言基础》变量的概念和使用
date: 2020-07-16 14:11:57
categories:
  - Golang
  - Golang语言基础
tags:
  - Golang
  - Golang语言基础
  - 变量的概念和使用
---

《Golang语言基础》变量的概念和使用
<!--more-->

#### 变量的使用
1、声明变量
第一种：
var名称类型是申明单个变量的语法
> 已字母或者下划线开头，由一个或多个字母、数字、下划线组成```
```
var name type
name = value
```

第二种：
根据值自行判定变量类型
```
var name = value
```

第三种：
省略var
```
name := value
```

#### 多变量申明
第一种，声明与赋值分开：
```
var name1,name2,name3 type
name1,name2,name3 = v1,v2,v3
```

第二种，直接赋值：
```
var name1,name2,name3 = v1,v2,v3
```

第三种，集合类型
```
var (
	name1 type1
	name2 type2
)
```
