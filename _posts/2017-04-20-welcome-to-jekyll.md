---
layout: post
title: "Docker镜像制作"
---

>常用镜像源，Docker操作命令和镜像制作

# 1. Docker镜像源
## 1.1 清华源
```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu focal stable
```
## 1.2 阿里镜像源
```
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

# 2. Docker常用命令
```
# 创建镜像
sudo docker build -t <img> .
# 镜像打标签
sudo docker image tag ssh-agent:v1.8 <url>/ssh-agent:v1.8
# 镜像推送/拉取
su <user>/<passwd>
docker push <url>/ssh-agent:v1.8
docker image push <url>/ssh-agent:v1.8
docker image pull <url>/ssh-agent:v1.8

# 启动容器
docker run --device /dev/fuse --privileged -dit \
    -p 4444:22 \
    -v /data/mnt/docker/gitadmin/ccache:/home/gitadmin/ccache \
    -e "JENKINS_AGENT_SSH_PUBKEY=ssh-rsa <pub_key>" \
    --name=cibuild1.8 \
    <url>/ssh-agent:v1.8
# /bin/bash进入容器
sudo docker exec -it cibuild1.8 /bin/bash

# 宿主和容器数据传输
docker cp source_directory/. mycontainer:/path/to/container/
docker cp mycontainer:/path/to/file_in_container.txt destination_directory/

# 搜索镜像
docker search --limit 5 <img_name>
# 删除单个镜像
docker image rm my-image:v1
# 批量删除仓库名为repo_name的镜像
docker rmi $(docker images | grep "repo_name" | awk '{print $3}')
# 批量删除镜像:通过创建时间筛选
docker image rm $(docker image ls -q -f before=mongo:3.2)
# 条件查看容器
docker ps -qa --filter="ancestor=ci-report.x-ringtek.com:27009/public/jenkins/ssh-agent:v1.8"
# 批量删除exited状态的容器
docker rm $(docker ps -q -f status=exited)
```

# 3. Dockerfile镜像制作实例
```
FROM ubuntu:focal
LABEL maintainer="xxx@163.com" \
      version="0.2" \
      description="cicd docker environment 1.1"

# Generate locale C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
ENV LANG C.UTF-8
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY ./sources.list /etc/apt/
    
RUN apt-get update && apt upgrade -y && \
	apt-get install -y --no-install-recommends \
	ca-certificates language-pack-en apt-utils software-properties-common \
    openssh-server openssh-client bash vim patch netcat-traditional iputils-ping \
    less sed wget

RUN apt-get install -y gcc-11 gcc-11-multilib g++-11 g++-11-multilib \
	npm nodejs xxd \
	kconfig-frontends libpulse-dev:i386  libasound2-dev:i386 libasound2-plugins:i386 \
	libusb-1.0-0-dev  libusb-1.0-0-dev:i386 libmad0-dev:i386 libv4l-dev libv4l-dev:i386 \
	libuv1-dev libmp3lame-dev:i386 libmad0-dev:i386 libv4l-dev:i386 \
	qemu-system-arm qemu-efi-aarch64 qemu-utils nasm yasm \
	libdivsufsort-dev  libc++-dev libc++abi-dev \
	libprotobuf-dev protobuf-compiler protobuf-c-compiler \
	gcc-multilib g++-multilib zip busybox

RUN apt-get install -y python2 && ln -s /usr/bin/python2 /usr/bin/python
RUN apt-get install -y cppcheck rsync jq

RUN  locale-gen en_US.UTF-8 \
	&& dpkg-reconfigure locales \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR "${JENKINS_AGENT_HOME}"s
EXPOSE 22
ENTRYPOINT ["setup-sshd-unix"]
```
