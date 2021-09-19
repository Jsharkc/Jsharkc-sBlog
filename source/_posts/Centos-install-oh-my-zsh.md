---
title: Centos 安装 oh-my-zsh
date: 2019-11-17 14:29:38
tags:
  - centos
  - oh-my-zsh
---

## 查看、安装 zsh

查看是否安装了 zsh

```
# 方法一：
chsh -l
# 方法二：
cat /etc/shells
# 可能结果：
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin
/bin/zsh
# 如果有 /bin/zsh 代表已经安装，反之则没有
```

安装 zsh

```shell
yum install -y zsh
```

切换 shell 为 zsh

```
chsh -s /bin/zsh
```

## 安装 oh-my-zsh

安装需要 `git`，没有安装需要先安装：

```shell
yum install -y git
```

1、可以通过别人已经写好的脚本安装，用 `curl` 或者 `wget` 下载脚本来安装：

* 通过 `curl`

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

* 通过 `wget`

```shell
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

2、也可以自己用 `git` 安装

```shell
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

## 查看、修改主题

可以选择自己喜欢的主题，查看主题命令如下：

```shell
ls ~/.oh-my-zsh/themes
```

修改主题

```
vim ~/.zshrc
# 找到 ZSH_THEME 行，修改为自己想用的主题名称即可
```

默认的主题是 `ZSH_THEME="robbyrussell"` ，改成自己喜欢的即可，也可以用我自定义的一个主题，安装方法：

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/Jsharkc/jacobin-zsh-theme/master/install.sh)" 
source ~/.zshrc
```

**截图**

![image](https://user-images.githubusercontent.com/17465198/69112005-2938bb00-0aba-11ea-8c16-12fb9181b93f.png)

## 安装想用的插件

我想安装「自动补全」和「语法高亮」插件

### 自动补全插件 `zsh-autosuggestions`

1. 下载该插件到`.oh-my-zsh`的插件目录

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

2. 编辑`.zshrc`文件

找到 `plugins=` 这一行，添加 `zsh-autosuggestions`，例如：

```
plugins=(
  git
  zsh-autosuggestions
)
```

3. 使插件生效

```
source ~/.zshrc
```

### 语法高亮插件 `zsh-syntax-highlighting`

1. 下载该插件到`.oh-my-zsh`的插件目录

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```

2. 编辑`.zshrc`文件

找到 `plugins=` 这一行，添加 `zsh-syntax-highlighting`，例如：

```
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

3. 使插件生效

```
source ~/.zshrc
```



想了解更多 `oh-my-zsh` 内容请前往 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

