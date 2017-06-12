<properties
    pageTitle="监视应用程序网关的访问、后端运行状况和指标 | Azure"
    description="了解如何启用和管理应用程序网关的访问日志"
    services="application-gateway"
    documentationcenter="na"
    author="amitsriva"
    manager="rossort"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid="300628b8-8e3d-40ab-b294-3ecc5e48ef98"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/17/2017"
    wacn.date="05/22/2017"
    ms.author="amitsriva"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="ed0f03066a8067b88152ea4fe9ab722732fb4384"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="backend-health-diagnostics-logging-and-metrics-for-application-gateway"></a>应用程序网关的后端运行状况、诊断日志记录和指标

Azure 提供使用日志记录和指标来监视资源的功能。 应用程序网关提供使用后端运行状况、日志记录和指标的相关功能。

[**后端运行状况**](#backend-health) - 应用程序网关提供通过门户和 powershell 监视后端池中的服务器运行状况的功能。

[**日志记录**](#enable-logging-with-powershell) - 通过日志记录，可出于监视目的从资源保存或使用访问及其他日志。

[**度量值**](#metrics) - 应用程序网关当前有一个度量值。 此度量值可衡量应用程序网关的吞吐量，以每秒字节数为单位。

## <a name="backend-health"></a>后端运行状况

应用程序网关提供通过门户、PowerShell 和 CLI 监视后端池的各个成员的运行状况的功能。 后端运行状况报告反映了应用程序网关运行状况探测到后端实例的输出。 如果探测成功且可向后端提供流量，则可认为运行状况正常，否则不正常。

> [AZURE.IMPORTANT]
> 如果应用程序网关子网上存在 NSG，则应在入站流量的应用程序网关子网上打开端口范围 65503-65534。 这些端口是确保后端运行状况 API 正常工作所必需的。

### <a name="view-backend-health-through-the-portal"></a>通过门户查看后端运行状况

无需执行任何操作即可查看后端运行状况。 在现有的应用程序网关中，导航到“监视” > “后端运行状况”。 后端池中的每个成员都列在此页上（不管其是 NIC、IP 还是 FQDN）。 会显示后端池名称、端口、后端 http 设置名称以及运行状况。 运行状况的有效值为“正常”、“不正常”、“未知”。

> [AZURE.WARNING]
> 如果后端运行状况状态为“未知”，请确保网络安全组 (NSG) 规则或 VNet 中的自定义 DNS 未阻止访问后端。

![后端运行状况][10]

### <a name="view-backend-health-with-powershell"></a>使用 PowerShell 查看后端运行状况

后端运行状况也可通过 PowerShell 检索。 以下 PowerShell 代码显示了如何通过 `Get-AzureRmApplicationGatewayBackendHealth` cmdlet 拉取后端运行状况。

    Get-AzureRmApplicationGatewayBackendHealth -Name ApplicationGateway1 -ResourceGroupName Contoso

系统会返回结果。以下代码片段中显示了一个响应示例。

    {
    "BackendAddressPool": {
        "Id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/appGatewayBackendPool"
    },
    "BackendHttpSettingsCollection": [
        {
        "BackendHttpSettings": {
            "Id": "/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings"
        },
        "Servers": [
            {
            "Address": "hostname.chinanorth.chinacloudapp.cn",
            "Health": "Healthy"
            },
            {
            "Address": "hostname.chinanorth.chinacloudapp.cn",
            "Health": "Healthy"
            }
        ]
        }
    ]
    }

## <a name="diagnostic-logging"></a>诊断日志记录

可在 Azure 中使用不同类型的日志来对应用程序网关进行管理和故障排除。 可通过门户访问其中某些日志，且可从 Azure Blob 存储提取并在 Excel 和 PowerBI 等各种工具中查看所有日志。 可从以下列表了解有关不同类型日志的详细信息：

* **活动日志：**可以使用 [Azure 活动日志](/documentation/articles/insights-debugging-with-events/)（以前称为操作日志和审核日志）查看提交到 Azure 订阅的所有操作及其状态。 默认收集活动日志条目，并可在 Azure 门户预览中查看。
* **访问日志：** 可以使用此日志来查看应用程序网关访问模式并分析重要信息，包括调用方的 IP、请求的 URL、响应延迟、返回问代码、输入和输出字节数。 每隔 300 秒会收集一次访问日志。 此日志包含每个应用程序网关实例的一条记录。 应用程序网关实例可以由“instanceId”属性标识。
* **防火墙日志：** 可以使用此日志来查看通过应用程序网关的检测或阻止模式（通过 Web 应用程序防火墙配置）记录的请求。

> [AZURE.WARNING]
> 日志仅适用于在 Resource Manager 部署模型中部署的资源。 不能将日志用于经典部署模型中的资源。 若要更好地了解两种模型，请参考[了解 Resource Manager 部署和典型部署](/documentation/articles/resource-manager-deployment-model/)一文。

对于日志，若要存储日志，可使用两个不同的选项。

* 存储帐户 - 如果日志存储时间较长并且需要进行查看，最好使用存储帐户。
* 事件中心 - 若要集成其他 SEIM 工具以获取对资源的警报，最好使用事件中心

### <a name="enable-logging-with-powershell"></a>使用 PowerShell 启用日志记录

每个 Resource Manager 资源都会自动启用活动日志记录。 必须启用访问日志记录才能开始收集通过这些日志提供的数据。 若要启用日志记录，请参阅以下步骤：

1. 记下存储帐户的资源 ID，其中存储日志数据。 此值的形式如下：/subscriptions/\<subscriptionId\>/resourceGroups/\<resource group name\>/providers/Microsoft.Storage/storageAccounts/\<存储帐户名称\>。 订阅中的所有存储帐户均可使用。 可以使用门户预览来查找此信息。

    ![门户预览 - 应用程序网关诊断](./media/application-gateway-diagnostics/diagnostics1.png)

2. 记下应用程序网关的资源 ID（会为其启用日志记录）。 此值的形式如下：/subscriptions/\<subscriptionId\>/resourceGroups/\<resource group name\>/providers/Microsoft.Network/applicationGateways/\<应用程序网关名称\>。 可以使用门户预览来查找此信息。

    ![门户预览 - 应用程序网关诊断](./media/application-gateway-diagnostics/diagnostics2.png)

3. 使用以下 PowerShell cmdlet 启用诊断日志记录：

        Set-AzureRmDiagnosticSetting  -ResourceId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name> -StorageAccountId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name> -Enabled $true     

> [AZURE.TIP] 
>活动日志不需要单独的存储帐户。 使用存储来记录访问需支付服务费用。

### <a name="activity-log"></a>活动日志

Azure 默认生成此日志（以前称为“操作日志”）。  日志在 Azure 的事件日志存储区中保留 90 天。 若要深入了解这些日志，请阅读[查看事件和活动日志](/documentation/articles/insights-debugging-with-events/)一文。

### <a name="access-log"></a>访问日志

只有按照上述步骤基于每个应用程序网关启用了此日志，才会生成此日志。 数据存储在你启用日志记录时指定的存储帐户中。 应用程序网关的每次访问均以 JSON 格式记录下来，如以下示例所示：

    {
        "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
        "operationName": "ApplicationGatewayAccess",
        "time": "2016-04-11T04:24:37Z",
        "category": "ApplicationGatewayAccessLog",
        "properties": {
            "instanceId":"ApplicationGatewayRole_IN_0",
            "clientIP":"37.186.113.170",
            "clientPort":"12345",
            "httpMethod":"HEAD",
            "requestUri":"/xyz/portal",
            "requestQuery":"",
            "userAgent":"-",
            "httpStatus":"200",
            "httpVersion":"HTTP/1.0",
            "receivedBytes":"27",
            "sentBytes":"202",
            "timeTaken":"359",
            "sslEnabled":"off"
        }
    }

### <a name="firewall-log"></a>防火墙日志

只有按照上述步骤基于每个应用程序网关启用了此日志，才会生成此日志。 此日志还需要在应用程序网关上配置 Web 应用程序防火墙。 数据存储在你启用日志记录时指定的存储帐户中。 将记录以下数据：

    {
      "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
      "operationName": "ApplicationGatewayFirewall",
      "time": "2017-03-20T15:52:09.1494499Z",
      "category": "ApplicationGatewayFirewallLog",
      "properties": {
        "instanceId": "ApplicationGatewayRole_IN_0",
        "clientIp": "104.210.252.3",
        "clientPort": "4835",
        "requestUri": "/?a=%3Cscript%3Ealert(%22Hello%22);%3C/script%3E",
        "ruleSetType": "OWASP",
        "ruleSetVersion": "3.0",
        "ruleId": "941320",
        "message": "Possible XSS Attack Detected - HTML Tag Handler",
        "action": "Blocked",
        "site": "Global",
        "details": {
          "message": "Warning. Pattern match \"<(a|abbr|acronym|address|applet|area|audioscope|b|base|basefront|bdo|bgsound|big|blackface|blink|blockquote|body|bq|br|button|caption|center|cite|code|col|colgroup|comment|dd|del|dfn|dir|div|dl|dt|em|embed|fieldset|fn|font|form|frame|frameset|h1|head|h ...\" at ARGS:a.",
          "data": "Matched Data: <script> found within ARGS:a: <script>alert(\\x22hello\\x22);</script>",
          "file": "rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf",
          "line": "865"
        }
      }
    } 


### <a name="view-and-analyze-the-activity-log"></a>查看和分析活动日志

可使用以下任意方法查看和分析活动日志数据：

* **Azure 工具：**通过 Azure PowerShell、Azure 命令行接口 (CLI)、Azure REST API 或 Azure 门户预览检索活动日志中的信息。  [使用 Resource Manager 的活动操作](/documentation/articles/resource-group-audit/)一文中详细介绍了每种方法的分步说明。
* **Power BI：** 如果尚无 [Power BI](https://powerbi.microsoft.com/pricing) 帐户，可免费试用。 使用 [适用于 Power BI 的 Azure 活动日志内容包](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-audit-logs/) ，可借助预配置的仪表板（直接使用或进行自定义）分析数据。

## <a name="view-and-analyze-the-access-performance-and-firewall-log"></a>查看并分析访问和防火墙日志

可以连接到存储帐户并检索访问日志的 JSON 日志条目。 下载 JSON 文件后，你可以将它们转换为 CSV 并在 Excel、PowerBI 或任何其他数据可视化工具中查看。

> [AZURE.TIP]
> 如果熟悉 Visual Studio 和更改 C# 中的常量和变量值的基本概念，则可以使用 GitHub 提供的[日志转换器工具](https://github.com/Azure-Samples/networking-dotnet-log-converter)。
> 
> 

## <a name="next-steps"></a>后续步骤

* [Visualize your Azure Activity Log with Power BI](http://blogs.msdn.com/b/powerbi/archive/2015/09/30/monitor-azure-audit-logs-with-power-bi.aspx) （使用 Power BI 直观显示 Azure 活动日志）博客文章。
* [View and analyze Azure Activity Logs in Power BI and more](https://azure.microsoft.com/zh-cn/blog/analyze-azure-audit-logs-in-powerbi-more/) （在 Power BI 和其他组件中查看和分析 Azure 活动日志）博客文章。

[1]: ./media/application-gateway-diagnostics/figure1.png
[2]: ./media/application-gateway-diagnostics/figure2.png
[3]: ./media/application-gateway-diagnostics/figure3.png
[4]: ./media/application-gateway-diagnostics/figure4.png
[5]: ./media/application-gateway-diagnostics/figure5.png
[6]: ./media/application-gateway-diagnostics/figure6.png
[7]: ./media/application-gateway-diagnostics/figure7.png
[8]: ./media/application-gateway-diagnostics/figure8.png
[9]: ./media/application-gateway-diagnostics/figure9.png
[10]: ./media/application-gateway-diagnostics/figure10.png

<!--Update_Description: wording update-->