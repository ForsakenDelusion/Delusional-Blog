---
title: 在Docker中访问vite构建的vue项目
date: 2025-01-08 10:53:06 +08:00
filename: 2025-01-08-Vue-in-docker
categories:
  - Misc
tags:
  - Docker
  - Vue
  - Web
dir: Misc
share: true
---

使用场景介绍。本地mac作为开发机器，同局域网内的一台Linux主机中的Docker作为开发环境。

需求：可以使用mac远程进Linux机器的docker进行开发。

想法：利用简单的端口映射完成。

## 步骤一：Docker端口映射

1. 保存运行中的开发容器

```shell
docker commit <容器ID> <输出的镜像名>
```

停止容器

```shell
docker stop <容器 ID> # 停止当前运行的容器
docker rm <容器ID> # （可选）删除不需要的容器
```

2. 使用新的端口配置创建新容器

```shell
docker run -d --name <新容器的ID> --hostname <新容器的主机名> -p 2222:22 -p 5173:5173 <刚刚commit的镜像名>
```

## 步骤二：防火墙开放端口

这一步我们直接开放vite所需的端口，这里大家可以根据自己的实际情况选择不同的方式。我的本地机器上也安装了宝塔面板，所以我直接使用了宝塔，很方便。

## 步骤三：修改package.json

```json
 "scripts": {
    "dev": "vite --host 0.0.0.0", // 修改了这一行
    "build": "run-p type-check \"build-only {@}\" --",
    "preview": "vite preview",
    "build-only": "vite build",
    "type-check": "vue-tsc --build",
    "lint": "eslint . --fix"
  },
```

至此，大功告成，使用ip:端口的方式即可访问新创建的vue项目。
