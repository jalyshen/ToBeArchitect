# Docker基础: Dockerfile

https://www.cnblogs.com/sparkdev/p/6357614.html

内容来自：

作者：[sparkdev](http://www.cnblogs.com/sparkdev/)

出处：http://www.cnblogs.com/sparkdev/

=====

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

作用：Dockerfile的**第一条指令必须**为FROM指令 ，并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令。

#### MAINTAINER

格式：

```shell
MAINTAINER <name>
```

作用：指定维护者信息。

注意：MAINTAINER指令已经被抛弃，建议使用LABEL。

#### LABEL

格式：

```shell
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

作用：为镜像添加标签。一个LABEL就是一个键值对。

举例如下：

```shell
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \that label-values can span multiple lines."
```

可以给镜像添加多个LABEL。需要注意的是，每条LABEL指令都会生成一个新的层。最好是把添加的多个LABEL合并为一条指令：

```shell
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

也可以分行写：

```shell
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

如果新添加的LABEL和已有的LABEL同名，则新值会覆盖旧值。

可以使用docker inspect命令查看镜像的LABEL信息。

#### RUN

格式：

```shell
RUN <command>
```

或者：

```shell
RUN ["executable", "param1", param2"]
```

第一种将在shell终端中运行，即/bin/sh -c， 第二种则使用**exec**执行。指定使用其他终端可以通过第二种方式实现，例如：RUN ["bin/bash", "-c", "echo hello"]

每条RUN指令将在当前镜像的基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

#### CMD

支持三种格式：

```shell
CMD [“executable”, “param1”, “param2”] ：使用 exec 执行，推荐方式。

CMD command param1 param2              ：在 /bin/sh 中执行，提供给需要交互的应用。

CMD [“param1”, “param2”]               ：提供给 ENTRYPOINT 的默认参数。
```

指定启动容器时实行的命令，每个Dockerfile只能有一条CMD命令。如果指定了多条CMD命令，只有最后一条会被执行。

如果用户在启动容器时制定了要运行的命令，则会覆盖掉CMD指定的命令。

#### EXPOSE

格式：

```shell
EXPOSE <port> [<port>…]
```

例如：EXPOSE 22 80 8443

这个命令告诉Docker服务，容器需要暴露的端口号，供互联系统使用。在启动容器时需要通过 **-P** 参数让Docker主机分配一个端口转发指定的端口。使用 -P 参数则可以具体指定主机上哪个端口映射过来。

#### ENV

格式：

```shell
ENV <key> <value>
```

用来指定一个环境变量，这个变量提供给后续的RUN指令使用，并在容器运行时保持。例如：

```shell
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

#### ADD

格式：

```shell
ADD <src> <dest>
```

该指令将复制指定的<src>到容器中的<dest>。<src>可以是Dockerfile所在目录的一个相对路径(文件或者目录)，也可以是一个URL；还可以是一个tar文件（自动解压为目录）。

#### COPY

格式：

```shell
COPY <src> <dest>
```

复制本地主机的<src>（为Dockerfile所在目录的相对路径，文件或目录）为容器中的<dest>。目标路径不存在时，会自动创建。当使用本地目录为源目录时，推荐使用COPY。

#### ENTRYPOINT

格式：

```shell
ENTRYPOINT [“executable”, “param1”, “param2”]

ENTRYPOINT command param1 param2 (shell 中执行)
```

配置容器启动后执行的命令，并且**不可**被docker run提供的参数覆盖。

每个dockerfile中只能有一个ENTRYPOINT，当指定多个ENTRYPOINT时，只有最后一个生效。

#### VOLUME

格式：

```shell
VOLUME ["/data"]
```

可以使用VOLUME指令添加多个数据卷：

```shell
VOLUME ["/data1", "/data2"]
```

创建一个可以从本地或者其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

#### USER

格式：

```shell
USER daemon
```

指定运行容器时的用户名或者UID，后续的RUN也会使用指定用户。当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以在之前创建所需要的用户。

例如：RUN groupadd -人postgres && useradd -r -g postgres postgres

#### WORKDIR

格式：

```shell
WORKDIR /path/to/workdir
```

为后续的RUN、CMD、ENTRYPOINT指令配置工作目录。可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会给予之前命令指定的路径。例如：

```shell
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

则最终路径是： /a/b/c

#### ONBUILD

格式：

```shell
ONBUILD [INSTRUCTION]
```

配置当所创建的镜像作为其他新创建镜像的基础镜像时，所执行的操作指令。例如：Dockerfile使用如下的内容创建了镜像image-A:

```shell
…
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build –dir /app/src
…
```

如果基于image-A创建新的镜像时，新的Dockerfile中使用FROM image-A指定基础镜像时，会自动执行ONBUILD指令内容，等价于在后面添加了两条指令。

```shell
FROM image-A
#automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build –dir /app/src
```

## 创建镜像

编写完成Dockerfile之后，可以通过docker build命令来创建镜像。

基本的格式为：

```shell
docker build [optional]路径
```

该命令将读取指定路径下（包括子目录）的Dockerfile，并将该路径下所有内容发送给docke服务端，由服务端来创建镜像。因此一般建议放置Dockerfile的目录为空目录。

另外，可以通过.dockerignore文件来让docker忽略路径下的目录和文件。要指定镜像的标签信息，可以通过 -t 选项来实现。

例如：指定Dockerfile所在路径为 /tmp/docker_builder/，并且希望生成镜像标签为 build_repo/first_image, 可以使用如下命令：

```shell
$ sudo docker build -t build_repo/first_image /tmp/docker_builder/
# 如果不想使用上次 build 过程中产生的 cache 可以添加 --no-cache 选项
$ sudo docker build --no-cache -t build_repo/first_image /tmp/docker_builder
```

