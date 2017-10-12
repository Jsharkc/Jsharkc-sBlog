---
title: 初探 Cockroachdb
date: 2017-03-17 14:50:37
tags: cockroach
thumbnail: https://raw.githubusercontent.com/cockroachdb/cockroach/master/docs/media/cockroach_db.png
---
# 初探 Cockroachdb

## 1.创建网桥

由于在单个主机上运行多个 Docker 容器，因此每个容器有一个 CockroachDB 节点，需要创建Docker所指的桥接网络。桥接网络将使容器能够作为单个群集进行通信，同时保持与外部网络的隔离。

``` docker
docker network create -d bridge roachnet
```

我们在 `roachnet` 这里和随后的步骤中使用了网络名称，但是请随时给您的网络任何您喜欢的名字。

## 2.启动第一个节点

``` docker
docker run -d \
--name=roach1 \
--hostname=roach1 \
--net=roachnet \
-p 26257:26257 -p 8080:8080  \
-v "${PWD}/cockroach-data/roach1:/cockroach/cockroach-data"  \
cockroachdb/cockroach:v1.0.4 start --insecure
```

此命令创建一个容器并启动其中的第一个 CockroachDB 节点。我们来看看每个部分：

`docker run`：Docker 命令启动一个新的容器。

`-d`：这个标志在后台运行容器，所以你可以在同一个shell中继续下一步。

`--name`：容器的名称。这是可选的，但是自定义名称使得在其他命令中引用容器更容易，例如在容器中打开Bash会话或停止容器时。

`--hostname`：容器的主机名。您将使用它将其他容器/节点连接到集群。

`--net`：用于容器加入的网桥。有关详细信息，请参阅步骤1。

`-p 26257:26257 -p 8080:8080`：这些标志将用于节点间和客户端节点通信（`26257`）的默认端口和 `8080` 从容器到主机的管理UI（）的 HTTP 请求的默认端口映射。这允许集装箱间通信，并可以从浏览器调用管理 UI。

`-v "${PWD}/cockroach-data/roach1:/cockroach/cockroach-data"`：该标志挂载作为数据卷的主机目录。这意味着该节点的数据和日志将存储在 ``${PWD}/cockroach-data/roach1`` 主机上，并在容器停止或删除后持续。有关更多详细信息，请参阅 Docker 将主机目录作为数据卷主题。 
`cockroachdb/cockroach:v1.0.4 start --insecure：CockroachDB` 命令以不安全的方式启动容器中的一个节点。

## 3.将节点添加到集群

在这一点上，您的群集是实时和可操作的。只需一个节点，您就可以连接一个 SQL 客户端并开始构建数据库。然而，在实际部署中，您总是希望3个或更多节点可以利用 CockroachDB 的自动复制，重新平衡和容错能力。

要模拟真正的部署，通过添加另外两个节点来扩展集群：

``` docker
# Start the second container/node:
docker run -d \
--name=roach2 \
--hostname=roach2 \
--net=roachnet \
-v "${PWD}/cockroach-data/roach2:/cockroach/cockroach-data" \
cockroachdb/cockroach:v1.0.4 start --insecure --join=roach1

# Start the third container/node:
docker run -d \
--name=roach3 \
--hostname=roach3 \
--net=roachnet \
-v "${PWD}/cockroach-data/roach3:/cockroach/cockroach-data" \
cockroachdb/cockroach:v1.0.4 start --insecure --join=roach1
```

这些命令添加了两个容器，并在其中启动 CockroachDB 节点，并将它们连接到第一个节点。从步骤2中只需要注意几点：

`-v`：该标志挂载作为数据卷的主机目录。数据和日志这些节点将被存储在 `${PWD}/cockroach-data/roach2与${PWD}/cockroach-data/roach3` 主机上和容器停止或删除之后将继续存在。
`--join`：该标志使用第一个容器将新节点连接到集群 `hostname`。否则，所有 `cockroach start` 默认值都被接受。请注意，由于每个节点都在唯一的容器中，所以使用相同的默认端口不会引起冲突。

## 4.测试集群

现在已经扩展到3个节点，可以使用任何节点作为集群的 SQL 网关。为了演示这一点，使用 docker exec 命令在第一个容器中启动内置的 SQL shell：

``` docker
docker exec -it roach1 ./cockroach sql --insecure
# Welcome to the cockroach SQL interface.
# All statements must be terminated by a semicolon.
# To exit: CTRL + D.
```

运行一些基本的 CockroachDB SQL 语句：

``` sql
> CREATE DATABASE bank;

> CREATE TABLE bank.accounts (id INT PRIMARY KEY, balance DECIMAL);

> INSERT INTO bank.accounts VALUES (1, 1000.50);

> SELECT * FROM bank.accounts;
```

``` sql
+----+---------+
| id | balance |
+----+---------+
|  1 |  1000.5 |
+----+---------+
(1 row)
```

退出节点1上的 SQL shell：

``` sql
> \q
```

然后在第二个容器中启动 SQL shell：

``` docker
docker exec -it roach2 ./cockroach sql --insecure
# Welcome to the cockroach SQL interface.
# All statements must be terminated by a semicolon.
# To exit: CTRL + D.
```

现在运行相同的 SELECT 查询：

``` sql
> SELECT * FROM bank.accounts;
```

``` sql
+----+---------+
| id | balance |
+----+---------+
|  1 |  1000.5 |
+----+---------+
```

``` sql
> (1 row)
```

如您所见，节点1和节点2的行为与 SQL 网关相同。

完成后，退出节点2上的 SQL shell：

``` sql
> \q
```

## 5.监控集群

当启动第一个容器/节点时，将节点的默认HTTP端口映射8080到8080主机上的端口。要查看集群的管理UI，请将浏览器指向该端口localhost，即 http://localhost:8080。

![CockroachDB管理界面](https://www.cockroachlabs.com/docs/images/admin_ui.png)

如前所述，CockroachDB会自动将您的数据复制到幕后。要验证上一步中写入的数据是否已成功复制，请向下滚动到每个商店的副本图表并将其悬停在该行上：

![CockroachDB管理界面](https://www.cockroachlabs.com/docs/images/admin_ui_replicas.png)

每个节点上的副本计数相同，表示集群中的所有数据都被复制3次（默认值）。

## 6.停止集群

使用 docker stop 和 docker rm 命令停止和删除容器（因此集群）：

``` docker
# Stop the containers:
docker stop roach1 roach2 roach3

# Remove the containers:
docker rm roach1 roach2 roach3
```