---
title: 利用Docker搭建傲来训练营环境
date: 2025-04-11 21:31:31 +08:00
filename: 2025-04-11-Docker-EulixOS
categories:
  - OpenCourse
tags:
  - EulixOS
dir: OpenCourse
share: true
---

## Docker镜像构建

```Dockerfile
FROM swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/ubuntu:24.04

# Install necessary packages
RUN apt-get update && apt-get install -y \
    openssh-server \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Configure SSH
RUN mkdir /var/run/sshd
# Allow root login with password
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# Enable password authentication
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Set root password (change 'your_password' to your desired password)
RUN echo 'root:your_password' | chpasswd

# Expose SSH port
EXPOSE 22

# Start SSH service
CMD ["/usr/sbin/sshd", "-D"]
```

构建镜像，运行容器

```shell
docker build -t ubuntu-eulix .
docker run --restart=unless-stopped -d \
  --name my-ssh-container \
  --hostname my-container-host \
  -p 2224:22 \
  ubuntu-eulix:latest
```

然后你就可以通过ssh登陆了

```shell
ssh root@localhost -p 2222
# 密码是你设置的，比如 rootpass
```

## Eulix实验环境搭建

接下来就是安装傲来训练营所需要的环境，我们根据[教程](https://gitee.com/LearningEulixOS/2025-exercises-stage-1)的指示安装必要环境。

本地需要配置部分环境，Ubuntu/Debain 配置参考如下

```
sudo apt install git opensbi u-boot-qemu sshpass openssh-client jq curl qemu-system-misc
```

验证 qemu 是否配置成功

```
qemu-system-riscv64 --version
```

而后拉取交叉编译工具链镜像

```
git clone https://isrc.iscas.ac.cn/gitlab/learningeulixos/2024-exercises-virtual-machines.git
```

通过 qemu 启动工具链与测试环境

```
qemu-system-riscv64 \
    -machine 'virt' \
    -cpu 'rv64' \
    -m 1G \
    -device virtio-blk-device,drive=hd \
    -drive file=qcow2镜像路径,if=none,id=hd \
    -virtfs local,id=lee,path=实验工程路径,mount_tag=lee,security_model=passthrough \
    -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.elf \
    -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
    -object rng-random,filename=/dev/urandom,id=rng \
    -device virtio-rng-device,rng=rng \
    -nographic \
    -append "root=LABEL=rootfs console=ttyS0"

```

qemu 启动后进行测试

实验被挂载到 /lee 目录下，需要切换目录进行测试

```
cd /lee
```

使用 GNU Make 进行构建。

```
# 构建所有（exercise-xx）。
make all
# 构建一个或多个 exercise，比如，exercise-01, exercise-02。
make exercise-xx # 
```

清除产物：

```
make clean
```

测试

```
# 运行所有测试
make test
# 运行一个或多个测试，比如，test-exercise-01, test-exercise-02。
make test-exercise-xx
```

测试完成后可退出按 ctrl+A 然后按 X 退出 qemu

