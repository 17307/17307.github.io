---
title: 忘记mysql密码
date: 2018-08-30 10:53:04
categories: db
---

# 忘记xampp mysql的密码

- 停止mysql服务器

    `sudo /opt/lampp/lampp stopmysql`

- 使用`--skip-grant-tables` 参数来启动 mysqld

    `sudo /opt/lampp/sbin/mysqld --skip-grant-tables`
- 再开一个终端(在终端中直接右键+B) 进入mysql

    `sudo /opt/lampp/bin/mysql -uroot`

- 修改密码

    `use mysql;`
    `update user set password=password("123456") where user="root";`
    `flush privileges;`

- 重启服务