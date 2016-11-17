<properties 
   pageTitle="在 Resource Manager 中使用模板创建内部负载均衡器 | Azure"
   description="了解如何在 Resource Manager 中使用模板创建内部负载均衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="03/04/2016"
   wacn.date="08/29/2016"
   ms.author="sewhee" />  


# 开始使用模板创建内部负载均衡器

[AZURE.INCLUDE [load-balancer-get-started-ilb-arm-selectors-include.md](../../includes/load-balancer-get-started-ilb-arm-selectors-include.md)]
<BR>
[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代 [classic deployment model](/documentation/articles/load-balancer-get-started-ilb-classic-ps/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## 通过单击部署方式部署模板

公共存储库中提供的示例模板采用包含用于生成上述方案的默认值的参数文件。若要使用单击部署来部署此模板，按照[此链接](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-2-vms-internal-load-balancer%2Fazuredeploy.json)，单击“部署至 Azure”，如有必要替换默认参数值，然后按照门户中的说明进行操作。

## 使用 PowerShell 部署模板

若要使用 PowerShell 部署下载的模板，请执行以下步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。

2. 从浏览器导航到[模板文件](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json)，复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.json** 的文件，并将其中的 windows.net 路径全部修改为 chinacloudapi.cn。

3. 从浏览器导航到
 [模板参数文件](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.parameters.json)，复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.parameters.json** 的文件，并将其中的 **GEN-UNIQUE** 修改为需要设定的值。

4. 运行 **New-AzureRmResourceGroupDeployment** cmdlet 以使用模板创建资源组。

		New-AzureRmResourceGroupDeployment -Name TestRG `
		    -TemplateFile 'c:\lb\azuredeploy.json' `
		    -TemplateParameterFile 'c:\lb\azuredeploy.parameters.json'

		
## 使用 Azure CLI 部署模板

若要使用 Azure CLI 部署模板，请执行以下步骤。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

2. 从浏览器导航到[模板文件](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json)，复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.json** 的文件，并将其中的 windows.net 路径全部修改为 chinacloudapi.cn。

3. 从浏览器导航到
 [模板参数文件](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.parameters.json)，复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.parameters.json** 的文件，并将其中的 **GEN-UNIQUE** 修改为需要设定的值。


4. 运行 **azure group deployment create** 命令以使用你在前面下载并修改的模板和参数文件部署新的内部负载均衡器。在输出后显示的列表说明了所用的参数。

		azure group create -n TestRG -l chinaeast --template-uri 'c:\lb\azuredeploy.json' -e 'c:\lb\azuredeploy.parameters.json'


## 后续步骤

[使用源 IP 关联配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0822_2016-->
