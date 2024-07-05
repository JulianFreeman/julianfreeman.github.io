---
title: Linux 备忘笔记
date: 2024-07-05 15:27:00 -0400
categories: [笔记, Linux]
tags: [linux, kubuntu]
description: 记录一些使用 Linux 桌面系统过程中遇到的问题
media_subpath: /assets/img/linux-backup/
---

## VeraCrypt 挂载乱码

挂载的加密盘里有中文，出现乱码的解决方法：

在 **设置-参数选择-装载选项-文件系统-装载选项** 填 `iocharset=utf8` 。

## KUbuntu 修改语言

安装语言包：

```shell
sudo apt install language-pack-gnome-zh-hans language-pack-kde-zh-hans
# 必要时还可以安装
# firefox-locale-zh-hans
# libreoffice-l10n-zh-cn
# thunderbird-locale-zh-hans
# kdevelop-l10n
```
{: .nolineno}

改变系统语言，除了 **系统设置-区域设置-语言** 里面添加中文之外，**格式** 里的 **区域** 也得改成中文，否则有些软件的语言变不过来。

## KUbuntu 修改输入法

推荐使用 rime ，如果是 ibus-rime ，安装后使用 ``ctrl + ` `` 调出方案选择菜单，可以选简化字。或者参考官网教程。

如果安装 fcitx ，在 `~/.bashrc`{: .filepath} 文件中添加

```shell
export GTK_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export QT_IM_MODULE=fcitx
```
{: file="~/.bashrc"}

然后运行

```shell
source ~/.bashrc
```
{: .nolineno}

注销。

## KUbuntu 不能创建加密分区

- Try Ubuntu
- 安装 ubiquity-frontend-gtk 
- 运行命令 `ubiquity gtk_ui` ，注意后面是下划线不是中划线
- 在弹出的安装界面中就可以正常添加加密分区（设置密码）了
- 加密分区指 physical volume for encryption ，分区设置完密码后会出现一个 `/dev/mapper/?`{: .filepath} ，在那下面点击添加根分区
- 注意，留 500M 给 `/boot`{: .filepath} 分区，不要加密

## 手动添加字体

将字体文件复制到 `/home/user/.local/share/fonts/`{: .filepath} 下，然后运行 `fc-cache -fv` 生成字体缓存，就可以用了。

## 中文显示异体字的问题

参考这个 [文档](https://wiki.archlinux.org/title/Localization/Simplified_Chinese#Chinese_characters_displayed_as_variant_(Japanese)_glyphs)，或者也可以先试试把 fonts-noto-cjk 软件包卸载了重新安装，能解决就不看这个文档了。

## 修改打开方式里的可选程序

安装 Skype 之后几乎所有文件的打开方式里都会有使用 Skype 分享的选项，关键这个中文还很奇怪：

![share-with-skype](share-with-skype.png)

分享就分享吧，为啥还是个“已注销”，还加个句号。看着很难受。

说一下删除这个选项的方式。

找到 `skypeforlinux-share.desktop`{: .filepath} 文件，我的 Skype 用 Snap 安装的，所以在 `/snap/skype/263/meta/gui/skypeforlinux-share.desktop`{: .filepath} 这里，当然 `263` 可能是版本号之类的，不过什么版本的都无所谓，设置是一样的，主要是打开这个文件，看到里面的 MimeType 是 application/octet-stream。

![skype-mimetype](skype-mimetype.png)

然后找到系统设置，找到应用程序，找到文件关联，找到 application/octet-stream，

![octet-stream](octet-stream.png)

在程序优先顺序里删掉 Skype 就可以了。

这样在所有文件关联中都没有 Skype 共享了。

## 关于 apt-key

apt-key 只支持到 Ubuntu 22.04 ，但是目前有些软件还是会把软件仓库的密钥放在 `/etc/apt/trusted.gpg`{: .filepath} 文件中，这种方式在下一个版本中会被弃用。下面说一下怎么更改。

### 支持的

首先，如果已经拿到了软件仓库的密钥文件，一个 `.gpg`{: .filepath} 或者 `.asc`{: .filepath} 文件，那么可以把这个文件移动到 `/etc/apt/trusted.gpg.d/`{: .filepath} 目录下。

如果没有密钥文件，而是直接添加到了 `trusted.gpg`{: .filepath} 文件中，则可以使用 `apt-key export <email>` 命令导出密钥内容，并导入 KGpg 软件中，再导出公钥文件（或者直接保存为文件）。

### 推荐的

不过现在推荐把软件管理的仓库密钥放在 `/usr/share/keyrings/`{: .filepath} 目录下，然后在软件的 `sources.list`{: .filepath} 文件中，添加参数 `signed-by` 来指定密钥的路径。

比如在 `google-chrome.list`{: .filepath} 中原本的软件源命令是这样的：

```conf
deb [arch=amd64] https://dl.google.com/linux/chrome/deb/ stable main
```

添加参数后就是这样

```conf
deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] https://dl.google.com/linux/chrome/deb/ stable main
```

这里谷歌直接提供了密钥文件 `google-chrome.gpg`{: .filepath} 并放在 `trusted.gpg.d`{: .filepath} 目录中，我们可以直接转移。

其实这个密钥文件的位置是无所谓的，只不过大家约定俗成了都放在 `/usr/share/keyrings/`{: .filepath} 里面。如果不是软件管理的仓库密钥，则可以放在 `/etc/apt/keyrings/`{: .filepath} 里面。

## 双系统修改默认启动

以下操作需要管理员权限。

1. 备份一下 `/etc/default/grub`{: .filepath} 文件。
2. 打开该文件，将 `GRUB_DEFAULT` 修改为 Windows 启动项的序号（以 `0` 起始，假如 Windows 是第三个启动项，则此处改为 `2`）。
3. 运行 `update-grub` 。
4. 重启。

## Linux 挂载 U 盘后里面的中文文件名乱码

```shell
sudo mount -o iocharset=utf8 /dev/sdb1 /mnt/some
```
{: .nolineno}

## 检测 Shell 文件是否以管理员启动

```shell
#!/bin/bash

if [ "$UID" -ne "0" ]; then
    echo "Permission Denied. Try to run as root."
    exit
fi

# 以上条件也可以换为 "$(id -u)" -ne "0"
```
