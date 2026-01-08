---
title: 面对教学场景自动同步课件的jupyterhub快速部署
date: 2026-01-08 00:57:02 +08:00
filename: 2026-01-08-jupyterhub-courseware-sync-deploy
categories:
  - Misc
tags:
  - Environment
  - ML
dir: Misc
share: true
archive: false
priority: 0
---


参考文献：https://www.cnblogs.com/lyjun/p/18884521

项目已开源在：https://github.com/ForsakenDelusion/my-jupyter-deploy

## 简述

该项目面向于小规模团队培训/教学场景，利用docker容器化部署，通过**Git 同步钩子**: 容器启动时会自动执行 `hooks/sync-materials.sh`拉取指定仓库，这样多用户登陆之后都能看见管理员设定的教学仓库。

### 场景1：企业内部培训
痛点 ：新员工技术培训环境配置复杂，版本不一致 解决方案 ：使用my-jupyter-deploy，新员工只需打开浏览器，输入账号密码即可获得统一的环境

### 场景2：高校教学
痛点 ：学生电脑配置各异，实验环境难以统一 解决方案 ：部署在校园服务器，学生通过校园网访问，确保实验环境完全一致

### 场景3：团队协作开发
痛点 ：数据科学家环境差异导致代码运行结果不一致 解决方案 ：统一开发环境，代码评审时无需担心环境问题

### 核心特性：

-  一键部署 ：通过Docker Compose快速启动
- 自动课件同步 ：Git仓库自动拉取最新教学材料
-  中文环境支持 ：开箱即用的中文界面和编码支持
- 环境隔离 ：开发与生产环境完全隔离
- 机器学习生态 ：预装完整的ML/DL工具链

## 1. 容器化设计理念
项目采用 双镜像架构 ：

- JupyterHub镜像 ：负责用户认证和容器调度
- Notebook镜像 ：包含完整的数据科学环境

这种设计让团队可以独立更新两个部分，比如升级Python包时只需重建Notebook镜像，而用户认证系统保持不变。

## 2. 配置简单

各项配置都可以在`docker-compose.yml`的字段进行设置

```yaml
    environment:
      # This username will be a JupyterHub admin
      JUPYTERHUB_ADMIN: admin
      # All containers will join this network
      DOCKER_NETWORK_NAME: jupyterhub-network-dev
      # JupyterHub will spawn this Notebook image for users
      # ！！！notebook的镜像也换！！！
      # DOCKER_NOTEBOOK_IMAGE: quay.io/jupyter/base-notebook:latest
      DOCKER_NOTEBOOK_IMAGE: my-custom-notebook-dev:latest
      # Notebook directory inside user image
      DOCKER_NOTEBOOK_DIR: /home/jovyan/work
      # Git 同步脚本配置 (请在此处填入你的仓库地址)
      # REPO_URL: https://github.com/your-org/your-repo.git
      # REPO_BRANCH: main
      # 代理配置 
      # HTTP_PROXY: http://proxy.example.com:7890
      # HTTPS_PROXY: http://proxy.example.com:7890
```

## 3. 部署简单

几行命令即可完成镜像构建和部署

```shell
docker build -t my-custom-notebook-dev:latest -f Dockerfile.notebook .

docker build -t my-custom-notebook-dev:latest -f Dockerfile.notebook .

docker compose up -d
```

---

欢迎大家使用！一起来测测！