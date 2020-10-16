---
title: 《Golang语言基础》数据类型
date: 2020-07-16 15:11:57
categories:
  - Golang
  - Golang语言基础
tags:
  - Golang
  - Golang语言基础
  - iota
  - 数据类型
---

《Golang语言基础》数据类型
<!--more-->

#### iota
iota，特殊常量，可以认为是一个可以被编译器修改的常量
```
const (
	A = iota // 0
	B // 1
	C // 2
	D = "haha" // haha iota = 3
	E // haha iota = 4
	F = 100 // 100 iota = 5
	G // 100 iota = 6
	H = iota // 7
	I // iota 8
)

const (
	J = iota // 0
)
```

#### 基本数据类型
- bool
- Numeric Types
		int8,int16,int32,int64,int
		uint8,uint16,uint32,uint64,uint
		float32,float64
		complex64,complex128
		byte
		rune
- string

1、布尔型bool
布尔型的值只可以是常量true或者false

2、数值型
- int8
有符号8位整型（-128到127）
- int16
有符号16位整型（-32768到32767）
- int32
有符号32位整型（-2147483648到2147483647）
- int64
有符号64位整型（-9223372036854775808到9223372036854775807）
- uint8
无符号8位整型（0到255）
- uint16
无符号16位整型（0到65535）
- uint32
无符号32位整型（0到4294967295）
- uint64
无符号64位整型（0到18446744073709551615）

> int和uint根据底层平台，表示32位或64位证书。除非需要使用特定大小的证书，否则通常应该使用int来表示整数
> 大小：32位系统32位，64位系统64位

2、浮点型
- float32
IEEE-754 32位浮点型数
- float64
IEEE-754 64位浮点型数
- complex64
64位实数和虚数

3、其他
- byte
类似uint8
- rune
类似int32
- uint
32或64位
- int
与uint一样大小
- uintptr
无符号整形，用于存放一个指针

#### 字符串类型
字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的。Go语言的字符串的字节使用UTF-8编码标识Unicode文本
```
var str string
str = "hello world"
```

#### 复合类型（派生类型）
1、指针类型（Pointer）
2、数组类型
3、结构化类型（struct）
4、Channel类型
5、函数类型
6、切片类型
7、接口类型（interface）
8、Map类型
