---
title: Mac 使用 V2ray 科学上网
date: 2018-04-07 21:42:23
tags: Golang
---
# Mac 使用 V2ray 科学上网

## Install v2ray-core

step 1: Add official tap

```Shell
brew tap v2ray/v2ray
```

step 2: Install v2ray-core:

```shell
brew install v2ray-core
```

## Update v2ray-core

step 1: update tap

```shell
brew update
```

step 2: update v2ray-core

```Shell
brew upgrade v2ray-core
```

## 使用

直接在命令行上输入 `v2ray` 就可以运行 v2ray-core。（配置文件在当前目录则不用写参数，直接 v2ray）

默认配置文件位于：/usr/local/etc/config.json

编辑默认配置文件：

```
vim /usr/local/etc/config.json
```

配置文件也可以从 `https://free-ss.site/` 下载：

![](/img/src/free-ss.png)

config.json 添加 http 代理：

```Json
  "inboundDetour": [{
    "protocol": "http",
    "port": 1081,
    "settings": {
      "udp": true
    }
  }],
```

.zshrc 添加

```
# proxy
alias proxy='export https_proxy=http://127.0.0.1:1081;export http_proxy=https://127.0.0.1:1081;export socks5_proxy=socks5://127.0.0.1:1080'
alias unproxy='unset https_proxy http_proxy socks5_proxy'
```

然后配置一下 chrome 插件 Proxy SwitchyOmega

![](/img/src/switchOmega.png)

OK，现在你可以科学上网了！