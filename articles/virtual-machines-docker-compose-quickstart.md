<properties pageTitle="在 Azure 虚拟机上使用 Docker 和 Compose 入门" description="对在 Azure 上使用 Compose 和 Docker 的简要介绍" services="virtual-machines" documentationCenter="" authors="dlepow" manager="timlt editor=""/>

<tags ms.service="virtual-machines" ms.date="05/07/2015" wacn.date="08/29/2015"/>

# 在 Azure 虚拟机上使用 Docker 和 Compose 入门


你可以在 Azure 中的 Linux 虚拟机上快速开始将 [Compose](http://github.com/docker/compose)（*Fig* 的后继）与 Docker 一起使用以定义和运行复杂的应用程序。使用 Compose，可在单个文件中定义一个多容器应用程序，然后可通过单个命令启动该应用程序，此命令会执行所有操作让该应用程序在 VM 上运行。




## 使用 Docker VM 扩展创建 Azure VM 并安装 Compose

可以使用各种 Azure 过程和 Azure 应用商店中提供的映像创建 Azure VM 并在其上安装 Docker 和 Compose。有关示例，请参阅[从 Azure 命令行界面使用 Docker VM 扩展](virtual-machines-docker-with-xplat-cli)，了解通过适用于 Mac、Linux 和 Windows 的 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展创建 Ubuntu VM 的快速过程。该文章中的示例在服务管理 (**asm**) 模式下使用 CLI。


在 Ubuntu VM 使用 Docker 运行后，使用 SSH 连接到该 VM，然后通过运行两个命令安装 [Compose](https://github.com/docker/compose/blob/882dc673ce84b0b29cd59b6815cb93f74a6c4134/docs/install)：

```
curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```
>[AZURE.NOTE]如果你收到“权限被拒绝”错误，则很可能 /usr/local/bin 目录不可写，你将需要以超级用户身份安装 Compose。依次运行 `sudo -i`、上述两个命令和 `exit`。

若要测试你的 Compose 安装，请运行

```
docker-compose --version
```

你将看到与
```
docker-compose 1.2.0
``` 
类似的输出。



## 创建 docker-compose.yml 配置文件

Compose 使用名为 `docker-compose.yml` 的文本配置文件来定义将在 VM 中运行的容器。该文件指定要在每个容器中运行的映像（或者它可能从 Dockerfile 生成）、必要的环境变量和依赖关系、端口、容器之间的链接等。有关 yml 文件语法的详细信息，请参阅 [docker-compose.yml 参考](http://docs.docker.com/compose/yml/)。

在 VM 上创建工作目录，并使用你最喜欢的文本编辑器创建 `docker-compose.yml`。若要试用一个简单示例，请将以下文本复制到该文件中。此配置将使用 [DockerHub 注册表](https://registry.hub.docker.com/_/wordpress/)中的映像安装 WordPress（开源博客和内容管理系统）和链接后端 MariaDB SQL 数据库。

 ```
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

```

## 使用 Compose 启动容器

在你的工作目录中直接运行

```
docker-compose up -d

```

以启动 docker-compose.yml 中指定的 Docker 容器。你将看到类似于以下内容的输出：

```
Creating wordpress_db_1...
Creating wordpress_wordpress_1...
```

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

现在可以通过浏览到 `http://localhost:8080` 在 VM 上直接连接到 WordPress。如果要通过 Internet 连接到 VM，请先在 VM 上配置 HTTP 终结点，以便将公共端口 80 映射到专用端口 8080。例如，运行以下 Azure CLI 命令：

```
azure vm endpoint create <machine-name> 80 8080

```

现在，你应看到 WordPress 开始屏幕，你可以在其中完成安装。

![WordPress 开始屏幕][wordpress_start]




## 后续步骤

* 有关构建和部署多容器应用程序的更多示例，请查阅 [Compose 命令参考](http://docs.docker.com/compose/cli/)和[用户指南](http://docs.docker.com/compose/)。
* 请尝试将 Docker Compose 与 [Docker Swarm](/documentation/articles/virtual-machines-docker-swarm) 群集集成。有关方案，请参阅 [Docker Compose/Swarm 集成](https://github.com/docker/compose/blob/master/SWARM)。
* 使用 Azure 资源管理器模板（你自己的或[社区](http://azure.microsoft.com/documentation/templates/)提供的），通过 Docker 部署 Azure VM 和使用 Compose 设置的应用程序。例如，[包含 Docker 和三个容器的 Ubuntu VM 模板](http://azure.microsoft.com/documentation/templates/docker-simple-on-ubuntu/)可帮助你使用 [DockerHub 注册表](https://registry.hub.docker.com/)中的映像快速部署三个服务。

<!--Image references-->

[wordpress_start]: ./media/virtual-machines-docker-compose-quickstart/WordPress.png

<!---HONumber=67-->