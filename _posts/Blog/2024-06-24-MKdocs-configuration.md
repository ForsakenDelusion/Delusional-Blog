---
title: MKdocs指南
date: 2024-06-24 00:34:08 +08:00
filename: 2024-06-24-MKdocs-configuration
categories:
  - Blog
tags:
  - Blog
dir: Blog
share: true
---
# 安装

先随便进个目录，我个人习惯用venv创建个虚拟环境

```shell
mkdir {project-name}
cd {project-name}
#创建python虚拟环境
python -m venv .venv
#激活虚拟环境
source .venv/bin/active
#下载所需包
pip install mkdocs
pip install material-mkdocs
```
这样，新的mkdocs就被创建了

再关联一下你的远程仓库，这样基本的初始化就完成了

# 配置

我们需要改一下mkdocs的yml配置，这里直接给到一个实例

```yml
site_name: AUST计算机求生指南

site_url: https://aust-cs-plan.forsakendelusion.online

site_author: Delusion

site_description: >-

AUST计算机求生指南

  

repo_name: AUST-CS-Plan

repo_url: https://github.com/ForsakenDelusion/AUST-CS-Plan.git

  

theme:

name: material

language: zh

custom_dir: overrides

icon:

repo: fontawesome/brands/github

  

features:

- header.autohide

- navigation.tracking

- navigation.top

- search.highlight

- search.share

- search.suggest

- content.code.annotate

  

palette:

# Palette toggle for light mode

- media: "(prefers-color-scheme: light)"

scheme: default

primary: pink

toggle:

icon: material/toggle-switch

name: Switch to dark mode

  

# Palette toggle for dark mode

- media: "(prefers-color-scheme: dark)"

scheme: slate

primary: pink

toggle:

icon: material/toggle-switch-off-outline

name: Switch to light mode
```

简单设置一下，我们就可以使用

```shell
mkdocs gh-deploy
```

将其推送到你的github仓库中，你会发现仓库中多了一个gh-pages分支。并且终端会给出你的githubpages的地址，点进去就能访问了。

但是此时会出现一个问题，你目录下的site文件也被推送到main仓库，但其实site里面的文件已经被推送到分支里面了，所以我们并不需要，因此，配置一下`.gitignore`

```.gitignore
.DS_Store
.vscode/

/venv
/site
```

至此，基本设置已经完成了

# 设置workflows实现自动化

进入到你的{mkdocs-project}目录下

```shell
mkdir .github
cd .github
mkdir workflows
#名字可以随便起xxx.yml
vim auto.yml
```

将以下内容复制进去

```yml
name: update site

on: # 在什么时候触发工作流

push: # 在从本地main分支被push到GitHub仓库时

branches:

- main

pull_request: # 在main分支合并别人提的pr时

branches:

- main

jobs: # 工作流的具体内容

deploy:

runs-on: ubuntu-latest # 创建一个新的云端虚拟机 使用最新Ubuntu系统

steps:

- uses: actions/checkout@v2 # 先checkout到main分支

- uses: actions/setup-python@v2 # 再安装Python3和相关环境

with:

python-version: 3.x

- run: pip install mkdocs-material # 使用pip包管理工具安装mkdocs-material

- run: mkdocs gh-deploy --force # 使用mkdocs-material部署gh-pages分支
```

这样，自动化部署就完成了.

# 绑定二级域名


我的主域名是`forsakendelusion.online`但是我想把新的界面放到另一个地址里，怎么办？？

此时我们就可以创建一个二级域名，并且解析到github pages上。

首先来到项目仓库的Settings-Pages设置，将你的域名设置一下。

![MKdocs指南-20250122.png](../../assets/images/MKdocs%E6%8C%87%E5%8D%97-20250122.png)

然后再去我们的域名商设置一下解析

![MKdocs指南-20250122-1.png](../../assets/images/MKdocs%E6%8C%87%E5%8D%97-20250122-1.png)

我这里直接解析域名，也可以选择解析ip

过一会就好了。

你以为这样就完了？

注意有坑

如果你以为这样就完了，等你下次再通过 `git push` 提交文档/博客的时候，你就会发现你的自定义域名已经变成 `404`了。  
此时你打开 GitHub 对应仓库的 Settings 一看，好家伙，框框里什么都没有了。

先说解决方法再说原因。

解决方法是，在 `docs` 文件夹里创建一个 `CNAME` 文件，内容写上你的自定义域名，下次 Actions 运行的时候，Pages 就会自动把这个域名设置成自定义域名了。

这里给出我通常使用的插件

```yml
features:
    - header.autohide
    - navigation.tracking
    - navigation.top
    - navigation.indexes#让大章节也能点进去看
    - navigation.tabs#让章节在上面显示
    - navigation.instant
    - navigation.sections#隔离一级标题
    - search.highlight
    - search.share
    - search.suggest
    - content.code.annotate
```

https://blog.arisa.moe/blog/2022/220407-github-pages/#github-actions
[另一份参考文档](https://blog.wangriyu.wang/2018/01-githubpage.html)
[为你的页面加上评论区](https://blog.highp.ing/p/giscus/)
https://squidfunk.github.io/mkdocs-material/reference/admonitions/#+type:abstract