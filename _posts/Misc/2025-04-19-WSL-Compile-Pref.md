---
title: WSL内核编译Pref工具
date: 2025-04-19 00:37:22 +08:00
filename: 2025-04-19-WSL-Compile-Pref
categories:
  - Misc
tags:
  - WSL
  - Misc
  - OS
dir: Misc
share: true
---
参考文章：https://www.arong-xu.com/posts/wsl2-install-perf-with-manual-compile/

## 准备阶段

### 1. 更新 WSL2

打开 Windows PowerShell, 更新 WSL2:

```powershell
wsl --update
```

得到输出:

```text
正在检查更新.
已安装最新版本的适用于 Linux 的 Windows 子系统.
```

## 编译内核代码

这部分步骤需要小心操作, 毕竟是内核代码.

需要注意的是, **编译内核代码的时候不要使用 root 权限**. 只需要在安装`perf`的时候使用`root`权限.

### 1. 检查内核版本

首先我们需要查看当前`Ubuntu`的内核版本:

```bash
uname -r
```

得到输出:

```text
5.15.146.1-microsoft-standard-WSL2
```

这个版本号将决定我们下载的内核代码版本, 即`5.15.146.1`版本. **请注意这个版本号可能会有变化, 你的实际操作结果跟这里的版本号可能会有所不同.**

设置一个环境变量`KERNEL_VERSION`, 方便后续引用:

```bash
export KERNEL_VERSION=$(uname -r | cut -d'-' -f1)
```

### 2. 下载内核代码

这里提供两种可选方式:

1. 从[Github Release](https://github.com/microsoft/WSL2-Linux-Kernel/releases)下载, 找到对应版本的内核代码.
    
2. 从 Github 上 clone 代码:

```bash
git clone \
--depth 1 \
--single-branch --branch=linux-msft-wsl-${KERNEL_VERSION} \
https://github.com/microsoft/WSL2-Linux-Kernel.git
```

### 3. 编译并安装

使用`make`命令编译内核代码:

```bash
cd WSL2-Linux-Kernel
make KCONFIG_CONFIG=Microsoft/config-wsl
```

为了加快编译速度, 可以使用`-j`参数:

```bash
make -j $(nproc) KCONFIG_CONFIG=Microsoft/config-wsl
```

编译`perf`工具:

```bash
cd tools/perf
make
```

编译完成后安装到系统目录:

```bash
sudo cp perf /usr/bin/
```

## 全功能的`perf`

`perf`编译的时候会根据你本地安装的库的版本来决定是否支持某些功能. 如果你需要全功能的`perf`则需要进一步安装依赖库.

### 1. 安装依赖库

```bash
sudo apt install binutils-dev debuginfod default-jdk default-jre libaio-dev libbabeltrace-dev libcap-dev libdw-dev libdwarf-dev libelf-dev libiberty-dev liblzma-dev libnuma-dev libperl-dev libpfm4-dev libslang2-dev libssl-dev libtraceevent-dev libunwind-dev libzstd-dev libzstd1 python-setuptools python3 python3-dev systemtap-sdt-dev zlib1g-dev
```

### 2. 重新编译`perf`

```bash
aronic@arong:~/WSL2-Linux-Kernel/tools/perf$ make clean && make
```

编译过程中会显示哪些功能被支持.

```txt
...
Auto-detecting system features:
...                         dwarf: [ on  ]
...            dwarf_getlocations: [ on  ]
...                         glibc: [ on  ]
...                        libbfd: [ on  ]
...                libbfd-buildid: [ on  ]
...                        libcap: [ on  ]
...                        libelf: [ on  ]
...                       libnuma: [ on  ]
...        numa_num_possible_cpus: [ on  ]
...                       libperl: [ on  ]
...                     libpython: [ on  ]
...                     libcrypto: [ on  ]
...                     libunwind: [ on  ]
...            libdw-dwarf-unwind: [ on  ]
...                          zlib: [ on  ]
...                          lzma: [ on  ]
...                     get_cpuid: [ on  ]
...                           bpf: [ on  ]
...                        libaio: [ on  ]
...                       libzstd: [ on  ]
...        disassembler-four-args: [ on  ]

...
```

### 3. 安装`perf`

```bash
aronic@arong:~/WSL2-Linux-Kernel/tools/perf$ sudo cp perf /usr/bin/
```

## 测试`perf`

```bash
aronic@arong:~$ perf --version
perf version 5.15.146.1.gee5b8e3dcbc6
```

## Docker内使用

参考：https://chinggg.github.io/post/docker-perf/

注意，在Docker内使用需要加上两行参数启动容器

- 在 `docker run` 时加上参数 `--cap-add CAP_SYS_ADMIN` 及 `--privileged`，赋予容器特权

例如

```shell
docker run -d --cap-add CAP_SYS_ADMIN --privileged --name Ubuntu-15445 --hostname 15445 -p 2222:22 -p 5173:5173 15445:latest
```