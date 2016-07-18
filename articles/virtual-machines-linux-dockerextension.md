<!-- ARM: tested -->

<properties
   pageTitle="了解 Azure 上的 Docker VM 扩展 | Azure"
   description="了解如何使用 Docker VM 扩展快速安全地在 Azure 中部署 Docker 环境"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/04/2016"
	wacn.date="06/20/2016"/>

# 使用 Docker VM 扩展部署环境

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

Docker 是流行的容器管理和映像处理平台，可让你在 Linux（和 Windows）上快速操作容器。通过 Azure，你可以根据需求使用几个不同的方式灵活部署 Docker：

- 若要快速制作应用原型，或如果你已经知道如何使用 Docker Machine，则可以[使用 Docker Machine Azure 驱动程序](/documentation/articles/virtual-machines-linux-docker-machine/)在 Azure 中部署 Docker 主机。
- 对于基于模板的部署，可以使用 Azure 虚拟机的 Docker VM 扩展。这种做法可以与 Azure Resource Manager 模板部署集成，包含所有相关的优点，例如基于角色的访问、诊断与部署后配置。

本文重点介绍如何使用 Resource Manager 模板在你定义的自定义生产就绪环境中部署 Docker VM 扩展。

## 适用于模板部署的 Azure Docker VM 扩展

Azure Docker VM 扩展在 Linux 虚拟机中安装并配置 Docker 守护程序、Docker 客户端和 Docker Compose。使用该扩展还可以配合 Docker Compose 来定义和部署容器应用程序。通过使用 Resource Manager 模板，可以使用一致的方式重新部署环境。Azure Docker VM 扩展非常适合在稳健的开发人员或生产环境中使用，因为相比于单纯使用 Docker Machine 或自行创建 Docker 主机，可以享有一些额外的控制功能。

通过 Azure Resource Manager，可以创建和部署定义整个环境结构的模板，如 Docker 主机、存储、基于角色的访问控制 (RBAC)、诊断，等等。你可以[阅读有关 Resource Manage 和模板的详细信息](/documentation/articles/resource-group-overview/)，以更好地了解其中的一些优点。相比于单纯使用 Docker Machine，使用 Resource Manager 模板的优点是可以定义其他 Docker 主机、存储、访问控制等，并且日后还能根据需要重新生成部署。

## 使用 Docker VM 扩展部署模板：

我们使用现有的快速入门模板来演示如何部署已安装 Docker VM 扩展的 Ubuntu VM。可以在此处查看模板：[使用 Docker 简易部署 Ubuntu VM](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu)

可以从 [GitHub](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/docker-simple-on-ubuntu/azuredeploy.json) 下载该模板并运行以下命令。

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

	azure group create --name myDockerResourceGroup --location "China North" \
	  --template-file /path/to/azuredeploy.json

回答为存储帐户命名、DNS 名称、用户名等的提示，然后等待几分钟时间完成部署。您应该会看到与下面类似的输出：

	info:    Executing command group create
	+ Getting resource group myDockerResourceGroup
	+ Updating resource group myDockerResourceGroup
	info:    Updated resource group myDockerResourceGroup
	info:    Supply values for the following parameters
	newStorageAccountName: mydockerstorage
	adminUsername: ops
	adminPassword: P@ssword!
	dnsNameForPublicIP: mydockergroup
	+ Initializing template configurations and parameters
	+ Creating a deployment
	info:    Created template deployment "azuredeploy"
	data:    Id:                  /subscriptions/guid/resourceGroups/myDockerResourceGroup
	data:    Name:                myDockerResourceGroup
	data:    Location:            chinanorth
	data:    Provisioning State:  Succeeded
	data:    Tags: null
	data:
	info:    group create command OK

## 部署第一个 nginx 容器

部署完成之后，使用在部署期间提供的 DNS 名称通过 SSH 连接到新 Docker 主机。由于 Docker 工具已安装，因此我们来试着运行 nginx 容器：

	docker run -d -p 80:80 nginx

您应该会看到与下面类似的输出：

	Unable to find image 'nginx:latest' locally
	latest: Pulling from library/nginx
	efd26ecc9548: Pull complete
	a3ed95caeb02: Pull complete
	a48df1751a97: Pull complete
	8ddc2d7beb91: Pull complete
	Digest: sha256:2ca2638e55319b7bc0c7d028209ea69b1368e95b01383e66dfe7e4f43780926d
	Status: Downloaded newer image for nginx:latest
	b6ed109fb743a762ff21a4606dd38d3e5d35aff43fa7f12e8d4ed1d920b0cd74

使用 `docker ps` 检验主机上运行的容器：

	CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                         NAMES
	b6ed109fb743        nginx               "nginx -g 'daemon off"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, 443/tcp   nostalgic_murdock

打开 Web 浏览器并输入部署期间指定的 DNS 名称，以查看容器的运行情况：

![运行 ngnix 容器](./media/virtual-machines-linux-dockerextension/nginxrunning.png)

有关 Docker VM 扩展的详细信息，例如配置 Docker 守护程序 TCP 端口、配置安全性、使用 Docker Compose 部署容器等主题，请参阅 [Azure Virtual Machine Extension for Docker GitHub project（适用于 Docker GitHub 项目的 Azure 虚拟机扩展）](https://github.com/Azure/azure-docker-extension/)。

## Docker VM 扩展 JSON 模板参考

本示例使用快速入门模板。可通过将以下内容添加到 JSON 定义文件，使用自己现有的 Resource Manager 模板将 Docker VM 扩展安装在模板定义的 VM 上：

	{
	  "type": "Microsoft.Compute/virtualMachines/extensions",
	  "name": "[concat(variables('vmName'), '/DockerExtension'))]",
	  "apiVersion": "2015-05-01-preview",
	  "location": "[parameters('location')]",
	  "dependsOn": [
	    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
	  ],
	  "properties": {
	    "publisher": "Microsoft.Azure.Extensions",
	    "type": "DockerExtension",
	    "typeHandlerVersion": "1.1",
	    "autoUpgradeMinorVersion": true,
	    "settings": {},
	    "protectedSettings": {}
	  }
	}

有关更详细的 Resource Manager 模板用法演练，请阅读 [Azure Resource Manager overview（Azure Resource Manager 概述）](/documentation/articles/resource-group-overview/)

## 后续步骤

阅读不同部署选项的详细步骤：

1. [通过 Azure 驱动程序使用 Docker 计算机](/documentation/articles/virtual-machines-linux-docker-machine/)  
2. [从 Azure 命令行界面 (Azure CLI) 使用 Docker VM 扩展](/documentation/articles/virtual-machines-linux-classic-cli-use-docker/)  

<!---HONumber=Mooncake_0613_2016-->