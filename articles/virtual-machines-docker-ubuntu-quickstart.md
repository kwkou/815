<properties
	pageTitle="如何快速将 Docker 用于 Ubuntu-Docker VM 映像"
	description="介绍并演示如何在几分钟内直接从 Azure 映像库“使用 Ubuntu Server 上的 Docker”"
	services="virtual-machines"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="virtual-machines"
	ms.date="10/04/2015"
	wacn.date="11/12/2015"/>

# 如何在 Azure 应用商店中快速开始使用 Docker

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]资源管理器模型。
 

开始使用 [Docker] 的最快方法是转到 Azure 应用商店，将 [Canonical] 创建的**“Ubuntu Server 上的 Docker”**映像模板与 [MSOpenTech] 结合使用来创建 VM。这将创建 Ubuntu Server VM 并自动安装 [Docker VM 扩展](/documentation/articles/virtual-machines-docker-vm-extension)，同时在 Azure 上预安装并运行**最新** Docker 引擎。

你可以使用 SSH 立即连接到该 VM，并直接开始使用 Docker 而不用执行任何其他操作。

> [AZURE.NOTE]Azure 应用商店模板创建的 VM 未托管 Docker 远程 API 以供远程 docker 客户端进行管理。若要在此 VM 上启用远程控制 Docker 主机，请参阅[使用 HTTPS 运行 Docker](https://docs.docker.com/articles/https/)，或者执行[通过 Azure 门户使用 Docker VM 扩展](/documentation/articles/virtual-machines-docker-with-portal)或[通过 Azure CLI 使用 Docker VM 扩展](/documentation/articles/virtual-machines-docker-with-xplat-cli)中的步骤。如果你特别技术狂，则可以从 GitHub 构建 [Windows Docker 客户端](https://github.com/ahmetalpbalkan/Docker.DotNet)，然后也对其进行试用（或直接从 [nuget](https://www.nuget.org/packages/Docker.DotNet/) 中获取它）。
<!-- -->
如果你想要从 Windows 自动化 Azure Docker VM，则可以[安装 Docker 工具箱](https://docs.docker.com/installation/windows/)或[从 Chocolatey](https://chocolatey.org/packages/docker) 获取 Docker.exe。

## 登录到门户

除非你没有 Azure 帐户，否则这部分很简单。[也可以轻松获取免费帐户](/pricing/1rmb-trial/)！

## 使用 Canonical 提供的 Docker 映像和 MSOpenTech 创建 VM

1. 现在，你已登录，请单击左下角的**“新建”**以创建新的 VM 映像。你可能会立即在横幅中看到相应映像：

> ![在横幅中选择 Docker Ubuntu 映像](./media/virtual-machines-docker-ubuntu-quickstart/CreateNewDockerBanner.png)

2. 但如果没有，请在映像库中搜索它：

> ![在映像库中找到该映像](./media/virtual-machines-docker-ubuntu-quickstart/DockerOnUbuntuServerMSOpenTech.png)

3. 提供实例的用户名和密码或 **.pub** 文件的内容（ssh-rsa 格式），以便使用证书启用 SSH。（下图显示了指定用户名和密码组合。） 然后，按底部的**“创建”**。

> ![配置 VM 实例](./media/virtual-machines-docker-ubuntu-quickstart/CreateVMDockerUbuntuPwd.png)

4. 并等到它运行。这根本不应该需要太多时间。

> ![在门户中运行的 Docker 映像](./media/virtual-machines-docker-ubuntu-quickstart/DockerUbuntuRunning.png)

## 使用 SSH 连接玩得开心

现在，快乐开始了。你可以使用 SSH 立即连接到该 VM：

> ![使用 SSH 进行连接](./media/virtual-machines-docker-ubuntu-quickstart/SSHToDockerUbuntu.png)

然后开始发出 Docker 命令，请记住在此 Azure VM 上，默认配置需要 **`sudo`**：

> ![提取映像](./media/virtual-machines-docker-ubuntu-quickstart/DockerPullSmallImages.png)

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

你将要开始使用 [Docker]！

<!--Anchors-->
[Log on to the Portal]: #logon
[Create a VM with the Docker Image from Canonical and MSOpenTech]: #createvm
[Connect with SSH and Have Fun]: #havingfun
[Next steps]: #next-steps


[Docker]: https://www.docker.com/
[BusyBox]: http://zh.wikipedia.org/wiki/BusyBox
[Docker scratch image]: https://docs.docker.com/articles/baseimages/#creating-a-simple-base-image-using-scratch
[Canonical]: http://www.canonical.com/
[MSOpenTech]: http://msopentech.com/
 

<!---HONumber=79-->