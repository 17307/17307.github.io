---
title: docker
date: 2018-10-28 10:46:48
categories: web
tags: docker
---

# 0x00 安装
[官网](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce)

# 0x01 基本命令

## image
```bash
# 拉取
docker image pull hello-world

# 运行image ， 产生 container
docker container run hello-world

```

## container
```bash

# 列出本机正在运行的容器
docker container ls

# 列出本机所有容器，包括终止运行的容器
docker container ls --all

# 杀死某个 container
docker container kill [containID]
```

## Dockerfile 

这里给出一个flask的小例子。
  
### 项目目录  
![image](413BC09A31334384851E03F517CDF8BB)

### app.py
```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello DockerFile!'


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

```

### Dockerfile
```
FROM python:3

#安装flask
RUN ["pip","install","flask"]
#创建文件夹
RUN mkdir -p  /flask/project
#复制文件到相应目录
COPY app.py /flask/project/
#切换目录到/flask/project
WORKDIR /flask/project
#暴露端口,允许外部链接
EXPOSE 5000
#容器启动时 执行的命令 运行flask程序
CMD ["python","app.py"]
```

### 创建镜像与启动

```bash
#首先把app.py Dockerfile文件一起放到一个文件夹下面 然后运行
#flask 是镜像的名字
#注意末尾的 .
docker build -t flask .  

#这条命令运行flask镜像 并把flask镜像中的5000端口和宿主机中的6000端口作映射
docker run -p 6000:5000 flask

#后台启动容器
docker run -d -p 6000:5000 flask

#查看容器中的标准输出
docker logs [ID]

#查看正在运行的容器
docker ps 

#杀死某个正在运行
docker kill [ID]

#进入docker的终端
# -i 允许你对容器内的标准输入 (STDIN) 进行交互
# -t （个人实验是用于交互）
docker run -i -t [image] /bin/bash

# docker container exec命令用于进入一个正在运行的 docker 容器。如果docker run命令运行容器的时候，没有使用-it参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的 Shell 执行命令了。
docker container exec -it [containerID] /bin/bash

```

## 某些变量

### Volumn


> 想要了解Docker Volume，首先我们需要知道Docker的文件系统是如何工作的。  
> Docker镜像是由多个文件系统（只读层）叠加而成。当我们启动一个容器的时候，Docker会加载只读镜像层并在其上（译者注：镜像栈顶部）添加一个读写层。如果运行中的容器修改了现有的一个已经存在的文件，那该文件将会从读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏。当删除Docker容器，并通过该镜像重新启动时，之前的更改将会丢失。在Docker中，只读层及在顶部的读写层的组合被称为Union File System（联合文件系统）。
> 
> 为了能够保存（持久化）数据以及共享容器间的数据，Docker提出了Volume的概念。简单来说，Volume就是目录或者文件，它可以绕过默认的联合文件系统，而以正常的文件或者目录的形式存在于宿主机上。

大概总结一下上面的内容，就是可以实现：**存储数据**和**共享数据**。

```bash
#将宿主机当前目录的db目录 与 镜像中的 /data/db 做挂载
docker run -p 27017:27017 -v $PWD/db:/data/db -d mongo:3.2
```

通过 VOLUME 指令创建的挂载点，无法指定主机上对应的目录，是自动生成的.语法为：
```
FROM ubuntu
VOLUME ["/data1","/data2"]
```
## 保存修改的容器

```bash
docker commit 614122c0aabb aoct/apache2
```

# 0x02 注意

今天遇到遇到了一个有趣的事情，我在构建一个镜像的时候，Dockerfile如下：
```
FROM ubuntu
RUN apt-get update

RUN apt-get install  python3
CMD /bin/bash
```
此时会报错退出，个人分析是因为在安装python3的过程中，需要用户的输入，导致docker中断。所以需要改成
```
FROM ubuntu
RUN apt-get update

RUN apt-get install -y python3
CMD /bin/bash
```
`-y`表示同意各种要求。