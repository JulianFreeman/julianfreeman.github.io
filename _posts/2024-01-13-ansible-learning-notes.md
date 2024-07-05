---
title: Ansible 学习笔记
date: 2024-01-13 20:08:00 -0500
categories: [学习笔记, Ansible]
tags: [linux, ansible]
description: 本文记录学习 Ansible 的一些笔记
---

## ansible 安装

```shell
pip3 install ansible
```
{: .nolineno}

这样安装的版本比 apt 安装的版本要新，但是不包含配置文件。

## ansible 模块

ansible 有很多模块，不同的模块完成不同的功能，有些是官方维护的，有些是社区维护的。

```shell
ansible localhost -m copy -a 'src=/etc/passwd dest=/tmp'
```
{: .nolineno}

`localhost` 指定主机，也可以是其他 IP 地址。

`-m copy` 指定要使用的模块，这里是关于复制的模块。

`-a` 后面接模块的参数，这里有 `src` 和 `dest` 两个参数，分别是复制的文件和目标路径。

```shell
ansible-doc -l | grep 'copy'  # 可以查找名字里有 copy 的模块
```
{: .nolineno}

```shell
ansible-doc copy  # 查看 copy 模块的文档
```
{: .nolineno}

```shell
ansible-doc -s copy  # 查看 copy 参数有关的文档（简化版）
```
{: .nolineno}

```shell
ansible localhost -m file -a 'path=/tmp/passwd state=absent'
```
{: .nolineno}

在本机使用 `file` 模块，`path` 参数指定要操作的文件，`state` 是指定文件状态，`absent` 代表删除，类似于 `rm -r`。

`state` 为 `touch` 表示创建文件，为 `directory` 表示创建目录，类似于 `mkdir -p`。

```shell
ansible localhost -m debug -a 'msg="hello world"'
```
{: .nolineno}

`debug` 模块有两个常用参数，一个是 `msg`，可以输出信息和变量，另一个是 `var`，只能输出变量。

```shell
ansible localhost -e 'str=world' -m debug -a 'msg="hello {{str}}"'
```
{: .nolineno}

`-e` 指定变量，字符串值没有特殊符号的话可以不加引号，`msg` 里引用变量值时需要用双大括号，`var` 则不需要。

```shell
ansible localhost -e 'str="hello world"' -m debug -a 'var=str'
```
{: .nolineno}

## 配置文件

（优先级从上往下）

- `ANSIBLE_CFG`：环境变量指定的配置文件
- `ansible.cfg`：当前目录下的 `ansible.cfg`
- `~/.ansible.cfg`：家目录下的 `ansible.cfg`
- `/etc/ansible/ansible.cfg`：默认的全局配置文件

配置文件的一部分

```ini
[defaults]
# some basic default values...
#inventory      = /etc/ansible/hosts
#library        = /usr/share/my_modules/
#module_utils   = /usr/share/my_module_utils/
#remote_tmp     = ~/.ansible/tmp
#local_tmp      = ~/.ansible/tmp
#plugin_filters_cfg = /etc/ansible/plugin_filters.yml
#forks          = 5

[inventory]
#enable_plugins = host_list, virtualbox, yaml, constructed
#ignore_extensions = .pyc, .pyo, .swp, .bak, ~, .rpm, .md, .txt, ~, .orig, .ini, .cfg, .retry
#ignore_patterns=
#unparsed_is_failed=False
```

## 演员表

演员表 inventory 用来指定主机节点，也就是 ansible 要操作的远程主机。

`ansible -i /tmp/myinv.ini` 可以指定演员表，如果不指定，则使用 `/etc/ansible/hosts`。

定义一个节点，类似如下

```plaintext
node2 ansible_host=192.168.113.28 ansible_port=22 ansible_user=node2 ansible_password=123456
```

保存在 `/etc/ansible/hosts` 中。

更多参数见 [How to build your inventory — Ansible Documentation](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters)

然后就可以使用
 `ansible node2 -m copy -a "src=/etc/passwd dest=/tmp"`
操作远程主机了。

如果密码明写在文件里，需要安装 sshpass 包，如果不写，就会在运行 ansible 的时候提示输入（或者如果远程主机上保存了本地的 ssh 公钥，就不用密码了）。

演员表分组

```ini
[node23]
192.168.113.28 ansible_user=node2
192.168.113.29 ansible_user=node3

[node34]
192.168.113.29 ansible_user=node3
192.168.113.30 ansible_user=node4
```

同一个主机可以同时存在于不同的组，这里 `ansible_user` 必须指定，不然运行时默认用本机的用户名登录远程主机。

此时可以运行 

```shell
ansible node34 -m copy -a "src=/etc/passwd dest=/tmp"
```
{: .nolineno}

就是同时操作了 3 和 4 两台主机了。

有两个特殊的分组，`all` 表示所有分组内的几点，`ungrouped` 表示不在任意分组的节点，但两者都不包含 `localhost` 这个节点。

```shell
ansible node34 --list  # 列出该分组的所有主机
ansible-inventory node34 --graph  # 等价
ansible-inventory node34 --graph --vars  # 再加上每个主机的参数
ansible-inventory --list  # 列出所有主机
```
{: .nolineno}

这样子可以给 node23 组里的所有主机统一设置一个参数

```ini
[node23:vars]
ansible_port=22
```

这样表示定义一个node234组，里面包含node23组和node34组

```ini
[node234:children]
node23
node34
```

注意，这个 `:children` 必须要有。

上面的 ansible 命令在执行时都可以使用 `-i` 来指定演员表，一般来说，演员表就是一个文件，但是也可以 `-i` 指定一个目录，
那么目录里的所有文件都会被当作演员表解析。不过默认会有些文件后缀名被忽略，这个由配置文件的 `ignore_extensions` 定义。
所以演员表最后不要有后缀。

## 剧本

上面是一条一条命令执行的，很慢，不自动化，ansible 可以定义剧本，包含更多操作。

- 一个 playbook 有很多 play
- 一个 play 有 pre_tasks，好多 tasks，还有 post_tasks

一个剧本示例：first.yaml

```yaml
---
- name: play 1
  hosts: node23
  gather_facts: false
  tasks: 
    - name: task1 in play1
      debug: 
        msg: "output task1 in play1"

    - name: task2 in play1
      debug: 
        msg: "output task2 in play1"

- name: play 2
  hosts: node34
  gather_facts: false
  tasks: 
    - name: task1 in play2
      debug: 
        msg: "output task1 in play2"

    - name: task2 in play2
      debug: 
        msg: "output task2 in play2"
```

剧本是 yaml 格式，比较好理解，一共 2 个 play，每个 play 有 2 个 task，每个 play 用 `hosts` 定义要执行这些任务的主机组。

执行这个剧本，使用

```shell
ansible-playbook first.yml
```
{: .nolineno}

再来个剧本

```yaml
---
- name: copy play
  hosts: node34
  gather_facts: false

  tasks:
    - name: copy passwd
      copy:
        src: /etc/passwd
        dest: /tmp
```

play 和 task 中的 `name` 不是必需的，但是建议有。

每个 play 必须包含的只有 `hosts` 和 `tasks` 指令。

`gather_facts` 会在执行任务前收集主机的一些信息，会花一些时间。如果确定用不到这些信息，可以指定 `false` 不收集。

`hosts` 可以接 IP，主机组，通配符，以 `~` 开头可以使用正则表达式。

配置文件里的 `forks` 参数，可以指定同时开几个进程处理主机的任务，一个进程处理一个主机的任务。

## 配置密钥认证

常规步骤

```shell
ssh-keyscan $host >>~/.ssh/known_hosts 2>/dev/null
sshpass -p'123456' ssh-copy-id $user@$host &>/dev/null
```
{: .nolineno}

使用剧本

```yaml
---
- name: configure ssh connection
  hosts: node23
  gather_facts: false
  connection: local
  tasks:
    - name: configure ssh connection
      shell: |
        ssh-keyscan {{inventory_hostname}} >>~/.ssh/known_hosts
        sshpass -p'123456' ssh-copy-id root@{{inventory_hostname}}
```



