<properties
    pageTitle="创建 Azure 应用程序网关 - 模板 | Azure"
    description="本页提供有关使用 Azure Resource Manager 模板创建 Azure 应用程序网关的说明"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="8192ee25-d9f0-4b32-a45e-1d74629c54e5"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/23/2017"
    wacn.date="05/22/2017"
    ms.author="gwallace"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="ed9ec5a3c0269b6cbc3ce84984845ebdf8c981f2"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="create-an-application-gateway-by-using-the-azure-resource-manager-template"></a>使用 Azure Resource Manager 模板创建应用程序网关
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/application-gateway-create-gateway-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-create-gateway-arm/)
- [Azure 经典 PowerShell](/documentation/articles/application-gateway-create-gateway/)
- [Azure Resource Manager 模板](/documentation/articles/application-gateway-create-gateway-arm-template/)
- [Azure CLI](/documentation/articles/application-gateway-create-gateway-cli/)

Azure 应用程序网关是第 7 层负载均衡器。 它在不同服务器之间提供故障转移和性能路由 HTTP 请求，而不管它们是在云中还是在本地。
应用程序网关提供许多应用程序传送控制器 (ADC) 功能，包括 HTTP 负载均衡、基于 cookie 的会话相关性、安全套接字层 (SSL) 卸载、自定义运行状况探测、多站点支持，以及许多其他功能。

若要查找支持功能的完整列表，请参阅[应用程序网关概述](/documentation/articles/application-gateway-introduction/)

了解如何从 GitHub 下载并修改现有 Azure Resource Manager 模板，以及如何通过 GitHub、PowerShell 和 Azure CLI 部署该模板。

如果只是直接从 GitHub 部署 Azure Resource Manager 模板，而不进行任何更改，请跳到“从 GitHub 部署模板”。

## <a name="scenario"></a>方案

在此方案中，将要：

* 创建具有 Web 应用程序防火墙的应用程序网关。
* 创建名为 VirtualNetwork1 且包含 10.0.0.0/16 保留 CIDR 块的虚拟网络。
* 创建名为 Appgatewaysubnet 且使用 10.0.0.0/28 作为其 CIDR 块的子网。
* 为需要用来进行流量负载均衡的 Web 服务器设置两个先前已配置的后端 IP。 在此模板示例中，后端 IP 是 10.0.1.10 和 10.0.1.11。

> [AZURE.NOTE]
> 这些设置是适用于此模板的参数。 若要自定义模板，可更改 azuredeploy.json 文件中的规则、侦听程序、SSL 以及其他选项。

![方案](./media/application-gateway-create-gateway-arm-template/scenario.png)

## <a name="download-and-understand-the-azure-resource-manager-template"></a>下载并了解 Azure Resource Manager 模板

可以从 GitHub 下载用于创建虚拟网络和两个子网的现有 Azure Resource Manager 模板，进行任何所需的更改，然后重用该模板。 为此，请使用以下步骤：

1. 导航到[创建启用了 Web 应用程序防火墙的应用程序网关](https://github.com/Azure/azure-quickstart-templates/tree/master/101-application-gateway-waf)。
1. 单击 **azuredeploy.json**，然后单击 **RAW**。
1. 将该文件保存到计算机上的本地文件夹中。
1. 如果熟悉 Azure Resource Manager 模板，则跳到步骤 7。
1. 打开保存的文件，并查看 **parameters** 下行中的内容
1. Azure Resource Manager 模板参数提供了在部署过程中可以填充的值的占位符。

    | 参数 | 说明 |
    | --- | --- |
    | **subnetPrefix** |应用程序网关子网的 CIDR 块。 |
    | **applicationGatewaySize** | 应用程序网关的大小。  WAF 仅允许中型和大型网关。 |
    | **backendIpaddress1** |第一个 Web 服务器的 IP 地址。 |
    | **backendIpaddress2** |第二个 Web 服务器的 IP 地址。 |
    | **wafEnabled** | 用于确定是否启用了 WAF 的设置。|
    | **wafMode** | Web 应用程序防火墙的模式。  可用选项有：“预防”或“检测”。|
    | **wafRuleSetType** | WAF 的规则集类型。  目前，OWASP 是唯一受支持的选项。 |
    | **wafRuleSetVersion** |规则集版本。 OWASP CRS 2.2.9 和 3.0 目前是支持的选项。 |

    > [AZURE.IMPORTANT]
    > 在 GitHub 中维护的 Azure Resource Manager 模板可能随着时间的推移发生变化。 请确保在使用该模板之前对其进行检查。

1. 检查 **resources** 下的内容，并注意以下属性：

    * **type**。 模板创建的资源的类型。 在这种情况下，类型为 `Microsoft.Network/applicationGateways`，它表示应用程序网关。
    * **name**。 资源的名称。 请注意 `[parameters('applicationGatewayName')]`的使用，这意味着该名称是在部署过程中由用户或参数文件作为输入提供的。
    * **properties**。 资源的属性列表。 此模板在应用程序网关创建过程中，使用虚拟网络与公共 IP 地址。

    > [AZURE.NOTE]
    > 有关模板的详细信息，请参阅：[Resource Manager 模板参考](https://github.com/Azure/azure-quickstart-templates/)

1. 导航回 [https://github.com/Azure/azure-quickstart-templates/blob/master/101-application-gateway-waf/](https://github.com/Azure/azure-quickstart-templates/blob/master/101-application-gateway-waf)。
1. 单击 **azuredeploy-parameters.json**，然后单击 **RAW**。
1. 将该文件保存到计算机上的本地文件夹中。
1. 打开保存的文件并编辑参数的值。 使用以下值部署本方案中所述的应用程序网关。

        {
            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "addressPrefix": {
                "value": "10.0.0.0/16"
                },
                "subnetPrefix": {
                "value": "10.0.0.0/28"
                },
                "applicationGatewaySize": {
                "value": "WAF_Medium"
                },
                "capacity": {
                "value": 2
                },
                "backendIpAddress1": {
                "value": "10.0.1.10"
                },
                "backendIpAddress2": {
                "value": "10.0.1.11"
                },
                "wafEnabled": {
                "value": true
                },
                "wafMode": {
                "value": "Detection"
                },
                "wafRuleSetType": {
                "value": "OWASP"
                },
                "wafRuleSetVersion": {
                "value": "3.0"
                }
            }
        }

1. 保存文件。 可以使用联机 JSON 验证工具（例如 [JSlint.com](http://www.jslint.com/)）测试 JSON 模板和参数模板。

## <a name="deploy-the-azure-resource-manager-template-by-using-powershell"></a>使用 PowerShell 部署 Azure Resource Manager 模板

如果从未使用过 Azure PowerShell，请参阅：[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)，并按照说明进行操作，登录到 Azure 并选择订阅。

### <a name="step-1"></a>步骤 1

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

### <a name="step-2"></a>步骤 2

检查该帐户的订阅。

    Get-AzureRmSubscription

系统会提示使用凭据进行身份验证。

### <a name="step-3"></a>步骤 3

选择要使用的 Azure 订阅。

    Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### <a name="step-4"></a>步骤 4

如有必要，请使用 **New-AzureResourceGroup** cmdlet 创建资源组。 在下面的示例中，将在“中国东部”位置创建名为 AppgatewayRG 的资源组。

    New-AzureRmResourceGroup -Name AppgatewayRG -Location "China North"

运行 **New-AzureRmResourceGroupDeployment** cmdlet，使用在前面下载并修改的模板和参数文件部署新虚拟网络。

    New-AzureRmResourceGroupDeployment -Name TestAppgatewayDeployment -ResourceGroupName AppgatewayRG `
    -TemplateFile C:\ARM\azuredeploy.json -TemplateParameterFile C:\ARM\azuredeploy-parameters.json

## <a name="deploy-the-azure-resource-manager-template-by-using-the-azure-cli"></a>使用 Azure CLI 部署 Azure Resource Manager 模板

若要使用 Azure CLI 部署下载的 Azure Resource Manager 模板，请执行以下步骤：

### <a name="step-1"></a>步骤 1

如果从未使用过 Azure CLI，请参阅[安装和配置 Azure CLI](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)，并按照说明进行操作，直到选择 Azure 帐户和订阅。

### <a name="step-2"></a>步骤 2

如有必要，请运行 `az group create` 命令创建一个资源组，如以下代码片段中所示。 请注意命令的输出。 在输出后显示的列表说明了所使用的参数。 有关资源组的详细信息，请访问 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

    az group create --location chinanorth --name appgatewayRG

**-n（或 --name）**。 新资源组的名称。 在本方案中为 *appgatewayRG*。

**-l（或 --location）**。 将创建新资源组的 Azure 区域。 在本方案中为 *chinanorth*。

### <a name="step-4"></a>步骤 4

运行 `az group deployment create` cmdlet，使用上述步骤中下载并修改的模板和参数文件部署新虚拟网络。 在输出后显示的列表说明了所用的参数。

    az group deployment create --resource-group appgatewayRG --name TestAppgatewayDeployment --template-file azuredeploy.json --parameters @azuredeploy-parameters.json

## <a name="deploy-the-azure-resource-manager-template-by-using-click-to-deploy"></a>使用“单击部署”来部署 Azure Resource Manager 模板

“单击部署”是另一种使用 Azure Resource Manager 模板的方式。 这是将模板与 Azure 门户预览配合使用的简便方法。

### <a name="step-1"></a>步骤 1

转到[创建具有 Web 应用程序防火墙的应用程序网关](https://github.com/Azure/azure-quickstart-templates/tree/master/101-application-gateway-waf/)。

### <a name="step-2"></a>步骤 2

单击 **“部署到 Azure”**。

[![“部署到 Azure”](./media/application-gateway-create-gateway-arm-template/deploytoazure.png)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-application-gateway-public-ip%2Fazuredeploy.json)

### <a name="step-3"></a>步骤 3

在门户上填写部署模板的参数，然后单击“确定”。

![Parameters](./media/application-gateway-create-gateway-arm-template/ibiza1.png)

### <a name="step-4"></a>步骤 4

选择“法律条款”，然后单击“购买”。

### <a name="step-5"></a>步骤 5

在“自定义部署”边栏选项卡上，单击“创建” 。

## <a name="providing-certificate-data-to-resource-manager-templates"></a>向 Resource Manager 模板提供证书数据

如果将 SSL 与模板一起使用，请提供 base64 字符串的证书，而不是上传证书。 若要将 .pfx 或 .cer 转换为 base64 字符串，请运行以下 PowerShell 命令。 此代码片段会证书将转换为 base64 字符串，以便将其提供给模板。 预期输出为一个字符串，它可以存储在变量中，并粘贴到模板中。

    [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("<certificate path and name>.pfx"))

## <a name="next-steps"></a>后续步骤

若要配置 SSL 卸载，请访问：[配置应用程序网关以进行 SSL 卸载](/documentation/articles/application-gateway-ssl/)。

若要将应用程序网关配置为与内部负载均衡器配合使用，请访问：[创建具有内部负载均衡器 (ILB) 的应用程序网关](/documentation/articles/application-gateway-ilb/)。

如需大体上更详细地了解负载均衡选项，请访问：

* [Azure 负载均衡器](/documentation/services/load-balancer/)
* [Azure 流量管理器](/documentation/services/traffic-manager/)

<!--Update_Description: wording update-->