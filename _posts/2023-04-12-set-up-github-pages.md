---
title: 创建 Github Pages 站点
date: 2023-04-12 19:27:08 -0400
categories: [教程, 网站]
tags: [github, web]
description: 免折腾创建 Github Pages 个人站点的简单步骤
---
## TL;DR

套用别人的模板。

## 预备工作

- 有一个 Github 账号以及一个名为 username.github.io 的仓库，其中 username 换成自己的 Github 用户名。
- 安装 jekyll（反正我是没打算尝试别的选择）。[这里][jekyll-on-ubuntu] 是 Ubuntu 上的安装方法。

## 挑选模板

因为本人技术不精，折腾不出来好看的网站，所以只能找现成的了。[这里][jekyll-themes] 有一些免费的主题。

找到一个喜欢的网站主题模板之后，去 Github 上把它克隆下来，把自己的 username.github.io 的仓库也克隆到本地，然后把模板网站仓库里的一些必要文件复制到自己的仓库里。理论上可以把所有文件都复制过去，但万一有一些功能自己不想要，或者想修改，那也可以酌情选择。

检查一下各个文件，把相关个人信息替换成自己的。然后加个原作者声明。

## 预览网站

[这][test-site-locally] 是官方教程网站。

简单说的话，

```shell
bundle install
bundle add webrick
bundle exec jekyll serve
```
{: .nolineno}

然后去 `http://localhost:4000` 就可以预览网站了。如果有修改的话，也是实时更新的。

## 发布网站

把仓库推送到 Github 上，等待几分钟就可以访问 `https://username.github.io` 了。

## 文件结构

个人来说喜欢那种 `header.html`、`footer.html`、`post.html` 什么的都给整好，我就只需要写个 Markdown 就行的网站模板。

这种类型的模板一般文件结构有 `_layout`、`_post` 之类的文件夹，前者会有各种 html 模板文件，后者则放着所有的帖子，都是 Markdown 文件，文件命名是 `YYYY-MM-DD-post-name.md` 这种的。

[jekyll-on-ubuntu]: https://jekyllrb.com/docs/installation/ubuntu/
[jekyll-themes]: https://jekyllthemes.io/free
[test-site-locally]: https://docs.github.com/zh/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll
