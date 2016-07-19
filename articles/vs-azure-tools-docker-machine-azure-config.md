<properties
   pageTitle="使用 Docker 计算机在 Azure 中创建 Docker 主机 | Azure"
   description="介绍如何使用 Docker 计算机在 Azure 中创建 Docker 主机。"
   services="azure-container-service"
   documentationCenter="na"
   authors="allclark"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="06/08/2016"
   wacn.date="" />

# 使用 Docker-Machine 在 Azure 中创建 Docker 主机

运行 [Docker](https://www.docker.com/) 容器时需要运行 Docker 守护程序的主机 VM。
本主题说明如何使用 [docker-machine](https://docs.docker.com/machine/) 命令，创建在 Azure 中运行并配置有 Docker 守护程序的新 Linux VM。

**注意：**
- 本文以 docker-machine 0.7.0 版或更高版本为基础
- 在不久的未来，将通过 docker-machine 支持 Windows 容器

## 使用 Docker 计算机创建 VM

通过 `azure` 驱动程序，使用 `docker-machine create` 命令在 Azure 中创建 Docker 主机 VM。

Azure 驱动程序需要订阅 ID。你可以使用 [Azure CLI](/documentation/articles/xplat-cli-install) 或 [Azure 门户](https://portal.azure.com)来检索 Azure 订阅。

**使用 Azure 门户**
- 从左导航页中选择“订阅”，并复制订阅 ID。

**使用 Azure CLI**
- 键入 `azure account list` 并复制订阅 ID。

键入 `docker-machine create --driver azure` 可查看选项及其默认值。
也可以查看 [Docker Azure 驱动程序文档](https://docs.docker.com/machine/drivers/azure/)了解详细信息。

以下示例依赖默认值，但会根据需要打开 VM 上的端口 80 进行 Internet 访问。

    docker-machine create -d azure --azure-subscription-id <Your AZURE_SUBSCRIPTION_ID> --azure-open-port 80 mydockerhost
    

## 使用 docker-machine 选择 Docker 主机
docker-machine 中有你主机的条目之后，即可在运行 docker 命令时设置默认主机。
##使用 PowerShell

    docker-machine env MyDockerHost | Invoke-Expression 

##使用 Bash

    eval $(docker-machine env MyDockerHost)

现在可以对指定的主机运行 docker 命令

    docker ps
    docker info

## 运行容器

配置主机之后，现在可以运行简单的 Web 服务器，来测试是否已正确配置主机。
我们在此处使用标准的 nginx 映像，指定它应侦听端口 80，并且如果主机 VM 重新启动，容器也将重新启动 (`--restart=always`)。

    docker run -d -p 80:80 --restart=always nginx

输入应如下所示：

    Unable to find image 'nginx:latest' locally
    latest: Pulling from library/nginx
    efd26ecc9548: Pull complete
    a3ed95caeb02: Pull complete
    83f52fbfa5f8: Pull complete
    fa664caa1402: Pull complete
    Digest: sha256:12127e07a75bda1022fbd4ea231f5527a1899aad4679e3940482db3b57383b1d
    Status: Downloaded newer image for nginx:latest
    25942c35d86fe43c688d0c03ad478f14cc9c16913b0e1c2971cb32eb4d0ab721

## 测试容器

使用 `docker ps` 检查正在运行的容器：

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
    d5b78f27b335        nginx               "nginx -g 'daemon off"   5 minutes ago       Up 5 minutes        0.0.0.0:80->80/tcp, 443/tcp   goofy_mahavira

查看正在运行的容器，并键入 `docker-machine ip <VM name>` 以查找要在浏览器中输入的 IP 地址：

    PS C:\> docker-machine ip MyDockerHost
    191.237.46.90

![运行 ngnix 容器](./media/vs-azure-tools-docker-machine-azure-config/nginxsuccess.png)

##摘要
使用 docker-machine，可以在 Azure 中轻松预配 Docker 主机来进行个别 Docker 主机验证。
有关容器的生产托管，请参阅 [Azure 容器服务](http://aka.ms/AzureContainerService)

若要使用 Visual Studio 开发 .NET Core 应用程序，请参阅 [Docker Tools for Visual Studio](http://aka.ms/DockerToolsForVS)

<!---HONumber=Mooncake_0711_2016-->