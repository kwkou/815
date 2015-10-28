<properties
	pageTitle="使用 Azure 资源管理器模板，部署高度可用的“Active Directory 域服务”域"
	description="使用资源管理器模板和 Azure 预览门户、Azure PowerShell 或 Azure CLI，轻松部署两个服务器以用作“Active Directory 域服务”域控制器。"
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


# 使用 Azure 资源管理器模板，部署高度可用的“Active Directory 域服务”域

按照本文中的说明，使用资源管理器模板部署高度可用的 Active Directory 域。此模板将在相同子网上的新虚拟网络中创建两个虚拟机。

![](./media/virtual-machines-workload-template-ad-domain/two-server-ad.png)

可以使用 Azure 预览门户、Azure PowerShell 或 Azure CLI 运行模板。

## Azure 预览门户

要使用资源管理器模板和 Azure 预览门户部署此工作负荷，请单击[此处](https://manage.windowsazure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Factive-directory-new-domain-ha-2-dc%2Fazuredeploy.json)。

![](./media/virtual-machines-workload-template-ad-domain/azure-portal-template.png)

1.	在“模板”窗格中，单击“保存”。
2.	单击“参数”。在“参数”窗格上，输入新值、从允许的值中选择，或者接受默认值，然后单击“确定”。
3.	如有需要，可单击“订阅”并选择正确的 Azure 订阅。
4.	单击“资源组”并选择现有的资源组。或者，单击“或新建”为此工作负荷创建新的资源组。
5.	如有需要，可单击“资源组位置”并选择正确的 Azure 位置。
6.	如有需要，可单击“法律条款”以查看使用模板的条款条件和协议。
7.	单击“创建”。

根据具体模板，Azure 可能需花费一些时间生成工作负荷。当模板执行完成时，你的现有或新资源组中将出现一个新的双服务器 Active Directory 域。

## Azure PowerShell

在开始之前，请确保安装了正确版本的 Azure PowerShell 且已登录，并切换到新的“资源管理器”模式。有关详细信息，请单击[此处](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell#setting-up-powershell-for-resource-manager-templates)。

在以下命令集中填写 Azure 部署名称、新的资源组名称以及 Azure 数据中心位置。删除引号内的所有内容，包括 < and > 字符。

	$deployName="<deployment name>"
	$RGName="<resource group name>"
	$locName="<Azure location, such as West US>"
	$templateURI="https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/active-directory-new-domain-ha-2-dc/azuredeploy.json"
	New-AzureResourceGroup -Name $RGName -Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

下面是一个示例。

	$deployName="TestDeployment"
	$RGName="TestRG"
	$locname="West US"
	$templateURI="https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/active-directory-new-domain-ha-2-dc/azuredeploy.json"
	New-AzureResourceGroup -Name $RGName -Location $locName
	New-AzureResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

接下来，在 Azure PowerShell 提示符中运行命令块。

当你运行 **New-AzureResourceGroupDeployment** 命令时，系统会提示你提供一系列参数的值。指定了所有参数值后，**New-AzureResourceGroupDeployment** 将创建和配置虚拟机。

当模板执行完成时，你的新资源组中将出现一个新的双服务器 Active Directory 域配置。

## Azure CLI

在开始之前，请确保安装了正确版本的 Azure CLI 且已登录，并切换到新的“资源管理器”模式。有关详细信息，请单击[此处](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli#getting-ready)。

首先，创建新的资源组。使用以下命令并指定组名称，以及要向其中部署的 Azure 数据中心位置。

	azure group create <group name> <location>

下一步，使用以下命令并指定新资源组的名称以及 Azure 部署的名称。

	azure group deployment create --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/active-directory-new-domain-ha-2-dc/azuredeploy.json <group name> <deployment name>

下面是一个示例。

	azure group create adtestbed eastus2
	azure group deployment create --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/active-directory-new-domain-ha-2-dc/azuredeploy.json adtestbed wldevtest

当你运行 **azure group deployment create** 命令时，系统会提示你提供一系列参数的值。指定了所有参数值后，Azure 将创建和配置虚拟机。

当模板执行完成时，你的新资源组中将出现一个新的双服务器 Active Directory 域服务域配置。


## 其他资源

[使用 Azure 资源管理器模板和 Azure PowerShell 部署和管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)

[Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)

<!--[-->Azure 资源管理器概述<!--](/documentation/articles/resource-group-overview)-->

[使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli)

[虚拟机文档](http://www.windowsazure.cn/documentation/services/virtual-machines/)

[How to install and configure Azure PowerShell](/documentation/articles/install-configure-powershell)

<!---HONumber=67-->