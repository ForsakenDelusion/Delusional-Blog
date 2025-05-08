---
title: 基于Docker的Ubuntu开发容器构建指南
date: 2025-05-09 00:10:12 +08:00
filename: 2025-05-09-Docker-Dev-Box
categories:
  - Misc
tags:
  - Docker
  - Linux
dir: Misc
share: true
---
## 前言

使用Docker容器作为隔离的开发环境有很多优势。相比虚拟机，Docker容器启动更快、资源占用更少，而且可以方便地在不同机器间迁移。今天就分享一下我构建Ubuntu开发容器的经验，希望能给大家一些启发。

## 为什么选择容器化开发环境？

在日常开发中，我们经常会遇到这些问题：

- 在不同项目间切换时，环境依赖冲突
- 新团队成员配置环境耗时且容易出错
- 本地开发环境和生产环境不一致导致的"在我机器上能跑"问题
- 想尝试新工具但又不想污染本机环境

容器化开发环境能很好地解决这些问题，让我们专注于代码而不是环境配置。

## Dockerfile详解

下面是我整理的一个基于Ubuntu 22.04的开发容器Dockerfile，我会逐段解释：

```dockerfile
FROM swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/ubuntu:22.04
LABEL maintainer="Delusion233 <wangcy0205@gmail.com>"
# 环境变量设置 - 将相关变量分组提高可读性
ENV DEV_USER=user \
    UID=1000 \
    GID=1000 \
    DEF_PASSWD=password \
    TZ=Asia/Shanghai \
    LANG=en_US.UTF-8
# 时区设置 - 单独设置时区减少因时区问题引起的后续错误
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone
# 系统扩展 - unminimize可能耗时较长，单独执行便于调试
RUN yes | unminimize
# 安装必要软件包 - 添加了更新和清理步骤，是Docker最佳实践
RUN apt-get update && \
    apt-get install -y systemd sudo openssh-server bash-completion zsh git curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# 用户和权限配置 - 用户创建逻辑独立处理
RUN addgroup --gid $GID $DEV_USER && \
    adduser --uid $UID --gid $GID --gecos "" --disabled-password $DEV_USER && \
    usermod -aG sudo $DEV_USER && \
    echo "$DEV_USER:$DEF_PASSWD" | chpasswd
# 系统配置 - 已移除securetty相关配置
RUN systemctl mask systemd-resolved.service && \
    echo "LANG=$LANG" > /etc/default/locale
CMD ["systemd"]
```

### 基础镜像选择

我选择了华为云镜像源的Ubuntu 22.04作为基础镜像。使用国内镜像源可以加快构建速度，Ubuntu 22.04 LTS是目前较为稳定且有长期支持的版本。

### 环境变量配置

```dockerfile
ENV DEV_USER=user \
    UID=1000 \
    GID=1000 \
    DEF_PASSWD=password \
    TZ=Asia/Shanghai \
    LANG=en_US.UTF-8
```

这里设置了几个关键环境变量：

- `DEV_USER`：容器内的开发用户名
- `UID`和`GID`：用户和组ID，设为1000是为了与宿主机的普通用户保持一致，避免权限问题
- `DEF_PASSWD`：默认密码（注意：生产环境应该使用更复杂的密码或SSH密钥）
- `TZ`：设置时区为上海，解决日志时间不一致等问题
- `LANG`：设置语言环境

### 时区设置

```dockerfile
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone
```

单独设置时区可以避免因时区问题引起的各种奇怪错误，特别是在处理定时任务或日志时。

### 系统扩展

```dockerfile
RUN yes | unminimize
```

Docker中的Ubuntu镜像通常是最小化的，`unminimize`命令会还原被移除的文档和man手册等内容，使容器更适合交互式使用。注意这一步可能比较耗时，但对于开发环境来说很有必要。

### 安装必要软件包

```dockerfile
RUN apt-get update && \
    apt-get install -y systemd sudo openssh-server bash-completion zsh git curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

这里安装了一些基本的开发工具：

- `systemd`：作为容器的init系统
- `sudo`：允许用户获取临时特权
- `openssh-server`：启用SSH远程访问
- `bash-completion`、`zsh`：提升shell使用体验
- `git`、`curl`、`vim`：基本开发工具

最后的清理步骤是Docker最佳实践，可以减小镜像体积。

### 用户和权限配置

```dockerfile
RUN addgroup --gid $GID $DEV_USER && \
    adduser --uid $UID --gid $GID --gecos "" --disabled-password $DEV_USER && \
    usermod -aG sudo $DEV_USER && \
    echo "$DEV_USER:$DEF_PASSWD" | chpasswd
```

这段创建了一个普通用户，并将其添加到sudo组以获取管理权限。设置UID和GID为1000是为了与宿主机保持一致，便于文件共享。

### 系统配置

```dockerfile
RUN systemctl mask systemd-resolved.service && \
    echo "LANG=$LANG" > /etc/default/locale
```

屏蔽了`systemd-resolved.service`以避免可能的DNS冲突，并设置了默认的语言环境。

## 构建与运行

### 构建命令

```shell
docker build -t dev-container:v0.1 .
```

这个命令将当前目录下的Dockerfile构建为一个名为`dev-container:v0.1`的镜像。

### 运行命令

```shell
docker run -d \
  --name Ubuntu-Dev \
  --hostname Dev-Box \
  --privileged \
  --security-opt seccomp=unconfined \
  --security-opt apparmor=unconfined \
  -p 2225:22 \
  --restart=unless-stopped \
  dev-container:v0.1
```

这个运行命令值得仔细讲解：

- `-d`：在后台运行容器
- `--name Ubuntu-Dev`：给容器指定一个易于识别的名称
- `--hostname Dev-Box`：设置容器内的主机名
- `--privileged`：给予容器额外权限，以便运行systemd
- `--security-opt seccomp=unconfined`和`--security-opt apparmor=unconfined`：放宽安全限制，允许systemd等系统服务正常运行
- `-p 2225:22`：将容器内的22端口映射到宿主机的2225端口，用于SSH连接
- `--restart=unless-stopped`：容器会在退出时自动重启，除非手动停止

## 扩展与优化建议

### 1. 持久化数据

为了保留开发成果，建议使用卷挂载：

```shell
docker run -d \
  --name Ubuntu-Dev \
  --hostname Dev-Box \
  --privileged \
  --security-opt seccomp=unconfined \
  --security-opt apparmor=unconfined \
  -p 2225:22 \
  -v ~/dev-workspace:/home/user/workspace \
  --restart=unless-stopped \
  dev-container:v0.1
```

这样，在`~/dev-workspace`目录的所有内容都会映射到容器内的`/home/user/workspace`，即使容器被删除数据也不会丢失。

### 2. 开发工具定制

可以在Dockerfile中添加更多开发工具，例如：

```dockerfile
# 安装开发工具
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    python3 python3-pip \
    nodejs npm \
    # 添加更多你需要的工具... \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. 安全加固

生产环境中不建议使用`--privileged`和放宽的安全选项。如果不需要运行systemd，可以使用更安全的配置：

```shell
docker run -d \
  --name Ubuntu-Dev \
  --hostname Dev-Box \
  -p 2225:22 \
  -v ~/dev-workspace:/home/user/workspace \
  --restart=unless-stopped \
  dev-container:v0.1
```

同时，应该使用更强的密码或SSH密钥认证：

```dockerfile
# 在Dockerfile中使用ARG接收构建参数
ARG SSH_PUB_KEY=""
# 设置SSH密钥
RUN mkdir -p /home/$DEV_USER/.ssh && \
    echo "$SSH_PUB_KEY" > /home/$DEV_USER/.ssh/authorized_keys && \
    chown -R $DEV_USER:$DEV_USER /home/$DEV_USER/.ssh && \
    chmod 700 /home/$DEV_USER/.ssh && \
    chmod 600 /home/$DEV_USER/.ssh/authorized_keys
```

构建时指定公钥：

```shell
docker build --build-arg SSH_PUB_KEY="$(cat ~/.ssh/id_rsa.pub)" -t dev-container:v0.1 .
```

### 4. 容器网络优化

如果需要多个开发容器协同工作，可以创建自定义网络：

```shell
# 创建开发网络
docker network create dev-network

# 运行容器并连接到网络
docker run -d \
  --name Ubuntu-Dev \
  --network dev-network \
  # 其他参数... \
  dev-container:v0.1
```

### 5. 镜像压缩

可以使用多阶段构建减小镜像体积，或者使用`docker-slim`等工具压缩镜像。

## 使用方法

构建并运行容器后，可以通过SSH连接到容器：

```shell
ssh user@localhost -p 2225
```

默认密码是之前在Dockerfile中设置的`password`。

## 总结

通过这个定制的Ubuntu开发容器，我们可以快速搭建一个隔离的、一致的开发环境。容器化的开发环境不仅提高了开发效率，还避免了环境不一致导致的问题。

这个配置还有很多可以改进的地方，比如添加更多开发工具、配置dotfiles、优化网络等。希望这篇笔记能给大家一些启发，欢迎分享你的改进建议！

---

_声明：本文仅供学习参考，生产环境中请注意安全加固。_