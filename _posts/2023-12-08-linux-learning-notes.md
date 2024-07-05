---
title: Linux 学习笔记
date: 2023-12-08 10:57:00 -0500
categories: [学习笔记, linux]
tags: [linux]
description: 兄弟连细说 Linux 视频教程的笔记。根据个人情况，会的内容就跳过了，所以章节间不全。
---

## 第四讲 Linux 常用命令

### 4.1 文件处理命令

#### 4.1.1 命令格式与目录处理命令 `ls`

使用 `ls -l /home/julian` 显示的是该目录下文件和目录的信息，要想查看该目录本身的信息，可以用 `-d` 参数。

```terminal
julian@ubuntu2204:~$ ls -l /home/julian
total 36
drwxr-xr-x 3 julian julian 4096 Nov 24 13:26 Desktop
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Documents
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Downloads
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Music
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Pictures
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Public
drwx------ 3 julian julian 4096 Nov 24 11:54 snap
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Templates
drwxr-xr-x 2 julian julian 4096 Nov 24 11:55 Videos
```

```terminal
julian@ubuntu2204:~$ ls -ld /home/julian
drwxr-x--- 14 julian julian 4096 Nov 24 13:31 /home/julian
```

#### 4.1.2 目录处理命令

##### `mkdir`

`mkdir` 的 `-p` 参数可以 **递归** 创建目录。

```terminal
julian@ubuntu2204:~/Documents$ mkdir -p ./abc/def
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    └── def

2 directories, 0 files
```

`mkdir` 可以同时创建 **多个** 目录。

```terminal
julian@ubuntu2204:~/Documents$ mkdir ./abc/ghi ./abc/jkl
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    └── jkl

4 directories, 0 files
```

##### `cp`

`cp` 可以同时复制 **多个** 文件，只要把目标目录放在最后就好。

```terminal
julian@ubuntu2204:~/Documents$ cp /etc/hosts /etc/hosts.allow ./abc/
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    ├── hosts
    ├── hosts.allow
    └── jkl

4 directories, 2 files
```

`cp` 默认只能复制文件，但是加 `-r` 参数可以 **递归** 复制目录。

```terminal
julian@ubuntu2204:~/Documents$ cp -r /etc/environment.d/ ./abc/jkl/
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    ├── hosts
    ├── hosts.allow
    └── jkl
        └── environment.d
            ├── 90atk-adaptor.conf
            └── 90qt-a11y.conf

5 directories, 4 files
```

当然同时复制文件和目录也是可以的，只要加上 `-r` 而且最后是目标目录就好。

```terminal
julian@ubuntu2204:~/Documents$ cp -r ./abc/jkl/environment.d/ ./abc/hosts ./abc/def/
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    │   ├── environment.d
    │   │   ├── 90atk-adaptor.conf
    │   │   └── 90qt-a11y.conf
    │   └── hosts
    ├── ghi
    ├── hosts
    ├── hosts.allow
    └── jkl
        └── environment.d
            ├── 90atk-adaptor.conf
            └── 90qt-a11y.conf

6 directories, 7 files
```

`cp` 复制时默认不复制源文件属性，加 `-p` 参数可以保留源文件属性（比如时间）。

```terminal
julian@ubuntu2204:~/Documents$ ls -l /etc/hosts
-rw-r--r-- 1 root root 225 Nov 24 11:47 /etc/hosts

julian@ubuntu2204:~/Documents$ ls -l ./abc/hosts
-rw-r--r-- 1 julian julian 225 Nov 24 14:08 ./abc/hosts

julian@ubuntu2204:~/Documents$ cp -p /etc/hosts ./abc/ghi/

julian@ubuntu2204:~/Documents$ ls -l ./abc/ghi/hosts 
-rw-r--r-- 1 julian julian 225 Nov 24 11:47 ./abc/ghi/hosts
```

`cp` 在复制文件或目录的同时也可以 **重命名**，只要 **目标路径不存在** 即可。

```terminal
julian@ubuntu2204:~/Documents$ cp /etc/hosts ./abc/myhosts
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── ghi
    └── myhosts

3 directories, 1 file
```

```terminal
julian@ubuntu2204:~/Documents$ cp -r /etc/environment.d/ ./abc/env.d
julian@ubuntu2204:~/Documents$ tree
.
└── abc
    ├── def
    ├── env.d
    │   ├── 90atk-adaptor.conf
    │   └── 90qt-a11y.conf
    ├── ghi
    └── myhosts

4 directories, 3 files
```

但是重命名只能在复制 **单个** 文件或者目录时进行，如果复制多个文件或目录，目标路径 **必须是一个存在的目录**。

##### 关于 `cp` 的一些使用情况说明

参数中涉及到目录时，最后加不加 `/` 应该影响不大，因为同一目录下的子文件和子目录之间也不能重名，所以程序能知道哪个是目录，哪个是文件。

当路径最后有 `/` 的时候，程序会把该路径当目录处理，如果找不到这个目录，哪怕能找到同名文件，也会报错，因为文件不是目录。

当路径最后没有 `/` 的时候，程序会根据情况，找到发现是目录，就当目录处理，找到发现是文件，就当文件处理，啥也没找到，就报错。这里目录和文件没有先后，因为不可能既有目录又有文件，前面说了，文件和目录也不能重名。

所以为了严谨，我们可以把目录路径后加 `/` 以作标识。

当使用 `cp` 复制单个路径时，

- 如果源路径是文件，
  - 如果目标路径是文件，
    - 如果目标文件不存在：复制并重命名
    - 如果目标文件存在：覆盖复制并可能重命名（取决于源和目标是不是重名）
  - 如果目标路径是目录，
    - 如果目标目录不存在：报错
    - 如果目标目录存在：复制到该目录下（不关心目标目录与源文件是否重名）
- 如果源路径是目录，
  - 如果目标路径是文件（这里指路径最后没有加 `/`），
    - 如果目标文件不存在：复制并重命名（目标路径当作目录处理）
    - 如果目标文件存在：报错（文件和目录不能重名）
  - 如果目标路径是目录：
    - 如果目标目录不存在：复制并重命名
    - 如果目标目录存在：复制到该目录下

当使用 `cp` 复制多个路径时，

- 如果目标路径是一个存在的目录：复制到该目录下
- 否则，
  - 如果目标路径是一个存在的文件：报错（文件不是目录）
  - 如果目标路径不存在：报错（必须存在）

所以，要判断一个 `cp` 命令会不会导致重命名，

1. 看源路径是不是只有一个
2. 看目标路径是不是不存在

不考虑报错且源路径只有一个的情况下，目标路径不存在 **一定会重命名**，存在则 **只有** 一种情况会重命名，也就是 **源和目标都存在且都是文件时**，也就是发生 **覆盖** 时。

##### `mv`

`mv` 命令可以理解为 `cp` 之后把源文件或目录也删掉，因此其对复制或者重命名的处理跟 `cp` 是一样的。只不过 `cp` 要处理目录的话得加 `-r` 参数，但 `mv` 不需要加参数就可以处理目录。

其实，把上面的分层结构中的“复制”改为“移动”就符合 `mv` 的情况了，无论源路径是单个还是多个，都符合。

#### 4.1.3 文件处理命令

##### `cat`

`cat` 就是打印 **所有** 文件内容，没法分页，也没法打印局部。其 `-n` 参数指的是打印时 **添加行号**。

`-A` 可以显示非打印字符，比如行尾换行符 `\n` 显示为 `$`，Windows 文件的行尾换行会显示为 `^M$`，有时候会导致一些错误。`dos2unix` 命令可以把 Windows 换行符转换为 Linux 换行符。这个包可能需要手动安装。

`batcat`（`sudo apt install bat`）可以根据不同的文件或编程语言高亮文件内容，同时也支持 `-A` 查看非打印字符。`batcat --help` 查看更多选项。

##### 命令行查看文档的一些操作

> `more` 好像也可以往上翻页。

使用 `less` 或者 `man` 之类的命令在命令行查看帮助文档时，

- 敲 `/`，输入要搜索的文本，回车，文档内能匹配的文本会高亮，此时敲 `n` 可以查看下一个匹配，敲 `N` 可以查看上一个匹配（`Shift+n`）

> `vim` 也可以，不过这个可以单独讲了。
{: .prompt-info}

#### 4.1.4 链接命令

`ln -s [源路径] [目标路径]` 可以创建软链接。

如果目标路径就在当前的工作目录中，则源路径是相对路径或者绝对路径都可以。

如果目标路径不在当前的工作目录中，则源路径必须是绝对路径，否则软链接不生效。

`ln` 创建硬链接则好像没有这个要求。不过硬链接只能作用于文件，且不能跨分区。

### 4.2 权限管理命令

#### 4.2.1 权限管理命令 `chmod`

同时进行多种授权时用逗号隔开：

```terminal
julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-r--r-- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp

julian@ubuntu2204:~/Documents$ chmod g+w,o-r ./abc/issue.cp 

julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-rw---- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp
```

授权按照先后顺序进行，比如上例，`g+w,o-r` 表示先给所属组加写权限，再给其他人减读权限。这样如果权限的设置前后有冲突，以在后的为准。

当然，用数字就没这些冲突了：

```terminal
julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-rw---- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp

julian@ubuntu2204:~/Documents$ chmod 644 ./abc/issue.cp 

julian@ubuntu2204:~/Documents$ ls -l ./abc/issue.cp 
-rw-r--r-- 2 julian julian 26 Nov 24 20:39 ./abc/issue.cp
```

文件或目录的权限只有其 **所有者和 root 用户** 可以修改。

##### 权限对文件和目录的含义

|权限|对文件|对目录|
|--:|--|--|
|读权限 `r`|可以查看文件内容|可以列出目录内容|
|写权限 `w`|可以修改文件内容|可以在目录创建、删除（也就是修改目录里的内容）|
|执行权限 `x`|可以执行文件|可以进入目录|

可能把目录理解为记录了其内所有文件和目录名称的名单文件，就好理解对目录的读写权限的含义了。读就是能看到名单文件的内容，也就是目录里有什么文件和目录，写就是修改这个名单文件，也就是修改目录里的文件和目录，即创建或删除。

**目录的读和执行权限都是同时出现的**，否则不能正常使用。

#### 4.2.2 其他权限管理命令

##### `chown`

只有 root 用户可以修改文件的所有者，所有者自己都不行。

`chown` 文档原话：

- `chown` 修改给定的每一个文件的所有者和/或所属组。
- 如果指定了用户名，这个用户名就被修改为文件的所有者，文件的所属组不变。

```terminal
root@ubuntu2204:/tmp/abc# ls -dl def/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:03 def/

root@ubuntu2204:/tmp/abc# chown julian def/

root@ubuntu2204:/tmp/abc# ls -dl def/
drwxrwxr-x 2 julian karl 4096 Nov 24 23:03 def/
```

- 如果用户名后面跟着一个冒号和一个组名，中间没有空格，那么文件的所属组也会改变。

```terminal
root@ubuntu2204:/tmp/abc# ls -dl ghi/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 ghi/

root@ubuntu2204:/tmp/abc# chown julian:julian ghi/

root@ubuntu2204:/tmp/abc# ls -dl ghi/
drwxrwxr-x 2 julian julian 4096 Nov 24 23:13 ghi/
```

- 如果用户名后面有冒号但没有指定组名，那么文件的所有者改为该用户，所属组改为该用户的登录组（login group）。

```terminal
root@ubuntu2204:/tmp/abc# ls -dl jkl/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 jkl/

root@ubuntu2204:/tmp/abc# chown julian: jkl/

root@ubuntu2204:/tmp/abc# ls -dl jkl/
drwxrwxr-x 2 julian julian 4096 Nov 24 23:13 jkl/
```

- 如果有冒号和组名，但没有指定用户名，那么只有文件的所属组被改变，这种情况下 `chown` 就跟 `chgrp` 作用一样了。

```terminal
root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 mno/

root@ubuntu2204:/tmp/abc# chown :julian mno/

root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl julian 4096 Nov 24 23:13 mno/
```

- 如果只给了一个冒号，或者啥也没给，那么什么都不会改变。

##### `chgrp`

修改所属组就很简单了，当然也只有 root 用户有权限修改。

```terminal
root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl julian 4096 Nov 24 23:13 mno/

root@ubuntu2204:/tmp/abc# chgrp karl mno/

root@ubuntu2204:/tmp/abc# ls -dl mno/
drwxrwxr-x 2 karl karl 4096 Nov 24 23:13 mno/
```

##### `umask`

`umask` 设定的是创建文件时的初始权限。加 `-S` 参数可以以更直观的方式显示。

```terminal
julian@ubuntu2204:~/Documents$ umask -S
u=rwx,g=rwx,o=rx
```

这意味着创建新的目录，其初始权限为 `rwxrwxr-x`。因为 Linux 规定缺省权限创建的新文件不能有执行权限，因此默认新文件的初始权限为 `rw-rw-r--`。

### 4.3 文件搜索命令

#### 4.3.1 文件搜索命令 `find`

##### 按名称查找

名称全匹配：

```terminal
root@ubuntu2204:~# find /etc -name init
/etc/init
/etc/apparmor/init
```

名称包含匹配：

```terminal
root@ubuntu2204:~# find /etc -name *init*
/etc/init
/etc/security/namespace.init
/etc/apparmor/init
/etc/gdb/gdbinit
/etc/gdb/gdbinit.d
/etc/X11/xinit
/etc/X11/xinit/xinitrc
...
```

名称不区分大小写：

```terminal
root@ubuntu2204:~# find /etc -iname init
/etc/init
/etc/apparmor/init
/etc/gdm3/Init
```

##### 按大小查找

`-size` 后的参数的单位是块，1 个块是 512B，因此 204800 个块是 204800 * 512B = 100MB 大小，加号代表大于，减号代表小于

```terminal
root@ubuntu2204:~# find / -size +204800
/snap/firefox/2987/usr/lib/firefox/libxul.so
/snap/gnome-42-2204/141/usr/lib/x86_64-linux-gnu/libLLVM-15.so.1
/snap/gnome-42-2204/120/usr/lib/x86_64-linux-gnu/libLLVM-15.so.1
...

root@ubuntu2204:~# ls -lh /snap/firefox/2987/usr/lib/firefox/libxul.so
-rwxr-xr-x 1 root root 138M Aug  7 06:31 /snap/firefox/2987/usr/lib/firefox/libxul.so
```

##### 按用户或组查找

`-user` 按照所有者查找，`-group` 按照所属组查找。

##### 按时间查找

`-amin` 访问文件时间（access），`-cmin` 修改文件属性时间（change），`-mmin` 修改文件内容时间（modify），后面的数字单位都是分钟，加号代表超过这个时间，减号代表在这个时间之内

下例表示 `/etc` 目录中 100 分钟之内修改过文件属性的文件

```terminal
root@ubuntu2204:~# find /etc -cmin -100
/etc/cups
/etc/cups/subscriptions.conf
/etc/cups/subscriptions.conf.O
```

##### 结合多个条件查找

`-a` 表示 and，即多个条件同时满足

下例表示查找大小大于 80MB 同时小于 100MB 的文件

```terminal
root@ubuntu2204:~# find / -size +163840 -a -size -204800
/snap/gnome-3-38-2004/143/usr/lib/x86_64-linux-gnu/libLLVM-12.so.1
...

root@ubuntu2204:~# ls -hl /snap/gnome-3-38-2004/143/usr/lib/x86_64-linux-gnu/libLLVM-12.so.1
-rw-r--r-- 1 root root 88M Mar  8  2022 /snap/gnome-3-38-2004/143/usr/lib/x86_64-linux-gnu/libLLVM-12.so.1
```

`-o` 表示 or，即多个条件满足一个即可

##### 按类型查找

`-type` 可以接 `f`（文件），`d`（目录），`l`（软链接）

下例表示查找名称为 `init` 开头的目录

```terminal
root@ubuntu2204:~# find /etc -name init* -a -type d
/etc/init
/etc/apparmor/init
/etc/init.d
/etc/initramfs-tools
/etc/initramfs-tools/scripts/init-premount
/etc/initramfs-tools/scripts/init-bottom
/etc/initramfs-tools/scripts/init-top
```

##### 查找后执行

`-exec` 后接要执行的命令，前面查找的结果用 `{}` 替换，最后加一个 `\;`

下例查找 `/etc` 目录下所有以 `init` 开头的目录后执行 `ls -dl` 列举了它们的属性

```terminal
root@ubuntu2204:~# find /etc -name init* -a -type d -exec ls -dl {} \;
drwxr-xr-x 2 root root 4096 Nov 24 11:49 /etc/init
drwxr-xr-x 3 root root 4096 Aug  7 19:53 /etc/apparmor/init
drwxr-xr-x 2 root root 4096 Nov 24 12:22 /etc/init.d
drwxr-xr-x 5 root root 4096 Nov 24 12:22 /etc/initramfs-tools
drwxr-xr-x 2 root root 4096 Jun 14 03:54 /etc/initramfs-tools/scripts/init-premount
drwxr-xr-x 2 root root 4096 Jun 14 03:54 /etc/initramfs-tools/scripts/init-bottom
drwxr-xr-x 2 root root 4096 Jun 14 03:54 /etc/initramfs-tools/scripts/init-top
```

将 `-exec` 替换为 `-ok` 会在对每一个查找到的结果执行命令前进行确认，通常如果查找后执行的是删除命令，`-ok` 可能会有用。

##### 按 i 结点查找

`-inum` 可以按照文件结点查找。

如果有一个名称很怪的文件，不知道怎么手打出来，此时可以先查看其 i 结点号，然后用 `find` 找到后再执行某些操作

```terminal
root@ubuntu2204:/tmp/abc# ls -i
655376 'weird name file'

root@ubuntu2204:/tmp/abc# find . -inum 655376 -exec cat {} \;
pretend this is a file with a weird name
```

同时使用 i 结点查找也能找到一块分区内某一个文件的所有硬链接。

#### 4.3.2 其他搜索命令

##### `locate`

在 Ubuntu 22.04.3 中这个命令没有默认安装，需要安装 `plocate` 包。

使用前先用 `updatedb` 命令更新索引库，默认的索引文件在 `/var/lib/plocate/plocate.db`{: .filepath}。

`locate` 区分大小写，但并不是全匹配的

```terminal
julian@ubuntu2204:~/temp$ locate to_be
/home/julian/temp/to_be_or_not
```

`-i` 可以不区分大小写

```terminal
julian@ubuntu2204:~/temp$ locate -i to_be
/home/julian/temp/TO_BE_OR_NOT
/home/julian/temp/to_be_or_not
```

`locate` 默认匹配的是全路径，也就是说，只要一个目录名符合条件，该目录下的所有文件和目录都符合条件，因为它们的路径中一定会包含这个符合条件的父目录

```terminal
julian@ubuntu2204:~/temp$ tree
.
├── to_be
│   ├── bill.t
│   └── shakes.txt
├── to_be_or_not
└── TO_BE_OR_NOT

1 directory, 4 files
julian@ubuntu2204:~/temp$ locate to_be
/home/julian/temp/to_be
/home/julian/temp/to_be_or_not
/home/julian/temp/to_be/bill.t
/home/julian/temp/to_be/shakes.txt
```

`-b` 参数可以设置为只匹配文件名或目录名

```terminal
julian@ubuntu2204:~/temp$ locate -b to_be
/home/julian/temp/to_be
/home/julian/temp/to_be_or_not
```

`-c` 参数可以只计数而不罗列所有项

```terminal
julian@ubuntu2204:~/temp$ locate -bc init
3070
```

有些路径 `locate` 并不会索引，比如 `/tmp`{: .filepath} 目录。

##### `which`

搜索命令文件的绝对路径

```terminal
julian@ubuntu2204:~/temp$ which ls
/usr/bin/ls
```

##### `whereis`

搜素命令文件的绝对路径和该命令文档的绝对路径

```terminal
julian@ubuntu2204:~/temp$ whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

##### `grep`

文件内容查找命令

```terminal
julian@ubuntu2204:~/temp$ grep "to be" to_be_or_not 
So this is a to be or not to be question
so to be or not to be ?

julian@ubuntu2204:~/temp$ grep so to_be_or_not 
so to be or not to be ?
```

同样默认区分大小写，加 `-i` 不区分

```terminal
julian@ubuntu2204:~/temp$ grep -i so to_be_or_not 
So this is a to be or not to be question
so to be or not to be ?
```

`-v` 反选

```terminal
julian@ubuntu2204:~/temp$ grep -vi so to_be_or_not 
yes you're right.
nah, this is only a text, don't be serious
```

### 4.4 帮助命令

`man` 命令不仅可以看命令的帮助文档，也可以看配置文件的帮助文档。比如 `man services` 会打开 `/etc/services`{: .filepath} 配置文件的帮助文档。

查看 `passwd` 会发现它即是一个命令，也是一个配置文件

```terminal
julian@ubuntu2204:~/temp$ whereis passwd
passwd: /usr/bin/passwd /etc/passwd /usr/share/man/man1/passwd.1.gz /usr/share/man/man1/passwd.1ssl.gz /usr/share/man/man5/passwd.5.gz
```

此时调用 `man passwd` 默认显示的是命令的帮助文档，如果想看配置文件的帮助文档，可以用 `man 5 passwd`。

Linux 的帮助文档中 `1` 表示命令的帮助文档，`5` 表示配置文件的帮助文档。

`whatis` 可以查看命令和配置文件的 *一行* 帮助信息。

`apropos` 可以检索帮助文档

```terminal
julian@ubuntu2204:~/temp$ whatis apropos
apropos (1)          - search the manual page names and descriptions

julian@ubuntu2204:~/temp$ apropos passwd
chgpasswd (8)        - update group passwords in batch mode
chpasswd (8)         - update passwords in batch mode
fgetpwent_r (3)      - get passwd file entry reentrantly
getpwent_r (3)       - get passwd file entry reentrantly
gpasswd (1)          - administer /etc/group and /etc/gshadow
grub-mkpasswd-pbkdf2 (1) - generate hashed password for GRUB
openssl-passwd (1ssl) - compute password hashes
pam_localuser (8)    - require users to be listed in /etc/passwd
passwd (1)           - change user password
passwd (1ssl)        - OpenSSL application commands
passwd (5)           - the password file
passwd2des (3)       - RFS password encryption
update-passwd (8)    - safely update /etc/passwd, /etc/shadow and /etc/group
```

有些 Shell 内置命令无法用 `man` 查看帮助文档，比如 `cd`。

用 `help` 可以查看内置命令的帮助文档。比如 `help cd`。

比如 `help if` 也是可以的。

### 4.5 用户管理命令

`useradd` 是一个更底层的命令，常规添加用户应该用 `adduser`。

`adduser` 可以创建普通用户，创建系统用户，创建普通组，创建系统组，把一个存在的用户添加到一个存在的组。

另见 [`useradd`](#731-用户添加命令-useradd)。

`passwd` 可以修改用户的密码。普通用户只能修改自己的密码，而且还要符合复杂度的要求，但是 root 用户可以修改所有人的密码，且不用遵守复杂度要求。

另见 [`passwd`](#732-修改用户密码-passwd)。

`who` 可以查看现在有多少个用户在登录。

`w` 可以查看更多信息。其展示的信息都是什么意思，可以查看帮助文档 `man w`。

### 4.6 压缩解压命令

#### `gzip`

后缀一般为 .gz 的压缩文件。关于 `gzip` 的命令帮助可以查看 `gzip --help`。

压缩文件

```terminal
julian@ubuntu2204:~/temp$ ls
lorem

julian@ubuntu2204:~/temp$ gzip lorem 

julian@ubuntu2204:~/temp$ ls
lorem.gz
```

加 `-k` 保留源文件

```terminal
julian@ubuntu2204:~/temp$ ls
lorem

julian@ubuntu2204:~/temp$ gzip -k lorem 

julian@ubuntu2204:~/temp$ ls
lorem  lorem.gz
```

`-d` 解压文件（与 `gunzip` 相同），加 `-k` 也是保留源文件

```terminal
julian@ubuntu2204:~/temp$ ls
lorem.gz

julian@ubuntu2204:~/temp$ gzip -dk lorem.gz 

julian@ubuntu2204:~/temp$ ls
lorem  lorem.gz
```

`-l` 可以在解压文件的情况下查看压缩文件内的内容。

已经以 .gz 结尾的文件不能被压缩（哪怕并不是 gzip 压缩文件），不是以 .gz 结尾的文件不能被解压（哪怕实际是 gzip 压缩文件），但能用 `-l` 查看其内容。

压缩和解压貌似都不能重命名。

gzip 不能压缩目录。

只能压缩单文件，哪怕指定了多个文件名，也是分别压缩的。

#### `tar`

`tar` 是一个打包命令，可以把多个文件或目录打包成一个文件。打包的同时，也可以压缩。

`-c` 创建（create）包，`-f` 指定生成的文件名，必须要有

```terminal
julian@ubuntu2204:~/temp$ tree
.
├── elit
├── ipsum
│   └── lorem
└── sanc
    └── manga

2 directories, 3 files

julian@ubuntu2204:~/temp$ tar -cf my.tar ipsum/ sanc/ elit 

julian@ubuntu2204:~/temp$ ls
elit  ipsum  my.tar  sanc
```

`-t` 可以查看包的内容，`-v` 可以列举更详细的信息，`-f` 还是指定文件名

```terminal
julian@ubuntu2204:~/temp$ tar -tvf my.tar 
drwxrwxr-x julian/julian     0 2023-11-25 23:30 ipsum/
-rw-rw-r-- julian/julian  2457 2023-11-25 20:44 ipsum/lorem
drwxrwxr-x julian/julian     0 2023-11-25 23:34 sanc/
-rw-rw-r-- julian/julian  1712 2023-11-25 23:34 sanc/manga
-rw-rw-r-- julian/julian  3347 2023-11-25 23:35 elit
```

`-x` 解包，`-f` 还是指定文件名

```terminal
julian@ubuntu2204:~/temp/abc$ ls
my.tar

julian@ubuntu2204:~/temp/abc$ tar -xf my.tar 

julian@ubuntu2204:~/temp/abc$ tree
.
├── elit
├── ipsum
│   └── lorem
├── my.tar
└── sanc
    └── manga

2 directories, 4 files
```

解包时不能指定目标路径。

解包时可以只指定部分内容解包（下例只解了包中的一个目录）

```terminal
julian@ubuntu2204:~/temp/abc$ ls
my.tar

julian@ubuntu2204:~/temp/abc$ tar -xf my.tar ipsum/

julian@ubuntu2204:~/temp/abc$ ls
ipsum  my.tar
julian@ubuntu2204:~/temp/abc$ tree
.
├── ipsum
│   └── lorem
└── my.tar

1 directory, 2 files
```

删除包中的内容（添加的话把 `--delete` 改为 `--append`）

```terminal
julian@ubuntu2204:~/temp/abc$ tar -f my.tar --delete elit

julian@ubuntu2204:~/temp/abc$ tar -tvf my.tar 
drwxrwxr-x julian/julian     0 2023-11-25 23:30 ipsum/
-rw-rw-r-- julian/julian  2457 2023-11-25 20:44 ipsum/lorem
drwxrwxr-x julian/julian     0 2023-11-25 23:34 sanc/
-rw-rw-r-- julian/julian  1712 2023-11-25 23:34 sanc/manga
```

打包时使用 gzip 压缩

```terminal
julian@ubuntu2204:~/temp$ tar -cf my.tar.gz elit ipsum/ sanc/ --gzip

julian@ubuntu2204:~/temp$ ls
abc  elit  ipsum  my.tar.gz  sanc
```

当然使用 `tar -zcf` 也是可以的。

`tar -xf` 貌似也可以识别出被压缩过的包，并自动解压缩。

```terminal
julian@ubuntu2204:~/temp/abc$ ls
my.tar.gz

julian@ubuntu2204:~/temp/abc$ tar -xf my.tar.gz 

julian@ubuntu2204:~/temp/abc$ tree
.
├── elit
├── ipsum
│   └── lorem
├── my.tar.gz
└── sanc
    └── manga

2 directories, 4 files
```

选择其他压缩格式

```terminal
julian@ubuntu2204:~/temp/abc$ tar -cf my.tar.bz elit ipsum/ sanc/ --bzip2

julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  my.tar.bz  sanc

julian@ubuntu2204:~/temp/abc$ file my.tar.bz 
my.tar.bz: bzip2 compressed data, block size = 900k
```

```terminal
julian@ubuntu2204:~/temp/abc$ tar -cf my.tar.xz elit ipsum/ sanc/ --xz

julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  my.tar.xz  sanc

julian@ubuntu2204:~/temp/abc$ file my.tar.xz 
my.tar.xz: XZ compressed data, checksum CRC64
```

#### `zip`

压缩文件（不会删除源文件）

```terminal
julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  sanc

julian@ubuntu2204:~/temp/abc$ zip elit.zip elit 
  adding: elit (deflated 64%)

julian@ubuntu2204:~/temp/abc$ ls
elit  elit.zip  ipsum  sanc
```

加 `-r` 可以压缩目录

```terminal
julian@ubuntu2204:~/temp/abc$ ls -F
elit  ipsum/  sanc/

julian@ubuntu2204:~/temp/abc$ zip -r my.zip elit ipsum/ sanc/
  adding: elit (deflated 64%)
  adding: ipsum/ (stored 0%)
  adding: ipsum/lorem (deflated 62%)
  adding: sanc/ (stored 0%)
  adding: sanc/manga (deflated 59%)

julian@ubuntu2204:~/temp/abc$ ls
elit  ipsum  my.zip  sanc
```

#### `unzip`

`-l` 可以列举 zip 压缩包的内容，加 `-v` 可以显示更详细的信息

```terminal
julian@ubuntu2204:~/temp/abc$ unzip -l my.zip 
Archive:  my.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
     3347  2023-11-25 23:35   elit
        0  2023-11-25 23:30   ipsum/
     2457  2023-11-25 20:44   ipsum/lorem
        0  2023-11-25 23:34   sanc/
     1712  2023-11-25 23:34   sanc/manga
---------                     -------
     7516                     5 files
```

`-d` 可以指定解压目录

```terminal
julian@ubuntu2204:~/temp/abc$ tree
.
├── def
└── my.zip

1 directory, 1 file

julian@ubuntu2204:~/temp/abc$ unzip my.zip -d def/
Archive:  my.zip
  inflating: def/elit                
   creating: def/ipsum/
  inflating: def/ipsum/lorem         
   creating: def/sanc/
  inflating: def/sanc/manga          

julian@ubuntu2204:~/temp/abc$ tree
.
├── def
│   ├── elit
│   ├── ipsum
│   │   └── lorem
│   └── sanc
│       └── manga
└── my.zip

3 directories, 4 files
```

#### `bzip2`

bzip2 是 gzip 的升级版，文件后缀默认是 .bz2，更适合压缩大文件。

其操作方式及特点跟 gzip 很相似，比如

- 默认不保留源文件
- 不能压缩目录
- 只能压缩单文件
- 已经以 .bz2 结尾的文件不能再压缩（哪怕实际不是 bzip2 压缩文件）

但也有些地方不同，比如

- 没有 `-l` 列举压缩文件内容。
- 不以 .bz2 结尾但实际是 bzip2 压缩包的文件可以解压

#### 总结

总结来说，`zip` 是 Windows 和 Linux 通用的，有些相似之处，可以同时处理目录和文件。但是 `gzip` 和 `bzip2` 都只是压缩命令，不能打包，而 `tar` 负责打包，同时还能压缩。因此用 `tar` 的情况更多一些。

### 4.7 网络命令

#### `write`

`write [用户名]` 给指定的用户发消息，只要用户在线，就能收到，不在线则收不到。

信息可以是多行的，可以用 Enter 换行，删除需要用 Ctrl+Backspace，发送并退出用 Ctrl+D。

```terminal
julian@ubuntu2204:~/temp$ write karl
hello karl,    
this is julian
```

```terminal
karl@ubuntu2204:~$ 
Message from julian@ubuntu2204 on pts/0 at 00:56 ...
hello karl,
this is julian
EOF
```

对方收到消息后，回车一下就可以继续编写命令。

#### `wall`

`wall [信息]` 给所有在线用户发送消息，包括自己。即 write all

```terminal
julian@ubuntu2204:~/temp$ wall hello everyone

Broadcast message from julian@ubuntu2204 (pts/0) (Sun Nov 26 00:59:01 2023):   

hello everyone
```

```terminal
karl@ubuntu2204:~$

Broadcast message from julian@ubuntu2204 (pts/0) (Sun Nov 26 00:59:01 2023):   

hello everyone
```

不过这种信息会中断正在进行的命令操作，因此并不推荐。

#### `ping`

`-c` 可以设置次数。

#### `mail`

Ubuntu 22.04.3 中需要安装 `mailutils` 才能使用，默认没有安装。

`mail [用户名]` 开始撰写新邮件，Cc 是抄送给谁，Subject 填入主题，然后就可以写内容了。跟 `write` 一样，可以多行，Ctrl+D 结束发送。

对方输入 `mail` 可以查看收件箱里有多少邮件，进入后，`help` 可以查看支持的命令，`h` 列举所有邮件，输入邮件前的序号，可以查看邮件内容，`d 1` 删除序号为 1 的邮件。

退出时貌似只有用 `q[uit]` 才能正确保存邮件的查看和删除状态，用 `ex[it]` 不行。

#### `last`

查看所有用户以往的全部登录信息。

#### `lastlog`

查看每一个用户上一次的登录信息。（包括伪用户）

`-u` 参数加 uid 可以只查看这一个用户的上次登录信息。uid 可以通过 `id` 查看。

```terminal
karl@ubuntu2204:~$ id karl
uid=1001(karl) gid=1001(karl) groups=1001(karl)

karl@ubuntu2204:~$ lastlog -u 1001
Username         Port     From             Latest
karl             pts/1    192.168.113.15   Sun Nov 26 20:06:32 -0400 2023
```

#### `traceroute`

Ubuntu 22.04.3 需要安装 `traceroute` 才能使用，默认没有安装。

可以追踪访问到某个目标地址中间经过了多少路由节点。

> Windows 里叫 `tracert`
{: .prompt-tip}

#### `netstat`

`-t` 查看 TCP 连接，`-u` 查看 UDP 连接，`-l` 查看监听端口，`-n` 不解析为域名，即显示 IP 地址（不加就显示域名）

```terminal
julian@ubuntu2204:~$ netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
...
```

Recv-Q 和 Send-Q 为 0 表示一切正常。

`-a` 可以查看所有连接

```terminal
julian@ubuntu2204:~$ netstat -a | less
```

除了显示正在监听的端口外，也能显示已建立连接的端口（ESTABLISHED），还能显示其他程序的联网信息。内容很多，所以加了 `less`。

`-r` 显示路由表。

### 4.8 关机重启命令

`shutdown -h now` 现在立刻关机。

`shutdown -r now` 现在立刻重启。

不加 `now` 默认会等一分钟。

`shutdown` 会在关机前处理好正在运行的服务。

`init --help` 了解系统的 7 个运行级别。

`runlevel` 查看当前的运行级别。

## 第五讲 文本编辑器 Vim

### 5.1 Vim 常用操作

Vim 分为三个工作模式，分别为命令模式、插入模式、编辑模式。命令模式下每敲的一个字母都会被当作命令处理，比如敲 `i a o` 都会进入插入模式，插入模式即可以修改文本，编辑模式是在命令模式下敲 `:` 进入，可以编辑命令，比如保存退出 `:wq`。

#### 插入命令

| 命令  | 作用         |
|:--- | ---------- |
| `a` | 在光标所在字符后插入 |
| `A` | 在光标所在行行尾插入 |
| `i` | 在光标所在字符前插入 |
| `I` | 在光标所在行行首插入 |
| `o` | 在光标下插入新行   |
| `O` | 在光标上插入新行   |

#### 移动命令

| 命令          | 作用         |
|:----------- | ---------- |
| `:set nu`   | 设置行号       |
| `:set nonu` | 取消行号       |
| `gg`        | 到第一行       |
| `G`         | 到最后一行      |
| `nG`        | 到第 n 行     |
| `:n`        | 到第 n 行（推荐） |
| `$`         | 到行尾        |
| `0`         | 到行首        |

#### 删除命令

| 命令        | 作用              |
|:--------- | --------------- |
| `x`       | 删除光标所在处的字符      |
| `nx`      | 删除光标所在处后 n 个字符  |
| `dd`      | 删除光标所在行         |
| `ndd`     | 删除 n 行          |
| `dG`      | 删除光标所在行到文件末尾的内容 |
| `D`       | 删除光标所在处到行尾的内容   |
| `:n1,n2d` | 删除第 n1 行到第 n2 行 |

#### 复制粘贴命令

| 命令    | 作用          |
|:----- | ----------- |
| `yy`  | 复制当前行       |
| `nyy` | 复制当前行以下 n 行 |
| `dd`  | 剪切当前行       |
| `ndd` | 剪切当前行以下 n 行 |
| `p`   | 粘贴在当前光标所在行下 |
| `P`   | 粘贴在当前光标所在行上 |

> 提示：其实 `dd` 是剪切命令，当不需要粘贴时，它可以当删除命令用。

#### 替换和取消命令

| 命令  | 作用                                |
|:--- | --------------------------------- |
| `r` | 替换光标所在处字符（无需进入插入模式）               |
| `R` | 进入替换模式，类似于 Windows 的插入模式，`Esc` 结束 |
| `u` | 取消上一步操作，即撤销                       |

#### 字符串命令

| 命令                  | 作用                    |
|:------------------- | --------------------- |
| `/string`           | 从上往下搜索指定字符串           |
| `?string`           | 从下往上搜索指定字符串           |
| `:set ic`           | 搜索时忽略大小写              |
| `:set noic`         | 搜索时考虑大小写              |
| `n`                 | 搜索指定字符串的下一个出现位置       |
| `N`                 | 搜索指定字符串的上一个出现位置       |
| `:%s/old/new/g`     | 全文替换指定的字符串            |
| `:n1,n2s/old/new/g` | 在第 n1 行到第 n2 行替换指定字符串 |

后两个命令的最后把 `g` 换成 `c` 会在每次替换时要求确认。

#### 保存退出命令

| 命令                  | 作用                       |
|:------------------- | ------------------------ |
| `:w`                | 保存修改                     |
| `:w /new/file/path` | 另存为指定文件                  |
| `:wq`               | 保存并退出                    |
| `ZZ`                | 保存并退出的快捷键                |
| `:q!`               | 不保存退出                    |
| `:wq!`              | 强制保存退出（文件所有者和 `root` 可用） |

### 5.2 Vim 使用技巧

#### 追加内容

`:r /path/to/file` 可以把指定的文件内容追加到当前文件中。

`:!command` 可以在不退出 Vim 的情况下执行命令。

`:r !command` 可以把命令的执行结果追加到当前文件。如 `:r !date` 可以把当前时间追加到文件中。

#### 定义快捷键

例如想要添加一个注释行的快捷键，可以使用 `:map ^P I#<ESC>`，其中 `^P` 的输入方式是 `Ctrl+V, Ctrl+P`，有颜色高亮，并非一个尖角加一个大写的 P，`^P` 代表 `Ctrl+P`，后面就是依次要执行的命令。`I` 是跳到行首并进入插入模式，`#` 是插入一个井号，然后 `<ESC>` 是退出插入模式。这样就完成了行注释。此时在行的任意位置键入 `Ctrl+P` 就会在行首增加一个井号，即注释。

如果取消行注释，可以用 `:map ^B 0x`。其中 `^B` 指快捷键 `Ctrl+B`，`0x` 指跳到行首并删除一个字符。

#### 连续行注释

原理是字符串替换。

`:n1,n2s/^/#/g` 可以在第 n1 行到第 n2 行的行首添加一个 `#`，也就是连续注释了 n1 到 n2 行。这里 `^` 代表行首，也就是替换行首为某字符。

`:n1,n2s/^#//g` 可以取消 n1 到 n2 行的行首井号。这里的 `^#` 指的是只位于行首的井号，行中的不管（类似于正则），新字符串不写代表空。

`:n1,n2s/^/\/\//g` 可以给 C 文件加注释。这里斜杠是要转义的。

#### 替换

`:ab mymail 123@gmail.com` 表示在文档中敲入 `mymail` 后就会被自动替换为 `123@gmail.com`。（敲入 `mymail` 后需要空格或回车才会生效。）

#### 永久生效

以上的所有设置只存储在了内存中，重启后就没了，想要保留设置就需要修改家目录下的 `.vimrc` 文件。

```vim
set nu
map ^P I#<ESC>
map ^B 0x
ab myname Julian
```

保存文件后，每次用 Vim 打开文件都会执行这些命令。注意，文件中 `^P` 的输入方式依然为 `Ctrl+V, Ctrl+P`，并非一个尖角加一个大写的 P。

## 第七讲 用户和用户组管理

### 7.1 用户配置文件

#### 7.1.1 用户信息文件 `/etc/passwd`{: .filepath}

Linux 的所有用户可以在 `/etc/passwd`{: .filepath} 中看到。该文件的每一行都是一个用户，其格式可以用 `man 5 passwd` 查看。

`/etc/passwd`{: .filepath} 的每一行可以用 `:` 分割。

第一部分是登录名，也就是用户名。

第二部分是有无密码的标识。如果是 `x` 表示有密码，该密码加密存放在 `/etc/shadow`{: .filepath} 中，这个文件只有管理员可以查看。

第三部分是用户 ID（UID），也是用户的唯一标识。即系统实际上是通过 UID 来区分用户的，而用户名是给人看的。用户名和 UID 的关系类似于域名和 IP 的关系。UID 为 `0` 表示管理员用户，之后有一定数量的系统用户，然后才是普通用户。比如 Ubuntu 22.04.3 的第一个普通用户的 UID 是 `1000`。

> 普通用户的 UID 从何开始又从何结束在 `/etc/login.defs`{: .filepath} 文件中由 `UID_MIN` 和 `UID_MAX` 字段设置。
{: .prompt-tip}

第四部分是初始用户组 ID（GID）。用户组分为初始组和附加组。一个用户一创建就拥有的组就是初始组，一般跟用户名相同。一个用户只能有一个且必须有一个初始组，可以改但不建议改。一个用户可以有很多附加组。所以如果一个用户想进入其它组，添加附加组即可，不要修改初始组。

第五部分是用户说明，也就是一个简短的字符串来解释当前用户是谁，便于人识别。

第六部分是用户家目录。

第七部分是用户的 Shell 解释器。也就是这个用户用来解释命令的解释器。如果改错，则该用户无法解释命令，也就无法开机了。

#### 7.1.2 影子文件 `/etc/shadow`{: .filepath}

影子文件，权限极低，只有管理员可以查看。每一行是一个用户，同样用 `:` 分割。

第一部分是用户名。

第二部分真正的用户密码。采用加密算法加密。如果是 `!` 或 `*` 则表示没有密码，不能登录。

第三部分是密码的最后一次修改日期。这个数字表示从 1970 年 1 月 1 日起到密码修改日所经过的天数。

第四部分是指两次密码的允许修改间隔，单位是天。默认是 `0`，也就是没有间隔。如果改为 `10` 表示密码修改完之后 10 天内不允许修改密码。

第五部分是密码有效期，单位是天。

第六部分是密码到有效期之前多少天要进行警告。假设第五部分设置的是 `90`，该部分设置 `7`，则到密码修改后的第 83 天开始，每次登录系统都会建议修改密码。

第七部分是密码到期之后的宽限天数。假设为 `5`，表示密码到期后五天内还是允许登录，如果还不修改，就禁止了。如果该部分为空或为 `0`，表示没有宽限期，密码到期不修改就禁止登录。如果为 `-1` 表示永远不会失效。跟有效期就无关了。

第八部分是账号的失效时间。用从 1970 年 1 月 1 日起所经过的天数表示。如果指定了该时间，则只要到了这个时间，不管账号在不在密码有效期内，都会被禁止。

第九部分保留。

#### 7.1.3 组信息文件 `/etc/group`{: .filepath} 和组密码文件 `/etc/gshadow`{: .filepath}

##### `/etc/group`{: .filepath}

用户组文件，每一行是一个用户组，同样用 `:` 分割。

第一部分是用户组名。

第二部分是用户组密码标志。真正的密码存放在 `/etc/gshadow`{: .filepath} 文件中。本来只有管理员可以把用户添加到用户组中，但如果一个用户组有密码，且有个普通用户知道这个组的密码，那么他就可以把用户添加到这个用户组中。但不建议这样，也不建议给用户组密码。

第三部分是用户组 ID（GID）。

第四部分是该组的附加用户（不包含初始用户）。


##### `/etc/gshadow`{: .filepath}

组密码文件，同 `/etc/shadow`{: .filepath}，只不过管理的是用户组的密码。每一行是一个用户组，用 `:` 分割。

第一部分是用户组名。

第二部分是用户组密码。

第三部分是组管理员的用户名。

第四部分是该组的附加用户。

### 7.2 用户管理相关文件

#### `/var/mail/`{: .filepath}

用户的邮箱目录。也可以在 `/var/spool/mail/`{: .filepath}。收到邮件后会在该目录下创建一个跟用户名相同的文件夹，存储邮件。

> Ubuntu 22.04.3 需要安装 `mailutils` 后才能使用。
{: .prompt-info}

#### `/etc/skel/`{: .filepath}

模板目录。用户在创建时会在其家目录下默认创建一些文件，这些文件就是从 `/etc/skel/`{: .filepath} 目录下拷贝过去的。如果想给新创建的用户一些提示信息，就可以在这个模板目录下新增一个提示文件，这样用户在创建后就会在其家目录下收到这个文件。

默认只有一些隐藏文件，使用 `ls -a /etc/skel/` 查看

### 7.3 用户管理命令

#### 7.3.1 用户添加命令 `useradd`

如之前所说，这个命令偏底层，常规创建用户用 `adduser`。查看帮助 `adduser --help`，比较详细且直观。

这里记录一下跟创建用户有关的一些配置文件：

`/etc/default/useradd`{: .filepath}

`/etc/login.defs`{: .filepath}

#### 7.3.2 修改用户密码 `passwd`

`passwd --help` 可以查看帮助。简单使用就是 `passwd [user]` 直接修改密码。

普通用户修改自己的密码，需要先验证当前密码，root 用户则不需要。

#### 7.3.3 修改用户信息 `usermod` 和修改用户密码状态 `chage`

`usermod` 用来修改一个已经存在的用户的相关信息。查看 `usermod --help` 帮助。

`chage` 查看用户密码的状态

```shell
chage -l [username]  # 列举更直观的密码信息
chage -d 0 [username]  # 该命令的主要作用，修改用户密码的上次修改时间为 0，
# 导致的结果就是该用户一旦登录就被要求修改密码，
# 可以用在当创建了大量默认用户后要求用户修改默认密码
```

#### 7.3.4 删除用户 `userdel` 和用户切换命令 `su`

`userdel` 删除用户

```shell
userdel -r [username]  # -r 选项可以删除用户的家目录和邮箱目录
```

`su` 切换用户。可以使用 `env` 查看当前的环境变量

对于一个属于 `sudo` 组的用户 julian，使用 `sudo su` 再加上 julian 的密码，可以暂时切换到 root 用户，但此时查看 `env` 会发现一些不同（以下省略了一些非关键内容）

```plaintext
SHELL=/bin/bash
SUDO_GID=1000
SUDO_COMMAND=/usr/bin/su
SUDO_USER=julian
LOGNAME=root
HOME=/root
USER=root
SUDO_UID=1000
MAIL=/var/mail/root
```

虽然 `USER` 是 root，但它知道 `SUDO_USER` 是 julian。如果是直接登录 root 并查看 `env`

```plaintext
SHELL=/bin/bash
LOGNAME=root
HOME=/root
USER=root
MAIL=/var/mail/root
```

可以看到没有 `SUDO_` 开头的变量。

如果在 julian 用户下直接切换到 root 用户，可以用 `su root` 并输入 root 的密码，查看 `env`

```plaintext
SHELL=/bin/bash
LOGNAME=julian
HOME=/root
USER=julian
MAIL=/var/mail/root
```

可以看到没有 `SUDO_` 但是 `USER` 还是 julian。也就是说部分环境变量没有切换过来。此时用 `su - root` 再试一遍

```plaintext
SHELL=/bin/bash
LOGNAME=root
HOME=/root
USER=root
MAIL=/var/mail/root
```

就会发现跟直接登录 root 是一样的了。

上述不加 `-` 不改变部分环境变量貌似只针对 root 用户，如果切换到其他普通用户，还是会跟着变的，比如 `su karl` 然后 `env`

```plaintext
SHELL=/bin/bash
LOGNAME=karl
HOME=/home/karl
USER=karl
MAIL=/var/mail/karl
```

### 7.4 用户组管理命令

#### 添加用户组 `groupadd`

创建新的用户组，`groupadd [new_group_name]`，`-g` 可以指定 GID。

#### 修改用户组 `groupmod`

修改已存在的用户组，`groupmod [old_group_name] -n [new_group_name]`，`-g` 可以修改 GID。

#### 删除用户组 `groupdel`

删除用户组 `groupdel [group_name]`，没啥参数。

如果这个组中有初始用户，则该组不能删除，如果没有，可以正常删除。因为初始用户和初始组是相互依存的。

貌似加 `-f` 就可以强制删除有初始用户的用户组。

#### 管理用户组 `gpasswd`

将用户添加到组或从组中删除。注意查看帮助，明确命令的选项格式。

`gpasswd -a [user] [group]` 添加用户到组（作为附加组）

`gpasswd -d [user] [group]` 从组中删除用户

## 第八讲 权限管理

### 8.1 ACL 权限

#### 8.1.1 ACL 权限简介与开启

一个文件或者目录对权限的管理只有 3 类：所有者，所属组，其他人。这里所属组只能有 1 个。但是如果有些情况下，有 2 个不同的组都需要对这个文件有一些权限，但是它们的权限不能一样，而且它们跟其他人的权限也不能一样，那现有的三个分类就不够用了。

比如说，有一个目录 `/projects`，其所有者是 `teacher`，所属组是 `students`，其中有 `student1` 和 `student2`，我们规定该目录的权限是 `rwxrwx---`，即老师和学生都有对这个目录的全部权限，但是其他人没有任何权限。此时有一个用户 `visti_stu`，来旁听的，不能修改这个目录，但是可以查看，因此它的权限应该是 `r-x`，但是这个权限跟上面的三类权限都不吻合。因此就需要一些额外的权限设置。

Access Control Lists（ACL）可以解决这个问题。

使用 ACL 前要查看分区是否支持并开启了 ACL（现在默认都是支持并开启的）。

`df -h` 可以查看分区使用情况（以更友好的方式）（下例省略了无关内容）

```terminal
julian@ubuntu2204:/tmp/root$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        50G   12G   35G  26% /
/dev/sda2       512M  6.1M  506M   2% /boot/efi
...
```

`dumpe2fs -h /dev/sda3` 可以查看分区的信息（需要 root 权限）

```terminal
root@ubuntu2204:/tmp/root# dumpe2fs -h /dev/sda3 | grep "Default mount options"
dumpe2fs 1.46.5 (30-Dec-2021)
Default mount options:    user_xattr acl
```

如果 `Default mount options` 中有 `acl` 字样，表示该分区支持 ACL。

现在 ACL 默认都是开启的，不过如果要手动开启：

临时开启分区的 ACL 权限：`mount -o remount,acl /`，即重新挂载后开启 ACL 功能。

写入文件永久生效：`/etc/fstab`{: .filepath} 里记录了开机时各分区该如何挂载。将 `acl` 参数写入即可保证每次开机挂载时都有开启 ACL。

不过如之前所说，现在都是默认开启了，因此没必要修改这个文件，而且这个文件 **极其** 重要，如果修改出错，系统在开机时将不能正确挂载分区，导致无法开机。

另见 [`/etc/fstab`{: .filepath}](#932-分区自动挂载与-fstab-文件修复)。

#### 8.1.2 查看与设定 ACL 权限

`setfacl` 用来设定 ACL 权限，使用 `--help` 查看帮助。

`-m` 用来修改文件或目录的 ACL 权限，格式为

`u:[user|uid]:[r|w|x|-]` 修改用户对文件或目录的权限

`g:[group|gid]:[r|w|x|-]` 修改用户组对文件或目录的权限

`o:[r|w|x|-]` 修改其他人对文件或目录的权限

```terminal
root@ubuntu2204:/tmp/root# ls -l
total 8
-rw-------  1 root root 2210 Nov 27 11:36 env.txt
-rw-r--r--  1 root root  665 Nov 27 13:00 fstab

root@ubuntu2204:/tmp/root# setfacl -m u:liz:r-- env.txt 

root@ubuntu2204:/tmp/root# ls -l
total 8
-rw-r-----+ 1 root root 2210 Nov 27 11:36 env.txt
-rw-r--r--  1 root root  665 Nov 27 13:00 fstab
```

```terminal
liz@ubuntu2204:/tmp/root$ cat env.txt 
SHELL=/bin/bash
XDG_SEAT=seat0
PWD=/root
...
```

```terminal
karl@ubuntu2204:/tmp/root$ cat env.txt 
cat: env.txt: Permission denied
```

`-x` 用来去除一个文件或目录的 ACL 权限，格式即 `-m` 格式去掉最后的权限位设置。

比如 `setfacl -x u:liz env.txt` 去除 `liz` 用户对 `env.txt` 的所有 ACL 权限。

`getfacl` 可以查看文件或目录的 ACL 权限。

#### 8.1.3 最大有效权限与删除 ACL 权限

用 `getfacl` 查看文件的 ACL 权限，可以看到如下内容

```terminal
root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
group::---
mask::---
other::---
```

> 以下是视频说的内容
{: .prompt-warning}

这里的 `mask` 就是最大有效权限。即，用命令设置权限之后，还需要与 `mask` 相与才是最终的权限。

比如 `mask::r-x`，那么即便我们设置 `-m u:liz:rwx`，用户最终得到的权限也只能是 `r-x`。

可以通过形如 `setfacl -m m::r-x file` 的方式设置 `mask`

```terminal
root@ubuntu2204:/tmp/root# setfacl -m m::r-x env.txt 

root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
group::---
mask::r-x
other::---
```

> 以下是实际测试结果
{: .prompt-warning}

先设置 `mask`，再设置某个用户的权限后，`mask` 会被覆盖。

```terminal
root@ubuntu2204:/tmp/root# setfacl -m m::r-x env.txt 

root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
group::---
mask::r-x
other::---

root@ubuntu2204:/tmp/root# setfacl -m u:liz:rwx env.txt 

root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
user:liz:rwx
group::---
mask::rwx
other::---
```

这里设置了用户的权限之后，`mask` 也变了。此时再设置一遍 `mask`

```terminal
root@ubuntu2204:/tmp/root# setfacl -m m::r-x env.txt 

root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
user:liz:rwx			#effective:r-x
group::---
mask::r-x
other::---
```

好像 `mask` 才生效。所以这里看 `mask` 好像是后作用的。此时再设置一个用户的

```terminal
root@ubuntu2204:/tmp/root# setfacl -m u:karl:rwx env.txt 

root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
user:karl:rwx
user:liz:rwx
group::---
mask::rwx
other::---
```

`mask` 又变了。而且之前对 liz 的限制也不起作用了。

所以测试来看，`mask` 并不是在设置新权限前必须与它相与，而是对已有的权限做一层遮罩，滤掉某些权限。

```terminal
root@ubuntu2204:/tmp/root# setfacl -m m::r-x env.txt 

root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
user:karl:rwx			#effective:r-x
user:liz:rwx			#effective:r-x
group::---
mask::r-x
other::---
```

`-x` 删除特定用户或组的权限，`-b` 可以删除目录下所有文件的 ACL 权限。

```terminal
root@ubuntu2204:/tmp/root# ls -l env.txt 
-rw-r-x---+ 1 root root 110 Nov 27 17:58 env.txt

root@ubuntu2204:/tmp/root# setfacl -b env.txt 

root@ubuntu2204:/tmp/root# getfacl env.txt 
# file: env.txt
# owner: root
# group: root
user::rw-
group::---
other::---

root@ubuntu2204:/tmp/root# ls -l env.txt 
-rw------- 1 root root 110 Nov 27 17:58 env.txt
```

#### 8.1.4 默认 ACL 权限和递归 ACL 权限

`-R` 后加目录可以把 ACL 的设置递归应用到目录下的所有文件和目录。这个参数只能在后，对顺序有点要求

```terminal
root@ubuntu2204:/tmp/root# setfacl -m u:liz:rwx project/

root@ubuntu2204:/tmp/root# getfacl project/abc.txt 
# file: project/abc.txt
# owner: root
# group: root
user::rw-
group::---
other::---

root@ubuntu2204:/tmp/root# setfacl -m u:liz:rwx -R project/

root@ubuntu2204:/tmp/root# getfacl project/abc.txt 
# file: project/abc.txt
# owner: root
# group: root
user::rw-
user:liz:rwx
group::---
mask::rwx
other::---
```

但是在运行命令后新添加的文件或者目录，不受这个限制。也就是说，这个 `-R` 的递归设置，只是一次性的快速设置，而不是一种规则，能时时保证目录下所有文件都遵守这个设置。

要想让其成为一种规则，需要设置默认 ACL 权限。如果给一个目录设置了默认 ACL 权限，那么其下创建的新文件和目录都会应用这个权限。

设置的方法就是在常规的权限规则设置之前加一个 `d:`，而且不用加 `-R`

```terminal
root@ubuntu2204:/tmp/root# setfacl -m d:u:liz:rwx project/

root@ubuntu2204:/tmp/root# getfacl project/
# file: project/
# owner: root
# group: root
user::rwx
user:liz:rwx
group::---
mask::rwx
other::---
default:user::rwx
default:user:liz:rwx
default:group::---
default:mask::rwx
default:other::---

root@ubuntu2204:/tmp/root# touch project/ghi.txt

root@ubuntu2204:/tmp/root# getfacl project/ghi.txt 
# file: project/ghi.txt
# owner: root
# group: root
user::rw-
user:liz:rwx			#effective:rw-
group::---
mask::rw-
other::---
```

> 注意，这里文件依然默认没有执行权限。
{: .prompt-tip}

这个命令作用于后来新添加的文件，所以如果有已存在的文件或目录，设置了父目录的默认权限，并不会让它们的权限发生改变。

### 8.2 文件特殊权限

#### 8.2.1 SetUID

SUID：让执行者临时拥有被执行文件 **属主** 的权限（仅对 **二进制文件** 有效，且执行者必须要有对该文件的执行权限）

比如所有用户都可以执行用于修改用户密码的 `passwd` 命令，但用户密码保存在 `/etc/shadow`{: .filepath} 文件中，默认除了超级用户外的所有用户都没有查看或编辑该文件的权限，所以对 `passwd` 命令加上 SUID 权限位，可让普通用户临时获得程序所有者的身份，即以 root 用户的身份将变更的密码信息写入到 `shadow`{: .filepath} 文件中。

`ll /usr/bin/passwd` 查看该命令的权限，看到所有者的权限为 `rws`，即拥有 SUID 权限。如果此文件没有执行权限，则为 `rwS`，表示设置的 SUID 权限无效。因为 SUID 只针对有执行权限的文件有效。  

SUID 权限 **非常危险**，不要轻易给任何命令加 SUID 权限。

##### 设置 SUID

`chmod u+s [file]` 或者形如 `chmod 4755 [file]`

```terminal
julian@ubuntu2204:/tmp/julian/cplus$ ls -l
total 20
-rwxr--r-- 1 julian julian 15960 Nov 24 13:31 hello
-rw-rw-r-- 1 julian julian    78 Nov 24 13:30 hello.c

julian@ubuntu2204:/tmp/julian/cplus$ chmod u+s hello

julian@ubuntu2204:/tmp/julian/cplus$ ls -l
total 20
-rwsr--r-- 1 julian julian 15960 Nov 24 13:31 hello
-rw-rw-r-- 1 julian julian    78 Nov 24 13:30 hello.c
```

但是这样其他用户依然无法执行 `hello`{: .filepath} 文件。因为 SUID 要求首先用户要对这个文件有执行权限，才会考虑这个用户在执行文件时能不能变为所有者身份。此时对于 `hello`{: .filepath} 来说，所属组和其他人都没有执行权限，因此也就不存在变不变身的说法了。

要做这个实验，可能只能模拟 `passwd` 了。

下例中 `modfile` 文件会向 `testfile.txt`{: .filepath} 中添加一行文本，但后者除了所有者其他人都没有权限修改

```terminal
julian@ubuntu2204:/tmp/julian/prog$ ls -l
total ..
-rwxr-xr-x 1 julian julian 30520 Nov 27 20:41 modfile
-rw-r--r-- 1 julian julian     0 Nov 27 20:40 testfile.txt

julian@ubuntu2204:/tmp/julian/prog$ ./modfile 

julian@ubuntu2204:/tmp/julian/prog$ cat testfile.txt 
hello there!
```

```terminal
liz@ubuntu2204:/tmp/julian/prog$ ls -l
total ..
-rwxr-xr-x 1 julian julian 30520 Nov 27 20:41 modfile
-rw-r--r-- 1 julian julian    13 Nov 27 20:42 testfile.txt

liz@ubuntu2204:/tmp/julian/prog$ ./modfile 
Fail to modify the file
```

此时添加 SUID 权限，其他人就可以修改文件了

```terminal
julian@ubuntu2204:/tmp/julian/prog$ chmod u+s modfile

julian@ubuntu2204:/tmp/julian/prog$ ls -l
total ..
-rwsr-xr-x 1 julian julian 30520 Nov 27 20:41 modfile
-rw-r--r-- 1 julian julian    13 Nov 27 20:42 testfile.txt
```

```terminal
liz@ubuntu2204:/tmp/julian/prog$ ls -l
total ..
-rwsr-xr-x 1 julian julian 30520 Nov 27 20:41 modfile
-rw-r--r-- 1 julian julian    13 Nov 27 20:42 testfile.txt

liz@ubuntu2204:/tmp/julian/prog$ ./modfile 

liz@ubuntu2204:/tmp/julian/prog$ cat testfile.txt 
hello there!
hello there!
```

当没有加 SUID 时，其他人对 `modfile` 有执行权限，但是对 `testfile.txt`{: .filepath} 没有写权限，因此执行时会报错，无法正确写入文本。当加入 SUID 之后，执行 `modfile` 时暂时变为所有者身份，因此就对 `testfile.txt`{: .filepath} 有写权限了。

如果其他人一开始压根对 `modfile` 没有执行权限，那么就没后面的 SUID 什么事了

```terminal
liz@ubuntu2204:/tmp/julian/prog$ ls -l
total ..
-rwsr-xr-- 1 julian julian 30520 Nov 27 20:41 modfile
-rw-r--r-- 1 julian julian    26 Nov 27 20:44 testfile.txt

liz@ubuntu2204:/tmp/julian/prog$ ./modfile
-bash: ./modfile: Permission denied
```

##### 取消 SUID

`chmod u-s [file]` 或者形如 `chmod 755 [file]`。

#### 8.2.2 SetGID

**SGID 功能一**：让执行者临时拥有被执行文件 **属组** 的权限（针对二进制文件，且执行者对该文件有执行权限）

举例来说，常用的查找命令 `locate`，其实体为 `/usr/bin/plocate`{: .filepath}，该命令所读取的数据库是 `/var/lib/plocate/plocate.db`{: .filepath}，但该数据库的属主是 `root`，属组是 `plocate`，权限为 `rw-r-----`。通常没有普通用户属于这个属组，也就是说，普通用户没有读取这个数据库的权限，但是用户又需要用 `locate` 查找文件，所以就给 `locate` 命令（实际是 `/usr/bin/plocate`{: .filepath}）加了 SGID 权限位。`locate` 的属组是 `plocate`，这样，当用户执行 `locate` 命令时，用户就临时属于 `plocate` 这个属组了，也就可以读取 `plocate.db`{: .filepath} 这个数据库了。


`ll /usr/bin/plocate` 查看该命令权限。看到所属组的权限为 `r-s`，表示拥有 SGID 权限。该权限 **同样危险**，也不要轻易给命令加 SGID 权限。

**SGID 功能二**：在该目录创建的文件自动继承此目录的属组（只对目录设置）

通常创建的文件的属组是创建者的基本组，但如果该目录被设置了 SGID 权限，那么该目录下的所有文件都会属于该目录的属组。

> 这个功能很方便管理目录和文件，可以用。
{: .prompt-info}

##### 设置 SGID

`chmod g+s [file]` 或者形如 `chmod 2755 [file]`

**针对目录**：

```terminal
julian@ubuntu2204:/tmp/julian$ mkdir proj

julian@ubuntu2204:/tmp/julian$ chmod g+s,o+w proj/

julian@ubuntu2204:/tmp/julian$ ls -dl proj/
drwxrwsrwx 2 julian julian 4096 Nov 28 12:30 proj/
```

```terminal
liz@ubuntu2204:/tmp/julian$ ls -dl proj/
drwxrwsrwx 2 julian julian 4096 Nov 28 12:30 proj/

liz@ubuntu2204:/tmp/julian$ touch proj/liz.txt

liz@ubuntu2204:/tmp/julian$ ls -l proj/liz.txt 
-rw-rw-r-- 1 liz julian 0 Nov 28 12:31 proj/liz.txt
```

这种情况下，liz 对 `proj`{: .filepath} 只有其他人的权限，因此首先其他人要对该目录有写权限，才能创建文件，其次如果该目录设置了 SGID，那么创建的文件属组是 julian 而不是 liz。

这样可以保证 julian 对 liz 创建的文件内容有一定操作权限（使用属组的权限），否则 julian 可能无法编辑其内容（其他人没有写权限）。

另一种情况是，用户是目录属组的一员，目录数组首先要对目录有写权限，如果目录设置了 SGID，该用户在目录下创建的文件依然是目录属组

```terminal
julian@ubuntu2204:/tmp/julian$ cat /etc/group | grep friends
friends:x:1004:julian,liz

julian@ubuntu2204:/tmp/julian$ ls -dl proj/
drwxrwsr-x 2 root friends 4096 Nov 28 13:06 proj/

julian@ubuntu2204:/tmp/julian$ touch proj/ju.txt

julian@ubuntu2204:/tmp/julian$ ls -l proj/ju.txt 
-rw-rw-r-- 1 julian friends 0 Nov 28 13:07 proj/ju.txt
```

这样属组的其他人也可以编辑这个文件

```terminal
liz@ubuntu2204:/tmp/julian$ ls -l proj/ju.txt 
-rw-rw-r-- 1 julian friends 0 Nov 28 13:16 proj/ju.txt

liz@ubuntu2204:/tmp/julian$ echo "hello julian" >> proj/ju.txt 

liz@ubuntu2204:/tmp/julian$ cat proj/ju.txt 
hello julian
```

所以这里给目录设置 SGID 是允许目录的属组对目录的里的文件也能执行属组权限，否则的话就只能执行其他人权限了。

> 添加新的用户组后可能需要注销重新登录才能使用该组的权限。
{: .prompt-tip}

**针对文件**：

跟 SUID 差不多，就不做测试了。

##### 取消 SGID

`chmod g-s [file]` 或者形如 `chmod 755 [file]`。

#### 8.2.3 Sticky BIT

SBIT：只可管理自己的文件而不能删除他人的文件（仅对目录有效）

`/tmp`{: .filepath} 的权限是 `777`，任何用户在该目录下都有全权限，但如果一个用户创建的文件被另一个用户删了怎么办？这就太混乱了，所以查看 `/tmp`{: .filepath} 的权限时我们实际看到它的其他人权限是 `rwt`，指该目录拥有 SBIT 权限位，用户在该目录下只能删除自己创建的文件，而不能删除其他人创建的文件。

> 该权限也可以用。
{: .prompt-info}

##### 设置 SBIT

`chmod o+t [dir]` 或者形如 `chmod 1777 [dir]`

上面说的一个用户不能删除另一个用户创建的文件，除了排除 root 用户外，**目录的所有者也不受此限制**。

比如 julian 创建一个目录，并赋予 SBIT 权限

```terminal
julian@ubuntu2204:/tmp/julian$ mkdir juhouse

julian@ubuntu2204:/tmp/julian$ chmod 1777 juhouse/

julian@ubuntu2204:/tmp/julian$ ls -dl juhouse/
drwxrwxrwt 2 julian julian 4096 Nov 28 13:50 juhouse/
```

但是把所属组改为 liz

```terminal
root@ubuntu2204:/tmp/julian# chgrp liz juhouse/

root@ubuntu2204:/tmp/julian# ls -dl juhouse/
drwxrwxrwt 2 julian liz 4096 Nov 28 13:50 juhouse/
```

这时 karl 来创建文件

```terminal
karl@ubuntu2204:/tmp/julian/juhouse$ touch karl1 karl2

karl@ubuntu2204:/tmp/julian/juhouse$ ls -l
total 0
-rw-rw-r-- 1 karl karl 0 Nov 28 13:52 karl1
-rw-rw-r-- 1 karl karl 0 Nov 28 13:52 karl2
```

liz 不能删除，但 julian 还是能删的。

```terminal
liz@ubuntu2204:/tmp/julian$ rm juhouse/karl1
rm: remove write-protected regular empty file 'juhouse/karl1'? y
rm: cannot remove 'juhouse/karl1': Operation not permitted
```

```terminal
julian@ubuntu2204:/tmp/julian$ rm juhouse/karl2
rm: remove write-protected regular empty file 'juhouse/karl2'? y

julian@ubuntu2204:/tmp/julian$ ls -l juhouse/
total 0
-rw-rw-r-- 1 karl karl 0 Nov 28 13:52 karl1
```

##### 取消 SBIT

`chmod o-t [dir]` 或者形如 `chmod 777 [dir]`。

### 8.3 文件系统属性权限

#### `chattr`

`chattr` 用于设置文件的隐藏权限。例如 `chattr +i file` 增加 `i` 权限，`chattr -i file` 取消 `i` 权限。

> 需要管理员权限。
{: .prompt-tip}

| 参数  | 描述 | 作用 |
| --- | -------------------- | ------------------------------------- |
| `i` | Immutable            | 对文件，无法删除或重命名文件，无法修改文件内容；<br />对目录，不能创建或删除或重命名文件，但可以修改文件内容 |
| `a` | Append_Only          | 对文件，仅允许追加内容，不能覆盖或删除已有内容；<br />对目录，可以创建新文件，但不能重命名或删除已有文件，对文件内容不影响 |
| `S` | Synchronous_Updates  | 文件内容变更后立即同步到硬盘 |
| `s` | Secure_Deletion      | 删除时用 `0` 填充原文件所在硬盘区域，不可恢复 |
| `A` | No_Atime             | 不再修改该文件的最后访问时间 |
| `d` | No_Dump              | 使用 `dump` 备份时忽略该文件/目录 |
| `c` | Compresson_Requested | 存储时默认将文件/目录进行压缩，读取时会自动解压缩 |
| `u` | Undelete             | 删除此文件后保留其在硬盘的数据，方便日后恢复 |

##### `i` 权限

对文件：

```terminal
root@ubuntu2204:/tmp/root# chattr +i abc

root@ubuntu2204:/tmp/root# cat abc
hello world

root@ubuntu2204:/tmp/root# echo "how are you" >> abc
bash: abc: Operation not permitted

root@ubuntu2204:/tmp/root# mv abc def
mv: cannot move 'abc' to 'def': Operation not permitted

root@ubuntu2204:/tmp/root# rm -f abc
rm: cannot remove 'abc': Operation not permitted
```

对目录：

```terminal
root@ubuntu2204:/tmp/root# tree
.
├── abc
└── def
    └── ghi

1 directory, 2 files

root@ubuntu2204:/tmp/root# chattr +i def/

root@ubuntu2204:/tmp/root# touch def/jkl
touch: setting times of 'def/jkl': No such file or directory

root@ubuntu2204:/tmp/root# rm -f def/ghi
rm: cannot remove 'def/ghi': Operation not permitted

root@ubuntu2204:/tmp/root# mv def/ghi def/jkl
mv: cannot move 'def/ghi' to 'def/jkl': Operation not permitted

root@ubuntu2204:/tmp/root# echo "how are you" >> def/ghi 
root@ubuntu2204:/tmp/root# cat def/ghi 
hello world
how are you
```

对目录加 `i` 权限，不影响修改目录里文件的内容，其实目录也可以看作是一个名单文件，保存里目录下的文件名称。那么父目录有 `i` 权限，不影响在子目录里创建文件，因为这种创建相当于修改目录（名单文件）的内容，是可以的。

```terminal
root@ubuntu2204:/tmp/root# chattr +i def/

root@ubuntu2204:/tmp/root# cd def/

root@ubuntu2204:/tmp/root/def# mkdir pqs
mkdir: cannot create directory ‘pqs’: Operation not permitted

root@ubuntu2204:/tmp/root/def# mkdir jkl/pqs

root@ubuntu2204:/tmp/root/def# tree
.
├── ghi
└── jkl
    ├── mno
    └── pqs

2 directories, 2 files

root@ubuntu2204:/tmp/root/def# mv jkl/ lkj
mv: cannot move 'jkl/' to 'lkj': Operation not permitted

root@ubuntu2204:/tmp/root/def# rm -rf jkl/
rm: cannot remove 'jkl/': Operation not permitted

root@ubuntu2204:/tmp/root/def# echo "hello" >> jkl/mno

root@ubuntu2204:/tmp/root/def# cat jkl/mno 
hello
```

##### `a` 权限

对文件：

使用像 vim 这样的编辑器无法对有 `a` 权限的文件进行任何修改，可能是它无法判定你是否覆盖了已有内容吧，用 `>>` 这种方式可以追加。

对目录：

当使用 vim 这样的编辑器打开文件时，它会自动创建一些交换文件，有些用完可能就删除了，但是如果这个目录有 `a` 权限，虽然可以正常创建交换文件，但是用完了却删除不了，因为 `a` 只允许目录创建新文件，禁止删除已有文件，因此 vim 检测到交换文件还保留着，会认为有其他用户正在编辑这个文件，就会报出警告。

### 8.4 系统命令权限

为什么安装系统时设置的第一个用户，不是 root 但是运行命令前加上 `sudo` 就可以有 root 的权限了呢？

通过查看 `id` 之后发现，该用户属于一个叫 `sudo` 的用户组。而且只要把其他用户加入这个组，那这个用户也可以用 `sudo` 获得 root 权限。

为什么加入这个组就可以使用 `sudo` 了呢？Linux 有一个文件来维护什么用户应该有什么权限，这个文件是 `/etc/sudoers`{: .filepath}，用 `visudo` 可以修改这个文件，或者设置新的文件来添加规则。

查看 `/etc/sudoers`{: .filepath} 文件会发现如下几行

```plaintext
root	ALL=(ALL:ALL) ALL

%admin ALL=(ALL) ALL

%sudo	ALL=(ALL:ALL) ALL

@includedir /etc/sudoers.d
```

`%sudo` 这一行即表示对这个组的所有用户拥有以 root 身份执行所有命令的权限。

最后一行表示新的规则也可以添加到 `/etc/sudoers.d`{: .filepath} 这个目录下，因此使用 `visudo /etc/sudoers.d/sudoer_liz` 命令开启一个新的文件，然后写入 `liz ALL=/usr/bin/cat`，这表示 liz 这个用户可以在任何 IP 下以 root 身份执行 `/usr/bin/cat`{: .filepath} 这个文件。如果要修改为特定 IP，要注意这里的 IP 指的是 Linux 本机的 IP，不是远程连接到 Linux 的 IP。

此时用 `sudo -l` 查看一下 liz 用户有哪些权限可用，会看到如下内容

```terminal
liz@ubuntu2204:/tmp/root$ sudo -l
[sudo] password for liz: 
Matching Defaults entries for liz on ubuntu2204:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User liz may run the following commands on ubuntu2204:
    (root) /usr/bin/cat
```

最后就是刚添加的内容。这表示当 liz 用户使用 `sudo cat [file]` 的时候，可以查看本来只有 root 才能查看的文件。但是此时 liz 只有在执行 `sudo cat` 的时候才能有 root 权限，执行其他命令还是普通的用户。

要移除这个权限，把 `/etc/sudoers.d/sudoer_liz`{: .filepath} 这个文件删掉就好了（因为里面只有这一行内容），删掉后再调用 `sudo cat abc`，就只有

```terminal
liz@ubuntu2204:/tmp/root$ sudo cat abc
liz is not in the sudoers file.  This incident will be reported.
```

此时 root 会受到一封 mail，说明此事。

## 第九讲 文件系统管理

### 9.2 文件系统常用命令

#### 9.2.1 各种分区命令

##### `df`

前面说过了，可以查看分区状态，包括占用率、挂载点等，`-h` 可以以更易读的方式显示分区大小。

##### `du`

可以查看文件和目录的占用磁盘空间的大小。但是因为看文件用 `ls` 也能看，因此 `du` 主要用来看目录的。

常用的 `-s` 只查看目录的总大小，`-h` 让大小显示更易读。

```terminal
root@ubuntu2204:/tmp/root# du -sh /etc
12M	/etc
```

> `du` 会扫描目录下的所有文件计算大小，因此如果目录比较大，将会是一个高负载的操作。
{: .prompt-warning}

##### `df` 和 `du` 的区别

`df` 统计占用率时是按从系统角度考虑的，除了文件本身占用的空间外，程序运行所占用的空间它也统计在内了，因此这个占有率是比较真实耳朵。

`du` 统计时只考虑文件本身的大小，不考虑程序运行时的占用，因此如果计算整个根分区的大小，其结果可能会比 `df` 显示的占用大小略小。

##### `fsck`

`fsck [选项] [分区设备文件名]` 可以自动修复文件系统，知道就行，一般用不着。

##### `dumpe2fs`

形如 `dumpe2fs /dev/sda1` 可以查看分区的详细信息。太详细了，细节到每一个块（block），太多了，建议导出为文件再看。不说了。

加 `-h` 选项可以只显示超级块（superblock）的信息，一般会用到的信息都在里面。比如 UUID 等。

#### 9.2.2 挂载命令

`mount` 直接执行可以查看当前都有哪些挂载点。

形如 `mount /dev/sda4 /home` 可以把一个设备挂载到一个目录下。

`-o` 指定特殊选项，比如 `remount`，`exec`，`noexec` 等。

`mount -o remount,noexec /home` 表示重新挂载 `/home`{: .filepath} 并且禁止该分区的执行功能。也就是说，该分区的任何执行文件都不能被执行，哪怕 root 也不行，因为整个分区都禁用了执行功能。

有时有的服务器允许用户上传文件，为了避免用户上传一些病毒脚本，可以把上传的文件都单独放在一个分区，然后禁用这个分区的执行功能。

#### 9.2.3 挂载光盘与 U 盘

可以把 Ubuntu 的镜像文件当作光盘加入到虚拟机中，然后 `mount /dev/sr0 /cdrom` 就可以挂载了。

`umount /dev/sr0` 或者 `umount /cdrom` 就可以卸载了。注意卸载时当前路径不能在挂载路径内。

`cat /proc/partitions` 可以查看当前有哪些设备。比如查看插入的 U 盘是否识别。

`fdisk -l /dev/sda` 可以查看某个设备的具体分区信息。

`/proc/filesystems`{: .filepath} 文件和 `/lib/modules/$(uname -r)/kernel/fs` 目录下记录了完整的文件系统列表，应该是指可支持的。

> `man mount` 并搜索 `type` 查看说明。
{: .prompt-tip}

### 9.3 `fdisk` 分区

#### 9.3.1 `fdisk` 命令分区过程

`df -h` 像是在 Windows 里打开“此电脑”查看每个盘的使用情况。

`fdisk -l` 像是在 Windows 里打开“管理”中的“磁盘管理”查看每个磁盘的分区情况，同时也能看到某个盘是否被正确识别了。

##### 分区

形如 `fdisk /dev/sdb` 可以给磁盘分区，注意，`sdb` 不需要加数字。

`m` 可以查看帮助。`w` 保存分区并退出，在退出之前，分区都不会生效，

> 退出后需不需要用 `partprobe` 重新读取分区表？

`g` 创建 GPT 分区表，就可以以 GPT 格式给磁盘分区，此时就不分主分区和扩展分区了。

##### 格式化

分区之后，需要格式化，使用命令形如 `mkfs -t ext4 /dev/sdb1` 给分区格式化为 `ext4` 格式。

##### 挂载

形如 `mount /dev/sdb1 /mnt/disk1` 可以给格式化好的分区挂载到某一目录下。

#### 9.3.2 分区自动挂载与 `fstab` 文件修复

新的磁盘分区并格式化后，需要手动挂载。要想实现开机自动挂载，需要写入 `/etc/fstab`{: .filepath} 文件。

`/etc/fstab`{: .filepath} 文件内容中每一行一共六列，  

第一列是分区设备文件名或 UUID（硬盘通用唯一识别码），  

第二列是挂载点，  

第三列是文件系统名称，  

第四列是挂载参数（`defaults` 表示使用默认挂载参数），跟 `mount` 命令的 `-o` 选项需要的参数一样，  

第五列是指定分区是否被 `dump` 备份，`0` 表示不备份，`1` 表示每天备份，`2` 表示不定期备份，（每个分区下都有一个 `lost+found` 目录，就是用来备份的）  

第六列是指定分区是否被 `fsck` 检测，`0` 表示不检测，其他数字表示检测的优先级，`1` 的优先级比 `2` 高。

**重要**：

前面说过，如果 `/etc/fstab`{: .filepath} 这个文件的内容有格式错误或其他错误，那么就直接导致无法开机。

所以在修改完该文件之后，建议 **执行一遍 `mount -a`**，该命令会依据 `/etc/fstab`{: .filepath} 文件重新挂载一遍所有分区，如果中间有错误，能显示出来，这样就不至于等重启后发现文件写错了而不能开机了。

如果真的把这个文件写错了，又没有发现，重新启动后无法开机了，此时主机应该会显示让输入 root 密码以修复错误。输入密码后可以看到终端界面，但是此时根分区可能被挂载为了只读模式，因此在修改 `/etc/fstab`{: .filepath} 文件之前，需要重新把根分区挂载为读写模式：`mount -o remount,rw /`。然后就可以修改 `/etc/fstab`{: .filepath} 以保证能正常开机了。

此种方法只适用于因为 `/etc/fstab`{: .filepath} 文件写错而无法开机这一种情况。**但是如果把这个文件中的根分区格式都写错了，那连上面的修复界面都不会有了。**

### 9.4 分配 swap 分区

`free` 命令可以查看内存和 swap 分区的使用情况，`-h` 可以让显示数据更易读。

要扩大 swap 分区，需要如下步骤

#### 分区

找一块硬盘，分一块分区给 swap，同样使用 `fdisk /dev/sdb` 命令，`n` 增加新的分区。默认增加的分区可能是 `Linux` 类型而不是 `Linux Swap` 类型，因此还需要用 `t` 命令修改分区类型。类型代码用 `l` 查看。

最后 `w` 保存。

> 如果分区正忙，无法保存分区表，可以尝试 `partprobe` 命令或重启电脑。
{: .prompt-tip}

#### 格式化

命令 `mkswap /dev/sdb3` 可以将该分区格式化为 swap 分区。

注意这里不使用 `mkfs` 而是 `mkswap`。

#### 挂载

`swapon /dev/sdb3` 挂载。

此时再查看 `free -h` 命令，就会发现 swap 分区变大了。

`swapoff /dev/sdb3` 可以取消这块 swap 分区。

使用命令挂载的 swap 分区，在电脑重启后不会自动挂载。要想自动挂载，依然需要修改 `/etc/fstab`{: .filepath} 文件。

swap 分区的格式类似于

```plaintext
UUID=.. swap swap defaults 0 0
```

其中第二项挂载点是 `swap`，没有根目录。

> 或者参考系统自己的 swap 分区的设置。
{: .prompt-info}

## 第十讲 Shell 基础

### 10.1 Shell 概述

Bourne Shell 从 1979 年起就在 Unix 中使用，其在 Linux 系统中被识别为 sh。

C Shell 主要在 BSD 版的 Unix 系统中使用，语法与 C 语言类似。

以上两种 Shell 的语法结构完全不同，互不兼容。

Bourne Shell 家族主要有 sh，ksh，bash，psh，zsh；C Shell 家族主要有 csh，tcsh。

`/etc/shells`{: .filepath} 文件中记录着系统支持的 Shell 类型。运行命令就可以切换不同的 Shell。

### 10.3 Bash 的基本功能

#### 10.3.1 历史命令与命令补全

`history` 命令可以查看最近的 1000 条终端命令。`~/.bash_history`{: .filepath} 文件里记录着最近的 2000 条终端命令。

这个数量的定义在 `~/.bashrc`{: .filepath} 文件中

```plaintext
HISTSIZE=1000
HISTFILESIZE=2000
```

`~/.bash_history`{: .filepath} 文件中默认只记录到上次关机时的终端命令，当前开机后的命令记录在内存中，使用 `history` 可以看到，但是还没记录到文件。

`history -w` 可以把内存里的历史命令记录到文件中。

`-c` 可以清空内存里的历史命令记录。

`!n` 可以执行历史命令里的第 n 条命令。n 可以是多少可以用 `history` 查看，每条历史命令都有个序号。

#### 10.3.2 命令别名与常用快捷键

`alias` 命令可以查看当前定义了多少别名。

形如 `alias vi="vim"` 可以定义别名。双引号内的是原命令。

命令执行顺序：

1. 使用绝对路径或相对路径执行的命令
2. 别名
3. bash 内部命令
4. `$PATH` 环境变量中定义的目录中找到的第一个命令

通过 `alias` 查看可以知道，`ls` 之所以会显示不同的颜色，是因为有一个别名 `alias ls="ls --color=auto"`，也就是说，直接执行 `/bin/ls`{: .filepath} 是不会有不同颜色的。

别名的顺位比内部命令还高，也就是说如果定义一个 `alias cd=".."` 那么会优先执行这个 `cd` 别名，而不是直接执行内部命令。

要取消某个别名，可以用 `unalias`。

常用快捷键：

| 快捷键 | 作用 |
|--|--|
| CTRL+A | 光标移动到命令开头 |
| CTRL+E | 光标移动到命令结尾 |
| CTRL+C | 终止当前命令 |
| CTRL+L | 清屏，相当于 `clear` |
| CTRL+U | 删除/剪切光标左边的所有字符 |
| CTRL+K | 删除/剪切光标右边的所有字符 |
| CTRL+Y | 粘贴剪切的内容 |
| CTRL+R | 在历史命令中搜索 |
| CTRL+D | 退出终端 |
| CTRL+Z | 暂停并放入后台 |
| CTRL+S | 暂停屏幕输出 |
| CTRL+Q | 恢复屏幕输出 |

#### 10.3.3 输入输出重定向

标准输出重定向：`>` 或 `>>`，此处输出的是正确运行的结果。

标准错误输出重定向：`2>` 或 `2>>`，此处输出的是错误运行的结果。

但是实际生产中，我们也不可能提前知道哪些命令是正确的，哪些是错误的，因此两种输出重定向分开写的情况不多。多数是合在一起写。写法有如下几种

正确输出和错误输出同时保存：

| 格式 | 说明 |
|--|--|
| `cmd > file 2>&1` | 覆盖 |
| `cmd >> file 2>&1` | 追加 |
| `cmd &> file` | 覆盖 |
| `cmd &>> file` | 追加 |
| `cmd >> file1 2>> file2` | 正确输出追加到 `file1`，<br />错误输出追加到 `file2` |

输入重定向即把原本从标准输入，也就是键盘获取的内容转而从文件获取。

`wc` 可以统计输入内容的行数、单词数和字符数。

`wc < [file]` 可以统计文件内容的行数、单词数和字符数。

#### 10.3.4 多命令顺序执行与管道符

`cmd1 ; cmd2` 两条命令之间没有任何关系，只是用分号写在一行而已。

`cmd1 && cmd2` 第一条命令正确执行，才会执行第二条命令。

`cmd1 || cmd2` 第一条命令错误执行，才会执行第二条命令。

小技巧：`cmd && echo yes || echo no` 可以根据输出判断命令是否正确执行。

#### 10.3.5 通配符与其他特殊符号

通配符跟正则还有点区别，通配符中的 `*` 像是任意 0 个或多个字符串的替身，而正则中的 `*` 表示前一个字符可以重复出现 0 次或多次。通配符中 `?` 必须匹配 1 个任意字符，0 个不行，正则中的 `?` 表示前一个字符可以出现 0 次或 1 次。

特殊符号：

| 符号 | 作用 |
|--|--|
| `''` | 单引号中的所有特殊符号都没有特殊含义 |
| `""` | 双引号中，除了 `$`、`` ` ``、`\` 之外的所有特殊符号都没有特殊含义 |
| ``` `` ``` | 反引号中的内容会当作命令对待（不推荐） |
| `$()` | 括号内的内容会当作命令对待（推荐） |
| `#` | 注释 |
| `$` | 在变量前加该符号，表示调用变量的值 |
| `\` | 转义符 |

```terminal
julian@ubuntu2204:~/temp/abc$ cmd=ls

julian@ubuntu2204:~/temp/abc$ echo $cmd
ls

julian@ubuntu2204:~/temp/abc$ echo '$cmd'
$cmd

julian@ubuntu2204:~/temp/abc$ echo "$cmd"
ls

julian@ubuntu2204:~/temp/abc$ echo `$cmd`
0abc 123 abc abcd

julian@ubuntu2204:~/temp/abc$ echo $($cmd)
0abc 123 abc abcd
```

### 10.4 Bash 的变量

#### 10.4.1 用户自定义变量

bash 的变量值默认都是字符串类型。如果要进行数值运算，需要明确指定。

变量的等号两侧 **不能** 有空格。

引用变量时可以使用 `$var` 或 `${var}` 的形式

```terminal
julian@ubuntu2204:~/temp/abc$ g=hello

julian@ubuntu2204:~/temp/abc$ n="$g world"

julian@ubuntu2204:~/temp/abc$ echo $n
hello world

julian@ubuntu2204:~/temp/abc$ m="${g}world"

julian@ubuntu2204:~/temp/abc$ echo $m
helloworld

julian@ubuntu2204:~/temp/abc$ p="$m:yes"

julian@ubuntu2204:~/temp/abc$ echo $p
helloworld:yes
```

`set` 命令查看所有变量（会很长）。

`unset [variable]` 可以删除变量

```terminal
julian@ubuntu2204:~/temp/abc$ echo $name
ls

julian@ubuntu2204:~/temp/abc$ unset name

julian@ubuntu2204:~/temp/abc$ echo $name

```

#### 10.4.2 环境变量

本地变量只能在当前的 Shell 中生效，环境变量可以在当前 Shell 以及所有子 Shell 中生效。

`pstree` 可以查看当前在哪一层 Shell。

如果是用远程终端连接的，可能在 sshd 那一支，如果使用的是本机终端，可能在 login 那一支。总之最后都是 bash-pstree，应该是表示该 Shell 最后执行的一条命令。

定义环境变量的方法：`export var=value`

```terminal
julian@ubuntu2204:~/temp/abc$ name=john

julian@ubuntu2204:~/temp/abc$ export age=18

julian@ubuntu2204:~/temp/abc$ sex=male

julian@ubuntu2204:~/temp/abc$ export sex

julian@ubuntu2204:~/temp/abc$ echo $name $age $sex
john 18 male

julian@ubuntu2204:~/temp/abc$ bash

julian@ubuntu2204:~/temp/abc$ echo $name $age $sex
18 male
```

`env` 可以查看当前 Shell 的环境变量。（`set` 则同时能看到本地变量和环境变量，范围比 `env` 大）

有两个比较重要的环境变量：`$PATH` 和 `$PS1`。（后者在 `env` 看不到，但也属于环境变量，用 `set` 可看）

`$PS1` 定义了终端的提示头的显示格式

| 符号 | 说明 |
|--|--|
| `\d` | 显示日期：“星期 月 日” |
| `\h` | 显示主机名 |
| `\t` | 显示 24 小时制时间：“HH:MM:SS” |
| `\T` | 显示 12 小时制时间：“HH:MM:SS” |
| `\A` | 显示 24 小时制时间：“HH:MM” |
| `\u` | 显示用户名 |
| `\w` | 显示当前所在目录的完整路径 |
| `\W` | 显示当前所在目录的最后一个目录 |
| `\#` | 显示当前执行的是第几条命令 |
| `\$` | 提示符。root 用户为 `#`，普通用户为 `$` |

#### 10.4.3 位置参数变量

| 变量 | 作用 |
|--|--|
| `$n` | `n` 为数字，`$0` 表示命令本身，`$1` 到 `$9` 表示第 1 到 9 个参数，10 以上的参数需要用 `${10}`
| `$*` | 表示命令中的所有参数，即把所有参数看作一个整体 |
| `$@` | 表示命令中的所有参数，但把每个参数区分对待 |
| `$#` | 表示命令中所有参数的个数 |

`param1.sh`：

```shell
##!/usr/bin/bash

echo "0=$0, 1=$1, 2=$2"
echo "Total: $#"
echo "params: $*"
echo "params: $@"
```

```terminal
julian@ubuntu2204:~/temp/abc$ ./param1.sh 11 22
0=./param1.sh, 1=11, 2=22
Total: 2
params: 11 22
params: 11 22
```

`$*` 和 `$@` 的区别可以在 `for` 循环中看出来，前者只循环一次，后者会循环“参数总个数”次。

#### 10.4.4 预定义变量

| 变量 | 作用 |
|--|--|
| `$?` | 上一个命令的返回状态。如果为 0，表示上一个命令正确执行，如果非 0，表示没有正确执行，具体是几，由命令编写者决定 |
| `$$` | 当前进程的进程号（PID） |
| `$!` | 后台运行的最后一个进程的进程号（PID） |

```terminal
julian@ubuntu2204:~/temp/abc$ ls
param1.sh

julian@ubuntu2204:~/temp/abc$ echo $?
0

julian@ubuntu2204:~/temp/abc$ lst
Command 'lst' not found, but there are 21 similar ones.

julian@ubuntu2204:~/temp/abc$ echo $?
127

julian@ubuntu2204:~/temp/abc$ ls abc
ls: cannot access 'abc': No such file or directory

julian@ubuntu2204:~/temp/abc$ echo $?
2
```

`&&` 和 `||` 也是通过判断上一个命令的返回值来确定是否有正确执行的。

`param2.sh`

```shell
##!/usr/bin/bash

echo "current process id: $$"

find /home/julian -name hello.sh &
echo "last background process id: $!"
```

命令最后加 `&` 表示把该命令放入后台运行

```terminal
julian@ubuntu2204:~/temp/abc$ ./param2.sh 
current process id: 3834
last background process id: 3835

julian@ubuntu2204:~/temp/abc$ /home/julian/temp/hello.sh

```

> 最后一条是后台运行的结果。
{: .prompt-tip}

##### `read`

从标准输入接收输入（类似于 Python 的 `input`）

`read [选项] [变量名]`

```terminal
julian@ubuntu2204:~/temp/abc$ read -p "Enter a number:" num
Enter a number:10

julian@ubuntu2204:~/temp/abc$ echo $num
10
```

`-p` 增加提示信息，`-t` 设定等待时间，`-n` 设定最大接收字符数，`-s` 隐秘输入。

### 10.5 Bash 的运算符

#### 10.5.1 数值运算与运算符

Shell 中的变量值默认都是字符串，因此要进行数值计算，如下方法行不通

```terminal
julian@ubuntu2204:~$ a=11

julian@ubuntu2204:~$ b=22

julian@ubuntu2204:~$ c=$a+$b

julian@ubuntu2204:~$ echo $c
11+22
```

##### `declare`

`declare -p [variable]` 可以查看变量的属性。`-x` 把变量变为环境变量（相当于 `export`）

```terminal
julian@ubuntu2204:~$ declare -p a
declare -- a="11"

julian@ubuntu2204:~$ declare -x a

julian@ubuntu2204:~$ declare -p a
declare -x a="11"

julian@ubuntu2204:~$ env | grep ^a
a=11
```

`-i` 把变量变为数值，因此便可以进行数值运算了。

```terminal
julian@ubuntu2204:~$ declare -i c=$a+$b

julian@ubuntu2204:~$ echo $c
33

julian@ubuntu2204:~$ declare -p c
declare -i c="33"
```

> 如果运算符两边有变量不是纯数字，会报错。
{: .prompt-warning}

##### `expr`

`expr` 也可以进行数值计算，不过运算符两边 **必须有空格**，可以结合 `$()` 进行赋值

```terminal
julian@ubuntu2204:~$ expr $a + $b
33

julian@ubuntu2204:~$ d=$(expr $c + $b)

julian@ubuntu2204:~$ echo $d
55
```

如果不加空格就会当字符串对待

```terminal
julian@ubuntu2204:~$ expr $a+$b
11+22
```

##### `let`

`let` 后跟的表达式不能有空格。

```terminal
julian@ubuntu2204:~$ let c=$a-$b

julian@ubuntu2204:~$ echo $c
-11
```

##### `$(())` 或 `$[]`

这样就无所谓空不空格了。

```terminal
julian@ubuntu2204:~$ e=$(($a+$b))

julian@ubuntu2204:~$ echo $e
33

julian@ubuntu2204:~$ e=$(( $a + $b ))

julian@ubuntu2204:~$ echo $e
33

julian@ubuntu2204:~$ e=$[$a + $b]

julian@ubuntu2204:~$ echo $e
33
```

#### 10.5.2 变量测试与内容替换

现将一个变量的状态分为三类：

- 未定义（undefined）
- 定义但为空（`y=`）
- 定义且有值（initialized）

则

| 变量置换 | 条件 | 符合 | 不符合 |
|--|--|--|--|
| `x=${y-val}` | `y` 未定义 | `x=val` | `x=$y` |
| `x=${y:-val}` | `y` 定义且有值 | `x=$y` | `x=val` |
| `x=${y+val}` | `y` 未定义 | `x=` | `x=val` |
| `x=${y:+val}` | `y` 定义且有值 | `x=val` | `x=` |
| `x=${y=val}` | `y` 未定义 | `x=val`<br/>`y=val` | `x=$y`<br/>`y` 不变 |
| `x=${y:=val}` | `y` 定义且有值 | `x=$y`<br/>`y` 不变 | `x=val`<br/>`y=val` |
| `x=${y?val}` | `y` 未定义 | `val` 输出到<br/>标准错误输出 | `x=$y` |
| `x=${y:?val}` | `y` 定义且有值 | `x=$y` | `val` 输出到<br/>标准错误输出 |

这个东西的作用就是根据 `x` 的值来判断 `y` 的值。其实要看用哪个，就根据需要什么效果：

- 如果不关心 `y` 是不是空，只关心其有没有定义，使用 `y` 未定义的条件（没有冒号的）
- 如果要求 `y` 不仅仅是定义了，还必须有值，使用 `y` 定义且有值的条件（有冒号的）
- 根据希望进行判断后 `x` 要一个什么样的值（是 `val` 还是 `$y` 还是空）来选择使用加号还是减号
- 等号和问号同理

### 10.6 环境变量配置文件

#### 10.6.1 环境变量配置文件简介

修改配置文件后一般需要重启才能让配置文件里修改的内容生效，但是 `source [filename]` 可以就地执行一遍配置文件的内容，让修改立刻生效。其等同于 `. [filename]`。

在 Ubuntu 22.04.3 上，配置文件有如下几个

- `/etc/profile`{: .filepath}
- `/etc/bash.bashrc`{: .filepath}
- `/etc/profile.d/*.sh`{: .filepath}
- `~/.profile`{: .filepath}
- `~/.bashrc`{: .filepath}

#### 10.6.3 其他配置文件和登录信息

注销时生效的环境变量配置文件 `~/.bash_logout`{: .filepath}

##### 本地终端登录前信息

`/etc/issue`{: .filepath}

| 转义符 | 作用 |
|--|--|
| `\d` | 当前系统日期 |
| `\s` | 操作系统名称 |
| `\l` | 登录的终端号 |
| `\m` | 硬件体系结构 |
| `\n` | 主机名 |
| `\o` | 域名 |
| `\r` | 内核版本 |
| `\t` | 当前系统时间 |
| `\u` | 当前登录用户的序列号 |

上述文件的内容显示在 **本地终端** 登录前的提示信息中。

##### 远程终端登录前信息

`/etc/issue.net`{: .filepath}

- 转义符不起作用，只能写纯文本信息
- 默认没有开启这个功能

要开启此功能，需要在 `/etc/ssh/sshd_config`{: .filepath} 文件中加入 `Banner /etc/issue.net` 行，然后重启 ssh 服务（`service sshd restart`）。

## 第十一讲 Shell 编程

### 11.1 基础正则表达式

`ls`、`find`、`cp` 等命令不支持正则表达式，因此只能使用简单的 Linux 通配符。

`grep`、`awk`、`sed` 等命令支持正则表达式，可以用正则匹配文件中的字符串。

Shell 的基础正则表达式中不包含 `()`、`+` 和 `?`。这些在扩展正则表达式中。

### 11.2 字符截取命令

#### 11.2.1 `cut` 字段提取命令

`grep` 会在文本中提取符合条件的行，也就是说 `grep` 是按行操作的。

`cut` 则会先把每一行分成不同的段，然后提取每一行的同一段。

分段方式有三种，`-b` 按照字节分段，`-c` 按照字符分段，`-f` 按照 `-d` 指定的分段符分段，默认是 `\t` 制表符。

测试发现在对待中文上 `-b` 和 `-c` 没啥差别。

提取分段的方式有四种：

- n，第 n 段，从 1 计数
- n-，从第 n 段到最后
- n-m，从第 n 段到第 m 段（包含）
- -m，从第 1 段到第 m 段（包含）

要提取多个范围时，用逗号分隔。

`ctext` 是用制表符分隔的。

```terminal
julian@ubuntu2204:~/temp$ cat ctext 
ID	Name	Gender	Mark
1	Liming	M	86
2	Sc	M	90
3	Gao	F	83

julian@ubuntu2204:~/temp$ cut -f 2,3 ctext 
Name	Gender
Liming	M
Sc	M
Gao	F
```

`dtext` 是用空格分隔的。

```terminal
julian@ubuntu2204:~/temp$ cat dtext 
ID Name Sex Mark
1 ZS M 90
2 LS M 23
3 WW F 30

julian@ubuntu2204:~/temp$ cut -d " " -f -3 dtext 
ID Name Sex
1 ZS M
2 LS M
3 WW F
```

某些命令的输出，比如 `df -h`，其格式是经过程序美化的，每列之间的分隔符不是单纯的一个制表符或者一个空格，而是根据界面的不同而有所不同，因此无法用 `cut` 进行分段。

#### 11.2.2 `printf` 命令

`printf` 会按照匹配符的顺序，从参数里一个一个拿数据匹配并格式化输出，匹配完一轮就再重新开始。比如

```terminal
julian@ubuntu2204:~/temp$ printf "%s %i\n" abc 12 wed 45
abc 12
wed 45
```

#### 11.2.3 `awk` 命令

入门查看 [AWK 命令]({{site.url}}/posts/awk-command/)。

因为 AWK 按照空白字符分隔段，所以可以处理 `df -h` 这样的输出

```terminal
julian@ubuntu2204:~/temp$ df -h | grep "/$" | awk '{print "root disk part used " $5}'
root disk part used 29%
```

等同于

```terminal
julian@ubuntu2204:~/temp$ df -h | awk '/\/$/ {print "root disk part used " $5}'
root disk part used 29%
```

#### 11.2.4 `sed` 命令

sed 是一个轻量级流编辑器，不同于 vim 只能编辑文件，sed 除了能编辑文件，也能编辑其他命令的输出结果。

`sed` 命令的格式

```plaintext
sed [选项] '[动作]' 流
```

选项

- `-n`：只输出经过 `sed` 命令处理的行
- `-i`：用 `sed` 的修改结果替换原文件

动作

- `a`：追加行
- `c`：替换行
- `i`：插入行
- `d`：删除行
- `p`：打印行
- `s`：字符串替换，格式为 `s/old/new/g`

后面的示例中使用的文件

```terminal
julian@ubuntu2204:~/temp$ cat ctext 
ID	Name	Gender	Mark
1	Liming	M	86
2	Sc	M	90
3	Gao	F	83
```

打印第 2 行内容

```terminal
julian@ubuntu2204:~/temp$ sed '2p' ctext 
ID	Name	Gender	Mark
1	Liming	M	86
1	Liming	M	86
2	Sc	M	90
3	Gao	F	83
```

但是除了第二行，其他行也都打印了，因此要加 `-n`

```terminal
julian@ubuntu2204:~/temp$ sed -n '2p' ctext 
1	Liming	M	86
```

删除第 2-3 行

```terminal
julian@ubuntu2204:~/temp$ sed '2,3d' ctext 
ID	Name	Gender	Mark
3	Gao	F	83
```

但是此时只是输出内容没了 2-3 行，实际的文件还是没变的，因为没加 `-i` 选项。

在第 2 行后追加 `hello world`

```terminal
julian@ubuntu2204:~/temp$ sed '2a hello world' ctext 
ID	Name	Gender	Mark
1	Liming	M	86
hello world
2	Sc	M	90
3	Gao	F	83
```

追加多行

```terminal
julian@ubuntu2204:~/temp$ sed '2a hello \
> world' ctext
ID	Name	Gender	Mark
1	Liming	M	86
hello 
world
2	Sc	M	90
3	Gao	F	83
```

在第 2 行前加入 `hello world`

```terminal
julian@ubuntu2204:~/temp$ sed '2i hello world' ctext 
ID	Name	Gender	Mark
hello world
1	Liming	M	86
2	Sc	M	90
3	Gao	F	83
```

将第二行替换为 `hello world`

```terminal
julian@ubuntu2204:~/temp$ sed '2c hello world' ctext 
ID	Name	Gender	Mark
hello world
2	Sc	M	90
3	Gao	F	83
```

替换特定字符串

```terminal
julian@ubuntu2204:~/temp$ sed 's/Liming/LM/g' ctext 
ID	Name	Gender	Mark
1	LM	M	86
2	Sc	M	90
3	Gao	F	83
```

多个条件用分号分隔

```terminal
julian@ubuntu2204:~/temp$ sed 's/Liming/LM/g;3c hello world' ctext 
ID	Name	Gender	Mark
1	LM	M	86
hello world
3	Gao	F	83
```

以上的所有操作如果想修改原文件，加 `-i` 选项，不过一般不会去修改原文件。

### 11.3 字符处理命令

#### `sort`

`sort` 命令可以把文件或者流中的数据按照行的字母顺序排序

```terminal
julian@ubuntu2204:~/temp$ cat ctext 
root:x:0
julian:x:10
liz:x:2
karl:x:11

julian@ubuntu2204:~/temp$ sort ctext 
julian:x:10
karl:x:11
liz:x:2
root:x:0
```

`-t` 可以指定行的分隔符，默认是制表符，`-k n[,m]` 可以指定按照从第 n 行到第 m 行排序

```terminal
julian@ubuntu2204:~/temp$ sort -t ":" -k 3,3 ctext 
root:x:0
julian:x:10
karl:x:11
liz:x:2
```

`-n` 可以指定按照数值排序，默认是字符串

```terminal
julian@ubuntu2204:~/temp$ sort -n -t ":" -k 3 ctext 
root:x:0
liz:x:2
julian:x:10
karl:x:11
```

### 11.4 条件判断

#### 按照文件类型判断

| 选项 | 作用 |
|--|--|
| `-b` | 存在且为块设备文件 |
| `-c` | 存在且为字符设备文件 |
| `-d` | 存在且为目录文件 |
| `-e` | 存在 |
| `-f` | 存在且为普通文件 |
| `-L` | 存在且为符号链接文件 |
| `-p` | 存在且为管道文件 |
| `-s` | 存在且文件非空 |
| `-S` | 存在且为套接字文件 |

判断有两种写法

```plaintext
test -e file
[ -e file ]
```

两者都不会有返回结果，要查看结果需要用 `echo $?`，为 0 则存在，其他则不存在。

因为条件判断主要用在 Shell 脚本中，所以第二种更常用。（第二种用法的中括号两侧必须有空格）

常用的选项有 `-d`，`-e`，`-f`。

#### 按照文件权限判断

| 选项 | 作用 |
|--|--|
| `-r` | 存在且有读权限 |
| `-w` | 存在且有写权限 |
| `-x` | 存在且有执行权限 |
| `-u` | 存在且有 SUID 权限 |
| `-g` | 存在且有 SGID 权限 |
| `-k` | 存在且有 SBIT 权限 |

这里的权限并不区分所有者或者所属组或者其他人，只要任意一方有相应权限，就会返回真。

常用的选项有 `-r`，`-w`，`-x`。

#### 两个文件之间比较

| 选项 | 作用 |
|--|--|
| `file1 -nt file2` | file1 的修改时间比 file2 新 |
| `file1 -ot file2` | file1 的修改时间比 file2 旧 |
| `file1 -ef file2` | file1 和 file2 的 inode 号一致 |

#### 两个整数之间比较

| 选项 | 作用 |
|--|--|
| `int1 -eq int2` | int1 等于 int2 |
| `int1 -ne int2` | int1 不等于 int2 |
| `int1 -gt int2` | int1 大于 int2 |
| `int1 -lt int2` | int1 小于 int2 |
| `int1 -ge int2` | int1 大于等于 int2 |
| `int1 -le int2` | int1 小于等于 int2 |

#### 字符串判断

| 选项 | 作用 |
|--|--|
| `-z string` | 字符串为空 |
| `-n string` | 字符串非空 |
| `str1 == str2` | str1 等于 str2 |
| `str1 != str2` | str1 不等于 str2 |

`==` 会把两侧的值当作字符串判断，`-eq` 会把两侧的值当作数值判断。

#### 多重条件判断

| 选项 | 作用 |
|--|--|
| `cond1 -a cond2` | 条件1 与 条件2 |
| `cond1 -o cond2` | 条件1 或 条件2 |
| `! cond` | 非 条件 |

### 11.5 流程控制

#### 11.5.1 `if` 语句

查看 `help if` 可以看到比较简洁的 `if` 语句写法

```plaintext
if COMMANDS; then COMMANDS; [ elif COMMANDS; then COMMANDS; ]... [ else COMMANDS; ] fi
```

比如

```terminal
julian@ubuntu2204:~/temp$ if [ -e gtext ]; then echo yes; else echo no; fi
yes
```

分开层次之后，可以这样

```shell
if [ -e gtext ]; then
    echo yes
else
    echo no
fi
```

其中 `then` 前面的分号必须要有

也可以这样

```shell
if [ -e gtext ]
then
    echo yes
else
    echo no
fi
```

此时 `if` 那一行最后就可以不要分号。

#### 11.5.2 `case` 语句

如下例

```shell
##!/usr/bin/bash

read -p "Enter y/n: " choice

case $choice in
    "y")
        echo "you choose yes"
        ;;
    "n")
        echo "you choose no"
        ;;
    *)
        echo "don't know what you choose"
        ;;
esac
```

以 `case $var in` 开头，每个条件用 `pattern)` 指定，每一分支最后用 `;;` 表示中断，类似于 `break`，最后的 `*)` 表示前面都不符合时执行的分支。

`pattern` 也可以有好几个，比如

```shell
##!/usr/bin/bash

read -p "Enter y/n: " choice

case $choice in
    "y" | "yes")
        echo "you choose yes"
        ;;
    "n" | "no")
        echo "you choose no"
        ;;
    *)
        echo "don't know what you choose"
        ;;
esac
```

通常在提供不同的选项列表时会用 `case` 判断用户选择了哪个选项。

#### 11.5.3 `for` 循环

##### 语法一：不定次数循环

语法格式

```plaintext
for NAME [in WORDS ... ] ; do COMMANDS; done
```

`WORDS` 会按照 **空格或换行符** 分隔成段并分别赋值给 `NAME`

```shell
##!/usr/bin/bash

for time in morning noon afternoon evening; do
    echo "now is $time"
done
```

当然，跟 `if` 一样，如果 `for` 中的 `do` 写在第二行，就不需要分号了

```shell
##!/usr/bin/bash

for time in morning noon afternoon evening
do
    echo "now is $time"
done
```

示例：批量解包脚本

```shell
##!/usr/bin/bash

ls *.tar.gz > tar_gz.log
for t in $(cat tar_gz.log)
do
    tar -zxf $t &>/dev/null
done

rm -f tar_gz.log
```

##### 语法二：确定次数循环

如下例

```shell
##!/usr/bin/bash

s=0
for (( i=1;i<=100;i++ ))
do
    s=$(( $s+$i ))
done
echo "The sum of 1..100 is $s"
```

注意，`for` 后面是双括号，只有双括号里的才是数值运算。

示例：批量创建用户

```shell
##!/usr/bin/bash

read -p "The default user name: " -t 30 name
read -p "The number of users you wanna create: " -t 30 num

if [ -n "$name" -a -n "$num" ]
then
    y=$( echo $num | sed 's/[0-9]\+//1' )
    if [ -z $y ]
    then
        for (( i=1;i<=$num;i++ ))
        do
            echo "$name$i is created"
        done
    else
        echo "$num is not a pure number; ($y)"
    fi
else
    echo "name or num is empty; ($name)($num)"
fi
```

这里几个点要注意下：

第一个 `if` 中判断变量是不是空时，`"$name"` 用双引号括起来，因为如果 `$name` 真的是空，且没有双引号，那么这个 `if` 可能就会变成形如 `if [ -n -a -n "3" ]` 的格式，而这是有语法错误的，用双引号括起来之后，即便为空，也是个空字符串，不会造成语法错误。

`sed 's/[0-9]\+//1'` 表示匹配 **一个或多个** 数字 **一次**，并替换为空，其等同于 `sed 's/[0-9]//g'`，表示匹配 **一个** 数字，但 **多次** 替换为空。也就是说，前者匹配多个连续的数字，但只替换一次，后者只匹配一个数字，但全部替换。

但是这里说的等同，是指在这个示例中作用是等同的，但它们两个的实际效果并不等同，比如

```terminal
julian@ubuntu2204:~/temp$ echo 123g456 | sed 's/[0-9]//g'
g

julian@ubuntu2204:~/temp$ echo 123g456 | sed 's/[0-9]\+//1'
g456
```

如果末尾是 `2`，表示替换第二个匹配，而不是前两个

```terminal
julian@ubuntu2204:~/temp$ echo 123g456 | sed 's/[0-9]\+//2'
123g
```

#### 11.5.4 `while` 循环与 `until` 循环

`while` 语法格式

```plaintext
while COMMANDS; do COMMANDS; done
```

```shell
##!/usr/bin/bash

s=0
i=1
while [ $i -le 100 ]
do
    s=$(( $s+$i ))
    i=$(( $i+1 ))
done
echo "The sum of 1..100 is $s"
```

> `i=$(( $i+1 ))` 等同于 `i=$(( i+1 ))` 等同于 `i=$(( ++i ))`
{: .prompt-tip}

`until` 语法格式

```plaintext
until COMMANDS; do COMMANDS; done
```

跟 `while` 差不多，但是判断条件相反，`until` 是当条件不成立时执行循环

```shell
##!/usr/bin/bash

s=0
i=1
until [ $i -gt 100 ]
do
    s=$(( $s+$i ))
    i=$(( $i+1 ))
done
echo "The sum of 1..100 is $s"
```

## 第十二讲 Linux 服务管理

### 12.4 服务管理总结

第一次接触服务管理，记录下目前为止自己对于服务的理解。

以前会觉得执行文件都是软件，说一个文件可以执行，那么就会自然而然地觉得它执行起来就会出现一个交互界面，供用户操作。但实际上也有很多执行文件并不是软件，没有交互界面，执行之后只在后台运行，但是却保证了系统或者某些软件的正常运作。

这些程序，被称作服务。

其实服务也好理解，软件一般是一个由交互界面的程序，服务则一般是看不到的、在后台默默运行的程序。管理服务也跟管理软件差不多。软件可以打开关闭，服务也可以打开关闭。只不过因为服务没有交互界面，没有一个叉让我们点。

`/etc/init.d/`{: .filepath} 下保存有很多独立服务，使用

```plaintext
/etc/init.d/[service] start|stop|status|restart
```

可以让服务开启、停止、显示状态、重启。

操作服务本质上可以使用上述这种用绝对路径直接指明服务文件的方法，但是也可以使用一些服务管理工具，比如 `service` 和 `systemctl`，两者用法上有些不同，不过功能差不多。

通过二进制包安装的服务，可以由系统管理，通过源码包安装的服务，一般默认在 `/usr/local`{: .filepath} 下，而不是分散在各个目录中，这些服务默认不由系统管理，需要手动管理。

服务除了常规的操作其开启或关闭之外，也可以设置是否开机自启动。

> 其他内容研究研究再来补充。

## 第十三讲 Linux 进程管理

### 13.1 进程管理

#### 13.1.1 进程查看

正在运行当中的程序，叫做进程。程序要想运行，至少会启动一个进程，有些比较复杂的程序，可能会启动多个进程。

进程管理主要是 **监控进程的健康状态，也就是服务器的健康状态**。

如果某些进程异常占据了很多资源，就需要检查一下是什么问题。

但很多问题都不能只是终止进程了事，哪怕发现了病毒进程，也不是终止就完了的，而是需要找到这个进程是从哪里启动的，病毒文件在哪里，彻底清除之后，再终止进程，否则的话，可能即便终止了进程它自己过一会就又起来了。

好的工程师在服务器处在亚健康时就能发现并解决问题，而不是等服务器出现明显的问题了才开始解决。

如果服务器少，可能手动执行一些进程查看命令还能应付的过来，但如果服务器上千台，不可能手动执行命令监控进程了，这时候就需要搭建一些监控服务器来辅助了。

##### `ps aux`

`ps aux` 是 BSD 操作系统格式，等同于 `ps -le`，后者是 Linux 标准命令格式。用哪个都行。

该命令可以查看当前系统中的所有进程。其格式比较复杂：

第一列，启动进程的用户。

第二列，进程 ID，也就是 PID。

第三列，该进程占用的 CPU 资源百分比。

第四列，该进程占用的物理内存的百分比。

第五列，VSZ，进程占用的虚拟内存大小，单位 KB。

第六列，RSS，进程占用实际物理内存大小，单位 KB。

第七列，TTY，该进程在哪个终端运行的。绝大部分系统进程都是内核直接调用的，不是在终端运行的，因此显示 `?`。tty1-7 是本地终端，pts/0-255 是远程终端，也就是说一个主机支持 256 个远程终端同时在线。

第八列，STAT，进程状态。R：运行；S：睡眠；T：停止；s：包含子进程；+：位于后台。

第九列，进程启动时间，超过一天的会用日期表示，一天之内的会用时间表示。

第十列，该进程占用 CPU 的运算时间。

第十一列，产生此进程的命令名。

##### `top`

实时查看系统各进程的运行情况，类似于 Windows 的任务管理器。`top` 默认每 3 秒刷新一次。`-d` 可以指定间隔秒数。

> 其实可以看到，`top` 本身也是比较耗资源的。
{: .prompt-tip}

第一行信息包含如下

系统当前时间；`up` 后是系统持续运行时间；当前几个用户在线；系统在 1 分钟、5 分钟、15 分钟之前的平均负载。

这个平均负载的参考跟 CPU 核数有关，如果是单核，超过 1 就算高了，如果是八核，可能超过 8 才算高。

第二行信息包含如下

进程数：总共多少；正在运行的多少；睡眠的多少；停止的多少；僵尸进程多少。

僵尸进程表示没有终止成功的进程。如果很长时间僵尸进程依然还在，可能就是程序在终止时卡死了，需要手动处理。

第三行信息包含如下

CPU 信息：us 用户模式占用百分比；sy 系统模式占用百分比；ni 改变过优先级的用户进程占用百分比；id 空闲 CPU 百分比；wa 等待输入输出的进程占用百分比；hi 硬中断请求服务占用百分比；si 软中断请求服务占用百分比；st 虚拟时间百分比（有虚拟机的时候有用）。

第四行信息包含如下

内存信息：总共多少；空闲多少；使用多少；缓冲/缓存多少。

单位 MB。

第五行信息包含如下

Swap 信息：总共多少；空闲多少；使用多少；？。

单位 MB。

上述重点信息有：第一行的平均负载，第三行的 CPU 空闲百分比（id），第四行的空闲内存大小。

这五行信息之下的信息与 `ps aux` 类似，不过默认按照 CPU 使用率排序。

敲 `Shift+M` 可以切换为按照内存占用排序，`Shift+P` 按照 CPU 使用率排序，`shift+N` 按照 PID 排序。

##### `pstree`

查看进程树。`-p` 显示进程 PID，`-u` 显示用户。

#### 13.1.2 终止进程

##### `kill`

`-l` 可以查看所有支持的信号，其中常用的有

| 信号代号 | 信号名称 | 说明 |
|--:|--|--|
| `1` | SIGHUP | 关闭进程，然后重新读取配置文件后重启 |
| `9` | SIGKILL | 强制终止进程，不能被阻塞、处理或者忽略 |
| `15` | SIGTERM | 正常结束进程，是默认信号 |

`kill [PID]` 默认用 `15` 信号结束进程，如果想要强制，使用 `kill -9 [PID]`。

##### `killall`

格式 `killall [选项] [信号] [进程名]`

其中选项有 `-i` 结束进程时询问一次，`-I` 忽略进程名的大小写。

##### `pkill`

一方面用法跟 `killall` 一样，同时还支持根据终端号踢出用户：`pkill -t [TTY]`。也可以加 `-9` 强制踢出。

### 13.2 工作管理

有两种方式可以把命令放入后台，

1. 在命令后加 `&`，比如 `tar -zcf etc.tar.gz /etc &`
2. 在命令执行过程中摁 Ctrl+Z

区别是，前者放入后台后，命令仍在运行，后者放入后台后，命令是暂停的。

当我们用 Ctrl+Z 把命令放入后台之后，可以用 `jobs` 命令查看后台所有的命令。`-l` 选项可以显示 PID 等信息。

```terminal
julian@gallifrey:~$ jobs
[1]+  Stopped                 top

julian@gallifrey:~$ jobs -l
[1]+ 76011 Stopped (signal)        top
```

因为查询后台的命令是 `jobs` 我们可以把后台任务成为“工作”。每一个工作都有一个工作号，从 `1` 开始。最后一个加入的工作会用 `+` 标记，倒数第二个加入的工作会用 `-` 标记，其他则是空白。

形如 `fg 1` 可以把后台工作号为 `1` 的任务恢复到前台继续运行。

类似的，`bg 1` 可以把工作号为 `1` 的任务恢复到后台运行。不过涉及到需要跟用户交互的命令，比如 `top` 或 `vim` 这种，不能放在后台运行，因为没意义。

如果不指定工作号，按照标记为 `+` 的任务恢复。

### 13.3 系统资源查看

#### `vmstat` 监控系统资源

格式 `vmstat [刷新延时] [刷新次数]`

比如 `vmstat 1 3` 就是每秒输出一次，一共输出三次。

```terminal
julian@gallifrey:~$ vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 2882660  81104 2935752    0    0    57    52  359  814  4  2 94  0  0
 0  0      0 2882660  81104 2935728    0    0     0     0 1101 1446  1  1 98  0  0
 0  0      0 2882660  81104 2935728    0    0     0     0 1144 1617  1  0 99  0  0
```

其中有些信息与 `top` 类似。有用的还是内存剩余和 CPU 空闲百分比（id）。

#### `dmesg` 开机时内核检测信息

直接运行 `dmesg` 就行，但是内容较多，一般结合 `grep` 搜索。

需要 root 权限。

#### `free` 查看内存使用状态

`free -m` 将结果按照 MB 大小显示。

```terminal
julian@gallifrey:~$ free -m
               total        used        free      shared  buff/cache   available
Mem:            7798        2033        2818         638        2946        4838
Swap:            975           0         975
```

缓存（cache）是从硬盘读取内容时将内存保存在高速缓存中，以提高下次读取的速度。缓冲（buffer）是在往硬盘写入内容时先将内容攒一攒，攒到一定数量再一次性写入硬盘，避免频繁写入造成的性能下降。

#### `/proc/cpuinfo`{: .filepath} 查看 CPU 信息

`cat /proc/cpuinfo` 可以查看 CPU 信息，不过因为 `/proc`{: .filepath} 这个目录时从内存挂载的，所以关机之后就没了，下次开机文件会重新生成。

CPU 有几个核就会显示几个块。

#### `uptime`

```terminal
julian@gallifrey:~$ uptime
 20:02:37 up  2:01,  2 users,  load average: 0.13, 0.12, 0.12
```

其实就是显示的 `top` 的第一行。

#### `uname`

查看系统和内核版本的信息。

`-a` 查看所有信息，`-r` 查看内核版本，`-s` 查看内核名称。

#### 判断系统位数

Linux 没有一个专门的命令查看操作系统位数，不过可以通过 `file` 命令查看任意一个二进制执行文件来判断。

```terminal
julian@gallifrey:~$ file /bin/ls
/bin/ls: ELF 64-bit LSB pie executable, x86-64, ...
```

> 上述命令结果省略了部分内容。
{: .prompt-info}

#### 查看系统发行版本

使用 `lsb_release -a`。

```terminal
julian@gallifrey:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04.3 LTS
Release:	22.04
Codename:	jammy
```

#### 查看进程使用的文件

`lsof` 命令可以查看进程调用或打开的文件信息，`-c [str]` 只列出以某字符串开头的进程打开的文件，`-u [user]` 只列出某个用户的进程打开的文件，`-p [PID]` 只列出某个进程打开的文件。

某些可能需要 root 权限。

### 13.4 系统定时任务

使用 `systemctl status cron` 查看定时服务是否开启。（一般是默认开启的）

要设置定时任务，使用 `crontab` 命令，其中 `-e` 编辑定时任务，`-r` 删除全部定时任务，`-l` 查询当前定时任务。

> 第一次编辑定时任务会让选择一个编辑器。运行 `select-editor` 可以更改。
{: .prompt-info}

其实本质上，定时任务的配置就是一个文件，每一行是一个任务，按照正确的格式编写。`-e` 就是打开编辑这个文件，`-r` 就是清空这个文件。如果只想删除某一个任务，`-e` 打开然后删除那一行就可以了。

#### 格式

`* * * * * [command]`

| 项目 | 含义 | 范围 |
|--|--|--|
| 第一个 `*` | 一小时中的第几分钟 | 0-59 |
| 第二个 `*` | 一天中的第几个小时 | 0-23 |
| 第三个 `*` | 一个月中的第几天 | 1-31 |
| 第四个 `*` | 一年中的第几个月 | 1-12 |
| 第五个 `*` | 一周当中的星期几 | 0-7 |

> 最后一个 `*` 中 0 和 7 都代表星期日。
{: .prompt-info}

| 特殊符号 | 含义 |
|--|--|
| `*` | 任意时间，<br/>比如第一个是 `*` 就表示一小时中的每一分钟都执行一次 |
| `,` | 表示不连续的时间，<br/>比如第一个位置是 `5,10,15` 表示分别在每小时的第 5 分、10 分和 15 分执行 |
| `-` | 表示连续的时间，<br/>比如最后一个位置是 `1-6` 表示在每周的周一到周六每天都执行 |
| `*/n` | 表示每隔多久执行一次，<br/>比如第一个位置是 `*/10` 表示每隔 10 分钟执行一次 |

示例

| 任务 | 含义 |
|--|--|
| `45 22 * * * [cmd]` | 每天的 22:45 执行 |
| `0 17 * * 1 [cmd]` | 每周一的 17:00 执行 |
| `0 5 1,15 * * [cmd]` | 每月的 1 号和 15号的 5:00 执行 |
| `40 4 * * 1-5 [cmd]` | 每周一到周五的每天 4:40 执行 |
| `*/10 4 * * * [cmd]` | 每天 4 点，每隔 10 分钟执行一次 |
| `0 0 1,15 * 1 [cmd]` | 每月的 1 号和 15 号 **以及** 每周一的 0:00 执行 |

最后一种写法虽然是支持的，但是建议不要把第三个位置和最后一个位置同时指定，容易晕。

## 第十四讲 日志管理

### 14.1 日志管理简介

查询一下日志服务是否开启

```terminal
julian@gallifrey:~/temp$ ps aux | grep rsyslogd
syslog       472  0.0  0.0 222400  6144 ?        Ssl  18:01   0:00 /usr/sbin/rsyslogd -n -iNONE
julian     76478  0.0  0.0   9208  2560 pts/0    S+   21:22   0:00 grep --color=auto rsyslogd
```

有第一行就代表开启了。

#### 常见日志

| 日志文件 | 说明 |
|--|--|
| `/var/log/cups/`{: .filepath} | 记录打印信息的日志 |
| `/var/log/dmesg`{: .filepath} | 记录了系统开机内核自检信息，`dmesg` 命令就是直接查看的该文件 |
| `/var/log/btmp`{: .filepath} | 记录错误登录的日志，是二进制文件，需要用 `lastb` 命令查看 |
| `/var/log/lastlog`{: .filepath} | 记录所有用户最后一次登录时间的日志，用 `lastlog` 命令查看 |
| `/var/log/wtmp`{: .filepath} | 记录所有用户的登录和注销信息，用 `last` 查看 |

### 14.2 `rsyslogd` 日志服务

`/var/log`{: .filepath} 下的日志基本都是采用的相同的日志格式，也是 `rsyslogd` 能识别的格式：

- 事件产生的时间
- 发生事件的服务器主机名
- 产生事件的服务名或程序名
- 事件的具体信息

`/etc/rsyslog.conf`{: .filepath} 文件记录了要记录什么样的日志。具体的配置格式见 `/etc/rsyslog.d/50-default.conf`{: .filepath}。

日志分等级

| 等级名称 | 说明 |
|--|--|
| debug | 一般调试信息 |
| info | 基本通知信息 |
| notice | 普通信息，但有一定重要性 |
| warning | 警告信息，但还不会影响到系统或服务运行 |
| err | 错误信息，可能会影响系统或服务运行了 |
| crit | 临界状况信息 |
| alert | 预警信息，必须立刻采取行动 |
| emerg | 紧急信息，系统已无法使用了 |

日志格式一般为 `[服务名].[日志等级] [路径]`。

例如 `authpriv.*` 表示记录 `authpriv` 服务产生的所有日志信息。`mail.err` 表示记录 `mail` 服务产生的高于或等于 err 等级的日志信息（也就是 err、crit、alert 和 emerg）。`*.=emerg` 表示记录任何服务产生的 emerg 等级的日志，其他等级不记录。`.!` 表示除了指定的等级不记录，其他等级都记录。

路径可以是日志文件的绝对路径，也可以是设备文件，比如 `/dev/lp0`{: .filepath} 是个打印机设备，记录日志时直接打印，也可以是远程主机 `@192.168.x.y:abc`，也可以是用户名（用户要在线才能收到），或者丢弃用 `~`。

当然最常用的还是日志文件绝对路径。

### 14.3 日志轮替

日志如果始终都记录在一个文件中，总有一天这个文件会大到打不开。因此涉及到日志记录，有两样事情要做，一个是分割，也就是把大文件分成小文件，另一个是轮替，也就是隔一段时间生成新的日志文件，清理旧的日志文件。

日志轮替时对于日志文件的命名有两种方式，

第一种，假设今天的日志文件叫 `logfile`{: .filepath}，到第二天的时候，把 `logfile`{: .filepath} 改为 `logfile-{昨天的日期}`{: .filepath}，比如 `logfile-20231206`{: .filepath}，然后今天的日志保存在新文件 `logfile`{: .filepath} 中。这样当天的日志文件永远叫 `logfile`{: .filepath}，之前的日志文件则都有日期标注。

第二种，同样假设今天的日志文件叫 `logfile`{: .filepath}，到第二天时，把 `logfile`{: .filepath} 改为 `logfile.1`{: .filepath}，然后今天的日志保存在新文件 `logfile`{: .filepath} 中。到第三天时，把 `logfile.1`{: .filepath} 改为 `logfile.2`{: .filepath}，把 `logfile`{: .filepath} 改为 `logfile.1`{: .filepath}，然后当天日志保存在新文件 `logfile`{: .filepath} 中。这样当天的日志文件永远叫 `logfile`{: .filepath}，之前的则有数字标记，数字越小距离当天越近。

`/etc/logrotate.conf`{: .filepath} 配置文件保存了日志轮替的相关设置。

| 参数 | 作用 |
|--|--|
| `daily` | 轮替周期为每天 |
| `weekly` | 轮替周期为每周 |
| `monthly` | 轮替周期为每月 |
| `rotate [数字]` | 保留的日志文件个数，0 表示不保留 |
| `create` | 轮替时创建新的日志文件 |
| `dateext` | 使用上述第一种日志命名方式，否则使用第二种 |
| `compress` | 压缩旧日志文件 |
| `missingok` | 如果日志不存在，忽略警告信息 |
| `notifempty` | 如果日志文件为空，则不轮替 |
| `minsize [大小]` | 轮替的最小值，达到这个值才会发生轮替，否则哪怕时间到了也不轮替 |
| `size [大小]` | 日志大于指定大小才轮替，不按照时间轮替，比如 `size 100k` |

该配置文件也包含了 `/etc/logrotate.d`{: .filepath} 下的文件，方便各服务单独配置自己的轮替规则。

各文件的格式形如

```plaintext
[日志文件绝对路径] {
    各种参数设置
}
```

`logrotate -f [配置文件名]` 可以强制发生日志轮替。

## 第十五讲 启动管理

### 15.1 启动管理

#### 15.1.1 系统运行级别

| 运行级别 | 含义 |
|--|--|
| 0 | 关机 |
| 1 | 单用户模式，类似 Windows 的安全模式，启动最少的服务，主要用于系统修复 |
| 2 | 不完全的命令行模式，不包含 NFS 服务 |
| 3 | 完全的命令行模式 |
| 4 | 系统保留 |
| 5 | 图形模式 |
| 6 | 重启 |

NFS 服务是一个 Linux 系统之间文件共享的服务，不安全，所以单独有一个级别把它去掉了。

单用户模式虽然启动的服务少，但还是从硬盘启动的，有些错误还是修复不了，如果使用光盘启动，理论上能修复的错误更多一些。

`runlevel` 显示当前运行级别。`init [级别]` 可以切换运行级别。

参考 [Where is the inittab file?](https://askubuntu.com/questions/34308/where-is-the-inittab-file)

> **Back in the days** the "System-V" init service was used in Ubuntu, and it used the /etc/inittab file.  
> **Some time ago** (around 2006) the "Upstart" init service replaced SysV. During these days you could follow the top answer and use man inittab to get info on this change.  
> **At the time of writing** (e.g. for Ubuntu 16.04) the "systemd" boot process is in use and there is no reference left to "inittab" (e.g. if you do apropos inittab you'll probably not find anything). Instead you could do man runlevel to get similar information.  
> **Bottom line**: the /etc/inittab file is nowhere, likely because you use a newer version for Ubuntu that has a different init service, e.g. systemd.

> `systemctl get-default` 可以查看当前的默认运行级别，`systemctl set-default` 可以修改。
{: .prompt-tip}

#### 15.1.2 启动过程

略。

### 15.2 启动引导程序 `grub`

Ubuntu 22.04.3 的启动配置貌似在 `/etc/default/grub`{: .filepath} 文件中。

> 其他内容略。

服务器不同于个人机，频繁更新并不一定合适。一方面新的软件可能不稳定，另一方面更新后可能有些地方需要重新配置，也会耽误时间。

> grub 加密？

### 15.3 系统修复模式

进入单用户模式，可以修改 root 密码，修改系统默认运行级别。

实在不行，就用光盘修复模式。

## 第十六讲 备份和恢复

### 16.1 备份概述

完全备份：每次都备份完整数据，备份的数据量大，但是恢复时方便快捷。

增量备份：每次只备份与上一次相比新增加的数据，备份时数据量小，但恢复时需要逐步恢复，比较麻烦。

差异备份：每次都是跟第一次的完整备份相比，备份新增的数据。这算是一个完全备份和增量备份的折中策略。

### 16.2 `dump` 和 `restore` 命令

#### `dump`

在 Ubuntu 22.04.3 中，`dump` 默认没有安装，使用 `sudo apt install dump` 安装。

选项有 `-0` 完全备份，`-1` 第一次增量备份，`-9` 第九次增量备份。`-f` 指定备份后的文件名。`-u` 备份后把日志写在 `/var/lib/dumpdates`{: .filepath} 文件中。`-j` 按照 bz2 格式压缩。

`-W` 选项可以看到不同分区的备份信息。

如果按照分区备份，可以使用 `-u` 记录在文件，如果不按照分区备份，只是备份分区中的某一个子目录，则不能用 `-u` 记录日志，且只能使用 `-0` 完全备份。

```terminal
root@gallifrey:~/temp# dump -0uj -f boot.bak.bz2 /boot/
  DUMP: You can't update the dumpdates file when dumping a subdirectory
  DUMP: The ENTIRE dump is aborted.

root@gallifrey:~/temp# dump -0j -f boot.bak.bz2 /boot
  DUMP: Date of this level 0 dump: Fri Dec  8 10:40:53 2023
  DUMP: Dumping /dev/nvme0n1p4 (/ (dir boot)) to boot.bak.bz2
  DUMP: Label: none
  DUMP: Writing 10 Kilobyte records
  DUMP: Compressing output at transformation level 2 (bzlib)
  DUMP: mapping (Pass I) [regular files]
  DUMP: mapping (Pass II) [directories]
  DUMP: estimated 196288 blocks.
  DUMP: Volume 1 started with block 1 at: Fri Dec  8 10:40:53 2023
  DUMP: dumping (Pass III) [directories]
  DUMP: dumping (Pass IV) [regular files]
  DUMP: Closing boot.bak.bz2
  DUMP: Volume 1 completed at: Fri Dec  8 10:41:15 2023
  DUMP: Volume 1 took 0:00:22
  DUMP: Volume 1 transfer rate: 8016 kB/s
  DUMP: Volume 1 196260kB uncompressed, 176367kB compressed, 1.113:1
  DUMP: 196260 blocks (191.66MB) on 1 volume(s)
  DUMP: finished in 22 seconds, throughput 8920 kBytes/sec
  DUMP: Date of this level 0 dump: Fri Dec  8 10:40:53 2023
  DUMP: Date this dump completed:  Fri Dec  8 10:41:15 2023
  DUMP: Average transfer rate: 8016 kB/s
  DUMP: Wrote 196260kB uncompressed, 176367kB compressed, 1.113:1
  DUMP: DUMP IS DONE

root@gallifrey:~/temp# ls -lh
total 173M
-rw-r--r-- 1 root root 173M Dec  8 10:41 boot.bak.bz2

root@gallifrey:~/temp# dump -W
Last dump(s) done (Dump '>' file systems):
  /dev/nvme0n1p4	(     /) Last dump: never

root@gallifrey:~/temp# cat /var/lib/dumpdates 

root@gallifrey:~/temp# dump -1j -f boot.bak1.bz2 /boot
  DUMP: Only level 0 dumps are allowed on a subdirectory
  DUMP: The ENTIRE dump is aborted.
```

#### `restore`

该命令常用的模式有四种，不能混用

模式：

`-C` 比较备份数据和实际数据的变化（貌似比较分区下的子目录时会有些问题）

`-i` 交互模式，手动选择恢复的文件

`-t` 查看模式，查看备份文件中有哪些数据

`-r` 还原模式，用于数据还原（实际上就是把备份文件解压缩到当前目录，然后手动拷贝需要的）

选项：

`-f` 指定备份文件名
