---
title: INSERT INTO注入
date: 2018-09-08 18:27:35
categories:  ctf
tag: sql注入
---

## 题目描述
登陆,存在kindeditor编辑器,可以读取目录,无法上传,文件包含
## 题目分析
在文章post处,存在insert into的注入
## 解决

### INSERT TO 语句
`INSERT INTO 表名称 VALUES (值1, 值2,....)`
### payload构造
在本题中,发现无法闭合`'`.同时由于插入的content部分为`<textarea>`,所以需要将payload插入到标题.  
但是文章标题有长度限制,所以采取通过插入多条内容绕过.  
通过尝试,得到插入语句应该为:`insert into 表 values (id,title,contetn)`.同时无法闭合最后的`'与)`.所以只能构造出相应的闭合.  

![image](1.png)
可以查出用户的密码,解密后登陆到admin账户.  
进入文章管理界面,存在文件包含或者读取漏洞.通过php伪协议读出flag.php.

## 其他
### kindeditor漏洞
#### 上传漏洞(此题无法使用)
#### 目录扫描
payload `http://d478b0827d21432bae695d8f71ad9a799e6bc2c58d184cbe.game.ichunqiu.com/kindeditor/php/file_manager_json.php?path=/`
![image](2.png)
![image](3.png)
