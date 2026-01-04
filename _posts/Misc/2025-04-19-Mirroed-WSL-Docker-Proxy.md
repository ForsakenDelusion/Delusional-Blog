---
title: Mirroed网络模式下WSL内的Docker容器代理方案
date: 2025-04-19 01:01:36 +08:00
filename: 2025-04-19-Mirroed-WSL-Docker-Proxy
categories:
  - Misc
tags:
  - WSL
  - Environment
dir: Misc
share: true
archive: false
---
## 需求概述

环境WIN11，WSL 版本： 2.4.13.0，WSL网络模式：Mirroed

需要在WSL的Docker容器内访问Windows宿主机代理。

## WSL设置

这一步在Windows上进行。打开WSL Setting，在网络中打开`主机地址回环`这个选项

![Mirroed网络模式下WSL内的Docker容器代理方案-20250419.png](../../assets/images/Mirroed%E7%BD%91%E7%BB%9C%E6%A8%A1%E5%BC%8F%E4%B8%8BWSL%E5%86%85%E7%9A%84Docker%E5%AE%B9%E5%99%A8%E4%BB%A3%E7%90%86%E6%96%B9%E6%A1%88-20250419.png)

只有这样，才能在WSL的Docker容器内访问到我们主机的代理端口。

## 容器内代理设置

在设置代理前，大家要先看好Windows宿主机的IP，方法大家应该都会。我的ip就是`192.168.6.115`

我是放在`.zshrc`内的，大家可以参考一下。

```shell
ip="192.168.6.115"

alias proxy="
    export http_proxy=socks5://$ip:7890;
    export https_proxy=socks5://$ip:7890;
    export all_proxy=socks5://$ip:7890;
    export HTTP_PROXY=socks5://$ip:7890;
    export HTTPS_PROXY=socks5://$ip:7890;
    export ALL_PROXY=socks5://$ip:7890;"
alias unproxy="
    unset http_proxy;
    unset https_proxy;
    unset all_proxy;
    unset HTTP_PROXY;
    unset HTTPS_PROXY;
    unset ALL_PROXY;"

proxy
```

没错，就这么简单，可以`curl ipinfo.io`来验证一下。