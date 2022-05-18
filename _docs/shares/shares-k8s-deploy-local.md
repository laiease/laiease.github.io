---
title:  K8s部署本地项目
permalink: /docs/shares-k8s-deploy-local/
---


#### 一、编写Dockerfile，打包本地镜像

```dockerfile
FROM openjdk:17-jdk

RUN mkdir /opt/stress-test

COPY ./target/stress-test-*.jar /opt/stress-test/stress-test.jar

WORKDIR /opt/stress-test
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "stress-test.jar"]
```

使用命令`docker build`将项目打包成docker镜像

```shell
docker build -t stress-test ./Dockerfile
```



#### 二、搭建本地私有化docker镜像仓库，将镜像上传到其中

1. 搭建本地私有化docker仓库

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

2. 搭建好私有化仓库后需要修改`daemon.json`文件，将本地私有化仓库的地址添加到文件中，然后加载配置文件并重启docker服务

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

3. 将本地镜像私有化docker仓库中

    1. 将本地镜像打标签

        `docker tag xxx:xxx IP:PORT/xxx:xxx`

    2. 上传镜像

        `docker push IP:PORT/xxx:xxx `

4. 查看本地仓库

    1. 查看本地仓库中的镜像名称`curl -XGET http://IP:PORT/v2/_catalog`

    2. 查看本地仓库中的某一镜像的版本`curl -XGET http://IP:PORT/v2/xxx/tags/list`

         

#### 三、编写项目的yaml文件。

1. postgres的启动文件

    ```yaml
    $ cat postgres.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: postgres
      name: postgres
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: postgres
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: postgres
        spec:
          volumes:
            - name: task-v-postgres
              hostPath:
                path: /data/kubernetes/postgresql
          containers:
            - image: postgres
              name: postgres
              ports:
                - containerPort: 5432
                  name: "db-server"
              env:
                - name: POSTGRES_PASSWORD
                  value: Laie123.com
                - name: PGDATA
                  value: /var/lib/postgresql/data/pgdata
              volumeMounts:
                - name: task-v-postgres
                  mountPath: /var/lib/postgresql/data
    ```

2. 项目文件

    ```yaml
    $ cat stress-test.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: stress-test
    spec:
      selector:
        matchLabels:
          run: stress-test
      replicas: 2
      template:
        metadata:
          labels:
            run: stress-test
        spec:
          volumes:
            - name: task-v-stress-test
              hostPath:
                path: /data/kubernetes/stress-test
          containers:
            - name: stress-test
              image: 10.13.14.43:5000/stress-test:latest
              ports:
                - containerPort: 8080
              volumeMounts:
                - name: task-v-stress-test
                  mountPath: /opt/stress-test
    ```

    使用命令`kubectl apply`来发布编写好的文件

    ```shell
    kubectl apply -f postgres.yaml/stress-test.yaml
    ```

    

#### 四、开放端口访问（service）。

1. 数据库开放端口文件

    ```yaml
    $ cat postgres-svc.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: postgresql-client-service
      labels:
        app: postgres
    spec:
      type: NodePort
      ports:
        - port: 5432
          targetPort: 5432
          nodePort: 30001
          protocol: TCP
      selector:
        app: postgres
    ```

2. 项目开放端口文件

    ```yaml
    $ cat stress-test-svc.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: stress-test
      labels:
        run: stress-test
    spec:
      type: NodePort
      ports:
      - port: 8080
        targetPort: 8080
        nodePort: 30002
        protocol: TCP
      selector:
        run: stress-test
    ```

    使用命令`kubectl apply`来发布编写好的文件

    ```shell
    kubectl apply -f postgres-svc.yaml/stress-test-svc.yaml
    ```

    

### 五、部署成功。

​	可以通过任意节点的IP:30002来访问系统。

