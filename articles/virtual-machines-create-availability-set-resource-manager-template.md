<properties
	pageTitle="使用 Azure 资源管理器模板创建可用性集"
	description="介绍如何使用可用性集模板，并提供了模板语法"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor=""/>

<tags ms.service="virtual-machines"
	  ms.date="05/04/2015"
      wacn.date="08/29/2015"/>

# 使用 Azure 资源管理器模板创建可用性集

你可以使用 Azure PowerShell 或 Azure 命令行 (CLI) 与资源管理器模板，轻松为虚拟机创建可用性集。此模板将创建一个可用性集。

在开始之前，请确保你已安装 Azure、PowerShell 和 Azure CLI 并已准备就绪。

[AZURE.INCLUDE [arm-getting-setup-powershell](../includes/arm-getting-setup-powershell.md)]

[AZURE.INCLUDE [xplat-getting-set-up](../includes/xplat-getting-set-up.md)]


## 使用资源管理器模板创建可用性集

按照以下步骤，配合 Azure PowerShell 使用 Github 模板存储库中的资源管理器模板为虚拟机创建一个可用性集。

### 步骤 1：下载 JSON 文件

指定一个本地文件夹作为 JSON 模板文件的位置，然后创建该文件夹（例如，C:\Azure\Templates\availability）。

替换文件夹名称，然后复制并运行这些命令。

	$folderName="<folder name, such as C:\Azure\Templates\availability>"
	$webclient = New-Object System.Net.WebClient
	$url = "https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-2-FDs-no-resource-loops/azuredeploy.json"
	$filePath = $folderName + "\azuredeploy.json"
	$webclient.DownloadFile($url,$filePath)

### 步骤 2：收集所需参数的详细信息

当你使用模板时，需要提供位置和集名称等详细信息。若要找出模板需要哪些参数，请执行以下操作之一：

- 在[此处](http://azure.microsoft.com/zh-cn/documentation/templates/201-2-vms-2-FDs-no-resource-loops/)查看参数列表。
- 在所选的工具或文本编辑器中打开 JSON 文件。在该文件顶部查找“parameters”节，其中列出了该模板要配置虚拟机所需的参数集。

收集所需的信息，以便到时可以输入。当你运行命令来部署模板时，系统将提示你输入这些信息。

### 步骤 3：创建可用性集

以下部分说明如何使用 Azure PowerShell 或 Azure CLI 来执行此操作。

### 使用 Azure PowerShell

填写 Azure 部署名称、资源组名称、Azure 位置以及你保存 JSON 文件的文件夹，然后运行以下命令。

	$deployName="<deployment name>"
	$RGName="<resource group name>"
	$locName="<Azure location, such as West US>"
	$folderName="<folder name, such as C:\Azure\Templates\availability>"
	$templateFile= $folderName + "\azuredeploy.json"
	New-AzureResourceGroup –Name $RGName –Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateFile $templateFile

当你运行 **New-AzureResourceGroupDeployment** 命令时，系统会提示你提供 JSON 文件的 **"parameters"** 节中的参数值。完成此操作后，命令将创建资源组和可用性集。

下面是模板的 PowerShell 命令集示例。

	$deployName="TestDeployment"
	$RGName="TestRG"
	$locname="West US"
	$folderName="C:\Azure\Templates[thing]"
	$templateFile= $folderName + "\azuredeploy.json"
	New-AzureResourceGroup –Name $RGName –Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateFile $templateFile

你将看到类似于下面的内容。

	cmdlet New-AzureResourceGroup at command pipeline position 1
	Supply values for the following parameters:
	(Type !? for Help.)
	newStorageAccountName: saTest
	dnsNameForPublicIP: 131.107.89.211
	adminUserName: WebAdmin1
	adminPassword: *******
	vmSourceImageName: a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201503.01-en.us-127GB.vhd
	...

若要删除此资源组及其所有资源（存储帐户、虚拟机和虚拟网络），请使用以下命令。

	Remove-AzureResourceGroup –Name "<resource group name>"


## 使用 Azure CLI

按照以下步骤，配合 Azure CLI 命令使用 Github 模板存储库中的资源管理器模板创建可用性集。

	azure group deployment create <my-resource-group> <my-deployment-name> --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-2-FDs-no-resource-loops/azuredeploy.json

<!---HONumber=67-->