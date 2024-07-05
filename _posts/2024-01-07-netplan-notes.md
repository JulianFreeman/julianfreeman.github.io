---
title: netplan 命令
date: 2024-01-07 22:34:00 -0500
categories: [学习笔记, Linux]
tags: [linux, netplan]
description: 本文记录自己对 netplan 命令的一些理解
---

netplan 的配置文件一般在 `/etc/netplan`{: .filepath} 下，但是 `/run/netplan`{: .filepath} 会覆盖 `/etc/netplan`{: .filepath}，后者会覆盖 `/lib/netplan`{: .filepath}。

`/etc/netplan`{: .filepath} 下可以定义多个文件，文件以两位数字开头，数字大的会覆盖数字小的配置。

首先查看本机网卡叫啥名

```shell
ip -o a s
```
{: .nolineno}

比如我这台叫 enp0s3

则一个 netplan 的配置文件的基本部分可以有如下

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.113.18/24]
      routes:
        - to: default
          via: 192.168.113.1
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
  version: 2
  renderer: networkd
```

其中 `renderer: networkd` 是默认的，不写也行。

`dhcp4` 如果是 `true` 则会自动分配 IP，是 `false` 则需要手动设置 `addresses`，`routes` 这里相当于设置了网关，但我也不知道具体这个参数怎么用。看不懂文档。

修改完配置文件后，可以运行

```shell
sudo netplan try
```
{: .nolineno}

检测配置文件是否正确，如果正确，可以回车应用配置，如果有错误，则会保持原样。

如果足够自信，也可以直接应用配置

```shell
sudo netplan apply
```
{: .nolineno}
