---
title: Nginx配置不当
date: 2018-09-15 10:20:21
categories: ctf
tags: 文件包含
---

# 题目描述

文件包含,nginx配置读取

**title:** 百度杯2017二月-Zone

# 解题

首先找到了文件包含的漏洞.

![image](1.png)

发现无法使用伪协议读取.同时,module后面的路径会过滤,将 `../` 过滤为 `空`.所以需要构造 `..././` 可以绕过.

![image](2.png)

读取nginx配置文件.  

![image](3.png)

发现最后还有一个包含,继续读.  
![image](4.png)

发现内容
```nginx
location /online-movies {
            alias /movie/; # 改变地址
            autoindex on; # 文件夹读取
        }
```
会把 `/online-movies../`变为 `/moive/../` 

![image](5.png)

![image](6.png)


伪协议不可用是因为 
```php
include "/var/html/".$module....
```