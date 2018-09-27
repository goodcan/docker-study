# docker study note

## docker image 相关操作

```
# 显示所有 image
docker images = docker image ls

# 拉去 registry 中的已有 image
docker pull image-name

# 删除 image
docker rmi = docker image rm
docker rmi image-ID

# 根据一个 Dockerfile 构建一个 image （推荐使用该方式来创建，方便分享给他人使用） 
docker build = docker image build
# 制定文件 Dockerfile 创建 image
docker build -t create-image-name Dockerfile-path

# 查看 image 的创建历史
docker history image-ID
```

## docker container 相关操作

```
# 显示所有正在运行的容器
docker ps = docker container ls
# 显示所有容器（包括推出的容器）
docker ps -a

# docker rm = docker container rm
# 单个单个删除 容器
docker rm container-ID

# 显示所有容器的 ID，'-q' 只显示容器 ID 号
docker ps -aq
# 删除多个容器
docker rm $(docker ps -aq)

# 显示已经退出的容器 ID
docker ps -f "status=exited" -q
# 删除已经退出的容器
docker rm $(docker ps -f "status=exited" -q)

# 根据一个 container 的修改创建新的 image （不推荐使用该方式，建议使用 Dockerfile 的形式）
docker commit = docker container commit
docker commit old-container-namae new-image-name
```

## docker run 相关操作

```
# 交互式运行一个 image
docker run -it image-name
```

## Dockerfile 语法
### FROM 
> 尽量使用官方的 image 作为 base image （比较安全）

```
# 制作 base image
FROM scratch 

# 使用 base image 
FROM centos 
FROM UBUNTU:14.04
```

### LABEL
> Metadata 不可少

```
LABEL maintainer="caojiacan0618@gmail.com"
LABEL version="1.0"
LABEL description="This is description"
```

### RUN
> 为了美观，复杂的RUN请用反斜线换行<br>
> 避免无用分层，合并多条命令成一行

```
# 反斜线换行
RUN yum update && yum install -y vim \
	python-dev

# 注意清理 cache
RUN apt-get update && apt-get install -y perl \
	pwgen --no-install-recommends && rm -rf \
	/var/lib/apt/lists/*

RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```

### WORKDIR
> 作用类似 cd 命令<br>
> 用 WORKDIR，不要使用 RUN cd<br>
> 尽量使用绝对目录

```
WORKDIR /root

# 如果没有会自动创建 test 目录
# 以下三行将会输出 /test/demo
WORKDIR /test
WORKDIR demo
RUN pwd
```

### ADD and COPY
> 大部分情况，COPY 优于 ADD<br>
> ADD 除了 COPY 还有额外的功能（解压）<br>
> 添加远程文件/目录请使用 curl 或者 wget

```
# 将当前目录的 hello 文件添加到 image 的根目录下
ADD hello /

# 添加到根目录，ADD 会执行解压缩， COPY 不会执行解压缩
ADD test.tar.get /

# 讲当前目录的 hello 文件，添加到 /root/test/ 目录下
# 生成的容器中会存在一个 /root/test/hello 文件
WORKDIR /root
ADD hello test/

# 以下效果同上
WORKDIR /root
COPY hello test/
```

### ENV
> 尽量使用 ENV 增加可维护性 

```
# 设置常量 MYSQL_VERSION 为 5.6 
ENV MYSQL_VERSION 5.6 

# 引用常量
RUN apt-get install -y mysql-server="${MYSQL_VERSION}" \
	&& rm -rf /var/lib/apt/lists/*
```

### CMD and ENTRYPOINT
> RUN：执行命令并创建新的 Image Layer<br>
> CMD：设置容器启动后默认执行的命令和参数<br>
> ENTRYPOINT：设置容器启动时运行的命令

```
# 以下通过 docker run [image] 输出 hello $name 并不是想要结果
FROM centos
ENV name Docker
ENTRYPOINT echo "hello $name"

# 以下通过 docker run [image] 输出 hello Docker 正是想要的结果
FROM centos
ENV name Docker
ENTRYPOINT ["/bin/bash", "-c", "echo hello $name"]

# cmd 补充说明
# 容器启动时默认执行的命令
# 如果 docker run 制定了其他命令，CMD 命令被忽略
# 如果定义多个 CMD，只有最后一个会执行
# 以下通过 docker run [image] 输出 hello Docker 正是想要的结果
FROM centos
ENV name Docker
CMD echo "hello $name"

# entrypoint 补充说明
# 让容器以应用程序或者服务的形式运行
# 不会被忽略，一定执行
# 最佳实践：写一个 shell 脚本作为 entrypoint
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 27017
CMD ["mongod"]
```

### EXPOSE
```
# 使容器对外暴露 5000 端口
EXPOSE 5000
```

## 创建私有 Docker Hub
> 使用 [Docker Registry](https://docs.docker.com/registry/)

### [基本使用方法](https://docs.docker.com/registry/#basic-commands)
```
# 从 Docker Hub 拉取官方 registry image
docker pull registry
# 启动一个 registry container
docker run -d -p 5000:5000 --restart always --name registry registry:2
# 从 Docker Hub 拉取一个 ubuntu image
docker pull ubuntu
# 修改 ubuntu repository 名称为私有 Docker Hub 启动的主机 IP + PORT + / + name
docker tag ubuntu localhost:5000/ubuntu
# 推送 image 到私有 Docker Hub 上
docker push localhost:5000/ubuntu
```

### 校验
```
# 删除 localhost:5000/ubuntu 镜像
docker rmi localhost:5000/ubuntu
# 重新从私有 Docker Hub 上拉取
docker pull localhost:5000/ubuntu
```

### [常用 HTTP API V2 接口](https://docs.docker.com/registry/spec/api/)
* GET /v2/_catalogu 显示已有 image
