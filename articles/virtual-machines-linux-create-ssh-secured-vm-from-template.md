<!-- ARM: tested -->

<properties
	pageTitle="使用 Azure 模板创建安全 Linux VM | Azure"
	description="使用 Azure Resource Manager 模板在 Azure 上创建安全 Linux VM。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="vlivech"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/27/2016"
	wacn.date="06/29/2016"/>

# 使用 Azure 模板创建受保护的 Linux VM

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

若要从模板创建 Linux VM，需要使用 Resource Manager 模式的 [Azure CLI](/documentation/articles/xplat-cli-install/) (`azure config mode arm`)。

## 快速命令摘要

	chrisl@fedora$ azure group create -n <exampleRGname> -l <exampleAzureRegion> [--template-uri <URL> | --template-file <path> | <template-file> -e <parameters.json file>]

## 详细演练

模板可让你在 Azure 上使用要在启动期间自定义的设置（如用户名和主机名）创建 VM。本文着重在使用 Azure 模板启动 Ubuntu VM，此 VM 创建只打开一个端口（22，以用于 SSH）的网络安全组 (NSG)，且需要 SSH 密钥才能登录。

Azure Resource Manager 模板是一个 JSON 文件，可用于简单的一次性任务（例如本文中所述的启动 Ubuntu VM），或用于建构涵盖整个环境的复杂 Azure 配置（例如从网络、OS 到应用程序堆栈部署的测试、开发或生产部署）。

## 创建 Linux VM

以下代码示例演示如何调用 `azure group create`，以使用[此 Azure Resource Manager 模板](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json)同时创建资源组和部署受 SSH 保护的 Linux VM。请记住，在示例中必须使用环境中唯一的名称。本示例使用 `quicksecuretemplate` 作为资源组名称，使用 `securelinux` 作为 VM 名称，使用 `quicksecurelinux` 作为子域名。

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

	chrisl@fedora$ azure group create -n quicksecuretemplate -l chinaeast --template-file /path/to/azuredeploy.json
	info:    Executing command group create
	+ Getting resource group quicksecuretemplate
	+ Creating resource group quicksecuretemplate
	info:    Created resource group quicksecuretemplate
	info:    Supply values for the following parameters
	adminUserName: ops
	sshKeyData: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDDRZ/XB8p8uXMqgI8EoN3dWQw... user@contoso.com
	dnsLabelPrefix: quicksecurelinux
	vmName: securelinux
	+ Initializing template configurations and parameters
	+ Creating a deployment
	info:    Created template deployment "azuredeploy"
	data:    Id:                  /subscriptions/<guid>/resourceGroups/quicksecuretemplate
	data:    Name:                quicksecuretemplate
	data:    Location:            chinaeast
	data:    Provisioning State:  Succeeded
	data:    Tags: null
	data:
	info:    group create command OK

你可以使用 `--template-uri` 参数创建新的资源组和部署 VM，或者使用 `--template-file` 参数并以模板文件路径作为参数在本地下载或创建模板并传递模板。Azure CLI 将提示你输入模板所需的参数。

## 后续步骤

使用模板创建 Linux VM 后，你需要了解还有哪些应用框架可与模板配合使用。

<!---HONumber=Mooncake_0425_2016-->