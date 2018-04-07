---
title: Centos7 安装 Docker-CE 社区版
date: 2017-01-15 17:42:23
tags: Docker
---
# Centos7 安装 Docker-CE 社区版

今天在自己的阿里云服务器上装了 docker-ce，记录一下，以后再装的话可以参考。

## 安装相关依赖

yum-utils 提供 yum-config-manager 工具, devicemapper存储驱动依赖 device-mapper-persistent-data 和 lvm2.

```Sh
> yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 配置版本镜像库

季度更新的稳定stable版和月度更新的edge版

```sh
> yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo
> yum-config-manager --enable docker-ce-edge

```

由于docker.com服务器下载很慢,所以改为国内镜像.

```sh
> yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

如需禁止edge版本, 可以执行下面的命令

```Sh
> yum-config-manager --disable docker-ce-edge
```

##安装 Docker

```Sh
> sudo yum makecache fast

> sudo yum install docker-ce
Error: docker-ce conflicts with 2:docker-1.12.6-28.git1398f24.el7.centos.x86_64
Error: docker-ce-selinux conflicts with 2:container-selinux-2.12-2.gite7096ce.el7.noarch

```

出现冲突, 原因是直接安装过docker.

```Sh
> yum list docker

Installed Packages
docker.x86_64                      2:1.12.6-28.git1398f24.el7.centos                      @extras
> sudo yum erase docker.x86_64
Removed:
  docker.x86_64 2:1.12.6-28.git1398f24.el7.centos
> sudo yum list container-selinux-2.12-2.gite7096ce.el7.noarch

> sudo yum erase container-selinux.noarch
```

删除老版本 docker

```sh
> yum list installed | grep docker
docker-client.x86_64                   2:1.12.6-28.git1398f24.el7.centos
docker-common.x86_64                   2:1.12.6-28.git1398f24.el7.centos
> sudo yum erase -y docker-client.x86_64
> sudo yum erase -y docker-common.x86_64

> sudo yum remove docker \
                  docker-common \
                  container-selinux \
                  docker-selinux \
                  docker-engine
```

再安装

```sh
[zhouhh@mainServer ~]$ sudo yum install docker-ce
Loaded plugins: fastestmirror, langpacks
Installing:
 docker-ce               x86_64       17.05.0.ce-1.el7.centos         docker-ce-edge        19 M
Installing for dependencies:
 docker-ce-selinux       noarch       17.05.0.ce-1.el7.centos         docker-ce-edge        28 k

Complete!

```

如果生产系统需要稳定版本, 需要 `yum list` 进行查询. 但yum list只会显示二进制包, 加上.x86_64会显示包含源码包的全部的包. sort -r会按版本倒序排序.

```Sh
> yum list docker-ce.x86_64  --showduplicates |sort -r
 * updates: mirrors.tuna.tsinghua.edu.cn
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror, langpacks
 * extras: mirror.bit.edu.cn
docker-ce.x86_64            17.05.0.ce-1.el7.centos             docker-ce-edge
docker-ce.x86_64            17.04.0.ce-1.el7.centos             docker-ce-edge
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
 * base: mirror.bit.edu.cn
```

第二列是版本号. el7表示centos7. 第三列是库名.

安装指定版本: sudo yum install docker-ce-

安装稳定版本:

```Sh
> sudo yum install docker-ce-17.03.1.ce-1.el7.centos
Installed:
  docker-ce.x86_64 0:17.03.1.ce-1.el7.centos

Dependency Installed:
  docker-ce-selinux.noarch 0:17.05.0.ce-1.el7.centos

Complete!
```

## 删除 docker-ce 版和镜像

```sh
> sudo yum remove docker-ce
> sudo rm -rf /var/lib/docker
```

## 启动测试 docker

Hello world的镜像启动后会打印”Hello from Docker!”然后退出.

```Sh
> sudo systemctl start docker
> docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
78445dd45222: Pull complete
Digest: sha256:c5515758d4c5e1e838e9cd307f6c6a0d620b5e07e6f927b07d05f6d12a1ac8d7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

```

## 非root用户启动docker

默认情况下，`docker` 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 `docker` 组：

```Sh
$ sudo groupadd docker
```

将当前用户加入 `docker` 组：

```sh
$ sudo usermod -aG docker $USER
```

## 设置自启动

大部分最新的linux发行版(RHEL, CentOS, Fedora, Ubuntu 16.04 以上), 都用sytemd来管理启动.

```Sh
[zhouhh@mainServer ~]$ sudo systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.

```

## 禁止自启动

```Sh
[zhouhh@mainServer ~]$ sudo systemctl disable docker
```