##  Docker 简明教程

此教程来自熊猫酱对[官方文档](https://docs.docker.com/get-started/)的翻译和优质博客的学习，不涉及 dockfile 的制作

---

### 1.为什么要用 docker？

> 软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？

![](https://raw.githubusercontent.com/PandaClark/thinpanda-blog/master/img/docker.png)

想象一下以下几个场景：

当你辛辛苦苦完成一个项目，打算部署到服务器上时，发现自己按照同样的配置过程在服务器上进行环境部署，却运行不起来…...

当你想要将自己的代码运行在同学或者同事的电脑上时，发现"我的机器上就能跑，而你的却不行"…...

当你想要学习大数据、人工智能和区块链，光 Hadoop 的环境配置就足够让人焦头烂额…...(Hadoop 中有个 yarn.sh 文件，而熊猫酱最近在使用的 React中也有个 yarn.sh，没有 docker 无法想象如何让两者并存)

总而言之，docker 解决了"在我的机器上能行，在你的机器上却不行"的问题。

### 2.docker 和虚拟机的区别

docker 能解决上述问题的原因是 docker 不仅将你的项目打包成了镜像(image)，并且将你项目中所使用的依赖也打包在同一个镜像之中，使得你的项目能够自带环境运行，而不是使用宿主机的环境。

早在 docker 出现之前，就有虚拟机的解决方案，刚开始接触 Linux 的同学也大都不是直接操作服务器，而是先在 Windows 电脑上安装一个 Ubuntu 的虚拟机。

那为什么我们要抛弃常用的虚拟机，而去使用 docker 呢？

docker 相比较虚拟机有如下几个优点：

![image-20190603102821425](https://raw.githubusercontent.com/PandaClark/thinpanda-blog/master/img/docker_vs_vm.png'docker和虚拟机的比较')

![](https://raw.githubusercontent.com/PandaClark/thinpanda-blog/master/img/dockerVSvm.png)

### 3.docker 是什么？

> **Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**它是目前最流行的 Linux 容器解决方案。

docker 通过 namespace 实现镜像中的环境与宿主机的环境的隔离，通过 cgroup实现一个进程组群资源（如CPU、内存、磁盘输入输出等）的控制与分离，从而不会和其他进程相互干扰，抢夺系统资源

### 4.docker 安装

个人安装 docker 其实是安装 Docker CE，建议去[官网下载](https://docs.docker.com/install/)安装，并仔细阅读文档。

注意：Linux 下面有个坑，Docker 需要用户具有 sudo 权限，为了避免每次命令都输入`sudo`，可以把用户加入 Docker 用户组

安装完成之后校验是否安装成功：

```shell
$ docker --version
Docker version 18.09, build c97c6d6

$ docker-compose --version
docker-compose version 1.24.0, build 8dd22a9

$ docker-machine --version
docker-machine version 0.16.0, build 9ba6da9
```

注：也可用`docker version`查看详细信息，不带`—-`输出的是详细信息

### 5.几个概念的辨析

- docker

指 docker client 这个守护进程(daemon)

- docker image

打包好的带环境的镜像

- docker container

由镜像产生的可以运行的实例，一个镜像可以产生多个实例，设置不同的 name，每个镜像都会被分配不同的 container id 作为唯一标识

### 6.先开始一个 helloworld

```shell
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

在这个过程中发生了什么？

1. 首先，docker 去本地找 hello-world 镜像

2. 没有找到，于是去默认的仓库拉取 latest 版本的镜像(因为`docker run`未指定镜像版本，默认就会选择 latest)

3. 拉取成功，对镜像完整性进行校验，查看是否在传输过程中被篡改
4. 校验成功，执行`docker run`命令，产生一个  hello-world image 的container 实例

### 7.详细的 docker 操作

假如你现在已经有一个 docker 镜像，名字叫做`friendlyhello`，它设置的访问端口为 80，运行的结果是在网页上输出`hello world!`，镜像运行起来以后它只是主机上的一个进程，不可能直接通过80 端口访问到，需要进行映射。比如我想在主机的 4000 端口上访问这个镜像，那么命令应该是：

```shell
# -p 选项指定了端口的映射
$ docker run -p 4000:80 friendlyhello
```

官网相关的说明如下：

>You should see a message that Python is serving your app at `http://0.0.0.0:80`. But that message is coming from inside the container, which doesn’t know you mapped port 80 of that container to 4000, making the correct URL`http://localhost:4000`.

现在，shell 中已经成功启动这个镜像，但是又出现一个问题：这个镜像一直在 shell 的前台运行，你无法做其他的和这个镜像无关的事情，关闭这个 shell 会导致镜像停止，为了能做其他事，你只好又重开一个 shell。对于一个只提供服务的镜像（比如 Nginx，Apache）来说，开着 shell 又不使用，简直是一场灾难。

解决的办法一如既往：让镜像在后台运行。

```shell
# -d 选项让 docker 镜像在后台运行
$ docker run -d -p 4000:80 friendlyhello
```

也可以通过`--name`选项在启动的同时给 container 一个名字

```shell
# --name 选项命名这个镜像为 myhello
$ docker run -d --name myhello -p 4000:80 friendlyhello
```

如果是需要用到 shell 进行交互的镜像(例如在：Ubuntu)，为了使主机上的 shell 命令链接到 docker 内，需要使用`-it`参数

```shell
$ docker run -it ubuntu:16.04 /bin/bash
```

如何关闭后台运行的镜像？

```shell
# 列出正在运行的镜像的信息，获取 CONTAINER ID
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
# 根据 CONTAINER ID 让镜像停止
$ docker container stop 1fa4ab2cf395
```

如果在启动的同时给了 container 一个名字，那么就可以用这个名字来终止 container 而不需要知道 CONTAINER ID

```shell
# 根据 NAMES 来删除一个 container
$ docker container stop myhello
```

### 8.一份 docker 常用操作的小抄

下面列出了一份常见 docker 操作的命令：

```shell
## List Docker CLI(client) commands
$ docker
$ docker container --help

## Display Docker version and info
$ docker --version
$ docker version
$ docker info

## Execute Docker image
$ docker run hello-world
$ docker run -it ubuntu:16.04 /bin/bash
$ docker run -d -p 4000:80 --name myhello friendlyhello

## List Docker images
$ docker image ls

## Remove Docker images (specified, all)
$ docker image rm <image id>
$ docker image rm $(docker image ls -a -q)

## List Docker containers (running, all, all in quiet mode)
$ docker container ls
$ docker container ls -all
$ docker container ls -aq

## Stop Docker containers (gracefully stop, force shutdown)
$ docker container stop <hash/name>
$ docker container kill <hash/name>

## Remove Docker container
$ docker container rm <hash>
$ docker container rm $(docker container ls -a -q)

$ docker build -t friendlyhello .  # Create image using this directory's Dockerfile
$ docker run -p 4000:80 friendlyhello  # Run "friendlyhello" m docker run -d -p 4000:80 
$ docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
```



