---
title: WSL 本地部署 Github Pages
date: 2025-01-22 08:26:00 -0400
categories: [笔记, Github Pages]
tags: [github, ruby]
description: 如何在 WSL 上安装 Ruby，并部署 Github Pages 项目
---

## WSL 安装 Ruby

我们使用 rbenv 版本管理来安装 Ruby，但是 Ubunut 官方 apt 中的软件包的版本比较低了，我们根据 [官网](https://github.com/rbenv/rbenv?tab=readme-ov-file#basic-git-checkout) 的步骤用 git 安装。

### 安装 rbenv

```bash
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
~/.rbenv/bin/rbenv init
```

然后重启终端，测试是否安装成功

```bash
rbenv --version
```

### 安装 ruby-build

rbenv 本身不能直接安装 Ruby，需要安装一个 [ruby-build](https://github.com/rbenv/ruby-build?tab=readme-ov-file#clone-as-rbenv-plugin-using-git) 插件

```bash
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
```

测试是否成功

```bash
# 列举可安装的稳定版本
rbenv install -l
```

### 安装 Ruby

这里选择安装最新的稳定版本，但是安装之前需要一些预备措施：

```bash
sudo apt install build-essential
sudo apt install zlib1g-dev
sudo apt install libffi-dev
sudo apt install libyaml-dev
```

然后安装 Ruby

```bash
rbenv install 3.4.1
```
如果之前尝试过几次失败了，再安装又失败的话，可以试试

```bash
rbenv install 3.4.1 -f
```

激活该版本为默认版本，并测试是否成功

```bash
rbenv global 3.4.1
ruby -v
```

## WSL 部署 Github Pages

安装好 Ruby，一般来说 bundle 就默认安装了，可以测试一下

```bash
bundle --version
```

### 创建 SSH 密钥并添加到 Github

从某时开始 Github 就不推荐使用 https 协议下载代码库了，所以我们改换 ssh 协议。参考 [官网](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) 操作设置 SSH 密钥。

### 本地部署

下载代码库

```bash
git clone git@github.com:xxxyyy/xxxyyy.github.io.git
```

安装必要的 gem

```bash
bundle install
# 官网推荐的
bundle add webrick
# Ruby 3.4.0 之后不作为标准库提供的
bundle add csv logger base64
```

本地运行

```bash
bundle exec jekyll serve
```

如果有任何问题，查看 官方文档。

### 创建 GPG 密钥并添加到 Github

建议在提交代码到仓库的时候使用 GPG 密钥验证，参考 [官网](https://docs.github.com/zh/authentication/managing-commit-signature-verification/generating-a-new-gpg-key?platform=linux) 操作。

