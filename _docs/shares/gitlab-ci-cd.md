---
title:  Docker基础
permalink: /docs/gitlab-ci-cd/
---


## GitLab CI/CD基于Docker实现maven项目的自动化编译部署

### 介绍

#### CI/CD

- 持续集成（Continuous Integration，简称CI），每次在上传代码块到基于Git仓库时，持续集成会运行脚本去构建、测试、校验代码。
- 持续交付（Continuous Delivery，简称CD），在 **持续集成** 之后，持续交付将进行**手动部署**应用。
- 持续部署（Continuous Deployment，简称CD），在 **持续集成** 之后，持续部署将进行**自动部署**应用。

#### 原理

当开发者配置了 `GitLab CI/CD`，那么当开发者将代码提交（`commit`）之后，会触发 `CI/CD` 相关的一系列操作。 这一系列操作由 `GitLab Runner` 执行，相关配置记载于`.gitlab-ci.yml`文件中，执行的结果将在Gitlab页面中展示。每一次的提交（`commit`）将会出发一条**流水线（pipeline）**，流水线是不同**阶段（Stage）**的**任务（Job）**的一个集合。**阶段（Stage）**用于逻辑切割，同一阶段的任务以并行方式执行，阶段间是顺序执行，上一个阶段执行失败，下一个阶段将不会执行。`.pre` 为第一阶段（译为：之前） 和 `.post` 最后阶段（译为：提交时），这两个阶段不需要被定义，也无法被修改。



### GitLab Runner

`GitLab Runner` 是一项开源项目，用于执行**任务（Job）**，并将执行结果传输回Gitlab。

Runner 可安装在操作系统，也可以通过Docker的方式安装。当 Runner 安装后，需要将其注册在 GitLab 中，方可使用。Runner 有若干种执executor可供使用，如：Docker、Shell、SSH、Kubernetes等等。Runner 默认使用Shell，Shell模式下，所有构建都会发生在Runner安装的机器中，操作十分简单，但是缺点很多。

`.gitlab-ci.yml` 文件中通过 `tags` 关键词选择Runner。Runner 的相关配置在 `config.toml` 文件中记载。

#### 安装

使用 `Docker` 方式安装需要使用到 `gitlab/gitlab-runner` 镜像。

```shell
$ docker run -d --name gitlab-runner --restart always -p 8093:8093 -v /data/gitlab/gitlab-runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest
Unable to find image 'gitlab/gitlab-runner:latest' locally
latest: Pulling from gitlab/gitlab-runner
7b1a6ab2e44d: Pull complete 
5580ef77ebbe: Pull complete 
d7b21acbe607: Pull complete 
Digest: sha256:d2db6b687e9cf5baf96009e43cc3eaebf180f634306cdc74e2400315d35f0dab
Status: Downloaded newer image for gitlab/gitlab-runner:latest
a87e5e25bc1338ba4c14ad98b3c2143e6191870b27e8a74a91c658e44f1962d5
```

>`-v /data/gitlab/gitlab-runner/config:/etc/gitlab-runner`：Runner配置文件映射到宿主机，方便调整和查看配置
>
>`-v /var/run/docker.sock:/var/run/docker.sock`：为了让容器可以通过`/var/run/docker.sock`与`Docker`守护进程通信

#### 注册

```shell
$ docker run --rm -it -v /data/gitlab/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=7 revision=5316d4ac version=14.6.0
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):	

Enter the registration token:

Enter a description for the runner:
[c53b52b4e87d]: 
Enter tags for the runner (comma-separated):

Registering runner... succeeded                     runner=shoZu3fE
Enter an executor: parallels, shell, virtualbox, kubernetes, docker-ssh+machine, custom, docker, docker-ssh, ssh, docker+machine:
docker
Enter the default Docker image (for example, ruby:2.6):
alpine:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

> 注册共有6步：
>
> 1. 输入GitLab实例的URL
> 2. 输入注册的token
> 3. 输入Runner的描述
> 4. 输入Runner的标签
> 5. 选择执行器：因为我们是基于docker安装的runner，所以这里选择docker
> 6. 输入默认镜像：alpine:latest

其中是通过 `GitLab` 来找到的，首先进入到需要设置 `CI/CD` 的项目，然后按照下图流程即可找到。

<img src="C:\Users\zhang\AppData\Roaming\Typora\typora-user-images\image-20220614173622783.png" alt="image-20220614173622783" style="zoom:80%;" />

注册成功后，可以在`/data/gitlab/gitlab-runner/config`下找到Runner的配置文件`config.toml`

```toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "test CICD"
  url = "http://XXXXXXXXX/"
  token = "XXXXXXXXXXXXXXXXXXX"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
```

在runner配置文件中配置docker命令:

```js
"/data/gitlab/gitlab-runner/maven/.m2:/root/.m2", "/usr/bin/docker:/usr/bin/docker", "/var/run/docker.sock:/var/run/docker.sock"
```

> `/data/gitlab/gitlab-runner/maven/.m2:/root/.m2`将maven的依赖映射到宿主机，避免每次打包时都重新下载。
>
> `/usr/bin/docker:/usr/bin/docker`, `/var/run/docker.sock:/var/run/docker.sock`用于解决Docker镜像中操作宿主机Docker报错的问题。



### 配置

这里介绍一下一些常用的参数：

- `variables`:定义全局变量
- `stages`:pipeline的阶段列表，定义整个pipeline阶段
- `stage`:定义某个job的所在阶段
- `image`:指定一个基础Docker进行作为基础运行环境，比如:node,python,java
- `tags`:用于指定Runner，tags的取值范围是在该项目可惜可见的runner tags中，也就是前面我们设置的那个tag
- `script`:当前阶段需要执行的命令
- `artifacts`:保留文档。在每次 job 之前runner会清除未被 git 跟踪的文件。为了让编译或其他操作后的产物可以留存到后续使用，添加该参数并设置保留的目录，保留时间等。被保留的文件将被上传到gitlab以备后续使用。
- `dependencies`:任务依赖。指定job的前置job。添加该参数后，可以获取到前置job的artifacts。注意如果前置 job 执行失败，导致没能生成artifacts，则 job 也会直接失败。

示例：

```yml
variables:
  #指定maven本地仓库地址
  MAVEN_OPTS: "-Dmaven.repo.local=/root/.m2/repository"
  # 定义build shell
  BUILD_SHELL: "mvn clean package -Dmaven.test.skip=true $MAVEN_OPTS  --settings=/root/.m2/settings.xml"

stages:          # List of stages for jobs, and their order of execution
  - package
  - build
  - deploy

package-job:       # This job runs in the build stage, which runs first.
  stage: package
  image: maven:3.8.5-jdk-8
  tags:
    - runner-123
  script:
    - echo $CI_COMMIT_MESSAGE
    - echo `mvn --version`
    - echo $BUILD_SHELL
    - $BUILD_SHELL
  artifacts:
    paths:
      - target/*.jar

build-job:
  stage: build
  image: docker
  dependencies:
    - package-job
  tags:
    - runner-123
  script:
    - echo $CI_COMMIT_MESSAGE
    - java_file_src=$(ls target/ | sed "s:^:`pwd`/:");
      echo $java_file_src;
      echo 'build imgage begin';
      docker build --build-arg JAR_PATH=target/*.jar --build-arg PORT=8999  -t my-test . ;
      echo 'build image end';

deploy-job:      # This job runs in the deploy stage.
  stage: deploy
  image: docker
  tags:
    - runner-123
  script:
    - echo "Deploying application..."
    - if [ $(docker ps -aq --filter name=my-test) ];
      then docker rm -f my-test;
      fi;
    - docker run -d -p 8999:8999 --name my-test my-test:latest;
    - echo "Application successfully deployed."
```

> 需要注意的是：
> 
> 1. 是要自定义maven构建参数。指定 maven本地仓库地址，指定配置文件 settings.xml。这样可以在本地没有依赖的情况下，通过settings.xml中配置的中央仓库的地址去拉取依赖。
>
> 2. docker镜像。不要怀疑，镜像名字就是[docker](https://registry.hub.docker.com/_/docker)，要在docker中进行docker操作，需要docker in docker 。
>
> 3. artifacts 的配置，在job中传递构建物，使用artifacts是比较方便的，这里需要注意的就是在任务中取artifacts时的路径问题。

由于示例中使用了docker构建，这是提供一下Dockerfile的写法：

```dockerfile
FROM openjdk:8u322

RUN mkdir /opt/app

ARG JAR_PATH
COPY $JAR_PATH /opt/app/app.jar

WORKDIR /opt/app

ARG PORT
EXPOSE $PORT

CMD [ "java", "-jar", "app.jar" ]
```