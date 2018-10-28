---
title: nginx
date: 2018-10-28 10:44:12
categories: web
---

# nginx 配置

### 安装

`apt install nginx   `

`pip install uwsgi`



 ### 配置



- 创建uwsgi的配置文件

  ```ini
  [uwsgi]
  #项目的位置
  chdir = /home/yy/code/project/py/flask-nginx/onlinetools
  
  #要运行的文件
  wsgi-file = /home/yy/code/project/py/flask-nginx/onlinetools/main.py
  #要调用的函数
  callable = app
  
  # python的虚拟路径
  home = /home/yy/code/project/py/flask-nginx/onlinetools/venv
  
  #socket file's location
  socket = /home/yy/code/project/py/n.sock
  #permissions for the socket file
  chmod-socket    = 666
  ```

  `uwsgi --ini uwsgi.ini `

- 配置 nginx.conf 

  ```xml
  server {
      listen      8000;
      server_name localhost;
      charset     utf-8;
      client_max_body_size 75M;
  
      location ^~ /static {
          root /home/yy/code/project/py/flask-nginx/onlinetools/cmsscan; # static的上层目录
          expires 30d;
      }
  
      location / {
          include uwsgi_params;
          uwsgi_pass unix://home/yy/code/project/py/n.sock;
      }
  }
  ```

  

  - 首先删除默认的配置 `sudo rm /etc/nginx/sites-enabled/default`
  - 然后将写的配置建立软连接 `sudo ln -s /home/yy/code/project/py/nginx/nginx.conf /etc/nginx/conf.d`

- 关于如何配置静态资源的路径以及一些其他的配置：https://my.oschina.net/u/238296/blog/599706



### 配置uwsgi可以一直运行

- `nohup uwsgi --ini config2.ini & `



### nginx停止与开始

`sudo /etc/init.d/nginx stop`

`sudo /etc/init.d/nginx start`

### nginx其他命令

- 查看nginx路径
  - `ps aux|grep nginx `

- 查看nginx配置文件路径
  - `/usr/local/opt/nginx/bin/nginx -t `



#### 坑

- manjaro上，始终是502，将sock文件更换目录……权限问题……尚未有更好方法
- 无法加载静态资源，使nginx以root/项目用户  启动……尚未有其他方案；在配置文件添加：`user o1hy`



