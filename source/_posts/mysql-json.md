---
title: MYSQL 使用SQL语句解析JSON
date: 2020-07-01 14:11:57
categories:
  - MYSQL
tags:
  - MYSQL
  - JSON
---

MYSQL 使用SQL语句解析JSON
<!--more-->

## mysql根据json字段的内容检索查询数据
	1.使用 字段->'$.json属性'进行查询条件
	2.使用json_extract函数查询，json_extract(字段,"$.json属性")
	3.根据json数组查询，用JSON_CONTAINS(字段,JSON_OBJECT('json属性', "内容"))

## mysql的json相关函数
```
创建
json_array	创建json数组
json_object	创建json对象
json_quote	将json转成json字符串类型

查询
json_contains	判断是否包含某个json值
json_contains_path	判断某个路径下是否包json值
json_extract	提取json值
column->path	json_extract的简洁写法，mysql 5.7.9开始支持
column->>path	json_unquote(column->path)的简洁写法
json_keys	提取json中的键值为json数组
json_search	按给定字符串关键字搜索json，返回匹配的路径
```

**
mysql5.7以上支持json的操作，以及增加了json存储类型
一般数据库存储json类型的数据会用json类型或者text类型
**

```
注意：用json类型的话
1）JSON列存储的必须是JSON格式数据，否则会报错。
2）JSON数据类型是没有默认值的。
```

## 查询案例：

#### 条件查询json对象里面的id等于142的记录（JSON对象）
```
select  * from log where data->'$.id' = 142;
```

#### 提取JSON中的字段（JSON对象）
```
select data->'$.id' id,data->'$.name' name from log where data->'$.id' = 142;
```


#### 条件查询json数组里面对象的id等于142的记录（JSON数组）
```
select * from log2 where JSON_CONTAINS(data,JSON_OBJECT('id', "142"))
```

#### 提取JSON中的字段（JSON对象）
```
SELECT json_extract(json,"$.key") FROM table

去掉引号
SELECT JSON_UNQUOTE(json_extract(json,"$.key")) FROM table

结果类型转字符串
SELECT CAST(JSON_UNQUOTE(json_extract(json,"$.key"))AS CHAR) FROM table
```

#### 修改json中某个值的语句
```
首先找到json中的key 然后要修改的值 这里值得注意的是如果你的json中没有对应的key的话,他会把这个key追加到你的json中，如果有的话就是修改咯
update table set json=json_set(json,"$.key","改修的值") where ?=?(你的判断条件)
```
