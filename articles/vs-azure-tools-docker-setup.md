<properties
   pageTitle="使用 VirtualBox 配置 Docker 主机 | Azure"
   description="使用 Docker 计算机和 VirtualBox 配置默认 Docker 实例的逐步说明"
   services="azure-container-service"
   documentationCenter="na"
   authors="allclark"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="06/08/2016"
   wacn.date="06/20/2016" />

# 使用 VirtualBox 配置 Docker 主机

## 概述
本文将指导你逐步使用 Docker 计算机和 VirtualBox 配置默认 Docker 实例。
如果你使用 [Docker for Windows beta](http://beta.docker.com/)，则无需进行此配置。

## 先决条件
需要安装以下工具。

- [Docker 工具箱](https://www.docker.com/products/overview#/docker_toolbox)

## 使用 Windows PowerShell 配置 Docker 客户端

要配置 Docker 客户端，只需打开 Windows PowerShell，并执行以下步骤：

1. 创建默认 Docker 主机实例。

		docker-machine create --driver virtualbox default
 
1. 验证默认实例是否已配置且在运行。（你应该会看到名为“default”的实例正在运行。

		docker-machine ls 
		
	![][0]
 
1. 将“default”设置为当前主机，并配置你的 Shell。

        docker-machine env default | Invoke-Expression

1. 显示活动的 Docker 容器。列表应为空。

		docker ps


	![docker ps output][1]
 
> [AZURE.NOTE] 每次重启开发计算机时，都需要重启本地 Docker 主机。为此，请在命令提示符下发出以下命令：`docker-machine start default`。

[0]: ./media/vs-azure-tools-docker-setup/docker-machine-ls.png
[1]: ./media/vs-azure-tools-docker-setup/docker-ps.png

<!---HONumber=AcomDC_0718_2016-->