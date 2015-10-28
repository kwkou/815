<properties
	pageTitle="使用 Azure 资源管理器模板部署 SharePoint 场"
	description="使用资源管理器模板和 Azure 预览门户、Azure PowerShell 或 Azure CLI，轻松部署包含 3 个服务器或 9 个服务器的 SharePoint 场。"
	services="virtual-machines"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags 
	ms.service="virtual-machines"
	ms.date="06/29/2015"
	wacn.date="08/29/2015"/>

# 使用 Azure 资源管理器模板部署 SharePoint 场

按照本文中的说明，使用资源管理器模板部署一个新的包含 3 个服务器或 9 个服务器的 SharePoint Server 2013 场。

## 部署包含 3 个服务器的 SharePoint 场

对于基本的 SharePoint Server 2013 场，资源管理器模板将在 3 个不同子网上的新虚拟网络中创建 3 个虚拟机。

![](./media/virtual-machines-workload-template-sharepoint/three-server-sharepoint-farm.png)

可以使用 Azure 预览门户、Azure PowerShell 或 Azure CLI 运行模板。

### Azure 预览门户

要使用资源管理器模板和 Azure 预览门户部署此工作负荷，请单击[此处](https://manage.windowsazure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsharepoint-three-vm%2Fazuredeploy.json)。

![](./media/virtual-machines-workload-template-sharepoint/azure-portal-template.png)

1.	在“模板”窗格中，单击“保存”。
2.	单击“参数”。在“参数”窗格上，输入新值、从允许的值中选择，或者接受默认值，然后单击“确定”。
3.	如有需要，可单击“订阅”并选择正确的 Azure 订阅。
4.	单击“资源组”并选择现有的资源组。或者，单击“或新建”为此工作负荷创建新的资源组。
5.	如有需要，可单击“资源组位置”并选择正确的 Azure 位置。
6.	如有需要，可单击“法律条款”以查看使用模板的条款条件和协议。
7.	单击“创建”。

根据具体模板，Azure 可能需花费一些时间生成工作负荷。完成时，你的现有或新资源组中将出现一个新的包含 3 个服务器的 SharePoint 场。

### Azure PowerShell

在开始之前，请确保安装了正确版本的 Azure PowerShell 且已登录，并切换到新的“资源管理器”模式。有关详细信息，请单击[此处](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell#setting-up-powershell-for-resource-manager-templates)。

在以下命令集中填写 Azure 部署名称、新的资源组名称以及 Azure 数据中心位置。删除引号内的所有内容，包括 < and > 字符。

	$deployName="<deployment name>"
	$RGName="<resource group name>"
	$locName="<Azure location, such as West US>"
	$templateURI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-three-vm/azuredeploy.json"
	New-AzureResourceGroup -Name $RGName -Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

下面是一个示例。

	$deployName="TestDeployment"
	$RGName="TestRG"
	$locname="West US"
	$templateURI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-three-vm/azuredeploy.json"
	New-AzureResourceGroup -Name $RGName -Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

接下来，在 Azure PowerShell 提示符中运行命令块。

当你运行 **New-AzureResourceGroupDeployment** 命令时，系统会提示你提供一系列参数的值。指定了所有参数值后，**New-AzureResourceGroupDeployment** 将创建和配置虚拟机。

当模板执行完成时，你的新资源组中将出现一个新的包含 3 个服务器的 SharePoint 场。

### Azure CLI

在开始之前，请确保安装了正确版本的 Azure CLI 且已登录，并切换到新的“资源管理器”模式。有关详细信息，请单击[此处](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli#getting-ready)。

首先，创建新的资源组。使用以下命令并指定组名称，以及要向其中部署的 Azure 数据中心位置。

	azure group create <group name> <location>

下一步，使用以下命令并指定新资源组的名称以及 Azure 部署的名称。

	azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-three-vm/azuredeploy.json <group name> <deployment name>

下面是一个示例。

	azure group create sp3serverfarm eastus2
	azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-three-vm/azuredeploy.json sp3serverfarm spdevtest

当你运行 **azure group deployment create** 命令时，系统会提示你提供一系列参数的值。指定了所有参数值后，Azure 将创建和配置虚拟机。

现在，你的新资源组中将出现一个新的包含 3 个服务器的 SharePoint 场。

##部署包含 9 个服务器的 SharePoint 场

对于高可用性的 SharePoint Server 2013 场，资源管理器模板将在 4 个不同子网上的新虚拟网络中创建 9 个虚拟机。

![](./media/virtual-machines-workload-template-sharepoint/nine-server-sharepoint-farm.png)

### Azure 预览门户

要使用资源管理器模板和 Azure 预览门户部署此工作负荷，请单击[此处](https://manage.windowsazure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsharepoint-server-farm-ha%2Fazuredeploy.json)。

![](./media/virtual-machines-workload-template-sharepoint/azure-portal-template.png)

1.	在“模板”窗格中，单击“保存”。
2.	单击“参数”。在“参数”窗格上，输入新值、从允许的值中选择，或者接受默认值，然后单击“确定”。
3.	如有需要，可单击“订阅”并选择正确的 Azure 订阅。
4.	单击“资源组”并选择现有的资源组。或者，单击“或新建”为此工作负荷创建新的资源组。
5.	如有需要，可单击“资源组位置”并选择正确的 Azure 位置。
6.	如有需要，可单击“法律条款”以查看使用模板的条款条件和协议。
7.	单击“创建”。

根据具体模板，Azure 可能需花费一些时间生成工作负荷。完成时，你的现有或新资源组中将出现一个新的包含 9 个服务器的 SharePoint 场。

### Azure PowerShell

在开始之前，请确保安装了正确版本的 Azure PowerShell 且已登录，并切换到新的“资源管理器”模式。有关详细信息，请单击[此处](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell#setting-up-powershell-for-resource-manager-templates)。

在以下命令集中填写 Azure 部署名称、新的资源组名称以及 Azure 数据中心位置。删除引号内的所有内容，包括 < and > 字符。

	$deployName="<deployment name>"
	$RGName="<resource group name>"
	$locName="<Azure location, such as West US>"
	$templateURI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-server-farm-ha/azuredeploy.json"
	New-AzureResourceGroup -Name $RGName -Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

下面是一个示例。

	$deployName="TestDeployment"
	$RGName="TestRG"
	$locname="West US"
	$templateURI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-server-farm-ha/azuredeploy.json"
	New-AzureResourceGroup -Name $RGName -Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

接下来，在 Azure PowerShell 提示符中运行命令块。

当你运行 **New-AzureResourceGroupDeployment** 命令时，系统会提示你提供一系列参数的值。指定了所有参数值后，**New-AzureResourceGroupDeployment** 将创建和配置虚拟机。

当模板执行完成时，你的新资源组中将出现一个新的包含 9 个服务器的 SharePoint 场。

### Azure CLI

在开始之前，请确保安装了正确版本的 Azure CLI 且已登录，并切换到新的“资源管理器”模式。有关详细信息，请单击[此处](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli#getting-ready)。

首先，创建一个新的资源组。使用以下命令并指定组名称，以及要向其中部署的 Azure 数据中心位置。

	azure group create <group name> <location>

下一步，使用以下命令并指定新资源组的名称以及 Azure 部署的名称。

	azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-server-farm-ha/azuredeploy.json <group name> <deployment name>

下面是一个示例。

	azure group create sphaserverfarm eastus2
	azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/sharepoint-server-farm-ha/azuredeploy.json sphaserverfarm spdevtest

当你运行 **azure group deployment create** 命令时，系统会提示你提供一系列参数的值。指定了所有参数值后，Azure 将创建和配置虚拟机。

当模板执行完成时，现在你的新资源组中将出现一个新的包含 9 个服务器的 SharePoint Server 2013 场。


## 其他资源

[使用 Azure 资源管理器模板和 Azure PowerShell 部署和管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)

[Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)

<!--[-->Azure 资源管理器概述<!--](/documentation/articles/resource-group-overview)-->

[使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli)

[虚拟机文档](http://www.windowsazure.cn/documentation/services/virtual-machines/)

[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)

<!---HONumber=67-->