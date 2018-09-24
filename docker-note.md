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
docker container ls
# 显示所有容器（包括推出的容器）
docker container ls -a 

# docker rm = docker container rm
# 单个单个删除 容器
docker rm container-ID

# 显示所有容器的 ID，'-q' 只显示容器 ID 号
docker container ls -aq
# 删除多个容器
docker rm $(docker container ls -aq)

# 显示已经退出的容器 ID
docker container ls -f "status=exited" -q
# 删除已经退出的容器
docker rm $(docker container ls -f "status=exited" -q)

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

```
```
