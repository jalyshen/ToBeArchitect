# Docker网络

原文：https://blog.csdn.net/xx8093/article/details/117885345?utm_medium=distribute.pc_feed_v2.none-task-blog-hot-20.pc_personrecdepth_1-utm_source=distribute.pc_feed_v2.none-task-blog-hot-20.pc_personrec

### 目录

* Docker 0
* 原理
* 小结
* --link
* 自定义网络
* 网络连通
* 实战：部署Redis集群



## Docker0

清空所有环境

测试：查看宿主机的IP信息：

![1](./images/Docker_Network/1.png)

三个网络，分别是：***本地回环地址、虚拟机网络地址、docker0网络地址***

问题：docker 如何处理容器网络访问？

```shell
# 启动一个容器，这个容器运行Tomcat
[root@localhost ~] docker run -d -P --name tomcat01 tomcat

# 查看容器内部网络地址： ip addr ，	容器启动的时候会得到 eth0@if96 ip地址，docker 分配的
[root@localhost ~] docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
95: eth0@if96: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 思考，Linux 能不能 ping通 容器内部
[root@localhost ~] ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.215 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.054 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.119 ms
```

**linux 可以ping通 docker 容器内部**。

## 原理

​        每启动一个Docker容器，Docker就会给Docker容器分配一个IP。只要安装一个docker，就会有一个网卡docker0桥接模式，使用的是 ***evth-pair*** 技术。

​        再次测试 ip addr：

![2](./images/Docker_Network/2.png)

​        再次启动一个容器进行测试，发现再次多了一对网卡：

![3](./images/Docker_Network/3.png)

#### 总结

* 容器的网卡，都是一对对的
* evth-pair 就是一对的虚拟设备接口，成对出现，一端连着协议，一端彼此相连
* 正因为有这个特性，evth-pair 充当一个桥梁，连接各种虚拟网络设备
* OpenStac、Docker容器之间的连接，OVS的连接，都是使用 evth-pair 技术

​        现在对 tomcat01 和 tomcat02 进行测试，看看能否使用IP来ping通：

```shell
[root@localhost ~] docker exec -it tomcat02 ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.459 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.183 ms
```

**容器和容器之间可以ping通的！(ip来ping, 通过服务名需要另外加参数，下面会讲)**

### 网络模型

![4](./images/Docker_Network/4.png)

#### 结论：

​        tomcat01 和 tomcat02 是公用一个路由器，docker0

​        所有的容器在不指定网络的情况下，都是 docker0 路由的，docker 会给容器分配一个默认的可用ip

## 小结

​        Docker 使用的是 Linux的桥接，宿主机中 是一个 Docker容器的网桥 docker0

![5](./images/Docker_Network/5.png)

Docker 中所有网络接口都是虚拟的，虚拟的转发效率高！

![6](./images/Docker_Network/6.png)

## --link

​        思考一个场景：编写一个微服务，database url=ip，项目不重启，数据库ip换掉，期望：可以通过名字来访问容器

## 自定义网络

## 网络连通

## 实战：部署Redis集群