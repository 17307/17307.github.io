---
title: ubuntu基础配置
date: 2018-08-30 19:52:36
categories: linux
---

ubuntu的简单配置

<!--more-->

# <center>IP配置</center>

## 静态IP

`sudo gedit /etc/network/interfaces`

```
auto [自己网卡]
iface [自己网卡] inet static
address 192.168.8.100    //ip
netmask 255.255.255.0   //子网掩码
gateway 192.168.8.2 //网关
dns-nameserver 8.8.8.8 //dns 实测 8.8.8.8
```

`sudo /etc/init.d/networking restart`

`reboot`

## 设置动态IP

```
auto eth0
#iface eth0 inet static
iface eth0 inet dhcp
```

# <center>密码问题</center>

- ubuntu修改root密码，$ sudo passwd root
- su root    
  Password:   