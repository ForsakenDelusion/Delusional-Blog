---
title: Vercel等云平台部署Jekyll博客失败原因踩坑总结
date: 2026-01-02 18:27:42 +08:00
filename: 2026-01-02-Vercel-filed-reason
categories:
  - Misc
tags:
  - Environment
  - Blog
dir: Misc
share: true
archive: false
---

可能出现的几个关键报错，如果你也是这种报错，那大概率遇到了和我一样的问题。

```shell
Error: Command "jekyll build" exited with 1
```

将构建命令改为`jekyll build --trace`之后，可能还会出现之类的字眼，如下。

```shell
/opt/buildhome/.asdf/installs/ruby/3.4.4/lib/ruby/gems/3.4.0/gems/jekyll-4.4.1/lib/jekyll/url.rb:161:in 'String#encode': "\xEF" from ASCII-8BIT to UTF-8 (Encoding::UndefinedConversionError)
```

第一句话先说结论：个人推测是由于之前某次不小心将文章url设置成了中文，当时的构建工具链的版本允许中文构建，结果后续更新之后不支持了，jekyll试图删除旧文件的过程中扫到了这些url，所以还是会报错，非常恶心。

解决办法是在构建的环境变量里面加入UTF-8的支持，让系统能处理各种语言，不要回推倒ASCII

在个人用的Vercel上是这样设置的

![Vercel等云平台部署Jekyll博客踩坑总结-20260102.png](../../assets/images/Vercel%E7%AD%89%E4%BA%91%E5%B9%B3%E5%8F%B0%E9%83%A8%E7%BD%B2Jekyll%E5%8D%9A%E5%AE%A2%E8%B8%A9%E5%9D%91%E6%80%BB%E7%BB%93-20260102.png)

加上两个条目，分别是

| Key    | Value   |
| ------ | ------- |
| LANG   | C.UTF-8 |
| LC_ALL | C.UTF-8 |
