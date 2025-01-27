---
title: code-server绑定二级域名重定向问题
date: 2024-06-06 00:34:08 +08:00
filename: 2024-06-06-code-server-subdomain-redirect
categories:
  - Misc
tags:
  - Server
  - Nginx
dir: Misc
share: true
---
## 这里使用nginx的反向代理来实现.

在宝塔面板的网站新添加一个反向代理,域名就填你的二级域名,目标地址填相对应服务的端口.

然后来到域名解析,增添一条解析,主机记录设置为你的二级域名,记录类型设置为A,记录值设置为你的主机ip即可

# !!!!!!!!你以为这样就结束了吗???

以上内容是我下午17.00写的,然而现在已经是23.13,我终于才配好反向代理!!!!!

首先我是遇到了无限重定向问题,[解决方法](https://www.jb51.net/server/294776xqc.htm)点这里,原因就是https的服务器不能代理到http的服务器,一开始我的反向代理的目标地址填成http了,导致一直无限重定向.把目标地址换成https就好了.