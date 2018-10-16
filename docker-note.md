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

# 停止运行的容器
# docker stop = docker container stop
docker stop container-ID/container-name

# 单个单个删除容器 - 容器必须是退出状态
# docker rm = docker container rm
docker rm container-ID/container-name

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

# 查看容器详细信息
# docker inspect = docker contaienr inspect
docker inspect container-ID

# 查看容器运行时产生的日志
# docker logs = docker container logs
docker logs container-ID
```

## docker run 相关操作
```
# 交互式运行一个 image
docker run -it image-name

# 后台执行
docker run -d image-name

# 定义名字运行容器
# 定义名字的容器是唯一的
docker run -d --name=demo image-name
docker stop demo
docekr rm demo
docker start demo

# 类似网络 NDS ，通过名称连接容器之间的网络
docker run -d --name test1 image-name
dokcer run -d --name test2 --link test1 image-name
# 在 test2 容器中可以直接使用容器 test1 来访问网络
docker exec -it test2 ping test1

# 启动时指明使用的网络
# 新的网络可以通过 docker network create 来创建
docker run --network network-name image-name

# 端口映射
# 将容器的 80 端口映射到 docker 主机的 80 端口
docker run -p 80:80 image-name

# 设置环境变量
# 设置 demo 的环境 REDIS_HOST 的值为 redis 容器的 IP 地址
docker run --name demo --link redis -e REDIS_HOST=redis image-name

# 指定 volume 
# 在 Dockerfile 中定义 VOLUME ["/var/lib/mysql"]
# 指定从 mysql 镜像中创建的容器的 volume 在 /var/lib/mysql 中
docker run -v mysql:/var/lib/mysql --name mysql1 mysql
```

## docker exec 相关操作
```
# 进入一个运行中的 container 中
# 交互式进入后执行 /bin/bash 进行命令操作
docker exec -it container-ID /bin/bash
# 交互式进入后执行 python 进入 python 解释器
docker exec -it container-ID python
# 交互式进入后执行 ip a 查看运行中的容器 IP 地址
docker exec -it container-ID ip a
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

# CMD 和 ENTRYPOINT 结合使用
ENTRYPOINT ["/usr/bin/ls"]
CMD []
# 创建一个以上两个语句组成的 image 后执行
docker run iamge --> 输出当前目录文件
docker run iamge -a --> 输出当前目录所有文件
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

## 容器物理资源限制
### 内存 - Memory
> 只设置 memory 那么 memory-swap 默认等于 memory，即占用两倍设置的 memory 

```
# 设置 memory 200M 和 memory-swap 200M 一共最多可使用 400M 内存
# docker run --memory=200M = docker run -m 200M
docker run --memory=200M image-name
```

### CPU
> --cpu-shares 并不是设置 CPU 个数，而是设置相对权重  
> CPU 个数多的容器会优先去使用资源的 CPU  
> 在主机 CPU 跑满时，会根据设置的 CPU 个数呈现一个百分比关系  

```
# docker run --cpu-shares=5 = docker run -c 5
docker run --cpu-shares=5 image-name
```

## docker-machine 操作
```
# 列出所有docker 虚拟机
docker-machine ls

# 创建一个 docker 虚拟机
docker-machine create docker-machine-name

# 启动一个 docker 虚拟机
docker-machine start docker-machine-name

# 登录一个 docker 虚拟机
docker-machine ssh docker-machine-name

# 删除一个 docker 虚拟机
docker-machine rm docker-machine-name
```

## docker network
```
# 显示本机所有的 docker 网络
docke network ls

# 显示使用对应 docker 网络的容器
docker network inspect docker-network-id/docker-nwtwork-name
```

### bridge 网络
> 连接到自己创建的 bridge 网络中的容器，默认已经相互 link，可以直接使用容器名称来访问

```
# 创建一个 docker birdge 网咯
# docker network create -d 驱动名称 新的网络名称
docker  network create -d bridage my-bridage

# 将容器连接到指定的网络
docker network connect network-name container-name
```

### none 网络
> 安全性高

```
# 谁都不能访问的容器
docker run --network none image-name
```

### host 网络
> 容器出现端口冲突

```
# 和主机共享一套网络
docker run --network host image-name
```

### overlay 网络 - 实现多机通信
> 该网络基于 VXLAN 方式实现类似隧道的功能

使用 overlay 网络加分布式存储 etcd 实现

## 持久化存储和数据共享
### 持久化数据的方案
#### 基于本地文件系统的 volume
可以在执行 Docker create 或 Docker run 时，通过 -v 参数将主机的目录作为容器的数据卷。这部分功能便是基于本地文系统的 volume 管理
* 受管理的 data volume，由 docker 后台自动创建
* 绑定挂载的 volume，具体挂载位置可以由用户指定

```
# 查看所有 docker 管理的 volume
docker volume ls
# 删除一个 volume
docker volume rm volume-name
# 查看 volume 详细信息
docker volume inspect volume-name
```

##### data volume 方式
> 需要在 Dockerfile 中定义好存储的目录

```
# 在 Dockerfile 中定义 VOLUME ["/var/lib/mysql"]
# 指定从 mysql 镜像中创建的容器的 volume 在 /var/lib/mysql 中
docker run -v mysql:/var/lib/mysql --name mysql1 mysql
```

##### Bind Mouting 方式
> 不需要在 Dockerfile 中定义   
> 目录映射的关系  
> Centos 7 下 使用 chcon -Rt httpd_sys_content_t . 关闭文本安全  
> 或者使用 docker run --privileged=true 加特权

```
# 定义主机目录和容器目录
# -v 主机目录:容器目录
docker run -v /home/aaa:/root/aaa image-name
```

#### 基于 plugin 的 volume
支持第三方的存储方案，比如 NAS，aws

## docker-compose 相关操作 - 默认使用 docker-compose.yml 文件名
- Docker Compose 是一个工具  
- 这个工具可以通过一个 yml 文件定义多容器的 docker 应用  
- 通过一条命令就可以根据 yml 文件的定义去创建或者管理这个多容器  

### yml 文件编写规则
#### Services
> 一个 services 代表一个 container，这个 container 可以从 dockerhub 的 image 来创建，或者从本地的 Dockerfile build 出来的 image 来创建  
>  Service 的启动类似 container run，我们可以给其指定 network 和 volume，所以可以给 service 指定 network 和 volume 的应用  

```
# 以下作用和 docker run -d --network back-tier -v db-data:/var/lib/postgresql/data/ postgres:9.4 命令相同
services:
  db:
    image: postgres:9.4
	volumes:
	  - "db-data:/var/lib/postgresql/data"
	networks:
	  - back-tier
# 以下作用是从 ./worker 目录中的 Dockerfile 文件创建的 image 连接 db 和 reids 并连接在 back-tier 网络中
services:
  worker:
    build: ./worker
	links:
	  - db
	  - redis
	networks:
	  - back-tier
```

#### Networks
```
# 以下作用和 docker network create -d bridge front-tier && docker network create -d bridge back-tier 命令相同
networks:
  front-tier:
    driver: bridge
  back-tier:
    driver: bridge
```

#### Volumes
```
# 以下作用和 docker volume create db-data 命令相同
volumes:
  db-data:
```

#### 完整的例子
```
version: '3'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-bridge

  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-bridge

volumes:
  mysql-data:

networks:
  my-bridge:
    driver: bridge
```

### docker-compose 命令行工具使用方法
> 用于本地开发，不适用于生产环境

```
# 前台启动 container
docker-compose up

# 后台启动 container
docker-compose up -d

# 删除停止的 container
docker-compose down

# 查看运行的 docker compose
docker-compose ps

# 指定启动容器的个数
docker-compose up --scale container-name=num

# 预构建 Dockerfile 
docker-compose build
```

## Swarm mode - 容器编排
- 管理更多容器
- 方便横向扩展
- 容器 down 了自动恢复
- 更新容器而不影响业务
- 监控追踪容器
- 调度容器的创建
- 保护隐私数据

### 初始化
> 先初始化 manager 节点  
> worker 节点使用生成的 token 进行添加

```
# 进入 manager 节点创建 swarm manager
docker swarm init --advertise-addr=swarm-manager-ip-addr

# 进入 worker 节点加入 swarm manager
docker swarm join --token swarm-manager-join-token swarm-mananger-ip:swarm-manager-port

# 在 manager 节点管理 join token
docker swarm join-token manager 
```

### node 命令
```
# 显示当前 swarm 节点
docker node ls
```

### service 命令
> 在 swarm 模式下一般不用 run 而使用 service

```
# 创建一个 service，及运行一个 container 并运行
docker service create --name demo iamge

# 查看 service
docker service ls

# 删除 service
docker service rm service-name

# 查看具体 service 内容
docker service ps service-name

# 扩展 service
docker service scale service-name=num
```

mode - 模式
- replicated 可以可以横向扩展 
- global 不能做横向扩展

Routing Mesh 的两种体现
- Internal - Container 和 Container 之间的访问通过 overlay 网络 （通过 VIP 虚拟 IP）
- Ingress - 如果服务有绑定端口，则此服务可以通过任意 swarm 节点的相应接口访问

### docker-compose.yml 中配置 deploy
> [官方文档](https://docs.docker.com/compose/compose-file/#deploy)

```
deploy:

  # 定义模式为 replicated 可以横行扩展
  mode: replicated

  # 定义横向扩展 3 个容器
  replicas: 3
	
  # 配置运行位置
  placement:
    # 指定容器运行的节点
    constraints:
      - node.role == manager 

  # 配置重启方案
  restart_policy:
    # 条件
    condition: on-failure
	# 延迟
	delay: 5s
	# 最大尝试次数
	max_attempts: 3

  # 更新配置
  update_config:
    # 同时更新个数
    parallelism: 2
	# 延迟
    delay: 10s
	# 顺序
    order: stop-first
```

### docker stack 部署 docker-compose.yml
```
# 启动stack
docker stack deploy stack-name --compose-file=docker-compose.yml

# 查看所有 stack
docker stack ls

# 查看具体的 stack 运行详情
docker stack ps stack-name

# 查看概括的 service 运行情况
docker stack service stack-name
```
