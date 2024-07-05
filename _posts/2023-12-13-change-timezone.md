---
title: 如何在 Linux 中设置或修改时区
date: 2023-12-13 14:34:00 -0500
categories: [翻译, 时区]
tags: [linux, timezone]
description: 本文是 Linuxize 网站关于更改 Linux 时区的一篇文章的翻译
---

原文：https://linuxize.com/post/how-to-set-or-change-timezone-in-linux/

## 如何在 Linux 中设置或修改时区

一个时区是指拥有相同标准时间的一块地理区域。一般来说，时区是在安装操作系统时设置的，但是安装后再设置也是很容易的。

使用正确的时区对于许多跟系统有关的任务和进程来说是很重要的。比如说，一个定时命令的守护进程使用系统时区来执行定时任务。时区也被用在日志的时间戳上。

这个教程介绍了在 Linux 中设置或修改时区的必要步骤。

### 检查当前时区

`timedatectl` 是一个命令行工具，可以允许我们查看和修改系统的时间和日期。在现代所有基于 `systemd` 的 Linux 系统中都包含这个命令。

要查看当前时区，不加参数直接运行 `timedatectl`

```terminal
julian@basoom:~$ timedatectl
               Local time: Wed 2023-12-13 18:11:22 UTC
           Universal time: Wed 2023-12-13 18:11:22 UTC
                 RTC time: Wed 2023-12-13 18:11:23
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

上面的输出中显示系统的时区是 UTC。

系统的时区是通过创建一个 `/etc/localtime` 软链接并指向位于 `/usr/share/zoneinfo` 目录的某一个二进制时区标识文件来设置的。

另一种查看时区的办法是用 `ls` 命令查看软链接的指向

```terminal
julian@basoom:~$ ls -l /etc/localtime 
lrwxrwxrwx 1 root root 27 Dec 13 18:11 /etc/localtime -> /usr/share/zoneinfo/Etc/UTC
```

### 在 Linux 中修改时区

在修改时区之前，你需要知道你想要使用的时区的全名。时区的命名规则遵守“地区/城市”的格式。

要查看所有可用的时区，使用 `timedatectl list-timezones` 命令或者直接列举出 `/usr/share/zoneinfo` 目录下的所有文件。

```terminal
julian@basoom:~$ timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
Africa/Asmera
Africa/Bamako
Africa/Bangui
Africa/Banjul
...
```

当你知道了你要使用的时区的全名后，假如说是 `America/New_York`，使用管理员权限运行如下命令

```shell
sudo timedatectl set-timezone America/New_York
```
{: .nolineno}

此时再使用 `timedatectl` 命令查看是否修改成功

```terminal
julian@basoom:~$ timedatectl
               Local time: Wed 2023-12-13 13:21:16 EST
           Universal time: Wed 2023-12-13 18:21:16 UTC
                 RTC time: Wed 2023-12-13 18:21:16
                Time zone: America/New_York (EST, -0500)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

恭喜！你成功修改了系统的时区！

### 通过创建软链接修改时区

如果你运行的系统比较老，没有 `timedatectl` 这个工具，你也可以通过修改 `/etc/localtime` 软链接的指向来修改时区。

先移除当前的软链接文件

```shell
sudo rm -rf /etc/localtime
```
{: .nolineno}

找到你要设置的时区文件，并创建软链接

```shell
sudo ln -s /usr/share/zoneinfo/America/New_York /etc/localtime
```
{: .nolineno}

要检查是否成功，可以用 `date` 命令，输出中会包含时区信息。
