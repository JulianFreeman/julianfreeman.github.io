---
title: Nginx 学习笔记
date: 2024-01-05 22:35:00 -0500
categories: [学习笔记, Nginx]
tags: [linux, nginx]
description: 燕十八 nginx 视频教程的笔记
---

## 01 nginx 编译安装

官方下载源码包：https://nginx.org/download/nginx-1.24.0.tar.gz

解压进入，执行

```shell
./configure --prefix=/usr/local/nginx
```
{: .nolineno}

如果遇到 PCRE 没有安装的，执行

```shell
sudo apt install libpcre3-dev
```
{: .nolineno}

> 一般 libpcre3 包是默认安装的，如果没有，也一并安装
{: .prompt-tip}

如果遇到 zlib 没有安装的，执行

```shell
sudo apt install zlib1g-dev
```
{: .nolineno}

> 一般 zlib1g 包是默认安装的，如果没有，也一并安装
{: .prompt-tip}

如果正常检查通过了，执行

```shell
make
sudo make install
```
{: .nolineno}

没有问题的话，nginx 会安装在 `/usr/local/nginx`{: .filepath}{: .nolineno} 下。

使用源码安装的 nginx 默认不会自动启动，需要运行

```shell
sudo /usr/local/nginx/sbin/nginx
```
{: .nolineno}

手动启动。

### 补充一点关于服务的内容

#### systemd 管理服务

```shell
systemctl start foo       # 启动服务
systemctl stop foo        # 停止服务
systemctl restart foo     # 重启服务
systemctl status foo      # 查看服务状态
systemctl enable foo      # 设置服务开机自启动
systemctl disable foo     # 设置服务不要开机自启动
systemctl is-enabled foo  # 查看服务是否是开机自启动
```
{: .nolineno}

#### 服务分类

使用二进制包安装的软件，其程序文件分散在系统的各个目录中，比如 `/bin`{: .filepath}、`/var`{: .filepath} 等。由二进制包安装的服务也可以由系统识别管理，通常是在 `/lib/systemd/system`{: .filepath} 目录下以 `.service` 结尾的文件。这些服务可以使用 `systemctl` 管理。

使用源码包安装的软件，其程序文件不会分散在系统中，而是集中位于用户指定的位置（就像 Windows 里安装的软件一般在 `C:\Program Files\`{: .filepath} 下），这个位置一般默认是 `/usr/local`{: .filepath}，当然用户可以自己指定任意位置。源码包安装的服务不能被系统识别，默认不受 `systemctl` 管理，当然也不会自动启动了。

## 02 nginx 信号量

nginx 有一个文件用来保存当前 nginx 主进程号，如果是源码包安装的，默认是 `/usr/local/nginx/logs/nginx.pid`{: .filepath}，如果是二进制包安装的，是 `/var/run/nginx.pid`{: .filepath}。

该文件的位置可以通过 `nginx.conf`{: .filepath} 文件的 `pid` 标记修改。源码包：`/usr/local/nginx/conf/nginx.conf`{: .filepath}；二进制包：`/etc/nginx/nginx.conf`{: .filepath}。

关于 nginx 的信号量控制，其官方文档：https://nginx.org/en/docs/control.html

> 以下默认在用二进制包安装的环境下进行。
{: .prompt-tip}

查看 nginx 是否在运行

```terminal
julian@basoom:/etc/nginx$ ps aux | grep nginx
root         758  0.0  0.0  10548   988 ?        Ss   11:44   0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx        759  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        761  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        762  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        763  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        764  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        765  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        766  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
nginx        767  0.0  0.0  11304  3868 ?        S    11:44   0:00 nginx: worker process
julian      2770  0.0  0.0   6608  2292 pts/0    S+   22:57   0:00 grep --color=auto nginx
```

可以看到 nginx 有一个主进程和好几个工作进程。主进程不直接响应网页的请求，而是用来管理工作进程的；工作进程用来响应网页请求。

`kill` 命令的 man 手册是这样解释自己的：`kill - send a signal to a process`，所以 `kill` 不只是乱杀一通，它实际上的功能是给进程发送信号，这个信号老多，其中恰好包括强制终止进程的信号。`kill -l` 查看所有信号。

通过信号量控制 nginx，也就是用 `kill` 命令控制 nginx。

终止 nginx：

```terminal
julian@basoom:/etc/nginx$ sudo kill -INT 758

julian@basoom:/etc/nginx$ ps aux | grep nginx
julian      2857  0.0  0.0   6476  2424 pts/0    S+   23:04   0:00 grep --color=auto nginx
```

重新开启用 `sudo systemctl start nginx`。

`INT` 直接终止进程，`QUIT` 会等请求结束后再终止进程（优雅地终止进程）。

`HUP` 信号会用新的配置文件启动一个新的工作进程，然后优雅地关闭旧的工作进程。也就是说我们修改了配置文件之后不用重启整个 nginx 服务，只需要用 `HUP` 信号量就可以了。

比如我们修改一下配置文件 `/etc/nginx/conf.d/default.conf`{: .filepath}，将

```conf
location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
}
```

改为

```conf
location / {
    root   /usr/share/nginx/html;
    index  ab.html index.html index.htm;
}
```

然后在 `/usr/share/nginx/html/`{: .filepath} 目录下创建 `ab.html`{: .filepath} 文件写入一些内容。

此时刷新网站还没有任何变化，执行 `sudo kill -HUP 2871` 后再刷新，就会看到 `ab.html`{: .filepath} 文件的内容了。同时执行 `ps aux | grep nginx`，会看到 nginx 的所有工作进程号都跟之前不一样了，但是主进程的还一样。这也就是说，所有工作进程都重启了，但是主进程没有。

`USR1` 信号可以重开日志文件，也就是日志轮替。

默认 nginx 会把日志写在 `/var/log/nginx/access.log`{: .filepath} 中，每刷新一次网站都会加一条记录。

当需要日志轮替时，先把 `access.log`{: .filepath} 重命名，然后创建一个新的 `access.log`{: .filepath} 文件，然后告诉 nginx 日志文件变了。如果不通知 nginx 的话，仅仅是改日志文件的名字并不会让 nginx 察觉到需要改变存储日志的位置，因为其底层通过 inode 号来识别文件，仅仅修改文件名并不会更改文件的 inode 号，因此哪怕改名了，nginx 依然是往这个 inode 号写日志。

通知 nginx 的方式即 `sudo kill -USR1 <pid>`。

## 03 nginx 虚拟主机配置

nginx 本身也有一些命令，与信号控制功能类似，比如 `nginx -s reload` 重读配置文件，`nginx -s reopen` 日志轮替，`nginx -s stop` 终止服务等。

`nginx -t` 可以测试配置文件语法是否正确，避免在修改配置文件后改错导致服务起不来。这个需要管理员权限。

nginx 配置文件的相关参数：

`worker_processes`：二进制包安装的 nginx 该值为 `auto`，源码包安装的 nginx 该值为 `1`；表示工作进程的数量。一般为 CPU 数 * 核数。

`events`：配置 nginx 连接的属性。

`events/worker_connections`：每个工作进程允许的最大连接数量。默认是 `1024`，可以改大，但是太大也没意义。

`http`：配置 http 服务器的部分。其下又有好几个 `server` 的部分，实际上每个 `server` 就是一个虚拟主机。在二进制包安装的 nginx 下，每一个 `server` 被放置在 `/etc/nginx/conf.d/*.conf`{: .filepath} 文件中，在源码包安装的 nginx 下则直接在 `http` 字段下。

一个简单的 `server` 需要的内容不多，下面简单创建一个：

在 `/etc/nginx/conf.d/`{: .filepath} 目录下创建一个 `jhome.conf`{: .filepath} 文件，然后写入如下内容

```conf
server {
    listen 80;
    server_name jhome.me;

    location / {
        root /usr/share/nginx/jhome;
        index index.html;
    }
}
```

这是一个最简单的可用的 `server` 了：监听哪个端口，使用哪个域名（当然这里可以是纯 IP 地址），网站文件的根目录在哪里。

然后在 `/usr/share/nginx/jhome`{: .filepath} 下创建 `index.html`{: .filepath} 文件，并写入一些内容。

然后运行 `sudo nginx -s reload` 重读配置文件。

此时访问 `jhome.me` 肯定不行，因为浏览器不认识这个域名，我们需要在 hosts 文件中指定该域名的 IP 地址为服务器的地址。

添加完成后再访问 `jhome.me`，就能看到刚才 `/usr/share/nginx/jhome/index.html`{: .filepath} 文件中的内容了。

当然这个配置里没有什么是必须的，监听端口可以任意指定，域名也可以随便写，网站根目录也可以在任意目录，只不过有些是约定俗成的。

在浏览器里输入一个网址时如果不指定端口，默认就是 80 端口，但是也可以指定。比如如果上述 `server` 的监听端口是 `2023`，那么访问时就应该写 `jhome.me:2023`。

## 04 nginx 日志管理

继续来看 `/etc/nginx/nginx.conf`{: .filepath} 文件的 `http` 字段，有如下部分

```conf
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
access_log  /var/log/nginx/access.log  main;
```

这里定义了服务器的日志文件路径及格式。

第一部分 `log_format` 定义了一种日志格式，即最后那串很长的字符串，然后给这种格式命名为 `main`，在后面定义日志文件路径的同时指定使用 `main` 格式。

我们也可以在每一个 `server` 里单独指定日志文件的路径及格式。

比如修改前面的 `server` 为

```conf
server {
    listen 2023;
    server_name jhome.me;

    location / {
        root /usr/share/nginx/jhome;
        index index.html;
    }

    access_log /var/log/nginx/jhome.me.access.log main;
}
```

此时访问 `jhome.me:2023`，日志文件就会以 `main` 的格式写入到 `/var/log/nginx/jhome.me.access.log`{: .filepath} 中了。

## 05 nginx 定时任务完成日志切割

日志切割就是在信号量那里说的，重命名日志文件后创建新的日志文件，然后通知 nginx 进行日志轮替，只不过这个步骤换成用脚本完成，然后设置定时任务。具体脚本这里就不写了。

## 06 Location 详解之精准匹配

> 视频里讲的有点乱，这里按照自己理解的思路来。
{: .prompt-info}

首先，一个有路径的网址结尾带不带 `/` 是不一样的。

这里说有路径是因为 `http://jhome.me` 和 `http:/jhome.me/` 是一样的，都是找根路径，但是 `http://jhome.me/foo` 和 `http://jhome.me/foo/` 就不一样了，前者找的是根路径下的 `foo` 文件，而后者找的是根路径下的 `foo/` 目录。

访问一个网页，归根结底其实还是访问网站服务器上的一个文件，如果网址路径是以目录结尾的，那么访问的是什么呢？

这就要说到前面 `server` 字段中的 `location` 中的 `index` 了。[官网文档](https://nginx.org/en/docs/http/ngx_http_index_module.html)是这么说的：

> `ngx_http_index_module` 模块处理结尾是 `/` 的连接请求。这样的请求也能被其他模块处理，比如 `ngx_http_autoindex_module` 和 `ngx_http_random_index_module`。
> 
> **样例配置**
> 
> ```conf
> location / {
>     index index.$geo.html index.html;
> }
> ```
> 
> **声明**
> 
> ```conf
> Syntax:  index file ...;
> Default: index index.html;
> Context: http, server, location
> ```
> 
> `index` 关键字定义的文件会被用作索引。文件名字可以包含变量。文件会按照被指定的顺序检索。列表的最后一个元素可以是一个绝对路径的文件，比如：
> 
> ```conf
> index index.$geo.html index.0.html /index.html;
> ```
> 
> 要注意，使用索引文件会导致一个内部的链接跳转，请求可能就会被不同的 `location` 处理。比如如果使用下面的配置：
> 
> ```conf
> location = / {
>     index index.html;
> }
> 
> location / {
>     ...
> }
> ```
> 
> 访问 `/` 的请求实际上会跳转为 `/index.html` 然后被第二个 `location` 处理。

这里目前我们要理解两件事：

1. `index` 是用来处理网址结尾是 `/` 的请求的。也就是说，如果网址结尾不是 `/`，那么 `index` 就没啥用。
2. 使用 `index` 会导致链接跳转。比如官网上最后的例子，当我们用 `/` 请求时，先精准匹配到了第一个 `location` 上，但是因为这个请求不是一个具体的文件，所以会通过 `index` 进行一次跳转。根据第一个 `location` 的设置，这个跳转要访问的请求是 `/index.html`，这个路径就无法精准匹配第一个 `location` 了，因此匹配到了第二个 `location` 上，所以就会按照第二个 `location` 设置的情况进行处理。

现在，我们要来说一下匹配了。

`location` 的格式是 `location [modifier] pattern`，其中 `pattern` 就是要匹配的内容，也就是一个字符串，可选的 `modifier` 就是匹配模式。

根据匹配的优先级，`location` 的匹配模式有如下四种情况：

- 精准匹配，修饰符是 `=`，表示路径的格式从头到尾必须一模一样，差一个字符都不行。比如 `location = /foo/` 可以匹配 `http://jhome.me/foo/` 但不能匹配 `http://jhome.me/foo` 或者 `http://jhome.me/foo/index.html` 等。
- 优先级高的常规匹配，修饰符是 `^~`，跟常规匹配规则一致，只不过常规匹配的优先级比正则匹配低，加上这个修饰符可以把优先级提到正则匹配前面。
- 正则匹配，修饰符是 `~` 或者 `~*`，把匹配字符串当作正则表达式进行匹配。测试发现这个正则不是完全匹配，而是包含就行。比如 `location ~ /greet[0-9]` 可以匹配到 `http://jhome.me/greet123`，因为 `greet[0-9]` 匹配到了 `greet1`。如果想要防止这种情况而只想匹配一个数字，需要写作 `/greet[0-9]$`。`~` 是区分大小写的正则匹配，`~*` 是不区分大小写的正则匹配。
- 常规匹配，无修饰符。会从路径开头进行匹配，包含就行，不用完全一致。比如 `location /foo` 能匹配到 `http://jhome.me/foo`、`http://jhome.me/foo/`、`http://jhome.me/foo/index.html` 等。所以说，`location /` 可以匹配到所有路径，因为所有路径都肯定是从 `/` 开头的。

道理说的差不多了，下面来点实战了。

精准匹配一个文件：

```conf
location = /foo.html {
    root /var/www/html;
}
```

如上所述，直接访问文件时 `index` 没啥用，所以这里不写了。此时如果访问 `http://jhome.me/foo.html`，就能精准匹配到这个 `location`，因为是文件，所以不会发生跳转，而是根据设置的 `root` 直接去找 `/var/www/html/foo.html`{: .filepath} 这个文件，找到了就显示，找不到就 404。

精准匹配一个目录：

```conf
location = /foo/ {
    root /var/www/html;
    index bar.html;
}

location / {
    root /usr/share/nginx/html;
    index index.html;
}
```

这时访问 `http://jhome.me/foo/` 就发生有意思的事情了。首先这个请求精准匹配到了第一个 `location`，但因为这个请求是一个目录 `/foo/`，因此会进行一次跳转。根据第一个 `location` 的设置，这个跳转的请求是 `/foo/bar.html`，这一次匹配就无法精准匹配第一个 `location` 了，因此就匹配到了万能的 `location /`。因为这次的请求是一个文件 `/foo/bar.html`，因此不会再发生跳转，而是根据匹配到的 `location` 的 `root` 设置寻找文件，也就是 `/usr/share/nginx/html/foo/bar.html`{: .filepath}，找到则正常访问，找不到 404。

所以现在明白了，按照这个配置访问 `/foo/` 的时候，网站文件并不是在 `/var/www/html`{: .filepath} 中找的，而是在 `/usr/share/nginx/html`{: .filepath} 中。

常规匹配一个文件（？）：

```conf
location /foo {
    root /var/www/html;
    index bar.html;
}

location / {
    root /usr/share/nginx/html;
    index index.html;
}
```

这时访问 `http://jhome.me/foo`，首先常规匹配到第一个 `location`，然后因为请求是一个文件（虽然没有后缀名，但也不是 `/` 结尾的，因此是一个文件），所以不会发生跳转，根据第一个 `location` 的设置，寻找 `/var/www/html/foo`{: .filepath} 文件，找不到就 404。

此时注意，其实按照常规匹配，第一个和第二个 `location` 都能匹配上，两个优先级也都一样，但是匹配都有点贪婪的意思，也就是按照最长匹配，因此两者相比第一个优先。哪怕调换位置也一样。

这时如果访问 `http://jhome.me/foo/`，其实还是按照常规匹配第一个，但因为请求是一个目录，因此按照第一个的设置发生跳转，跳转请求的是 `/foo/bar.html`，这个请求还是常规匹配第一个，而且因为这次请求是一个文件，所以不再发生跳转，而是根据第一个的设置寻找文件 `/var/www/html/foo/bar.html`{: .filepath}，找不到就 404。

到目前为止一切看起来都很美好，但是还是有些问题的。比如我们来搞点事情，

```conf
location = /foo.html {
    root /var/www/html;
    index bar.html;
}

location / {
    root /usr/share/nginx/html;
    index index.html;
}
```

配置文件如上，如果访问 `http://jhome.me/foo.html`，很好，可以精准匹配第一个，然后寻找文件 `/var/www/html/foo.html`{: .filepath}。可是如果此时找到的不是一个文件而是一个目录会怎么样？

```terminal
julian@basoom:/var/www/html$ ls -dl foo.html
drwxr-xr-x 2 root root 4096 Dec 26 20:39 foo.html
```

会报 404 吗？测试发现，可能会，但原因并非没找到这个文件。推测来说，如果此时 nginx 发现找到的不是一个文件而是一个同名目录，那么这个请求就会被转为 `/foo.html/` 再次匹配。此时的匹配只能匹配到第二个 `location`，且请求是一个目录，因此会进行一次跳转，这个跳转请求为 `/foo.html/index.html`，然后再次匹配到第二个 `location`，不再跳转，寻找文件 `/usr/share/nginx/html/foo.html/index.html`{: .filepath}。

有意思，下面又要开始紧张刺激的分类讨论了。

---

`location /foo.html` 且实际存在 `foo.html`{: .filepath} 文件：

```conf
location /foo.html {
    root /var/www/html;
    index bar.html;
}

location / {
    root /usr/share/nginx/jhome;
    index index.html;
}
```

```terminal
julian@basoom:/var/www/html$ ls -l foo.html
-rw-r--r-- 1 root root 136 Dec 26 20:32 foo.html
```

访问 `/foo.html` 时：

比较简单，常规最长匹配到第一个 `location`，然后找到文件 `/var/www/html/foo.html`{: .filepath}，完。

访问 `/foo.html/` 时：

先常规匹配到第一个，因为请求是一个目录（以 `/` 结尾），所以发生跳转，请求 `/foo.html/bar.html`，再次匹配还是匹配到第一个，这次是文件，所以不跳转，寻找 `/var/www/html/foo.html/bar.html`{: .filepath} 文件。找不到会 404，但是能从错误日志里看到是想找这个文件的。

---

`location /foo.html` 且实际存在 `foo.html`{: .filepath} 目录：

```conf
location /foo.html {
    root /var/www/html;
    index bar.html;
}

location / {
    root /usr/share/nginx/jhome;
    index index.html;
}
```

```terminal
julian@basoom:/var/www/html$ ls -dl foo.html
drwxr-xr-x 2 root root 4096 Dec 26 21:24 foo.html
```

访问 `/foo.html` 时：

跟前面讨论的情况一样，本来匹配到第一个，想找 `/var/www/html/foo.html`{: .filepath} 这个文件，结果找到发现一看是个目录，转而将请求改为 `/foo.html/` 匹配，还是第一个，再跳转请求 `/foo.html/bar.html`，还是匹配第一个，因此寻找 `/var/www/html/foo.html/bar.html`{: .filepath} 文件。

从地址栏也能看到，虽然输入的是 `http://jhome.me/foo.html`，但是访问结束后显示的是 `http://jhome.me/foo.html/`。

访问 `/foo.html/` 时：

这就跟访问 `/foo.html` 被跳转为访问 `/foo.html/` 一样了。还是寻找 `/var/www/html/foo.html/bar.html`{: .filepath} 文件。

---

到此为止可以得出的结论是，如果本身是个文件，访问同名目录不会导致转换，如果本身是个目录，访问同名文件会转换为访问该目录。

这其实也合理，如果一个文件名后缀不加 `/`，那也不知道它到底是个文件还是目录，如果加了 `/` 那就明确是目录了。

其实这里也可以理解为，地址最后有 `/` 则明确是访问目录，找到就过，找不到就不过，哪怕有个同名文件也不行，因为地址末尾明确规定了 `/`，即要找目录。但是如果地址末尾没有 `/`，此时就处于不确定状态，到底找文件还是目录，根据找的结果，找到文件算文件，找到目录算目录。因为同名文件和目录不可能同时存在。

> 挺耳熟。

---

`location /foo.html/` 且实际存在 `foo.html`{: .filepath} 目录：

```conf
location /foo.html/ {
    root /var/www/html;
    index bar.html;
}

location / {
    root /usr/share/nginx/jhome;
    index index.html;
}
```

```terminal
julian@basoom:/var/www/html$ ls -dl foo.html
drwxr-xr-x 2 root root 4096 Dec 26 21:24 foo.html
```

访问 `/foo.html` 时：

无法匹配第一个，因为 `/foo.html/` 并不包含在 `/foo.html` 中，所以只能匹配第二个，然后去 `/usr/share/nginx/jhome`{: .filepath} 下寻找 `foo.html`，如果找到文件，直接显示。

如果找到目录，这里就有意思了。如果 `/usr/share/nginx/jhome/foo.html`{: .filepath} 是一个目录，哪么请求会被转换为 `/foo.html/`，此时突然就被第一个 `location` 给匹配去了，因此最后寻找的文件变成了 `/var/www/html/foo.html/bar.html`{: .filepath}。

防不胜防啊，所以同一个 `server` 下还是用同一个 `root` 比较好。

访问 `/foo.html/` 时：

跟上面的第二种情况一样，被第一个匹配去，然后寻找 `/var/www/html/foo.html/bar.html`{: .filepath}。

---

`location /foo.html/` 且实际存在 `foo.html`{: .filepath} 文件：

```conf
location /foo.html/ {
    root /var/www/html;
    index bar.html;
}

location / {
    root /usr/share/nginx/jhome;
    index index.html;
}
```

```terminal
julian@basoom:/var/www/html$ ls -l foo.html
-rw-r--r-- 1 root root 141 Dec 26 21:18 foo.html
```

访问 `/foo.html` 时：

跟上面的情况一样，无法匹配第一个，于是匹配第二个，如果 `/usr/share/nginx/jhome/foo.html`{: .filepath} 是一个文件，直接显示文件，如果是个目录，会转换请求 `/foo.html/`，于是又被第一个匹配了去，转而寻找 `/var/www/html/foo.html/bar.html`{: .filepath}。

访问 `/foo.html/` 时：

匹配第一个，寻找 `/var/www/html/foo.html/bar.html`{: .filepath}。

---

所以总结来说，如果要匹配目录，模式应该写作 `/foo` 而不是 `/foo/`。

## 0x nginx 如何确定哪个虚拟主机处理请求

这里再补充一点关于 `listen` 和 `server_name` 的内容，参考自 https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms

`listen` 定义的是该 `server` 要响应的地址和端口。

- 如果不写 `listen`，默认监听 `0.0.0.0:80`。
- 如果只写地址，比如 `111.222.33.444`，则监听 `111.222.33.444:80`。
- 如果只写端口，比如 `888`，则监听 `0.0.0.0:888`。

对于多个 `server`，当一个请求发过来的时候，由谁来响应这个请求，就是根据谁的 `listen` 最匹配这个请求的地址和端口。

所以说，如果一个 `server` 的监听是 `0.0.0.0`，也就是所有地址，那么其匹配度会比一个准确的 IP 匹配要低。

如果最后有两个 `server` 的匹配度等同，难分高下，这时会再进行比较 `server_name`。这里很重要，只有两个 `server` 的 `listen` 的匹配程度相同时，才会考虑 `server_name`。比如有如下设置

```conf
server {
    listen 192.168.1.10;
    ...
}

server {
    listen 80;
    server_name example.com;
    ...
}
```

这里第一个实际监听的是 `192.168.1.10:80`，第二个实际监听的是 `0.0.0.0:80`。

如果域名 `example.com` 实际指向的就是 `192.168.1.10`，那么当访问 `example.com` 的时候，nginx 实际上在对比的是这个域名背后的 IP 地址和端口，也就是 `192.168.1.10:80`，到底与哪一个 `server` 更匹配。显而易见，跟第一个更匹配。这里没有匹配度相同难分高下的情况，而是第一个 `server` 完胜，因此根本用不到 `server_name`，直接由第一个 `server` 响应。

## 07 Location 之正则匹配

关于匹配的优先级，[官网](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)是这么说的：

> `location` 要么定义为一个前缀字符串，要么定义为一个正则表达式。正则表达式用 `~*` 修饰符（不区分大小写的匹配），或者 `~` 修饰符（区分大小写的匹配）。要找到匹配一个请求的 `location`，nginx 首先检查使用前缀字符串定义的 `location`，然后找到能匹配最长的那个记下来。借着检查正则表达式，按照定义的先后顺序检查，找到第一个正则匹配后就会立即停止匹配，然后使用这个 `location` 块的配置。如果没有正则能匹配的，就使用之前记下来的那个最长的前缀字符串。
> 
> 如果找到的那个最长的前缀字符串有 `^~` 修饰符，那么就不会检查正则表达式。
> 
> 同样，如果能找到一个用 `=` 修饰的完全匹配，那么匹配也会立刻中止。

这里说前缀字符串，可能是指匹配总是从第一个字符开始吧。

可以理解为，先不管所有有 `~*` 和 `~` 的块，按照常规匹配匹配其他的块（即从第一个字符开始逐个匹配，而且块之间没有什么先后顺序），如果能找到完全一致的匹配，且该匹配有 `=` 或者 `^~` 修饰符，立刻中止匹配并使用这个块的设置。如果找不到完全一致的匹配，则找能匹配且前缀字符串最长的那个块，如果这个块有 `^~` 修饰符，则立刻中止匹配并使用这个块的设置。如果没有这个修饰符，则暂时记录下这个块，然后去按定义的先后顺序匹配带有 `~*` 或者 `~` 修饰符的正则表达式，如果有能匹配的，立刻停止匹配并使用这个块的设置，如果检查完了也没有能匹配的正则表达式，则使用之前记录的那个最长的常规匹配的块的设置。

所有这个跟上一节说的优先级还是有些区别的。

## 09 nginx rewrite 语法详解

> 笔记中断。
{: .prompt-danger}
