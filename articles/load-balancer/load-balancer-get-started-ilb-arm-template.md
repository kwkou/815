<properties
    pageTitle="创建内部负载均衡器 - Azure 模板 | Azure"
    description="了解如何在 Resource Manager 中使用模板创建内部负载均衡器"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="64150862-6ced-42de-85dc-89d323257d7c"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="04/17/2017"
    ms.author="kumud"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="0e8f0dda2b9a24fe0ac7bc9a85903ebf9f06c573"
    ms.lasthandoff="04/07/2017" />

# <a name="create-an-internal-load-balancer-using-a-template"></a>使用模板创建内部负载均衡器
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/load-balancer-get-started-ilb-arm-portal/)
- [PowerShell](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)
- [Azure CLI](/documentation/articles/load-balancer-get-started-ilb-arm-cli/)
- [模板](/documentation/articles/load-balancer-get-started-ilb-arm-template/)

[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [AZURE.NOTE]
> Azure 具有用于创建和处理资源的两个不同的部署模型：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是使用[经典部署模型](/documentation/articles/load-balancer-get-started-ilb-classic-ps/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## <a name="deploy-the-template-by-using-click-to-deploy"></a>通过单击部署方式部署模板

公共存储库中提供的示例模板采用包含用于生成上述方案的默认值的参数文件。 若要通过单击部署的方式来部署此模板，请访问[此链接](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-internal-load-balancer)，单击“部署至 Azure”，如有必要，请替换默认参数值，然后按照门户中的说明进行操作。

## <a name="deploy-the-template-by-using-powershell"></a>使用 PowerShell 部署模板

若要使用 PowerShell 部署下载的模板，请执行以下步骤。

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) （如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。
2. 将参数文件下载到本地磁盘。
3. 编辑该文件并将其保存。
4. 运行 **New-AzureRmResourceGroupDeployment** cmdlet 以使用模板创建资源组。

        New-AzureRmResourceGroupDeployment -Name TestRG -Location chinanorth `
            -TemplateFile 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json' `
            -TemplateParameterFile 'C:\temp\azuredeploy.parameters.json'

## <a name="deploy-the-template-by-using-the-azure-cli"></a>使用 Azure CLI 部署模板

若要使用 Azure CLI 部署模板，请执行以下步骤。

1. 如果从未使用过 Azure CLI，请参阅 [安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) ，并按照说明进行操作，直到选择 Azure 帐户和订阅。
2. 运行 **azure config mode** 命令以切换到 Resource Manager 模式，如下所示。

        azure config mode arm

    下面是上述命令的预期输出：

        info:    New mode is arm

3. 打开参数文件，选择其内容，然后将其保存到计算机上的文件中。 对于本示例，我们将参数文件保存到 *parameters.json*。
4. 运行 **azure group deployment create** 命令以使用你在前面下载并修改的模板和参数文件部署新的内部负载均衡器。 在输出后显示的列表说明了所用的参数。

        azure group create --name TestRG --location chinanorth --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json --parameters-file parameters.json

## <a name="next-steps"></a>后续步骤

[使用源 IP 关联配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[配置负载均衡器的空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)


<!--Update_Description:update meta properties; wording update -->