<properties
   pageTitle="在 Azure 虚拟机上使用 Docker 和 Compose 入门"
   description="在 Azure 上使用 Compose 和 Docker 的快速简介"
   services="virtual-machines"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""/>

<tags 
	ms.service="virtual-machines"
	ms.date="08/07/2015" 
	wacn.date="09/15/2015"/>

# 在 Azure 虚拟机上使用 Docker 和 Compose 入门

本文介绍如何开始使用 Docker 和 [Compose](http://github.com/docker/compose) 在 Azure 中的 Linux 虚拟机上定义和运行复杂的应用程序。借助 Compose（*Fig* 的后继版本），你可以使用简单的文本文件定义由多个 Docker 容器组成的应用程序。然后使用单个命令启动应用程序，该命令会执行使该应用程序在 VM 上运行的所有操作。作为示例，本文说明如何使用后端 MariaDB SQL 数据库快速设置 WordPress 博客，但你也可以使用 Compose 设置更复杂的应用程序。

<!--
如果你不熟悉 Docker 和容器，请参阅 [Docker 高级白板](/documentation/videos/docker-high-level-whiteboard/)。
-->
## 步骤 1：将 Linux VM 设置为 Docker 主机

可以使用各种 Azure 过程和 Azure 应用商店中提供的映像创建 Linux VM 并将其设置为 Docker 主机。有关示例，请参阅[从 Azure 命令行界面使用 Docker VM 扩展](/documentation/articles/virtual-machines-docker-with-xplat-cli)，了解使用 Docker VM 扩展创建 Ubuntu VM 的快速过程。使用 Docker VM 扩展时，你的 VM 将自动设置为 Docker 主机。该文中的示例演示如何在服务管理模式下使用[适用于 Mac、Linux 和 Windows 的 Azure 命令行界面](/documentation/articles/xplat-cli) (Azure CLI) 创建 VM。

## 步骤 2：安装 Compose

在 Linux VM 使用 Docker 运行后，从客户端计算机使用 SSH 连接到它。如果需要，可通过运行以下两个命令安装 [Compose](https://github.com/docker/compose/blob/882dc673ce84b0b29cd59b6815cb93f74a6c4134/docs/install.md)。

>[AZURE.TIP]如果你已使用 Docker VM 扩展创建 VM，则已为你安装 Compose。跳过这些命令并转到步骤 3。仅当你自己将 Docker 安装在 VM 上时，才需要安装 Compose。

```
$ curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ chmod +x /usr/local/bin/docker-compose
```
>[AZURE.NOTE]如果你收到“权限被拒绝”错误，则 VM 上的 /usr/local/bin 目录不是可写的，你将需要以超级用户身份安装 Compose。依次运行 `sudo -i`、上述两个命令和 `exit`。

若要测试 Compose 安装，请运行以下命令。

```
$ docker-compose --version
```

你将看到与 ```
docker-compose 1.3.2
``` 类似的输出。


## 步骤 3：创建 docker-compose.yml 配置文件

接下来，将创建 `docker-compose.yml` 文件，它只是一个文本配置文件，用于定义要在 VM 上运行的 Docker 容器。该文件指定要在每个容器中运行的映像（或者它可能从 Dockerfile 生成）、必要的环境变量和依赖关系、端口、容器之间的链接等。有关 yml 文件语法的详细信息，请参阅 [docker-compose.yml 参考](http://docs.docker.com/compose/yml/)。

在 VM 上创建工作目录，并使用你最喜欢的文本编辑器创建 `docker-compose.yml`。若要试用一个简单示例，请将以下文本复制到该文件中。此配置将使用 [DockerHub 注册表](https://registry.hub.docker.com/_/wordpress/)中的映像安装 WordPress（开源博客和内容管理系统）和链接的后端 MariaDB SQL 数据库。

 ``` wordpress: image: wordpress links: - db:mysql ports: - 8080:80

db: image: mariadb environment: MYSQL\_ROOT\_PASSWORD: <your password>

```

## Step 4: Start the containers with Compose

In the working directory on your VM, simply run the following command.

```
$ docker-compose up -d

```

This starts the Docker containers specified in `docker-compose.yml`. You'll see output similar to:

```
正在创建 wordpress\_db\_1...正在创建 wordpress\_wordpress\_1...```

>[AZURE.NOTE]启动时请务必使用 **-d** 选项，以使容器在后台继续运行。

若要验证容器是否已启动，请键入 `docker-compose ps`。你应看到类似如下的内容：

```
Name             Command             State              Ports
-------------------------------------------------------------------------
wordpress_db_1     /docker-           Up                 3306/tcp
             entrypoint.sh
             mysqld
wordpress_wordpr   /entrypoint.sh     Up                 0.0.0.0:8080->80
ess_1              apache2-for ...                       /tcp
```

现在可以通过浏览到 `http://localhost:8080` 在 VM 上直接连接到 WordPress。如果要通过 Internet 连接到 VM，请先在 VM 上配置 HTTP 终结点，以便将公共端口 80 映射到专用端口 8080。例如，在 Azure 服务管理部署中，运行以下 Azure CLI 命令：

```
$ azure vm endpoint create <machine-name> 80 8080

```

现在，你应看到 WordPress 开始屏幕，你可以在其中完成安装并开始使用应用程序。

![WordPress 开始屏幕][wordpress_start]




## 后续步骤

* 有关构建和部署多容器应用的更多示例，请查阅 <!--[-->Compose 命令参考<!--](http://docs.docker.com/compose/cli/)-->和[用户指南](http://docs.docker.com/compose/)。
* 使用 Azure 资源管理器模板（你自己的或<!--[-->社区<!--](/documentation/templates/)-->提供的），通过 Docker 部署 Azure VM 和使用 Compose 设置的应用程序。例如，<!--[-->使用 Docker 部署 WordPress 博客<!--](/documentation/templates/docker-wordpress-mysql/)-->模板使用 Docker 和 Compose 通过 Ubuntu VM 上的 MySQL 后端快速部署 WordPress。
* 请尝试将 Docker Compose 与 [Docker Swarm](/documentation/articles/virtual-machines-docker-swarm) 群集集成。有关方案，请参阅<!-- [-->Docker Compose/Swarm 集成<!--](https://github.com/docker/compose/blob/master/SWARM)-->。
* 使用 Azure 资源管理器模板（你自己的或<!--[-->社区<!--](http://azure.microsoft.com/documentation/templates/)-->提供的），通过 Docker 部署 Azure VM 和使用 Compose 设置的应用程序。例如，<!--[-->包含 Docker 和三个容器的 Ubuntu VM 模板<!--](http://azure.microsoft.com/documentation/templates/docker-simple-on-ubuntu/)-->可帮助你使用 [DockerHub 注册表](https://registry.hub.docker.com/)中的映像快速部署三个服务。

<!--Image references-->

[wordpress_start]: ./media/virtual-machines-docker-compose-quickstart/WordPress.png

<!---HONumber=69-->