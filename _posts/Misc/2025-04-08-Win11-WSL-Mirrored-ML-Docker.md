---
title: Win11下的WSL2配合Docker搭建深度学习环境
date: 2025-04-08 00:03:27 +08:00
filename: 2025-04-08-Win11-WSL-Mirrored-ML-Docker
categories:
  - Misc
tags:
  - Environment
  - Docker
  - ML
dir: Misc
share: true
---
WSL2，网络设置为Mirrored（超好用）！体现在虚拟机可以直接访问到主机的端口号！代理网络变得轻而易举，而且局域网访问也不需要再设置转发了，整个配置下来非常无感。

另外我并没有使用docker继承，因为在后续操作中docker继承会导致systemctl和nvidia-container-toolkit拿不到docker信息。

然后就是了解到了我之前在docker里面开发的流程其实有误，之前我是将docker的ssh端口映射出去。然后连接外面开放的端口控制的。实际上，更好的解决方案是，ssh到docker的宿主机上，然后通过宿主机的docker插件连进docker访问。但是这也各有优劣，毕竟开放docker端口我就能一步到位了。

## 第一步 配置宿主机CUDA 

和上一篇文章一样

https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exe_local

直接安装你看到的最新版本，无论你想用的深度学习库要求使用哪个版本的 CUDA

## 第二步 配置WSL

### 必要软件安装以及更新

首先启动 WSL 中的 Ubuntu（可以通过开始菜单启动，或在 PowerShell 中输入 `wsl`），然后执行：

```shell
sudo apt update && sudo apt upgrade -y
```

#### 安装基本开发工具

```shell
sudo apt install -y build-essential git cmake wget curl
```


### 配置WSL的SSH

用于局域网访问WSL

```shell
sudo apt update
sudo apt remove openssh-*
sudo apt install openssh-server
```

修改`sshd_config`允许访问

```shell
sudo vim /etc/ssh/sshd_config
```

修改如下

```shell
#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

PubkeyAuthentication yes
```

### 美化Shell

#### 安装zsh

```shell
sudo apt-get install zsh #Ubuntu Linux记得先升级下 apt-get
chsh -s /bin/zsh #安装完成后设置当前用户使用 zsh 并重启 wsl
```

#### 安装 oh my zsh

此时我们需要下载 [oh-my-zsh](https://link.zhihu.com/?target=https%3A//github.com/robbyrussell/oh-my-zsh)

```text
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```


克隆插件

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting 
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

修改

```text
ZSH_THEME="agnoster"
plugins=(git zsh-syntax-highlighting zsh-autosuggestions z)
```

再加上我们的代理脚本，大家根据自己的情况修改。得益于WIN11的WSL2新的网络工作模式Mirrored，我们可以很方便的连接到宿主机的代理端口

```shell
# >>>终端配置代理 START <<<
#host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
proxy_ip=http://localhost:7890

alias poff='
unset http_proxy;
unset https_proxy;
unset socket5;
'
# 运行 pon 即可快捷打开代理
alias pon='
export http_proxy=$proxy_ip;
export https_proxy=$proxy_ip;
export socket5=$proxy_ip;
'
```

保存后

```text
source ~/.zshrc
```

### 安装Docker

不采用集成Docker，而是在WSL里面手动安装

```shell
# $ curl -fsSL test.docker.com -o get-docker.sh
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
# $ sudo sh get-docker.sh --mirror AzureChinaCloud
```

### 安装WSL里的CUDA

将以下安装代码一行一行的复制进终端执行。注意安装之前要先在wsl里面输入

```shell
nvidia-smi
```

查看一下`CUDA Version`,上面显示的版本是你最高能安装到的版本

https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_local

![WSL2配置Docker配置机器学习环境-20250406.png](../../assets/images/WSL2%E9%85%8D%E7%BD%AEDocker%E9%85%8D%E7%BD%AE%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E7%8E%AF%E5%A2%83-20250406.png)


**第一步：查找 nvcc 安装位置**

```shell
sudo find /usr -name nvcc
```

你大概率会看到类似路径：

```shell
/usr/local/cuda-12.8/bin/nvcc
```

**第二步：添加到 PATH 中（推荐写进 .bashrc 或 .zshrc）**

```shell
echo 'export PATH=/usr/local/cuda-12.8/bin:$PATH' >> ~/.zshrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.8/lib64:$LD_LIBRARY_PATH' >> ~/.zshrc
source ~/.zshrc
```

### 安装WSL里的Nvidia container toolkit

用于Docker直接使用宿主GPU。

这是用于Docker访问GPU

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#installing-on-ubuntu-and-debian

在 WSL 中可以直接安装 Ubuntu 版本的 NVIDIA Container Toolkit。

```shell
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

```

>注: 使用 Docker 时，可以使用任意的比系统 CUDA 版本低的 CUDA 镜像。开发时请使用 devel 版本的 PyTorch 镜像，只有 devel 版本才包含编译器，可以编译 C++ 扩展。

## 第三步 Docker的使用

### 启动Docker方法一

这里博主自己写了一个Dockerfile，作为基础环境配置，仅作为抛砖引玉，大家可以参考，自行修改。

```dockerfile
# 基于 Ubuntu 22.04，这里我换成了国内源
FROM swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/ubuntu:22.04

# 避免安装过程中的交互式提示
ENV DEBIAN_FRONTEND=noninteractive

# 安装基础软件包
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    wget \
    git \
    vim \
    openssh-server \
    ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 配置 SSH 服务
RUN mkdir -p /var/run/sshd
RUN echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
RUN echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config

# 设置 root 密码为 'password'（建议在构建后修改）
RUN echo 'root:password' | chpasswd

# 创建工作目录
RUN mkdir -p /root/ml-workspace
WORKDIR /root/ml-workspace

# 安装 Miniconda
ENV CONDA_DIR=/opt/conda
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p ${CONDA_DIR} && \
    rm ~/miniconda.sh

# 将 Conda 加入环境变量
ENV PATH=${CONDA_DIR}/bin:$PATH

# 配置 pip 使用阿里云源
RUN mkdir -p ~/.pip && \
    echo '[global]' > ~/.pip/pip.conf && \
    echo 'index-url = https://mirrors.aliyun.com/pypi/simple/' >> ~/.pip/pip.conf && \
    echo 'trusted-host = mirrors.aliyun.com' >> ~/.pip/pip.conf

# 配置 Conda
RUN conda config --set always_yes yes && \
    conda update -q conda && \
    conda init bash

# 创建一个 Conda 环境，安装 Python、pip 以及 CUDA 相关组件
RUN conda create -n ml python=3.10 pip && \
    conda install -n ml \
    cudatoolkit=11.8 \
    cudnn && \
    conda run -n ml pip install --upgrade pip

# 设置 CUDA 环境变量
RUN mkdir -p /opt/conda/envs/ml/etc/conda/activate.d/ && \
    echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/' > /opt/conda/envs/ml/etc/conda/activate.d/env_vars.sh && \
    chmod +x /opt/conda/envs/ml/etc/conda/activate.d/env_vars.sh

# 使用 pip 安装机器学习相关库，指定 CUDA 版本
# 安装数据处理库
RUN conda run -n ml pip install numpy pandas

# 安装可视化库
RUN conda run -n ml pip install matplotlib

# 安装机器学习库
RUN conda run -n ml pip install scikit-learn

# 安装 PyTorch (GPU版本)
RUN conda run -n ml pip install torch torchvision torchaudio

# 安装 TensorFlow
RUN conda run -n ml pip install tensorflow

# 安装 Jupyter 相关组件
RUN conda run -n ml pip install jupyter jupyterlab jupyterlab-language-pack-zh-CN

# 为 Jupyter 注册环境内核
RUN /opt/conda/envs/ml/bin/python -m ipykernel install --name ml --display-name "Python (ML)"

# 生成并配置 Jupyter 配置文件
RUN mkdir -p /root/.jupyter && \
    /opt/conda/envs/ml/bin/jupyter notebook --generate-config
RUN echo "c.NotebookApp.ip = '0.0.0.0'" >> /root/.jupyter/jupyter_notebook_config.py && \
    echo "c.NotebookApp.allow_root = True" >> /root/.jupyter/jupyter_notebook_config.py && \
    echo "c.NotebookApp.open_browser = False" >> /root/.jupyter/jupyter_notebook_config.py

# 在 .bashrc 中添加自动激活 ml 环境的命令
RUN echo 'conda activate ml' >> /root/.bashrc

# 配置库路径，避免 libtinfo.so.6 版本警告,如果不想加这一句，就把上面conda activate ml那条语句去掉。不然会报错的。
ENV LD_LIBRARY_PATH=/lib:/usr/lib:/lib64:/usr/lib64:$LD_LIBRARY_PATH

# 暴露端口
EXPOSE 22 8888 6006

# 设置容器启动命令
CMD ["/usr/sbin/sshd", "-D"]

```

另外，如果你想要Docker有伪开机自启动进程的话，可以参考如下写法。下面是一个开机自启动jupyter的实例。

```dockerfile
# 创建启动脚本
RUN echo '#!/bin/bash\n\
service ssh start\n\
jupyter notebook --no-browser --allow-root &\n\
exec "$@"\n' > /start.sh && chmod +x /start.sh

# 设置容器启动命令
ENTRYPOINT ["/start.sh"]
CMD ["/usr/sbin/sshd", "-D"]
```

写好Dockerfile之后，我们只需要build镜像，然后启动镜像即可，参考如下命令

```shell
docker build -t ml-ssh-image . # 构建镜像命令
docker run --name ml-env --hostname ml-server --gpus all --shm-size=8g -d -p 8888:8888 -p 6006:6006 -p 2223:22 -v $(pwd):/root/ml-workspace ml-ssh-image # 启动命令
```

### 启动Docker方法二

如果你不想自己构建镜像，而是使用各大企业的官方镜像，也可以参考如下命令

首先，pull你想要的镜像。我这里以pytorch的镜像为例子

```shell
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel
```

然后，采用合适的参数启动容器

```shell
docker run -it --name env-test --hostname ml-server --gpus all --shm-size=8g -p 8888:8888 -p 6006:6006 -p 2223:22 -v $(pwd):/root/ml-workspace swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel /bin/bash
```

注意，这里的容器一定要加上`-it`和后面的`/bin/bash`参数，详情可见一下解释

```text
在这个命令中，-d 参数告诉 Docker 在后台运行容器，而不是在前台保持交互状态。这意味着：

容器启动后会立即返回控制权给终端，不会等待容器内的进程结束
容器会在后台持续运行
容器的标准输入、输出和错误会被分离，不会显示在当前终端上

此外，从你的 Dockerfile 中可以看出，容器的启动命令是 CMD ["/usr/sbin/sshd", "-D"]，这意味着容器启动后会运行 SSH 服务器，并且 -D 参数使其保持在前台运行。这个进程不需要交互输入，它会一直运行并监听 SSH 连接。
当我们对比使用 -it 和不使用的场景：

需要 -it 的情况：当你想直接在容器中获得一个交互式 shell（如 bash）时，需要 -it。例如：docker run -it ubuntu bash
不需要 -it 的情况：

启动一个后台服务（如 Web 服务器、数据库或你的例子中的 SSH 服务器）
运行一个批处理任务，不需要用户输入
使用 -d 参数将容器放在后台运行



在你的场景中，不需要 -it 是因为：

你用 -d 参数指定了容器在后台运行
容器的主进程（sshd）本身就设计为长期运行的服务
你不需要直接与容器的标准输入进行交互，而是通过映射的端口（SSH 端口 2223）来访问容器

如果你想直接与容器进行交互，你通常会通过两种方式：

使用 docker exec -it ml-env-test bash 来在运行中的容器中启动一个新的 bash 会话
使用 SSH 客户端连接到映射的 2223 端口：ssh root@localhost -p 2223

总结来说，-it 主要用于需要直接交互式终端的场景，而对于后台服务容器（用 -d 启动的），它们的主要交互方式是通过网络服务（如 SSH、HTTP 等），因此不需要 -it 参数。

你使用的是 PyTorch 官方镜像（pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel）
这个镜像可能默认的 CMD 设置为某个简短的命令，或者没有指定持续运行的进程
当容器的主进程执行完成后，容器就会停止

PyTorch 官方镜像主要设计用于开发和训练模型，并不是设计为持续运行的服务。它们通常预期的用法是：

使用 -it 参数进行交互式开发
或者显式指定一个命令来运行你的训练脚本

如何解决这个问题
要让基于 PyTorch 镜像的容器持续运行，你有几种选择：

指定一个长期运行的命令:
bashCopydocker run --name env-test ... pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel tail -f /dev/null
tail -f /dev/null 是一个简单的命令，它会一直运行而不做任何事情。
以交互模式运行并指定 shell:
bashCopydocker run -it --name env-test ... pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel /bin/bash
这会给你一个交互式终端，但这种方式不能与 -d（后台模式）一起使用。
在容器中安装和启动 SSH 服务:
首先，以交互模式启动容器：
bashCopydocker run -it --name env-test ... pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel /bin/bash
然后在容器内安装 SSH：
bashCopyapt-get update && apt-get install -y openssh-server
mkdir -p /var/run/sshd
echo 'root:password' | chpasswd  # 设置密码
echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
最后，提交这个容器为新镜像并以 SSH 服务方式启动：
bashCopydocker commit env-test pytorch-with-ssh
docker run -d --name env-ssh ... pytorch-with-ssh /usr/sbin/sshd -D

创建自己的 Dockerfile:
最佳实践是创建一个以 PyTorch 镜像为基础，但添加了你需要的额外服务（如 SSH）的 Dockerfile：
dockerfileCopyFROM pytorch/pytorch:2.6.0-cuda12.6-cudnn9-devel

# 安装 SSH 服务
RUN apt-get update && apt-get install -y openssh-server && \
    mkdir -p /var/run/sshd && \
    echo 'root:password' | chpasswd && \
    echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config

# 设置启动命令
CMD ["/usr/sbin/sshd", "-D"]
```

以下内容来自(https://duanyll.com/2024/6/30/WSL2-Docker-Deep-Learning/#docker-%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8)

### Docker 快速入门

一个 Docker 镜像包含了运行程序所需的完整文件系统和环境变量。Docker 容器是一个特殊的进程，使用 Docker 镜像提供的文件系统和环境变量，并且网络栈和主机一般是隔离的（实际上，几乎所有的用户态要素都被隔离了）。相比于虚拟机，Docker 能快速地启动和停止，而且 Docker 镜像的大小通常比虚拟机的小很多。

需要注意的是，Docker 容器被设计成无状态的，容器内的文件系统和环境变量都是临时的，**任何修改都会在容器停止后丢失**。如果需要向容器内安装新的程序或者修改配置文件，应该重新构建一个新的 Docker 镜像。如果需要保存容器内的数据，应该把数据文件挂载（Mount）到宿主机的文件夹或者 Docker Volume。`Docker Commit` 命令可以把运行中的容器保存为新的 Docker 镜像，但是_不推荐用来制作镜像_，只适合用于调试或者紧急保存数据的需求。

可以方便地从 Docker Hub 上下载包含各类 Linux 发行版和软件的镜像，由于 Docker 提供了充分的隔离，几乎所有镜像都能下载后开箱即用，省去了手动安装各类环境的麻烦。如果需要自定义镜像，可以使用 Dockerfile 来描述镜像的构建过程，然后使用 `docker build` 命令构建镜像。Dockerfile 中的每一条指令都会生成一个新的镜像层，Docker 会尽量复用已有的镜像层，以减少镜像的大小。Dockerfile 通常包括 `FROM`, `RUN`, `COPY`, `CMD` 等指令。

- `FROM` 指定基础镜像
- `RUN` 在镜像中运行 Shell 命令，可以用来安装软件
- `COPY` 复制文件到镜像中。由于 Docker 的设计，要复制的文件必须在构建上下文中，所以通常需要把文件放在 Dockerfile 同一目录下
- `CMD` 指定容器启动时默认运行的命令
- `ENV` 设置环境变量

Docker 镜像的构建过程会被缓存，如果 Dockerfile 的某一步发生了变化，Docker 会重新构建这一步之后的所有步骤。下面是一个常见的 Dockerfile 示例：

```dockerfile
# 有了 Docker，可以使用任意老版本的 PyTorch 和 CUDA，而不会影响其他的项目
FROM pytorch/pytorch:1.7.1-cuda11.0-cudnn8-devel

# Dockerfile 中 APT 包的安装比较复杂，建议复制下面的格式
# rm /etc/apt/sources.list.d/cuda.list 如果镜像中没有 CUDA，则删掉这一行
# 如果不指定 DEBIAN_FRONTEND=noninteractive TZ=Asia/Shanghai 和 -y 参数，构建镜像会因为 apt 等待用户输入而卡死
RUN rm /etc/apt/sources.list.d/cuda.list \
    && sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list \
    && sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive TZ=Asia/Shanghai apt-get install -y git gdb vim curl wget tmux zip cmake ffmpeg libsm6 libxext6 \
    && rm -rf /var/lib/apt/lists/*

# environment.yml 需要放在 Dockerfile 同一目录下
COPY environment.yml /tmp/environment.yml
RUN conda env create -f /tmp/environment.yml
RUN conda init bash

# 除了 Conda，也可以直接用 pip 安装 Python 包，Docker 已经提供了环境隔离
RUN pip install -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com h5py einops tqdm matplotlib tensorboard torch-tb-profiler ninja scipy

```

可以用 `docker build` 命令构建 Docker 镜像：

```shell
docker build -t my-image-name .
```

`-t` 参数指定镜像的名字，只能包含小写字母和数字，可以用 `/` 分隔，`.` 指定 Dockerfile 所在的目录。构建完成后可以用 `docker images` 命令查看所有的镜像。

要启动 Docker 镜像，可以使用 `docker run` 命令：

```shell
docker run --rm -it --gpus all -v $(pwd):/workspace -p 8080:80 my-image-name bash
```

- `--rm` 容器停止后自动删除，一般不用留着因为数据都清除了
- `-it` 交互式启动，可以使用 Shell。如果需要作为后台进程运行，换成 `-d` 参数
- `--gpus all` 允许容器使用所有的 GPU，不加用不了 CUDA
- `-v $(pwd):/workspace` 把当前目录挂载到容器的 `/workspace` 目录，可以在容器内的 `/workspace` 目录中读写文件，数据会随时同步到宿主机，不会丢失
- `-p 8080:80` 把容器的 80 端口映射到宿主机的 8080 端口，可以通过 `localhost:8080` 访问容器内 `80` 端口的 Web 服务。如果不需要映射端口，可以不加这个参数。
    - 在 WSL2 上，这个命令只负责从容器映射到 WSL2 的网络栈
    - 但通常 WSL2 上的端口能被自动映射到 Windows 上，可以直接在 Windows 上访问 `localhost:8080`
- `my-image-name` 指定要启动的镜像名称
- `bash` 指定容器启动时运行的命令，可不加，默认是 `CMD` 指定的命令

使用 `docker ps` 命令可以查看所有正在运行的容器，使用 `docker stop` 命令可以停止容器（容器会在主进程退出后自动停止）。使用 `docker exec` 命令可以在运行中的容器中运行命令。使用 `docker cp` 命令可以从容器中复制文件到宿主机。具体用法略。

Docker Compose 可以用来管理多个容器，也能方便的把容器的启动参数写到文件里。具体用法略。

## 第四步 利用VSCODE开发

这里有几种选择，

第一个是单独开放WSL里的Docker的ssh端口，在上面的Docker run命令中，我也做了相关映射；

第二个是连接到WSL，再利用Docker插件连到WSL里面的docker。理论上来收方法二更加的官方，但是第一种方法更加的方便，所以大家自行选择。

Visual Studio Code 的 DevContainer 功能可以让你在容器中开发代码，能自动启动容器并使用 VS Code 在容器内进行开发调试。DevContainer 会自动挂载当前目录到容器内的 `/workspace` 目录，所以容器内的文件会和宿主机同步，不会丢失。VS Code 还能自动配置端口映射和 X11 显示并兼容 WSLg，`plt.show()` 能在 Windows 上显示图像。

要使用 DevContainer，需要安装 [Visual Studio Code](https://code.visualstudio.com/)、[Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) 和 [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) 插件。由于我们没有使用 Docker Desktop，所以需要先让 VSCode 连接到 WSL，才能使用 DevContainer 功能。

### 方法一

不过多介绍，如果有ssh需求的应该自己会

### 方法二

也很简单，流程就是先点击vscode左下角的连接图标，连接到WSL。

![Win11下的WSL2配合Docker搭建深度学习环境-20250408.png](../../assets/images/Win11%E4%B8%8B%E7%9A%84WSL2%E9%85%8D%E5%90%88Docker%E6%90%AD%E5%BB%BA%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E7%8E%AF%E5%A2%83-20250408.png)

然后打开Docker扩展，选中你想进入的容器，右键，接着选择带有`Visual Studio Code`字样的选项即可。

## 最后，内网穿透

这一步比较个性化，是我个人的需求23333.