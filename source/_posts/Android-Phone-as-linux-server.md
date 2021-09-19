---
title: Android手机作为linux服务器
date: 2019-11-16 12:59:00
updated: 2019-11-16 12:59:00
author: Jsharkc
tags: 
  - Termux
  - ssh
  - Android
  - linux
  - 服务器
---

有个闲置的「红米Note4」，回收也值不了几个钱，就想着是否能再利用一下，网上查了查，用 Termux 可以把 Android 手机当成 linux 服务器用，于是就有了接下来的部分了。

## 下载 Termux

[Termux 下载地址](https://www.coolapk.com/apk/com.termux)

下载之后安装即可，不 root 也可以用，但是很多目录没权限访问，所以能 root 还是 root 一下。

## 配置 Termux

Termux 自带 apt 包管理器，进行更新，安装 ssh 和用户管理模块

```shell
apt update
apt upgrade

apt install openssh
pkg install termux-auth
```

查看用户名，ip，设置密码

```
whoami
# 结果为：u0_a150，你的可能不一样，用自己的
ifconfig
# 找到 inet addr，我的是 192.168.0.104，也是用你自己的
passwd
# 这个是设置密码
sshd -p 9999
# 让 ssh 监听 9999 端口
```

设置好后，用电脑登录

```
ssh u0_a150@192.168.0.104 -p 9999
# 回车，输入密码就行了
```

之后通过 apt、pkg 安装 git、golang 等，就成服务器了，还可以通过 `termux-setup-storage` 插件把手机目录挂载到 `/data/data/com.termux/files/home/storage/shared`目录下，之后就可以随意操作了。用法是在命令行输入以下命令即可：

```
termux-setup-storage
```

当然如果你有公网服务器，还可以通过 frp 内网穿透，就可以在公网访问你的 Android 服务器了

## 利用 frp 内网穿透

安装 `Golang`

```shell
pkg install golang
```

[frp 下载地址](https://github.com/fatedier/frp/releases)

Android 下载以arm结尾的，服务器端根据自己的服务器选。

写个 Hello world http 服务

```go
package main

import (
        "fmt"
        "net/http"
)

func main() {
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
                fmt.Fprintf(w, "Hello World")
        })

        http.ListenAndServe(":8085", nil)
}
```

#### 配置 frp 服务端

```
# frps.ini
[common]
bind_port = 7001       # frp 服务绑定的端口
vhost_http_port = 8082 # 外网可访问的 web 服务端口
```

#### 配置 frp 客户端

```
# frpc.ini
[common]
server_addr = xx.xx.xx.xx  # 你的服务器地址
server_port = 7001         # 服务器绑定的端口，与服务端配置 bind_port 一致

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 9999          # 本地 ssh 监听端口
remote_port = 6000         # 远程连接用的端口

[web]
type = http
local_port = 8085          # 本地 web 服务监听端口
custom_domains = xx.xx.xx  # 服务器域名，有域名写域名，没域名写 ip
```

#### 运行 frp

**服务端**

```
./frps -c ./frps.ini
```

**客户端**

```
./frpc -c ./frpc.ini
```

**测试**

```
curl http://xx.xx.xx:8082
# 可以看到 Hello world
```

#### ssh 连接

```
ssh u0_a150@xx.xx.xx.xx -oPort=6000
```

格式：ssh  [Android用户名]@[服务器地址] -oPort=[frpc.ini 中 ssh 下 remote_port]

## 安装发行版 linux

可以再 Termux 上安装发行版 linux，包括 fedora、debian、alpine、aosc、arch、ubuntu、centos。

#### 安装 atilo

```
echo "deb [trusted=yes] https://yadominjinta.github.io/files/ termux extras" >> $PREFIX/etc/apt/sources.list
pkg in atilo-cn
```



#### 安装系统

```
atilo install centos
```

#### 删除系统

```
atilo remove centos
```

