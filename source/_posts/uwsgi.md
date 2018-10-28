---
title: uwsgi
date: 2018-10-28 10:45:33
categories: web
---

# 镇楼
https://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/index.html
# 个人理解

uwsgi是类似**nginx**或者**apache**这类容器与python交互的东西。
> WSGI 是一个 Python 协议，定义了应用程序（我们写的软件）如何与 Web 服务器（如 Nginx）通信，WSGI 只是一个接口
> 
> WSGI是一种通信协议。  
> uwsgi同WSGI一样是一种通信协议。  
> 而uWSGI是实现了uwsgi和WSGI两种协议的Web服务器。

从某种角度来看，uwsgi是一种**没有**nginx优秀的容器。(不知道说法是否准确)

https://lufficc.com/blog/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu

# 安装

```python
pip install uwsgi
```

# 例子

现在将实现一个简单的小例子，顺便进行发散式的扩充，因为正在学习docker，所以在docker的环境下进行。

## docker环境

**Dockerfile**
```
FROM python:3
RUN pip install uwsgi \
    && pip install flask
    
EXPOSE 5000
CMD /bin/bash
```

**在 $PWD/python 下创建一个简单的 ```app.py```**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "<span style='color:red'>Hello World !</span>"
```
**启动docker**
```bash
docker run -p 5000:5000 -v $PWD/python:/data -it uwsgi
```
**通过uwsgi启动flask**
``` 
uwsgi --http 0.0.0.0:5000 -w app:app
# 或者
uwsgi --socket 0.0.0.0:5000 --protocol=http -w app:app
```
- `-w app:app` 等价于 `--wsgi-file app.py --callable app`   
  指定 **哪个文件下**的**哪个变量**用来执行。

- `--http` 原本情况下，uwsgi是处理WSGI协议，所以需要指定   是`http`来处理。  
  如果是用`--socket`情况下，是建立起一个通信的socket，可以用来和`nginx`通信。此时，通过`protocol`来指定`http`协议。

https://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/HTTP.html
> http-socket <bind> 选项将会让uWSGI和原生HTTP通信。如果你的web服务器不支持 uwsgi protocol ，但是可以与上游HTTP代理通信，或者如果你正在用诸如Webfaction或者Heroku这样的服务来托管你的应用，那么你可以使用 http-socket 。如果你计划只通过uWSGI开放你的应用，那么用 http 选项来代替，因为路由器/代理/负载均衡器将会保护你。

**与nginx的通信**  

有两种方式：
- 用 **ip+端口** 方式来指定   socket，**ip**可能有问题，这里还不了解  
  `--socket 0.0.0.0:5000`
- 用 **文件方式** 指定socket  
  `socket = /home/yy/code/project/py/n.sock`  

此时需要nginx中使用不同的方法来配置对应socket。

![image](BD5E7E9AEDB04E409724C6AC1DCB237F)

下图中的 **ip**可能有问题，这里还不了解。  
![image](2794D18CD5E34ACE842DAA0C535AD7B3)

## 例子2
```
FROM python:3
RUN pip install uwsgi &&\
    pip install flask &&\
    mkdir /data

EXPOSE 5000
COPY ./app.py /data
WORKDIR /data
CMD ["uwsgi","--http","0.0.0.0:5000","-w","app:app"]
```
## 例子3

这个例子用于介绍 **uwsgi的配置文件** 如何写 
```ini
[uwsgi]
http = 0.0.0.0:5000
# 指定运行的文件的位置
chdir = /data/
wsgi-file = /data/app.py
# 指定变量
callable = app
processes = 4
threads = 2
```
当然也可如此
```ini
[uwsgi]
socket = 0.0.0.0:5000
protocol = http
chdir = /data/
wsgi-file = /data/app.py
callable = app
processes = 4
threads = 2
```
**执行命令**：
```bash
uwsgi uwsgi.ini
```

**所以新的Dockerfile为**:
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
**然后**
```bash
docker build -t uwsgi3 .
```
**然后**
```
docker run -p 5000:5000 uwsgi 
```

