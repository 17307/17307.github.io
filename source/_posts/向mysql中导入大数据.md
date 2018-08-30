---
title: 向mysql中导入大数据
date: 2018-08-30 10:55:13
categories: db
---

# <center>记一次导入数据中的问题</center>

## 由于某次需要导入一个超大的sql文件，遇到了许多问题。下面是解决办法

## 1. 需要对`my.ini`进行一些配置，包括

```
max_allowed_packet = 768M
wait_timeout=2880000 
interactive_timeout = 2880000
```

## 2.  提前设置数据库引擎

- MYISAM
- InnoDB
- 等等
- 已知：MYISAM插入速度>>>InnoDB

## 3.  如果实在是慢，可以先删掉索引、约束。