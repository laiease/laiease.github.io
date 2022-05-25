---
title:  Docker基础
permalink: /docs/docker-introduce/
---

## Docker基础

### 简介

​        Docker 是一个开源的、轻量级的容器引擎，用于创建、管理和编排容器。可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在个人电脑上编译测试通过的容器可以批量地在生产环境中部署。

**Docker通常用于如下场景** ：

- web应用的自动化打包和发布；
- 自动化测试和持续集成、发布；
- 在服务型环境中部署和调整数据库或其他的后台应用；


###  基本概念

#### 1. 镜像（Image）

**Docker 镜像** 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 **不包含** 任何动态数据，其内容在构建之后也不会被改变。

#### 2. 容器（Container）

镜像（`Image`）和容器（`Container`）的关系，就像操作系统的`安装文件`和我们使用的`操作系统`一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 [命名空间](https://en.wikipedia.org/wiki/Linux_namespaces)。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。

按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 **数据卷（Volume）** 、或者 **绑定宿主目录** ，在这些位置会直接对宿主机发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

#### 3. 仓库（Repository）

镜像构建完成以后，如果想要在其他宿主机上使用这个镜像，一种方式是将镜像导出，再导入到其他宿主机中；另一种方式就是使用 **Docker Registy** 服务，将镜像进行统一的存储和分发。

一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。

**Docker Registry**又分为两种：

- 公开服务

    Docker官方的 [Docker Hub](https://hub.docker.com/)、Google 的 [Google Container Registry](https://cloud.google.com/container-registry/)，其中Docker Hub也是Docker默认的Registry。

    由于一些不可抗力，从 *Docker Hub* 中下载镜像会比较缓慢，推荐配置**镜像加速器**，目前国内很多云服务商都提供了加速器服务，例如：

    - [阿里云加速器](https://www.aliyun.com/product/acr?source=5176.11533457&userCode=8lx5zmtu)
    - [网易云加速器](https://www.163yun.com/help/documents/56918246390157312)
    - [百度云加速器](https://cloud.baidu.com/doc/CCE/s/Yjxppt74z#使用dockerhub加速器)
    
- 私有服务

    Docker 官方提供了 [Docker Registry](https://hub.docker.com/_/registry/) 镜像，可以直接使用做为私有 Registry 服务。但是此服务只支持命令的方式来使用，没有图形界面、用户管理和权限控制等功能。

**Docker使用步骤：**

1. 获取镜像

   1. 从仓库下载镜像
   2. 本地构建镜像(Dockerfile)

2. 基于镜像启动容器

3. 将私有镜像上传到Repository，供他人使用

   

### 安装

#### 1. 离线安装

[Docker一键安装](https://gitlab.laiease.com/framework/docker-install)

#### 2. 在线安装

- [Install Docker Engine](https://docs.docker.com/engine/install/)
- [Install Docker Compose](https://docs.docker.com/compose/install/)



### 操作镜像

#### 1. 获取镜像

使用`docker pull`从镜像仓库中获取镜像，格式为：

```shell
$ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

> - Docker 镜像仓库地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub(`docker.io`)。
>
> - 仓库名：这里的仓库名是两段式名称，即 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。

例如：

```shell
$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
7b1a6ab2e44d: Pull complete 
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

> 命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub （`docker.io`）获取镜像。而镜像名称是 `ubuntu`，没有给出标签，则使用默认标签`latest`，因此将会获取官方镜像 `library/ubuntu` 仓库中标签为 `latest` 的镜像。`docker pull` 命令的输出结果最后一行给出了镜像的完整名称，即： `docker.io/library/ubuntu:latest`。

#### 2. 列出镜像

使用`docker image ls` 命令可以列出本地镜像，格式为:

```shell
$ docker image ls [OPTIONS] [REPOSITORY[:TAG]]
```

如果不给出仓库名，则默认查询所有的本地镜像，例如：

```shell
$ docker image ls
REPOSITORY                  TAG                      IMAGE ID       CREATED         SIZE
ubuntu                      latest                   d2e4e1f51132   2 weeks ago     77.8MB
```

>- SIZE的大小和Docker Hub中的大小是不一样，Docker Hub中存储的镜像是压缩后的，显示的下载所需的流量大小，本地显示的是实际大小。

#### 3. 删除镜像

使用`docker rmi`命令可以删除本地镜像，格式为：

```shell
$ docker rmi [OPTIONS] IMAGE [IMAGE...]
```

> 可以同时删除多个镜像，使用空格隔开，且可以使用仓库名或者镜像ID进行删除。

例如：

```shell
$ docker rmi d2e4e1f51132 nginx
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:26c68657ccce2cb0a31b330cb0be2b5e108d467f641c62e13ab40cbec258c68d
Deleted: sha256:d2e4e1f511320dfb2d0baff2468fcf0526998b73fe10c8890b4684bb7ef8290f
Deleted: sha256:e59fc94956120a6c7629f085027578e6357b48061d45714107e79f04a81a6f0c
Untagged: nginx:latest
Untagged: nginx@sha256:2d17cc4981bf1e22a87ef3b3dd20fbb72c3868738e3f307662eb40e2630d4320
Deleted: sha256:de2543b9436b7b0e2f15919c0ad4eab06e421cecc730c9c20660c430d4e5bc47
Deleted: sha256:e6643c9a23accf5c80d42301f00a7678bcd52883940e942d164441c967cdb5ad
Deleted: sha256:844c495fab6e4bfd0ae042a10fa4588adc0ffd0339a2952f58b0bbdd942b8c1c
Deleted: sha256:2deefd67059cad4693c3ec996e7db36c70b4b0aa77d6401ece78f7a6448a976c
Deleted: sha256:d0811e76f3fb47605235346df6d4c1d6877506b2019f5827d0a883822796acd0
Deleted: sha256:31352a5f24cf5e9926c20bebde4222154b95e1d060c2c57209c25537cd08021d
Deleted: sha256:fd95118eade99a75b949f634a0994e0f0732ff18c2573fabdfc8d4f95b092f0e
```

> 同时删除了镜像ID为`d2e4e1f51132`的镜像和`nginx:latest`镜像。

**可以使用`docker image ls`**命令来配合删除，例如：

```shell
$ docker image rm $(docker image ls -q ubuntu)
```

> 删除所有ubuntu镜像

#### 4. 使用Dockerfile构建镜像

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

```dockerfile
FROM openjdk:17-jdk

RUN mkdir /opt/stress-test

COPY ./stress-test.jar /opt/stress-test/stress-test.jar

WORKDIR /opt/stress-test
EXPOSE 8080

CMD ["java", "-jar", "stress-test.jar"]
```

> Dockerfile 尽量合并在一个指令中去写，尤其是`run`指令。 

使用命令`docker build`将项目打包成docker镜像

```
$ docker build -t stress-test -f ./Dockerfile .
```

> 如果在Dockerfile所在目录运行的话。也可以使用`docker build -t stress-test .`

#### 5. 镜像的导入导出

- 导出镜像

    使用命令`docker save`进行镜像的导出, 格式为：

    ```shell
    $ docker save [REPOSITORY[:TAG]]
    ```

    可是将其保存为文件，例如

    ```shell
    $ docker save ubuntu:18.04 > ubuntu-1804.tar
    ```

    > 命令成功后没有返回结果，在查看文件的保存目录即可看到文件。

- 导入镜像

    使用命令`docker load`进行镜像的导入， 格式为：

    ```shell
    $ docker load [OPTIONS]
    ```

    使用导出的文件，将镜像导入：

    ```shell
    $ cat ubuntu-1804.tar | docker load
    3e549931e024: Loading layer [==================================================>]  65.53MB/65.53MB
    Loaded image: ubuntu:18.04
    ```

    > 此时再查看镜像就可以发现镜像被导入。

### 操作容器

#### 1. 查看容器

使用命令`docker ps`命令来查当前宿主机存在的容器，格式为：

```shell
$ docker ps [OPTIONS]
```

通常，我们使用`docker ps -a`来查看所有的容器：

```shell
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                     PORTS                  NAMES
6765a8b1135f   nginx:latest   "/docker-entrypoint.…"   5 seconds ago    Up 4 seconds               0.0.0.0:8081->80/tcp   nginx2
12f0a748b4ff   nginx:latest   "/docker-entrypoint.…"   32 minutes ago   Exited (0) 7 minutes ago                          nginx
```

> `STATUS`代表着当前容器的运行状态, 状态以`Exited`开头表示处于终止状态；`Up`表示处于运行状态。
>
> `PORTS`表示当前容器所有映射的到宿主机的端口。

#### 2. 启动容器

Docker 容器启动有两种方式，一种是新建一个容器并启动，一种是将终止状态的容器重新启动。

- 新建并启动`docker run`

    使用`docker run`命令可以基于镜像新建一个容器并启动，格式为:

    ```shell
    $ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
    ```

    > docker run有很多参数，这里介绍一些常用参数：
    >
    > `-i`:以交互模式运行容器，通常与 -t 同时使用
    >
    > `-t`:为容器重新分配一个伪输入终端，通常与 -i 同时使用
    >
    > `-d`: 后台运行容器，并返回容器ID；
    >
    > `-v`: 目录映射，**宿主机目录:容器内部目录**
    >
    > `-p`: 端口映射，**宿主机端口:容器内部端口**
    >
    > `--restart`: 重启策略，always代表每次容器退出都重启
    >
    > `--name`: 启动后的容器名称

    例如：

    ```shell
    $ docker run -itd -p 8080:80 -v /Users/zhang/Download/nginx:/usr/share/nginx/html --name nginx --restart=always nginx:latest
    Unable to find image 'nginx:latest' locally
    latest: Pulling from library/nginx
    214ca5fb9032: Pull complete
    66eec13bb714: Pull complete
    17cb812420e3: Pull complete
    56fbf79cae7a: Pull complete
    c4547ad15a20: Pull complete
    d31373136b98: Pull complete
    Digest: sha256:2d17cc4981bf1e22a87ef3b3dd20fbb72c3868738e3f307662eb40e2630d4320
    Status: Downloaded newer image for nginx:latest
    12f0a748b4ff1733c90427787c1c1061f9021ae53856243a08300f12b616acb6
    ```

    > - 这里使用`nginx:latest	`镜像以后台运行的方式创建并启动了一个名为nginx的容器，将容器内部的80端口映射到了宿主机的8080端口，将容器内部的目录`/usr/share/nginx/html`映射到宿主机的`/Users/zhang/Download/nginx`。
    >
    > - 如果使用的镜像在本地是不存在的，那么会先将其下载下来，然后再运行。
    >
    > - 命令输出的最后一行为运行的容器的ID。

- 重新启动

    使用命令`docker start`命令可以让终止状态的容器重新启动，格式为：

    ```shell
    $ docker start [OPTIONS] CONTAINER [CONTAINER...]
    ```

    > 可以使用容器的ID或者容器的名称将容器重新启动。

    例如：

    ```shell
    $ docker start 12f0a748b4ff
    12f0a748b4ff
    ```

#### 3. 终止容器

使用命令`docker stop`来终止一个运行中的容器，格式为：

```shell
$ docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

> 可以使用容器的ID或者容器的名称将容器终止。

例如：

```shell
$ docker stop 12f0a748b4ff
12f0a748b4ff
```

#### 4. 进入容器

容器在新建的时候加上参数`-d`会使容器进入后台运行，这个时候我们想再进入容器的话就需要使用命令`docker attach` 或 `docker exec`。

- `docker attach`

    使用`docker attach`命令可以进入容器，格式为：

    ```shell
    $ docker attach [OPTIONS] CONTAINER
    ```

    > 使用`docker attach`退出后会导致容器进入终止状态。

- `docker exec`（推荐）

    使用`docker exec`命令可以进入容器，格式为：

    ```shell
    $ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
    ```

    > 这里需要着重注意一下`-i`和`-t`命令。只有同时使用这两个参数的时候才能是一个熟悉的Linux终端。
    >
    > 使用`docker exec`退出后不会导致容器进入终止状态，也是推荐的原因。

#### 5. 删除镜像

使用`docker rm`命令可以删除本地的容器，格式为：

```shell
$ docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

例如：

```shell
$ docker rm 12f0a748b4ff
12f0a748b4ff
```

> `docker rm`命令无法删除正在运行中的状态，需要先将容器终止或者需要使用参数`-f`强制进行删除。



### 私有仓库

#### 1. 安装私有仓库

此处我们使用Docker官方的registry安装私有化仓库。

```shell
$ docker run -itd -v /data/registry:/var/lib/registry -p 5000:5000 --restart=always --name registry registry:latest
Unable to find image 'registry:latest' locally
latest: Pulling from library/registry
79e9f2f55bf5: Pull complete 
0d96da54f60b: Pull complete 
5b27040df4a2: Pull complete 
e2ead8259a04: Pull complete 
3790aef225b9: Pull complete 
Digest: sha256:169211e20e2f2d5d115674681eb79d21a217b296b43374b8e39f97fcf866b375
Status: Downloaded newer image for registry:latest
b02c4ce152aae4828dae13f05a38b57bd1dd9189bbc1e5ac7108d1c6a9129ce3
$ docker ps -a
CONTAINER ID   IMAGE                                               COMMAND                  CREATED              STATUS                       PORTS                                       NAMES
b02c4ce152aa   registry:latest                                     "/entrypoint.sh /etc…"   About a minute ago   Up About a minute            0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   registry
```

> 默认情况下，registry的仓库会被创建在容器的`/var/lib/registry`目录下，我们使用`-v`将镜像文件存储到宿主机的`/data/registry`目录下。

#### 2. 配置仓库地址

```shell
$ vim /etc/docker/daemon.json
{
    ......
    "insecure-registries": ["IP:PORT"]
}
$ systemctl daemon-reload
$ systemctl restart docker
```

> 1. 默认docker拉取私有仓库是使用`https`协议，如果需要使用`http`就需要添加到`insecure-registries`中。
> 2. 所有使用此私有仓库的节点都需要此操作。

#### 3. 上传镜像

1. 将本地镜像打标签

    `docker tag xxx:xxx IP:PORT/xxx:xxx`

2. 上传镜像

    `docker push IP:PORT/xxx:xxx `

#### 4. 查看镜像

1. 查看本地仓库中的镜像名称`curl -XGET http://IP:PORT/v2/_catalog`

2. 查看本地仓库中的某一镜像的版本`curl -XGET http://IP:PORT/v2/xxx/tags/list`

#### 5. 下载镜像

`docker pull IP:PORT/xxx:xxx `



### 网络

#### 1. 外部访问容器

容器在创建时，可以使用`-p`参数将容器内的端口映射到宿主机中。通过访问宿主机相应的端口，就可以映射到容器中。

#### 2. 容器互联

使用 `--link` 参数来使容器互联，但是随着 Docker 网络的完善，建议大家将容器加入自定义的 Docker 网络来连接多个容器。

格式为：

```shell
$ docker network COMMAND
```

> - `connect` : 连接一个容器到已有的网络
> - `create` : 创建一个新的网络
> - `disconnect`: 断开一个网络中的容器
> - `ls` : 查看网络列表
> - `rm` : 删除一个或多个网络

