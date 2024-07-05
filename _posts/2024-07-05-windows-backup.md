---
title: Windows 备忘笔记
date: 2024-07-05 18:25:00 -0400
categories: [笔记, Windows]
tags: [windows]
description: 记录一些使用 Windows 过程中遇到的问题
---

## 检测 Batch 文件是否以管理员启动

```batch
@echo off

rem check if it is run as admin
if not "%1" == "am_admin" (
    rem you'd better keep the following line as it is.
    powershell start -verb runas '%0' am_admin & exit /b
)
```

## 在 Windows 中创建快捷方式时不添加“-快捷方式”字样

找到注册表位置 `\HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer`。点击 Explorer，在右侧找到 link，默认值为 `1e000000`，修改为 `00000000`，不需要重启。

## 通过注册表修改默认应用

打开注册表，进入 `HKEY_CLASSES_ROOT`，找到自己要修改的后缀名文件，查看其数据，比如 `.blend` 的数据是 blendfile，继续向下找 blendfile，找到 `shell/open/command`，在数据中修改执行文件路径即可。
