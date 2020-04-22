---
title: "Docker网络"
date: 2020-04-22T13:46:07+08:00
categories: ["docker"]
tags: ["docker", "docker网络" ,"docker多机网络"]
---

## 使用方式
您可以通过以下方式使用本书：
- Github地址：https://github.com/Knowledge-Precipitation-Tribe/Dive-into-Docker
- GitBook在线浏览：https://docs.docker.knowledge-precipitation.site/


## content
- <a href = "#Docker单机网络">Docker单机网络</a>
- <a href = "#Docker端口映射">Docker端口映射</a>
- <a href = "#flask-redis实战">flask-redis实战</a>
- <a href = "#Docker多机网络">Docker多机网络</a>
- <a href = "#flask-redis多机实战">flask-redis多机实战</a>


## [Docker网络](#content)

### [Docker单机网络](#content)

关于网络基础知识大家请系统学习计算机网络相关内容。

这里介绍两个几个常用命令：

- ping：查看指定IP是否可达 `ping 192.168.0.1`。
- telnet：查看服务是否可用 `telnet 192.168.0.1 80`，有的地址不可以ping但是可以用telnet。
- curl：用来请求 Web 服务器。它的名字就是客户端（client）的 URL 工具的意思。
- wget：一个下载文件的工具。
- ip a：显示IP地址。

查看docker的网络：

```bash
docker network ls
```

可以看到在docker中有三种网络：

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba002ff664?w=1222&h=190&f=png&s=180899)

默认情况下容器使用的是桥接也就是Bridge Network，之后我们启动一个容器并查看网络具体内容

```bash
docker run -d --name test1 busybox /bin/sh -c "while true;do sleep 3600;done"
docker inspect 0b464e45177b(改成你查看到的NETWORK ID)
```

在network的具体细节内我们可以看到containers中包含了我们刚才创建的容器

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba01d2ba19?w=1768&h=436&f=png&s=803028)

可以看到我们当前的容器是连接到了bridge的网络中。在我们的主机和容器之间会创建一对vethnet以便于容器和主机之间相互通信，在主机的终端上我们可以直接ping通容器的ip地址。

这时我们再创建一个容器

```bash
docker run -d --name test2 busybox /bin/sh -c "while true;do sleep 3600;done"
```



再次查看network的具体内容

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba020c84d6?w=1766&h=832&f=png&s=1555977)

同时test2也会创建一堆vethnet，test1和test2的vethnet端口会都连接在一个bridge上，所以他们两个之间可以相互ping通。

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba04dcad94?w=1400&h=716&f=png&s=113118)

在已经知道了两个容器之间可以相互访问的情况下，我们现在需要实现这样一个需求：

- 首先启动另一个容器db，在这个容器上运行数据库服务
- 同时启动一个容器web，这个容器中我们可以运行一个用户注册服务
- 用户点击注册后，web容器将数据存储到dbrongqi中

以上的这个操作必然要涉及两个容器的网络链接，我们知道可以通过IP地址相互ping通，但是在大规模集群的情况下不可能手动的去指定IP，这个时候我们如何让这两个容器知道对方的存在呢？

我们可以在创建容器时使用`--link`参数，来制定我们创建的这个容器要连接到哪个容器上。

```bash
docker run -d --name test2 --link test1 busybox /bin/sh -c "while true;do sleep 3600;done"
```

之后我们进入test2中，`ping test1`的IP地址和test1都可以ping通。也就是我们在使用`--link`时是创建的DNS服务，将test1映射到172.17.0.2上。

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba0506ceff?w=1284&h=738&f=png&s=1091403)

我们也可以手动创建一个network，在创建容器时将容器连接到我们自己创建的network上，此时如果我们在自建的网络上创建两个容器，他们两个之间是默认`link`好的。

### [Docker端口映射](#content)

我们启动一个容器里面一定会运行一些服务，但是这些服务我们如何才能让外界访问到呢？首先肯定是连接好网络，这之前我们已经说过了。另一个重要的就是端口映射。

假如说我们在服务器上启动nginx的服务，他默认是通过80端口来访问的，但是我们如何通过访问服务器的80端口来默认的访问到容器中的nginx服务呢？我们在运行容器时可以使用这个命令：

```bash
docker run --name web -d -p 80:80 nginx
```



我们通过-p参数来将容器的80端口映射到服务器的80端口上，这样我们访问服务器的80端口时就会直接请求nginx服务。

我们容器的IP地址：

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba0586cb20?w=1760&h=436&f=png&s=817116)

Linux服务器的IP地址分别是：

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba3a63053f?w=1636&h=92&f=png&s=116641)

然后我们访问Linux服务器的80端口：

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba45c4e6ba?w=1090&h=548&f=png&s=121872)

可以看到确实是可以成功映射的。

### [flask-redis实战](#content)

我们这个应用是使用flask搭建一个应用，这个应用是我们每访问一次网址就会在redis的数据库上加1。

首先我们创建一个启动redis服务的容器

```bash
docker run -d --name redis redis
```



之后我们编写一下python文件，名字叫做app.py

```python
from flask import Flask
from redis import Redis
import os
import socket

app = Flask(__name__)
redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)


@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Container World! I have been seen %s times and my hostname is %s.\n' % (redis.get('hits'),socket.gethostname())


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```



在创建Dockerfile，并编辑里面的内容

```dockerfile
FROM python:2.7
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 5000
CMD [ "python", "app.py" ]
```



然后根据Dockerfile创建我们自己的镜像

```bash
docker build -t su/flask-redis .
```



最后将我们自己创建的镜像加载成容器对外提供服务，并且将容器的5000端口映射到服务器的5000端口

```bash
docker run -d --link redis -p 5000:5000 --name flask-redis -e REDIS_HOST=redis su/flask-redis
```



我们来看一下效果，首次访问

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba557823c8?w=1282&h=56&f=png&s=15446)

再访问一次看一下效果

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba642b9c62?w=1284&h=56&f=png&s=14949)

可以看到我们已经实现了多容器应用的部署💯。

### [Docker多机网络](#content)

刚才我们已经成功的实现了flask-redis应用程序，但是还存在一个问题，我们的服务可能是访问量很大的一个服务，这时需要我们将redis和flask部署到不同的服务器上，我们怎么才能让这两个部署在不同服务器上的容器相互通信呢？

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba6ecf9727?w=2296&h=918&f=png&s=208184)

两个容器之间数据的传递使用的技术是VXLAN，他是一个overlay网络的实现，更多内容可以参考：[关于VLAN和VXLAN的理解](https://blog.csdn.net/octopusflying/article/details/77609199)。

想要实现两个容器进行通信我们需要一个技术来支持就是etcd，关于etcd的更多内容大家可以查看：[Etcd 使用入门](https://www.hi-linux.com/posts/40915.html)，[高可用分布式存储 etcd 的实现原理](https://draveness.me/etcd-introduction)。etcd 的官方将它定位成一个可信赖的分布式键值存储服务，它能够为整个分布式集群存储一些关键数据，协助分布式集群的正常运转。也就是我们的两台服务器要分别在etcd服务上进行注册，以便于互相识别。

### [flask-redis多机实战](#content)

接下来我们将之前的那个flask-redis实战转换为多机的实战

首先准备两台服务器，我这里准备了两台服务器分别是node1和node2，他们的ip地址分别是：`192.168.205.10`和`192.168.205.11`

接下来在node1节点上运行命令

```bash
vagrant@node1:~$ wget https://github.com/coreos/etcd/releases/download/v3.0.12/etcd-v3.0.12-linux-amd64.tar.gz
vagrant@node1:~$ tar zxvf etcd-v3.0.12-linux-amd64.tar.gz
vagrant@node1:~$ cd etcd-v3.0.12-linux-amd64
vagrant@node1:~$ nohup ./etcd --name docker-node1 --initial-advertise-peer-urls http://192.168.205.10:2380 \
--listen-peer-urls http://192.168.205.10:2380 \
--listen-client-urls http://192.168.205.10:2379,http://127.0.0.1:2379 \
--advertise-client-urls http://192.168.205.10:2379 \
--initial-cluster-token etcd-cluster \
--initial-cluster docker-node1=http://192.168.205.10:2380,docker-node2=http://192.168.205.11:2380 \
--initial-cluster-state new&
```

之后在node2上运行命令

```bash
vagrant@node2:~$ wget https://github.com/coreos/etcd/releases/download/v3.0.12/etcd-v3.0.12-linux-amd64.tar.gz
vagrant@node2:~$ tar zxvf etcd-v3.0.12-linux-amd64.tar.gz
vagrant@node2:~$ cd etcd-v3.0.12-linux-amd64/
vagrant@node2:~$ nohup ./etcd --name docker-node2 --initial-advertise-peer-urls http://192.168.205.11:2380 \
--listen-peer-urls http://192.168.205.11:2380 \
--listen-client-urls http://192.168.205.11:2379,http://127.0.0.1:2379 \
--advertise-client-urls http://192.168.205.11:2379 \
--initial-cluster-token etcd-cluster \
--initial-cluster docker-node1=http://192.168.205.10:2380,docker-node2=http://192.168.205.11:23
```

检查一下etcd的状态：

```bash
vagrant@node2:~/etcd-v3.0.12-linux-amd64$ ./etcdctl cluster-health
```

接下来我们要重启docker服务

在node1上运行命令

```bash
sudo service docker stop
sudo /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=etcd://192.168.205.10:2379 --cluster-advertise=192.168.205.10:2375&
```



在node2上运行命令

```bash
sudo service docker stop
sudo /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=etcd://192.168.205.11:2379 --cluster-advertise=192.168.205.11:2375&
```



接下来我们在node1上创建一个名为demo的overlay网络

```bash
docker network create -d overlay demo
```

我们现在在node1和node2上查看一下网络情况

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba699b2407?w=1478&h=264&f=png&s=434776)
![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba79d5156e?w=1462&h=270&f=png&s=440927)

我们虽然没有在node2上创建demo网络，但是通过etcd会同步的进行创建，这样我们两台服务器上都有了一个叫做demo的网络，接下来我们创建容器时就可以将demo作为容器的网络。

首先在node2上创建redis容器

```bash
docker run -d --name redis --net demo redis
```



接下来在node1上创建flask容器

```bash
docker run -d --net demo -p 5000:5000 --name flask-redis -e REDIS_HOST=redis su/flask-redis
```



接下来我们看一下实验结果

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba78dffec9?w=1280&h=56&f=png&s=15728)
![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/26/17116fba8411eeda?w=1296&h=58&f=png&s=15026)
可以看到我们已经实现了多容器应用的多机部署💯。