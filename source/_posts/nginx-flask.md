---
title: nginx-flask
date: 2018-10-28 10:48:58
categories: web
tags: [docker,nginx,flask]
---

# 一些思考

## 什么时候会用到compose

我有了这样的一个想法：我想创建两个镜像，一个安装`nginx`，另一个安装`python(flask)`,然后使用这2个容器做`web`服务，但是我发现我不会写。所以我去查阅compose的各种资料。我发现常常遇到的例子都是，一个web服务器配上一个数据库的服务器。如下：

```
version: "3"
services:

   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:
```
---

在wordpress的environment中，指定了数据库为：`db：3306`，这个会被解析成db容器的`ip:3306`。出现上面的原因，是因为在`wordpress`容器中使用了`depends_on`，当然也可以使用`link`方法。 
关于`depends_on`和`link`的大概为：通过打开端口，以及一些环境变量与另一个容器进行通信。  **当然**，除了上面两种方法，还可以用另一种方法，在下面将介绍到。

---

所以，`nginx`与`flask`之间的结合，到底适不适合使用`docker-compose`通信。然而，我并没有找到这个问题的答案，那就先看看它到底能不能使用`docker-compose`通信吧。

# 实现

## 思路

1. 拖一个`python`的镜像，在`python`中安装相应的包：`flask`,`uwsgi`。然后配置好这些东西，通过`uwsgi`开启`flask`，监听端口`0.0.0.0:5000`。  
2. 拖一个`nginx`的镜像，修改配置文件`nginx.conf`。在这里，最主要的地方就是如何在`proxy_pass`中，填入正确的 `flask`容器的`ip`。

```conf
#nginx.conf
http {
    server {
        listen 80; 
        server_name 0.0.0.0; 
        location / { 
            proxy_pass http://172.17.0.2:5000;
        }
    }
}
events {
  worker_connections  1024;  ## Default: 1024
}
```
### 方法一
为了解决上面**2**中的问题，当然可以先启动`flask`的容器，然后通过命令获得次容器的`ip`。
```bash
docker inspect <id> | grep IPAddre

docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID> 
# 查看所有容器ip
docker inspect --format='{{.NetworkSettings.IPAddress}}' $(docker ps -a -q)
```
获得上面的`ip`后，可以通过手工添加到上面的配置文件中。  
**但是**docker一个最大的优点就是自动化部署，如果通过这种方法，就显得太**蠢**了。
### 方法二

**通过 docker-compose实现**

#### 工程目录
![image](964E3656ED274796B159A979276AF1C2)

#### flask搭建
**Dockerfile**
```
FROM python:3
RUN pip install uwsgi &&\
    pip install flask &&\
    mkdir /data

EXPOSE 5000
COPY ./app.py /data
COPY ./uwsgi.ini /data
WORKDIR /data
CMD ["uwsgi","uwsgi.ini"]
```
**app.py**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "<span style='color:red'>Hello World !</span>"
```
**uwsgi.ini**
```
#uwsgi.ini
[uwsgi]
socket = 0.0.0.0:5000
#protocol = http
chdir = /data/
wsgi-file = /data/app.py
callable = app
processes = 4
threads = 2
```

#### nginx
**nginx.conf**
```
http {
    server {
        listen 80; 
        server_name 0.0.0.0; 
        location / { 
            proxy_pass http://${TEST_PATH}:5000;
        }
    }
}
events {
  worker_connections  1024;  ## Default: 1024
}
```
为了解决刚才提到的问题，可以看到，这里写的是 **`http://${TEST_PATH}:5000`**,其中的`${TEST_PATH}`是系统的环境变量。那么问题就是，如何取到这个环境变量了。下面将介绍。
#### docker-compose.xml
```
version: "3"
services:

   flask:
     build: ./flask/
     restart: always
     container_name: flask
     
   nginx:
    image: nginx
    # 或者
    # link: flask
    depends_on: flask
    environment:
        TEST_PATH : flask
    container_name: nginx
    ports:
        - 80:80
    volumes:
        - /root/t/nginx/nginx.conf:/1.conf
    command: /bin/bash -c "envsubst < /1.conf > /etc/nginx/nginx.conf && nginx -g 'daemon off;'"
```
或者：
```
version: "3"
services:

   flask:
     build: ./flask/
     restart: always
     container_name: flask
     networks:
        - code-network
     
   nginx:
    image: nginx
    environment:
        TEST_PATH : flask
    container_name: nginx
    ports:
        - 80:80
    volumes:
        - /root/t/nginx/nginx.conf:/1.conf
    networks:
        - code-network
        
    command: /bin/bash -c "envsubst < /1.conf > /etc/nginx/nginx.conf && nginx -g 'daemon off;'"
    
networks:
    code-network:
        driver: bridge
```

首先是`nginx`模块下的`depends_on`或`link`。  
两者的作用都是可以将不同的容器加入到同一个网络,这样，就可以将`docker-compose.xml`文件中的`flask`识别为容器的ip  

**link**  
在谋篇blog中看到了这样的描述：

> 其实就是在容器中的/etc/hosts添加容器id和别名的映射

但是在官网看到了这句话：

> One feature that user-defined networks do not support that you can do with --link is sharing environmental variables between containers. However, you can use other mechanisms such as volumes to share environment variables between containers in a more controlled way.

所以`link`似乎还可共享变量。  
**但是：**
> links provide a legacy interface to connect Docker containers running on the same host to each other without exposing the hosts’ network ports. Use the Docker networks feature instead

**depends_on**  
基本大部分的资料都是说解决**依赖关系**。但是我很好奇,当我在`docker-compose.xml`文件，没有加入`depends_on`时，`environment`中的`TEST_PATH`没有取到`flask`这个值。（这个值需要用到`network`来解决。）  
所以我不知道，我这次的失败是因为启动顺序导致的，还是没有再同一个网络导致的，如果是由于同一个网络导致，那么`depends_on`看来也可起到与`link`相似的功能。  
**总之**  
少用`link`，多用`depends_on`与`net_work`。  
**command**  
解决上面网络问题后，就是`commond`的问题了。最一开始的问题是，如何再`nginx`的配置文件中，获得环境变量。  
` command: /bin/bash -c "envsubst < /1.conf > /etc/nginx/nginx.conf && nginx -g 'daemon off;'"`  
后面的`nginx -g 'daemon off'`是开启`nginx`服务。那么前面呢？  


![image](8B97F365B8A04583918E8A4278EC59CE)  
存在环境变量 `${TERM}` 为 `xterm`，有文件 1.txt 中内容是 `${TERM}`  
![image](0114282659D54DB1BC818C31F5A28A58)    
所以`envsubst < /1.conf > /etc/nginx/nginx.conf `的意思是，将 `/1.conf`中的环境变量替换，然后输入到 `/etc/nginx/nginx.conf`中。  
如果需要指定特定的要替换的变量，可以用：
```
envsubst '$NGINX_HOST $NGINX_PORT $HOST_IP' < /1.conf > /etc/nginx/nginx.conf
```