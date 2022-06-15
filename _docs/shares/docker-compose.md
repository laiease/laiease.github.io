---
title:  Docker Compose
permalink: /docs/docker-compose/
---

## Docker Compose

### 简介

`Compose` 是一个用于定义和运行多容器 Docker 应用程序的工具。您可以通过 `Compose` 使用 YAML 文件来配置应用程序的服务。然后使用单个命令，您可以从配置中创建并启动所有服务。

`Compose` 适用于所有环境：生产、开发、测试以及 CI 工作流程。

使用 `Compose` 基本上是一个三步过程：

1. 使用 `Dockerfile` 定义应用程序的环境，以便可以在任何地方复制它。

2. 在 `docker-compose.yml` 中定义构成应用程序的服务，以便它们可以在隔离环境中一起运行。

3. 运行 `docker compose up`，`Docker Compose` 命令启动并运行整个应用程序。 也可以使用 `docker-compose` 二进制文件运行 `docker-compose up`。

`Compose` 具有用于管理应用程序整个生命周期的命令：

- 启动、停止和重建服务
- 查看运行服务的状态
- 流式传输正在运行的服务的日志输出   
- 在服务上运行一次性命令
   
`Compose` 中有两个重要的概念：

- 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。



### 安装

`Docker Compose` 的安装非常简单，`Docker Desktop for Mac/Windows` 自带 `docker-compose` 二进制文件，安装 Docker 之后可以直接使用。在 Linux 上从 [官方 GitHub Release](https://github.com/docker/compose/releases) 处直接下载编译好的二进制文件即可。

```shell
$ sudo curl -L https://github.com/docker/compose/releases/download/2.5.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 国内用户可以使用以下方式加快下载
$ sudo curl -L https://download.fastgit.org/docker/compose/releases/download/2.5.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

# 查看docker compose版本
$ docker-compose -v
Docker Compose version v2.5.0
```



### 使用

这里使用一个真实系统上的例子来给大家做一个演示

```yaml
version: "3"

networks:
  my_network:
    name: my_network

services:
  web:
    build: .
    image: stress-test:1.0.0
    container_name: stress-test
    ports:
      - "8090:8080"
    depends_on:
      - postgres
    networks:
      - my_network

  postgres:
    image: postgres
    container_name: postgres
    ports:
      - "15432:5432"
    volumes:
      - /data/postgresql/data:/var/lib/postgresql/data
    environment:
      - "POSTGRES_PASSWORD=1234567890"
    restart: always
    networks:
      - my_network
```

使用命令`docker compose up`运行`compose`文件，加上参数`-d`意为后台运行。

### 模板

模板是使用 `Compose` 的核心，涉及到的指令关键字也比较多。但是这里面大部分指令跟 `docker run` 相关参数的含义都是类似的。

下面我们介绍一些比较常用的模板指令：

#### version

这里指的是`Compose`文件格式的版本，下表中罗列了 `Compose` 文件版本支持特定的 `Docker` 版本

| **Compose file format** | **Docker Engine release** |
| ----------------------- | ------------------------- |
| 3.8                     | 19.03.0+                  |
| 3.7                     | 18.06.0+                  |
| 3.6                     | 18.02.0+                  |
| 3.5                     | 17.12.0+                  |
| 3.4                     | 17.09.0+                  |
| 3.3                     | 17.06.0+                  |
| 3.2                     | 17.04.0+                  |
| 3.1                     | 1.13.1+                   |
| 3.0                     | 1.13.0+                   |
| 2.4                     | 17.12.0+                  |
| 2.3                     | 17.06.0+                  |
| 2.2                     | 1.13.0+                   |
| 2.1                     | 1.12.0+                   |
| 2.0                     | 1.10.0+                   |

#### build

指定 `Dockerfile` 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 `Compose` 将会利用它自动构建这个镜像，然后使用这个镜像。

```yaml
version: '3'
services:
  webapp:
    build: ./dir
```

也可以使用 `context` 指令指定 `Dockerfile` 所在文件夹的路径，使用 `dockerfile` 指令指定 `Dockerfile` 文件名。

```YAML
version: '3'
services:

  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
```

#### image

指定为镜像名称, 如果镜像在本地不存在，`Compose` 将会尝试拉取这个镜像。

```yaml
image: ubuntu:18.04
image: sbg-nginx:1.0.0
```

> 如果和build同时使用，意为Dockerfile编译的镜像名称。

#### container_name

指定容器名称。

```yaml
container_name: sbg-nginx
```

>  指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

#### ports

暴露端口信息。

使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

```shell
ports:
 - "3000"
 - "8000:8000"
 - "127.0.0.1:8001:8001"
```

#### volumes

数据卷所挂载路径设置。可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)。

该指令中路径支持绝对路径和相对路径。

```yaml
volumes:
  - /data/mysql_data:/var/lib/mysql
  - ./binlog:/var/mysql/data
```

如果路径为数据卷名称，必须在文件中配置数据卷。

```yaml
version: "3"

services:
  mysql_db:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

#### depends_on

解决容器的依赖、启动先后的问题。以下例子中会先启动 `redis` `db` 再启动 `web`

```yaml
version: '3'

services:
  web:
    build: .
    depends_on:
      - db
      - redis

  redis:
    image: redis

  db:
    image: postgres
```

#### environment

设置容器内的环境变量。

```yaml
environment:
  - "POSTGRES_PASSWORD=1234567890"
```

#### networks

配置容器连接的网络。

```yaml
version: "3"
services:

  some-service:
    networks:
     - some-network

networks:
  some-network:
```



### 命令

对于 Compose 来说，大部分命令的对象既可以是项目本身，也可以指定为项目中的服务或者容器。如果没有特别的说明，命令对象将是项目，这意味着项目中所有的服务都会受到命令影响。

`docker compose` 的基本使用格式为:

```shell
$ docker compose [options] [COMMAND]
```

> - 如果安装的`Compose`版本为V2的话，可以是用命令`docker compose`，也可以使用二进制文件`docker-compose`。V1版本无法使用`docker compose`命令。

默认使用的是命令执行时所在的目录下的 `docker-compose.yaml` 文件，也可以通过`-f`参数指定文件，例如:

```shell
$ docker compose -f ./docker-compose.yml up -d
```

#### up

根据 `docker-compose.yaml` 文件启动项目，格式为：

```shell
$ docker compose up [options] [SERVICE...]
```

该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。

选项：

- `-d` 在后台运行服务容器。
- `--no-deps` 不启动服务所链接的容器。
- `--force-recreate` 强制重新创建容器，不能与 `--no-recreate` 同时使用。
- `--no-recreate` 如果容器已经存在了，则不重新创建，不能与 `--force-recreate` 同时使用。

#### down

此命令将会停止并删除 `up` 命令所启动的容器，并移除网络。

#### restart

格式为：

```shell
$ docker compose restart [options] [SERVICE...]
```

重启项目中的服务。

#### stop

格式为：

```shell
$ docker compose stop [SERVICE...]
```

停止已经处于运行状态的容器，但不删除它。通过 `docker-compose start` 可以再次启动这些容器。

#### start

格式为：

```shell
$ docker compose start [SERVICE...]
```

启动已经存在的服务容器。

#### scale

格式为 :

```shell
$ docker compose scale [options] [SERVICE=NUM...]
```

设置指定服务运行的容器个数。当指定数目多于该服务当前实际运行容器，将新创建并启动容器；反之，将停止容器。

#### rm

格式为 :

```shell
$ docker compose rm [options] [SERVICE...]
```

删除所有（停止状态的）服务容器。推荐先执行 `docker-compose stop` 命令来停止容器。

选项：

- ``-f` 强制直接删除，包括非停止状态的容器。
- `-v` 删除容器所挂载的数据卷。

