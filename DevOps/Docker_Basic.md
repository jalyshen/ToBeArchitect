# Docker基础: Dockerfile

https://www.cnblogs.com/sparkdev/p/6357614.html



Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile快速创建自定义的镜像。

先介绍Dockerfile的基本结构及其支持的众多指令，然后具体讲解通过执行指令来编写定制镜像的Dockerfile。

## 基本结构 

Dockerfile由一行行命令语句组成，并且支持以“#”开头的注释行。一般而言，Dockerfile的内容分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

例如：

```shell
# This dockerfile uses the Ubuntu image
# VERSION 2
# Author: docker_user
# Command format: Instruction [arguments / command] …

# 第一行必须指定基于的容器镜像
FROM ubuntu

# 维护者信息
MAINTAINER docker_user docker_user@email.com

# 镜像的操作指令
RUN echo “deb http://archive.ubuntu.com/ubuntu/ raring main universe” >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo “\ndaemon off;” >> /etc/nginx/nginx.conf

# 容器启动时执行指令
CMD /usr/sbin/nginx
```

其中，一开始必须指明所基于的镜像名称，接下来一般会说明维护者的信息。后面则是镜像操作指令，例如RUN指令，RUN指令将对镜像执行跟随的命令。每运行一条RUN指令，镜像添加新的一层，并提交。最后CMD指令，指定运行容器是的操作命令。

下面是两个dockerhub上的例子。

第一个例子，在Ubuntu镜像的基础上安装inotify-tools、nginx、apache2、openssh-server等软件，从而创建一个新的nginx镜像：

```shell
# nginx
# VERSION 0.0.1
FROM ubuntu
MAINTAINER Victor Vieus <victor@docker.com>
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
```

第二个例子，在ubuntu镜像上，安装firefox和vnc，启动后，用户可以通过5900端口通过vnc方式使用firefox：

```shell
# Firefox over VNC
# VERSION 0.3
FROM Ubuntu
# Install vnc, xvfb in order to create a ‘fake’ display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir /.vnc
# setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox
RUN bash -c ‘echo “firefox” >> /.bashrc’
EXPOSE 5900
CMD [“x11vnc”, “-forever”, “-usepw”, “-create”]
```

## 指令

指令的一般格式为INSTRUCTION arguments，指令包括FROM、MAINTAINER、RUN等，下面分别介绍。

#### FROM

格式：

```shell
FROM <image>
```

或者：

```shell
FROM <image>:<tag>
```

Dockerfile的**第一条指令必须**为FROM指令 ，并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令。

#### MAINTAINER

#### LABEL

#### RUN

#### CMD

#### EXPOSE

#### ENV

#### ADD

#### COPY

#### ENTRYPOINT

#### VOLUME

#### USER

#### WORKDIR

#### ONBUILD

## 创建镜像