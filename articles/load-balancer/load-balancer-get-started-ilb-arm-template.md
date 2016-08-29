<properties 
   pageTitle="在 Resource Manager 中使用模板创建内部负载平衡器 | Azure"
   description="了解如何在 Resource Manager 中使用模板创建内部负载平衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.date="03/04/2016"
   wacn.date="08/29/2016" />  


# 开始使用模板创建内部负载平衡器

[AZURE.INCLUDE [load-balancer-get-started-ilb-arm-selectors-include.md](../../includes/load-balancer-get-started-ilb-arm-selectors-include.md)]
<BR>
[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/learn-about-deployment-models-rm-include.md)] [classic deployment model](/documentation/articles/load-balancer-get-started-ilb-classic-ps/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## 通过单击部署方式部署模板

公共存储库中提供的示例模板采用包含用于生成上述方案的默认值的参数文件。若要使用单击部署来部署此模板，按照[此链接](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-2-vms-internal-load-balancer%2Fazuredeploy.json)，单击“部署至 Azure”，如有必要替换默认参数值，然后按照门户中的说明进行操作。

## 使用 PowerShell 部署模板

若要使用 PowerShell 部署下载的模板，请执行以下步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](/documentation/articles/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。


2. 将[参数](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.parameters.json)文件下载到本地磁盘。
3. 编辑该文件并将其保存。
4. 运行 **New-AzureRmResourceGroupDeployment** cmdlet 以使用模板创建资源组。


		New-AzureRmResourceGroupDeployment -Name TestRG -Location chinaeast `
		    -TemplateFile 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json' `
		    -TemplateParameterFile 'C:\temp\azuredeploy.parameters.json'
	
                
		
## 使用 Azure CLI 部署模板

若要使用 Azure CLI 部署模板，请执行以下步骤。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

3. 打开[参数文件](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.parameters.json)，选择其内容，然后将其保存到计算机上的文件中。对于本示例，我们将参数文件保存到 *parameters.json*。

4. 运行 **azure group deployment create** 命令以使用你在前面下载并修改的模板和参数文件部署新的内部负载平衡器。在输出后显示的列表说明了所用的参数。

		azure group create -n TestRG -l chinaeast --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json -e parameters.json


## 后续步骤

[使用源 IP 关联配置负载平衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载平衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0822_2016-->
