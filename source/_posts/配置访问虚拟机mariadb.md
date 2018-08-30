---
title: 配置访问虚拟机mariadb
date: 2018-08-30 10:54:12
categories: db
---

## 防火墙开放3306端口  

`iptables -A INPUT -p tcp --dport 3306 -j ACCEPT`

## 修改数据库

`user mysql;`
`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY '123456' WITH GRANT OPTION;`  
说明：root是登陆数据库的用户，123456是登陆数据库的密码，*就是意味着任何来源任何主机反正就是权限很大的样子。  
`flush privileges;`

