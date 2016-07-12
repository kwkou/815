<properties
   pageTitle="虚拟机上的 Docker 和 Compose | Azure"
   description="在 Azure 中的 Linux 虚拟机上使用 Compose 和 Docker 的快速简介。"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""
   tags="azure-service-management"/>

<tags
   ms.service="virtual-machines-linux"
   ms.date="03/02/2016"
   wacn.date="05/24/2016"/>

# 开始使用 Docker 和 Compose，在 Azure 虚拟机上定义和运行多容器应用程序

开始使用 Docker 和 [Compose](http://github.com/docker/compose) 在 Azure 中的 Linux 虚拟机上定义和运行复杂的应用程序。借助 Compose（ *Fig* 的后继版本），你可以使用简单的文本文件定义由多个 Docker 容器组成的应用程序。然后使用单个命令启动应用程序，该命令会执行使该应用程序在 VM 上运行的所有操作。作为示例，本文说明如何使用后端 MariaDB SQL 数据库快速设置 WordPress 博客，但你也可以使用 Compose 设置更复杂的应用程序。

作为示例，本文展示了怎么使用后台 MariaDB SQL 数据库快速设置 WordPress 博客，然而你也可以使用 Compose 来设置更加复杂的应用程序。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-wordpress-mysql)。

## 步骤 1：将 Linux VM 设置为 Docker 主机

可以使用各种 Azure 过程和 Azure 应用商店中提供的映像创建 Linux VM 并将其设置为 Docker 主机。有关示例，请参阅[从 Azure 命令行界面使用 Docker VM 扩展](/documentation/articles/virtual-machines-linux-classic-cli-use-docker/)，了解使用 Docker VM 扩展创建 Ubuntu VM 的快速过程。使用 Docker VM 扩展时，你的 VM 将自动设置为 Docker 主机。该文中的示例演示如何在服务管理模式下使用[适用于 Mac、Linux 和 Windows 的 Azure 命令行界面](/documentation/articles/xplat-cli-install/) (Azure CLI) 创建 VM。

## 步骤 2：安装 Compose

在 Linux VM 使用 Docker 运行后，从客户端计算机使用 SSH 连接到它。如果需要，可通过运行以下两个命令安装 [Compose](https://github.com/docker/compose/blob/882dc673ce84b0b29cd59b6815cb93f74a6c4134/docs/install.md)。

>[AZURE.TIP] 如果你已使用 Docker VM 扩展创建 VM，则已为你安装 Compose。跳过这些命令并转到步骤 3。仅当你自己将 Docker 安装在 VM 上时，才需要安装 Compose。

	$ curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
	
	$ chmod +x /usr/local/bin/docker-compose

>[AZURE.NOTE]如果你收到“权限被拒绝”错误，则 VM 上的 /usr/local/bin 目录不是可写的，你将需要以超级用户身份安装 Compose。依次运行 `sudo -i`、上述两个命令和 `exit`。

若要测试 Compose 安装，请运行以下命令。

	$ docker-compose --version

你将看到与 `docker-compose 1.5.1` 类似的输出。


## 步骤 3：创建 docker-compose.yml 配置文件

接下来，将创建 `docker-compose.yml` 文件，它只是一个文本配置文件，用于定义要在 VM 上运行的 Docker 容器。该文件指定要在每个容器中运行的映像（或者它可能从 Dockerfile 生成）、必要的环境变量和依赖关系、端口、容器之间的链接等。有关 yml 文件语法的详细信息，请参阅 [docker-compose.yml 参考](http://docs.docker.com/compose/yml/)。

在 VM 上创建工作目录，并使用你最喜欢的文本编辑器创建 `docker-compose.yml`。若要试用一个简单示例，请将以下文本复制到该文件中。此配置将使用 [DockerHub 注册表](https://registry.hub.docker.com/_/wordpress/)中的映像安装 WordPress（开源博客和内容管理系统）和链接的后端 MariaDB SQL 数据库。

	 wordpress:
	  image: wordpress
	  links:
	    - db:mysql
	  ports:
	    - 8080:80
	
	db:
	  image: mariadb
	  environment:
	    MYSQL_ROOT_PASSWORD: <your password>

## 步骤 4: 使用“撰写”启动容器

在 VM 上的工作目录中，直接运行以下命令。（取决于你的环境，也许需要用 `sudo` 来运行 `docker-compose`。）

	$ docker-compose up -d

这将启动“docker-compose.yml”中指定的 Docker 容器。你将看到类似于以下内容的输出：

	Creating wordpress\_db\_1...
	Creating wordpress\_wordpress\_1...

>[AZURE.NOTE] 启动时请务必使用 **-d** 选项，以使容器在后台继续运行。

若要验证容器是否已启动，请键入 `docker-compose ps`。你应看到类似如下的内容：

	Name             Command             State              Ports
	-------------------------------------------------------------------------
	wordpress_db_1     /docker-           Up                 3306/tcp
	             entrypoint.sh
	             mysqld
	wordpress_wordpr   /entrypoint.sh     Up                 0.0.0.0:8080->80
	ess_1              apache2-for ...                       /tcp

现在可以通过浏览到 `http://localhost:8080` 在 VM 上直接连接到 WordPress。如果要通过 Internet 连接到 VM，请先在 VM 上配置 HTTP 终结点，以便将公共端口 80 映射到专用端口 8080。例如，在 Azure 服务管理部署中，运行以下 Azure CLI 命令：

	$ azure vm endpoint create <machine-name> 80 8080

现在，入宫你尝试连接 `http://<cloudservicename>.chinacloudapp.cn`，你应看到 WordPress 开始屏幕，你可以在其中完成安装并开始使用应用程序。

![WordPress 开始屏幕][wordpress_start]


## 后续步骤

* 到 [Docker VM 扩展用户指引](https://github.com/Azure/azure-docker-extension/blob/master/README.md)中查看你的 Docker VM 中更多配置 Docker 和 Compose 的选项。
* 有关构建和部署多容器应用的更多示例，请查阅 [Compose CLI 参考](http://docs.docker.com/compose/reference/)和[用户指南](http://docs.docker.com/compose/)。
* 请尝试将 Docker Compose 与 [Docker Swarm](/documentation/articles/virtual-machines-linux-docker-swarm/) 群集集成。有关方案，请参阅 [使用 Swarm 配合 Compose](https://docs.docker.com/compose/swarm/)。

<!--Image references-->

[wordpress_start]: ./media/virtual-machines-linux-docker-compose-quickstart/WordPress.png

<!---HONumber=Mooncake_0118_2016-->