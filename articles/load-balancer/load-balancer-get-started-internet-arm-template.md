<properties 
   pageTitle="使用模板在 Resource Manager 中创建面向 Internet 的负载均衡器 | Azure"
   description="了解如何使用 ARM 模板在 Resource Manager 中创建面向 Internet 的负载均衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.date="02/09/2016"
   wacn.date="08/29/2016" />

# 开始使用 ARM 模板创建面向 Internet 的负载均衡器

[AZURE.INCLUDE [load-balancer-get-started-internet-arm-selectors-include.md](../../includes/load-balancer-get-started-internet-arm-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍资源管理器部署模型。你还可以[了解如何使用经典部署模型创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-classic-portal/)


[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## 通过使用单击部署来部署 ARM 模板

公共存储库中提供的示例模板采用包含用于生成上述方案的默认值的参数文件。若要使用单击部署来部署此模板，按照[此链接](http://go.microsoft.com/fwlink/?LinkId=544801)，单击“部署至 Azure”，如有必要替换默认参数值，然后按照门户中的说明进行操作。

## 使用 PowerShell 部署 ARM 模板

若要使用 PowerShell 部署下载的 ARM 模板，请执行以下步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。

3. 从浏览器导航到[模板文件](https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.json)，复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.json** 的文件，并将其中的 windows.net 路径全部修改为 chinacloudapi.cn。

4. 从浏览器导航到
 [TemplateParameterFile](https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.parameters.json)复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.parameters.json** 的文件，并将其中的 **GEN-UNIQUE** 修改为需要设定的值。

5. 运行 **New-AzureRmResourceGroupDeployment** cmdlet 以使用模板创建资源组。

		New-AzureRmResourceGroupDeployment -Name TestRG `
		    -TemplateFile 'c:\lb\azuredeploy.json' `
		    -TemplateParameterFile 'c:\lb\azuredeploy.parameters.json'

## 使用 Azure CLI 部署 ARM 模板

若要使用 Azure CLI 部署 ARM 模板，请执行下列步骤。

1. 如果你从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到资源管理器模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

3. 从浏览器导航到[模板文件](https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.json)，复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.json** 的文件，并将其中的 *windows.net* 路径全部修改为 *chinacloudapi.cn*。

4. 从浏览器导航到
 [TemplateParameterFile](https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.parameters.json)复制 json 文件内容并粘贴到计算机中的一个新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.parameters.json** 的文件，并将其中的 **GEN-UNIQUE** 修改为需要设定的值。

4. 运行 **azure group deployment create** cmdlet 以使用你在前面下载并修改的模板和参数文件部署新的负载均衡器。在输出后显示的列表说明了所用的参数。

		azure group create -n TestRG -l chinaeast -f 'c:\lb\azuredeploy.json' -e 'c:\lb\azuredeploy.parameters.json'

## 后续步骤

[开始配置内部负载均衡器](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0822_2016-->
