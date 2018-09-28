## custom centos base image
> 基于基础的 centos image 创建自己的 image

```
FROM centos
RUN yum update -y \
	&& yum install -y epel-release \
	&& yum install -y which \
	&& yum install -y tree
```
