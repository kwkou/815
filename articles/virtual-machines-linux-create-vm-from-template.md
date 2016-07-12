<!-- ARM: tested -->

<properties
	pageTitle="使用 Azure 模板创建 Linux VM | Azure"
	description="使用 Azure Resource Manager 模板在 Azure 上创建 Linux VM。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="vlivech"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/29/2016"
	wacn.date="06/27/2016"/>

# 使用 Azure 模板创建 Linux VM

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

本文说明如何使用 Azure 模板在 Azure 上快速部署 Linux 虚拟机。本文中的操作需要一个 Azure 帐户（[获取试用帐户](/pricing/1rmb-trial/)）、已登录 (`azure login -e AzureChinaCloud`) 且处于 Resource Manager 模式 (`azure config mode arm`) 的 [Azure CLI](/documentation/articles/xplat-cli-install/)。也可以使用 [Azure 门户预览](/documentation/articles/virtual-machines-linux-quick-create-portal/)或 [Azure CLI](/documentation/articles/virtual-machines-linux-quick-create-cli/) 快速部署 Linux VM。


## 快速命令

在以下命令示例中，请将 &lt; 与 &gt; 之间的值替换为你自己环境中的值。

可以从 [GitHub](https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json) 下载该模板并运行以下命令。

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

	azure group create \
	-n quicksecuretemplate \
	-l chinaeast \
	--template-file /path/to/azuredeploy.json

## 详细演练

模板可让你在 Azure 上使用要在启动期间自定义的设置（如用户名和主机名）创建 VM。本文着重在使用 Azure 模板启动 Ubuntu VM，此 VM 创建只打开一个端口（22，以用于 SSH）的网络安全组 (NSG)，且需要 SSH 密钥才能登录。

Azure Resource Manager 模板是一个 JSON 文件，可用于简单的一次性任务（例如本文中所述的启动 Ubuntu VM），或用于建构涵盖整个环境的复杂 Azure 配置（例如从网络、OS 到应用程序堆栈部署的测试、开发或生产部署）。

## 创建 Linux VM

以下代码示例演示如何调用 `azure group create`，以使用[此 Azure Resource Manager 模板](https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json)同时创建资源组和部署受 SSH 保护的 Linux VM。请记住，在示例中必须使用环境中唯一的名称。本示例使用 `quicksecuretemplate` 作为资源组名称，使用 `securelinux` 作为 VM 名称，使用 `quicksecurelinux` 作为子域名。

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

	azure group create \
	-n quicksecuretemplate \
	-l chinaeast \
	--template-file /path/to/azuredeploy.json

输出

	info:    Executing command group create
	+ Getting resource group quicksecuretemplate
	+ Creating resource group quicksecuretemplate
	info:    Created resource group quicksecuretemplate
	info:    Supply values for the following parameters
	sshKeyData: ssh-rsa AAAAB3Nza<..ssh public key text..>VQgwjNjQ== vlivech@azure
	+ Initializing template configurations and parameters
	+ Creating a deployment
	info:    Created template deployment "azuredeploy"
	data:    Id:                  /subscriptions/<..subid text..>/resourceGroups/quicksecuretemplate
	data:    Name:                quicksecuretemplate
	data:    Location:            chinaeast
	data:    Provisioning State:  Succeeded
	data:    Tags: null
	data:
	info:    group create command OK

你可以使用 `--template-uri` 参数创建新的资源组和部署 VM，或者使用 `--template-file` 参数并以模板文件路径作为参数在本地下载或创建模板并传递模板。Azure CLI 将提示你输入模板所需的参数。

## 后续步骤

使用模板创建 Linux VM 后，建议你了解使用模板还可以部署其他哪些应用框架。

<!---HONumber=Mooncake_0620_2016-->