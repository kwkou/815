<properties
    pageTitle="创建面向 Internet 的负载均衡器 - Azure 模板 | Azure"
    description="了解如何使用模板在 Resource Manager 中创建面向 Internet 的负载均衡器"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    tags="azure-resource-manager" />
<tags
    ms.assetid="b24f4729-4559-4458-8527-71009d242647"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="03/03/2017"
    ms.author="kumud" />  


# 使用模板创建面向 Internet 的负载均衡器

> [AZURE.SELECTOR]
- [门户](/documentation/articles/load-balancer-get-started-internet-portal/)
- [PowerShell](/documentation/articles/load-balancer-get-started-internet-arm-ps/)
- [Azure CLI](/documentation/articles/load-balancer-get-started-internet-arm-cli/)
- [模板](/documentation/articles/load-balancer-get-started-internet-arm-template/)

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文介绍资源管理器部署模型。你还可以[了解如何使用经典部署模型创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-classic-portal/)

[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## 通过单击部署方式部署模板

公共存储库中提供的示例模板采用包含用于生成上述方案的默认值的参数文件。若要使用单击部署来部署此模板，按照[此链接](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-loadbalancer-natrules)，单击“部署至 Azure”，如有必要替换默认参数值，然后按照门户中的说明进行操作。

## 使用 PowerShell 部署模板

若要使用 PowerShell 部署下载的模板，请执行以下步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。
2. 运行 **New-AzureRmResourceGroupDeployment** cmdlet 以使用模板创建资源组。

        New-AzureRmResourceGroupDeployment -Name TestRG -Location chinanorth `
            -TemplateFile 'https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.json' `
            -TemplateParameterFile 'https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.parameters.json'

## 使用 Azure CLI 部署模板

若要使用 Azure CLI 部署模板，请执行以下步骤。

1. 如果从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到 Resource Manager 模式，如下所示。

		azure config mode arm

	下面是上述命令的预期输出：

		info:    New mode is arm

3. 从浏览器导航到[快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-loadbalancer-lbrules)，复制 json 文件的内容并粘贴到计算机中的新文件。对于此方案，将下面的值复制到名为 **c:\\lb\\azuredeploy.parameters.json** 的文件。
4. 运行 **azure group deployment create** cmdlet 以使用你在前面下载并修改的模板和参数文件部署新的负载均衡器。在输出后显示的列表说明了所用的参数。

        azure group create --name TestRG --location chinanorth --template-file 'https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.json' --parameters-file 'c:\lb\azuredeploy.parameters.json'

## 后续步骤

[开始配置内部负载均衡器](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description:update meta properties, wording update; update code with full-paremeter format -->