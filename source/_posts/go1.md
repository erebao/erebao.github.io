---
title: 《Golang语言基础》编码规范
date: 2020-07-15 14:11:57
categories:
  - Golang
  - Golang语言基础
tags:
  - Golang
  - Golang语言基础
  - 编码规范
---

《Golang语言基础》编码规范
<!--more-->

#### 一、命名规范
1、包命名
包名应该为小写单词，不要使用下划线或者混合大小写
	
2、文件命名
应该为小谢单词，使用下划线分隔各个单词
	
3、结构体命名
- 采用驼峰命名法，首字符根据访问控制大写或者小写
- struct申明和初始化采用多行，如下所示：
```
//多行申明
type User struct{
	Username string
	Email string
}
//多行初始化
u := User{
	Username: "erebao",
	Email: "erebao@gmail.com"
}
```

4、接口命名
- 命名规则基本和上面的结构体类似
- 单个函数的结构名以“er”作为后缀，例如 Reader， Writer
```
type Reader interface{
	Read(p []byte)(n int, err error)
}
```

5、变量命名
· 和结构体类似，变量名称一般遵循驼峰法，首字母根据访问控制原则大写或者小写，但遇到特有名词时，需要遵循一下规则：
- 如果变量为私有，切特有名词为首个单词，则使用小写，如：apiClient
- 其他情况都应当使用该名词原有的写法：如ApiClient、repolD、UserID
- 错误实例UrlArray应写成urlArray或者URLArray
- 若变量类型为bool类型，则名称应以Has、Is、Can或Allow开头
```
var isExist bool
var hasConflict bool
var canManage bool
var allowGitHook bool
```

6、常量命名
常量均需使用全部大写字母组成，并使用下划线分词
```
const APP_VER = "1.0"
```
如果是枚举类型的常量，需要先创建相应类型：
```
type Scheme string
const {
	HTTP Scheme = "http"
	HTTPS Scheme = "https"
}
```

7、关键字

| |   |   |   |   |
| ------------ | ------------ | ------------ | ------------ | ------------ |
|break   |default   |func   |interface   |select   |
|case   |defer   |Go   |map   |Struct   |
|chan   |else   |Goto   |package   |Switch   |
|const   |fallthrough   |if   |range   |Type   |
|continue   |for   |import   |return   |Var   |

#### 二、注释
```
多行注释：/* */
单行注释：//
```
1、包注释
```
// util包，该包包含了项目共用的一些常量，封装了项目中一些共用函数
// 创建人：erebao
// 创建时间：20200715
```

2、结构、接口注释
```
// User，用户对象，定义了用户的基础信息
type User struct{
	Username string // 用户名
	Email string // 邮箱
}
```

3、函数、方法注释
```
// NetAttrModel，属性数据层操作类的工厂方法
// 参数：
//		ctx：上下文信息
// 返回值
//		属性操作类指针
func NetAttrModel(ctx *common.Context) *AttrModel{

}
```

4、代码逻辑注释
```
// 从Redis中批量读取属性，对于没有读取到的id，记录到一个数组里面，准备从DB中读取
xxxxx
xxxxxxx
xxxxxxx
```

5、注释风格
- 建议全部使用单行注释
- 和代码的规范一样，单行注释不要过长，禁止超过120字符

#### 三、代码风格
1、缩进和折行
tab

2、语句的结尾
不要使用冒号结尾
如果你打算将多个语句写在同一行，则必须使用冒号

3、括号和分隔
```
// 正确的方式
if a > 0 {

}
// 错误的方式
if a>0	// a、0和>之间应该空格
{	// 左大括号不可以换行，会报语法错误

}
```

#### 四、import规范
如果你在一个文件里面引入了一个package，建议采用如下格式：
```
import (
	"fmt"
)
```
如果你的包映入了三种类型的包，标准库包，程序内部包，第三方包，建议采用如下格式：
```
import (
	"encoding/json"
	"strings"
	
	"myproject/models"
	"myproject/controller"
	"myproject/utils"
	
	"github.com/astaxie/beego"
	"github.com/go-sql-drive/mysql"
)
```

#### 五、错误处理
接收到错误，要么返回err，或者使用log记录下来
```
// 错误写法
if err != nil {
	// error handling
} else {
	// normal code
}

// 正确写法
if err != nil {
	// error handling
	return // or continue, etc.
}
// normal code
```

#### 六、测试
单元测试文件名命名规范为 example_test.go
测试用例函数名称必须以Test开头，例如：TestExample

#### 七、常用工具
**gofmt**
大部分格式问题都可以通过gofmt解决，gofmt自动格式化代码，保证所有的go代码与官方推荐的格式保持一致，于是所有格式有关问题，都以gofmt的结果为准

**goimport**
该工具在gofmt的基础上增加了自动删除和引入包
```
go get golang.org/x/tools/cmd/goimports
```

**go vet**
vet工具可以帮我们静态分析我们的源码存在的各种问题，例如多余的代码，提前return的逻辑，struct的tag是否符合标准等
```
go get golang.org/x/tools/cmd/vet
```
使用如下：
```
go vet .
```
