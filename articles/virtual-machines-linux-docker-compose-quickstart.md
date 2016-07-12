<properties
   pageTitle="虚拟机上的 Docker 和 Compose | Azure"
   description="在 Azure 中的 Linux 虚拟机上使用 Compose 和 Docker 的快速简介"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="06/10/2016"
	wacn.date="07/11/2016"/>

# 开始使用 Docker 和 Compose，在 Azure 虚拟机上定义和运行多容器应用程序

开始使用 Docker 和 [Compose](http://github.com/docker/compose) 在 Azure 中的 Linux 虚拟机上定义和运行复杂的应用程序。借助 Compose（*Fig* 的后继版本），你可以使用简单的文本文件定义由多个 Docker 容器组成的应用程序。然后使用单个命令启动应用程序，该命令会执行使该应用程序在 VM 上运行的所有操作。

作为示例，本文说明如何在 Ubuntu VM 上使用后端 MariaDB SQL 数据库快速设置 WordPress 博客，但你也可以使用 Compose 设置更复杂的应用程序。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


如果你不熟悉 Docker 和容器，请参阅 [Docker 高级白板](https://azure.microsoft.com/documentation/videos/docker-high-level-whiteboard/)。

## 步骤 1：将 Linux VM 设置为 Docker 主机

可以使用各种 Azure 过程和 Azure 应用商店中提供的映像或 Resource Manager 模板创建 Linux VM，并将其设置为 Docker 主机。例如，请参阅[使用 Docker VM 扩展部署环境](/documentation/articles/virtual-machines-linux-dockerextension/)，了解使用[快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu)通过 Azure Docker VM 扩展来创建 Ubuntu VM 的快速过程。使用 Docker VM 扩展时，VM 将自动设置为 Docker 主机，并且已安装 Compose。该文中的示例演示如何在 Resource Manager 模式下使用[适用于 Mac、Linux 和 Windows 的 Azure 命令行接口](/documentation/articles/xplat-cli-install/) (Azure CLI) 创建 VM。

## 步骤 2：确认已安装 Compose

在 Linux VM 使用 Docker 运行后，从客户端计算机使用 SSH 连接到它。

若要测试是否在 VM 上安装了 Compose，请运行以下命令。

	$ docker-compose --version

你将看到与 `docker-compose 1.6.2, build 4d72027` 类似的输出。

>[AZURE.TIP] 如果使用另一种方法创建 Docker 主机，而且需要自行安装 Compose，请参阅 [Compose 文档](https://github.com/docker/compose/blob/882dc673ce84b0b29cd59b6815cb93f74a6c4134/docs/install.md)。


## 步骤 3：创建 docker-compose.yml 配置文件

接下来，将创建 `docker-compose.yml` 文件，它只是一个文本配置文件，用于定义要在 VM 上运行的 Docker 容器。该文件指定要在每个容器中运行的映像（或者它可能从 Dockerfile 生成）、必要的环境变量和依赖关系、端口、容器之间的链接等。有关 yml 文件语法的详细信息，请参阅 [Compose 文件参考](http://docs.docker.com/compose/yml/)。

在 VM 上创建工作目录，并使用你最喜欢的文本编辑器创建 `docker-compose.yml`。若要尝试对简单示例进行概念证明，请将以下文本复制到该文件。此配置将使用 [DockerHub 注册表](https://registry.hub.docker.com/_/wordpress/)中的映像安装 WordPress（开源博客和内容管理系统）和链接的后端 MariaDB SQL 数据库。

	 wordpress:
	  image: wordpress
	  links:
	    - db:mysql
	  ports:
	    - 80:80
	
	db:
	  image: mariadb
	  environment:
	    MYSQL_ROOT_PASSWORD: <your password>

## 步骤 4：使用 Compose 启动容器

在 VM 的工作目录中，只需运行以下命令。（视你的环境而定，可能需要使用 `sudo` 运行 `docker-compose`。）

	$ docker-compose up -d

这会启动 `docker-compose.yml` 中指定的 Docker 容器。你将看到如下输出：

	Creating wordpress_db_1...
	Creating wordpress_wordpress_1...

>[AZURE.NOTE] 启动时请务必使用 **-d** 选项，以使容器在后台继续运行。

若要验证容器是否已启动，请键入 `docker-compose ps`。你应看到类似如下的内容：

	Name             Command             State              Ports
	-------------------------------------------------------------------------
	wordpress_db_1     /docker-           Up                 3306/tcp
	             entrypoint.sh
	             mysqld
	wordpress_wordpr   /entrypoint.sh     Up                 0.0.0.0:80->80
	ess_1              apache2-for ...                       /tcp

现在可以在 VM 的端口 80 上直接连接到 WordPress。如果创建 VM 时使用的是 Resource Manager 模板，请尝试连接到 `http://<dnsname>.<region>.chinacloudapp.cn`；如果创建 VM 时使用的是经典部署模型，请尝试连接到 `http://<cloudservicename>.chinacloudapp.cn`。现在，你应看到 WordPress 开始屏幕，你可以在其中完成安装并开始使用应用程序。

![WordPress 开始屏幕][wordpress_start]


## 后续步骤

* 转到 [Docker VM 扩展用户指南](https://github.com/Azure/azure-docker-extension/blob/master/README.md)，了解用于在 Docker VM 中配置 Docker 和 Compose 的更多选项。例如，其中一个选项是将 Compose yml 文件（已转换为 JSON）直接放在 Docker VM 扩展的配置中。
* 有关构建和部署多容器应用的更多示例，请查阅 [Compose 命令行参考](http://docs.docker.com/compose/reference/)和[用户指南](http://docs.docker.com/compose/)。
* 使用 Azure 资源管理器模板（你自己的或[社区](https://github.com/Azure/azure-quickstart-templates/)提供的），通过 Docker 部署 Azure VM 和使用 Compose 设置的应用程序。例如，[使用 Docker 部署 WordPress 博客](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-wordpress-mysql)模板使用 Docker 和 Compose 通过 Ubuntu VM 上的 MySQL 后端快速部署 WordPress。
* 请尝试将 Docker Compose 与 [Docker Swarm](/documentation/articles/virtual-machines-linux-docker-swarm/) 群集集成。相关方案请参阅 [Using Compose with Swarm（将 Compose 与 Swarm 搭配使用）](https://docs.docker.com/compose/swarm/)。

<!--Image references-->

[wordpress_start]: ./media/virtual-machines-linux-docker-compose-quickstart/WordPress.png

<!---HONumber=Mooncake_0704_2016-->