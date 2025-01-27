---
title: Jeklly主题Chirpy踩坑汇总（错误代码块，Vercel构建失败，Liquid Warning）
date: 2025-01-28 00:08:37 +08:00
filename: 2025-01-28-Summary-of-Jekyll-Chirpy-Pitfalls
categories:
  - blog
tags:
  - jekyll
dir: blog
share: true
---
## 错误代码块

详情可见此[Discuss](https://github.com/cotes2020/jekyll-theme-chirpy/discussions/1943)

表现为

![Jeklly主题Chirpy错误代码块-20250128.png](../../assets/images/Jeklly%E4%B8%BB%E9%A2%98Chirpy%E9%94%99%E8%AF%AF%E4%BB%A3%E7%A0%81%E5%9D%97-20250128.png)

![Jeklly主题Chirpy错误代码块-20250128-1.png](../../assets/images/Jeklly%E4%B8%BB%E9%A2%98Chirpy%E9%94%99%E8%AF%AF%E4%BB%A3%E7%A0%81%E5%9D%97-20250128-1.png)

这种情况。解决办法也很简单， 把以下内容添加到 `assets/css/jekyll-theme-chirpy.scss` 文件中。如果你还没有这个[文件](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/assets/css/jekyll-theme-chirpy.scss)，那就创建一个：

\n    {% raw %}\n    ```scss
---
---

@use 'main
{%- if jekyll.environment == 'production' -%}
  .bundle
{%- endif -%}
';

/* append your custom style below */
html[data-mode=light] .highlight .err {
  color: unset;  /* I use unset to disable err?It seems to be working.. */
  background-color: unset;
}

html:not([data-mode]) .highlight .err, html[data-mode=dark] .highlight .err {
  color: unset;
  background-color: unset;
}

```\n    {% endraw %}\n    

## Vercel构建失败

在`Github Pages`上构建成功，`Vercel`却失败，不妨把`Vercel`构建设置中的`nodejs`版本改为18.x。

## Liquid Warning: Liquid syntax error报错

这个很简单，我们直接禁用`Liquid`这个功能就好了，详细请看[官网教学](https://chirpy.cotes.page/posts/write-a-new-post/#liquid-codes)，直接在文章`frontmatter`里面加上

```yaml
render_with_liquid: false
```

或者在全局`_config.yml`中修改内容，如下，这样就不用每篇文章都加个属性了。

```yml
defaults:
  - scope:
      path: "" # An empty string here means all files in the project
      type: posts
    values:
      layout: post
      comments: true # Enable comments in posts.
      toc: true # Display TOC column in posts.
      # DO NOT modify the following parameter unless you are confident enough
      # to update the code of all other post links in this project.
      permalink: /posts/:title/
      render_with_liquid: false # 加上这一行
```
