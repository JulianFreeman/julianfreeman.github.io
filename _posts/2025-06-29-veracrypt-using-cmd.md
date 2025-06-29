---
title: 在命令行中使用 VeraCrypt
date: 2025-06-29 09:00:00 -0400
categories: [笔记, VeraCrypt]
tags: [veracrypt]
description: 如何在不明文暴露密码的情况下使用命令行打开 VeraCrypt 加密卷
---

## Windows 版本

使用命令行打开加密卷，就涉及到指定密码，直接到脚本中指定密码不太安全，因此使用凭据管理器保存密码。

打开凭据管理器，在 Windows 凭据下，添加 **普通凭据**，其中网络地址可以理解为引用这个凭据的变量，用户名随便填，用不到，密码就是要保存使用的密码，此处也就是加密卷的密码。也可以再创建一个凭据，把 PIM 也存一下。

要使用凭据，需要安装一个 PowerShell 的模块，地址是：[CredentialManager 2.0](https://www.powershellgallery.com/packages/CredentialManager/2.0)。

但是这个模块只能在 PowerShell 5.1 中使用，在 PowerShell 7 中无法使用，所以需要在 PowerShell 5.1 的窗口环境下安装。运行

```powershell
Install-Module -Name CredentialManager -Scope CurrentUser
```

可能提示安装 nuget，可以同意，然后再次同意从 PowerShell Gallery 中安装。

可以通过如下命令查看是否安装成功。

```powershell
Get-Command -Module CredentialManager
```

编写 ps1 脚本打开加密卷

```powershell
# 确保加载 CredentialManager 模块
Import-Module CredentialManager

# 获取密码
$creds_pwd = Get-StoredCredential -Target "<pwd_cred_name>"
$securePassword = $creds_pwd.Password
$ptr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($securePassword)
$password = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ptr)

# 获取 PIM
$creds_pim = Get-StoredCredential -Target "<pim_cred_name>"
$securePim = $creds_pim.Password
$pimPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($securePim)
$pim = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($pimPtr)

# VeraCrypt 挂载参数
$vcPath = "C:\Program Files\VeraCrypt\VeraCrypt.exe"
$volumePath = "<C:\path\to\volume\file>"
$driveLetter = "A" # 挂载的盘符

$args = @(
    "/v", $volumePath,
    "/l", $driveLetter,
    "/p", "`"$password`"", # 防止特殊字符干扰命令行解析
    "/pim", $pim,
    "/hash", "sha-512", # 指定挂载参数，可以加快挂载速度
    "/q", "/s"
)

# $args | ForEach-Object { Write-Host $_ }
Start-Process -FilePath $vcPath -ArgumentList $args
```

如果加密卷没有 PIM 可以省略。

如果不好直接执行 ps1 文件，可以写一个 bat 文件

```batch
@echo off
powershell -ExecutionPolicy Bypass -Command "& '.\mount_vl.ps1'"
```

卸载加密卷的脚本比较简单

```powershell
$vcPath = "C:\Program Files\VeraCrypt\VeraCrypt.exe"
$driveLetter = "A" # 要卸载的盘符

Start-Process -FilePath $vcPath -ArgumentList @(
    "/u", $driveLetter, # 使用 /u 参数，而不是 /d
    "/q", "/s"
)
```

## MacOS 版本

将密码保存在 KeyChain 中。打开钥匙串访问，在 **登录** 的项目下，新增一个密码

名称可以理解为变量名，后续需要引用，账户名称随便填，密码即要保存的密码。

写一个脚本，同时赋予执行权限

```bash
#!/bin/bash

VOLUME_PATH="</path/to/volume/file>"
ACCOUNT_NAME="<acc_name>"
PWD_SERVICE_NAME="<pwd_key_name>"
PIM_SERVICE_NAME="<pim_key_name>"
VERACRYPT="/Applications/VeraCrypt.app/Contents/MacOS/VeraCrypt"

PWD=$(security find-generic-password -a "$ACCOUNT_NAME" -s "$PWD_SERVICE_NAME" -w)
PIM=$(security find-generic-password -a "$ACCOUNT_NAME" -s "$PIM_SERVICE_NAME" -w)

"$VERACRYPT" --text --non-interactive \
  --mount "$VOLUME_PATH" \
  --password "$PWD" \
  --pim "$PIM" \
  --slot 1

```

上面挂载的时候如果指定一个挂载点，但这个挂载点又需要提权限才能访问的话，用起来就比较麻烦，因此上述挂载脚本就没有指定挂载点，这样 VC 也会自己找地方挂载。

```bash
chmod u+x mount_vl.sh
```

现在命令行中执行一下看看正不正常，可能需要输入密码访问钥匙串，可以点始终允许，避免后续每次都输入。

打开 脚本编辑器，输入

```bash
do shell script "/path/to/mount_vl.sh"
```

这里最好用完整路径。

然后导出为应用程序即可。

