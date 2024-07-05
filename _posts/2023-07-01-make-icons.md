---
title: 制作图标文件
date: 2023-07-01 10:02:00 -0400
categories: [教程, 图标]
tags: [icons]
description: 本文介绍如何制作一个图标文件
---
## 2023-07-11 更新

发现了一个网站，[Icon Archive][iconarchive]，可以直接保存 .ico 和 .icns 图标，也可以保存其他不同大小的图片，十分方便。

---

## 原文

这里说的图标文件，指的是在 Windows 上或者 MacOS 上附加在应用程序上的图标文件。

应用程序的图标文件通常是一个包含多个大小不同的图标文件的集合，比如通常可能会有 16x16、32x32、48x48、128x128、256x256 等大小，为的是使应用在不同视图下都能以最合适的大小显示程序图标。在 [Windows](#windows-版) 上后缀为 .ico，在 [MacOS](#macos-版) 上后缀为 .icns。

## 准备一个 SVG 文件

在制作一个图标文件之前，当然是先准备一下图标，这里选择 SVG 格式的图标，因为这是一种矢量格式，在生成不同大小的图片文件时不会损失清晰度。下面推荐几个可以找图标的网站：

- [Ionicons][ionicons]
- [Freeicons][freeicons]

## Windows 版

### 准备工作

安装 [Inkscape][inkscape]，安装 [ImageMagick][imagemagick]。

Inkscape 并不会把自己的 `bin`{: .filepath} 目录自动添加到环境变量，所以需要手动添加一下。该目录通常是 `C:\Program Files\Inkscape\bin`{: .filepath}。

安装 ImageMagick 时记得勾选“安装常规辅助工具，例如 convert”。

### Batch 脚本

```batch
@echo off
setlocal EnableDelayedExpansion

@rem 获取 .svg 文件名
set "file=%~n1"
@rem 要生成的图片大小集合
set sizes=(16 32 48 128 256)
@rem 生成形如 " icon_16.png icon_32.png icon_48.png" 的字符串
set enames=
for %%s in %sizes% do (
    set "enames=!enames! %file%_%%s.png"
)

@rem 调用 inkscape 转换并生成不同大小的图片
for %%s in %sizes% do (
    inkscape --export-type=png --export-filename="%file%_%%s.png" -w %%s -h %%s %~1
)

@rem 调用 imagemagick 的 convert 将一系列不同大小的 .png 图片转为一张 .ico 图片
convert %enames% -colors 256 %file%.ico
@rem 等待 3 秒，让上一个命令转换完（可能不是很必要）
timeout /t 3

@rem 将生成的 .png 图片文件归个类（额外的处理）
mkdir %file%
echo y | del %file%\*
for %%s in %sizes% do (
    move %file%_%%s.png %file%
)

setlocal DisableDelayedExpansion
```

将上述脚本保存为 `svg2png.bat`{: .filepath} 文件，使用方法为 

```batch
svg2png.bat icon.svg
```
{: .nolineno}

与 .svg 文件同级的目录下会出现一个同名的 .ico 文件。

## MacOS 版

### 准备工作

安装 [Inkscape][inkscape]，安装 [Image2Icon][image2icon]。

安装完 Inkscape 之后貌似默认无法使用命令行调用 `inkscape` 命令，因此先执行以下命令创建一个软链接：

```shell
ln -s /Applications/Inkscape.app/Contents/MacOS/inkscape /usr/local/bin/inkscape
```
{: .nolineno}

这样就可以直接在终端调用 `inkscape` 命令了。

### SVG to PNG

[Image2Icon][im2icns-suggest] 建议使用 1024x1024 大小的图片创建 .icns 图标，因此我们先用 Inkscape 把 .svg 文件转成 1024x1024 大小的 .png 文件。在终端运行如下命令：

```shell
inkscape --export-type=png --export-filename="1024.png" -w 1024 -h 1024 icon.svg
```

其中 `icon.svg`{: .filepath} 就是我们提前准备的 SVG 图标文件了。此命令在该文件的同级目录下生成一个 `1024.png`{: .filepath} 文件。

### PNG to ICNS

打开 Image2Icon 软件，将刚才创建的 `1024.png`{: .filepath} 文件拖放进软件界面，点击“导出”，选择 ICNS 格式，“保存图标”就可以了。

[ionicons]: https://ionic.io/ionicons
[freeicons]: https://freeicons.io/
[inkscape]: https://inkscape.org/
[image2icon]: https://img2icnsapp.com/
[im2icns-suggest]: https://img2icnsapp.com/how-to-create-the-best-mac-icons/
[imagemagick]: https://imagemagick.org/

[iconarchive]: https://www.iconarchive.com/
