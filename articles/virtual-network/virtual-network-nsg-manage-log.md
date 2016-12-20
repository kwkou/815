<properties 
   pageTitle="监视 NSG 的操作、事件和计数器 | Azure"
   description="了解如何为 NSG 启用计数器、事件和操作日志记录"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager"
/>
<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="07/14/2016"
   wacn.date="12/16/2016"
   ms.author="jdial" />

#网络安全组 (NSG) 的日志分析

可以在 Azure 中使用不同类型的日志对 NSG 进行管理和故障排除。可通过门户访问其中某些日志，且可从 Azure Blob 存储提取并在 Excel 和 PowerBI 等各种工具中查看所有日志。可从下表了解有关各种类型日志的详细信息。

- **审核日志：**可以使用 [Azure 审核日志](/documentation/articles/insights-debugging-with-events/)（以前称为操作日志）查看提交到 Azure 订阅的所有操作及其状态。审核日志默认情况下启用，并且可以在 Azure 门户预览中查看。
- **事件日志：**可以使用此日志查看基于 MAC 地址向虚拟机和实例角色应用的 NSG 规则。每隔 60 秒收集一次这些规则的状态。 
- **计数器日志：**可以使用此日志查看应用每个 NSG 规则拒绝或允许流量的次数。

>[AZURE.WARNING] 日志仅适用于 Resource Manager 部署模型中部署的资源。不能将日志用于经典部署模型中的资源。若要更好地了解两种模型，请参考[了解 Resource Manager 部署和典型部署](/documentation/articles/resource-manager-deployment-model/)一文。

##启用日志记录
始终为所有 Resource Manager 资源自动启用审核日志记录。需启用事件和计数器日志记录，才能开始收集通过这些日志提供的数据。若要启用日志记录，请按以下步骤操作。

1.  登录到 [Azure 门户预览](https://portal.azure.cn)。若尚无网络安全组，请[创建 NSG](/documentation/articles/virtual-networks-create-nsg-arm-ps/) 再继续。 

2.  在门户预览中，单击“浏览”>“网络安全组”。

	![门户预览 - 网络安全组](./media/virtual-network-nsg-manage-log/portal-enable1.png)

3. 选择现有网络安全组。

	![门户预览 - 网络安全组设置](./media/virtual-network-nsg-manage-log/portal-enable2.png)

4. 在“设置”边栏选项卡中，单击“诊断”，然后在“诊断”窗格中，单击“状态”旁边的“启用”。
5. 在“设置”边栏选项卡中，单击“存储帐户”，然后选择现有的存储帐户，或者创建新的存储帐户。  

>[AZURE.INFORMATION] 审核日志不需要单独的存储帐户。使用存储来记录事件和规则需支付服务费用。

6. 在“存储帐户”正下方的下拉列表中，选择是否需要记录事件和/或计数器，然后单击“保存”。

	![门户预览 - 诊断日志](./media/virtual-network-nsg-manage-log/portal-enable3.png)

## 审核日志
Azure 默认生成此日志（以前称为“操作日志”）。日志在 Azure 的事件日志存储区中保留 90 天。通过阅读[查看事件和审核日志](/documentation/articles/insights-debugging-with-events/)一文可了解有关这些日志的详细信息。

## 计数器日志
仅当如上所述按 NSG 启用此日志后，才会生成此日志。数据存储在启用日志记录时指定的存储帐户中。以 JSON 格式记录应用于资源的每个规则，如下所示。

	{
		"time": "2015-09-11T23:14:22.6940000Z",
		"systemId": "e22a0996-e5a7-4952-8e28-4357a6e8f0c5",
		"category": "NetworkSecurityGroupRuleCounter",
		"resourceId": "/SUBSCRIPTIONS/D763EE4A-9131-455F-8C5E-876035455EC4/RESOURCEGROUPS/INSIGHTOBONRP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/NSGINSIGHTOBONRP",
		"operationName": "NetworkSecurityGroupCounters",
		"properties": {
			"vnetResourceGuid":"{DD0074B1-4CB3-49FA-BF10-8719DFBA3568}",
			"subnetPrefix":"10.0.0.0/24",
			"macAddress":"001517D9C43C",
			"ruleName":"DenyAllOutBound",
			"direction":"Out",
			"type":"block",
			"matchedConnections":0
			}
	}

## 事件日志
仅当如上所述按 NSG 启用此日志后，才会生成此日志。数据存储在启用日志记录时指定的存储帐户中。将记录以下数据：

	{
		"time": "2015-09-11T23:05:22.6860000Z",
		"systemId": "e22a0996-e5a7-4952-8e28-4357a6e8f0c5",
		"category": "NetworkSecurityGroupEvent",
		"resourceId": "/SUBSCRIPTIONS/D763EE4A-9131-455F-8C5E-876035455EC4/RESOURCEGROUPS/INSIGHTOBONRP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/NSGINSIGHTOBONRP",
		"operationName": "NetworkSecurityGroupEvents",
		"properties": {
			"vnetResourceGuid":"{DD0074B1-4CB3-49FA-BF10-8719DFBA3568}",
			"subnetPrefix":"10.0.0.0/24",
			"macAddress":"001517D9C43C",
			"ruleName":"AllowVnetOutBound",
			"direction":"Out",
			"priority":65000,
			"type":"allow",
			"conditions":{
				"destinationPortRange":"0-65535",
				"sourcePortRange":"0-65535",
				"destinationIP":"10.0.0.0/8,172.16.0.0/12,169.254.0.0/16,192.168.0.0/16,168.63.129.16/32",
				"sourceIP":"10.0.0.0/8,172.16.0.0/12,169.254.0.0/16,192.168.0.0/16,168.63.129.16/32"
			}
		}
	}

## 查看和分析审核日志
可使用以下任意方法查看和分析审核日志数据：

- **Azure 工具：**通过 Azure PowerShell、Azure 命令行界面 (CLI)、Azure REST API 或 Azure 门户预览检索审计日志中的信息。[使用 Resource Manager 审核操作](/documentation/articles/resource-group-audit/)一文中详细介绍了每种方法的分步说明。
- **Power BI：**如果尚无 [Power BI](https://powerbi.microsoft.com/pricing) 帐户，可免费试用。使用[适用于 Power BI 的 Azure 审核日志内容包](https://support.powerbi.com/knowledgebase/articles/742695)，可以借助预配置的仪表板（直接使用或进行自定义）分析数据。

## 查看和分析计数器和事件日志 
需要连接到存储帐户并检索事件和计数器日志的 JSON 日志条目。下载 JSON 文件后，可将其转换为 CSV，并在 Excel、PowerBI 或任何其他数据可视化工具中进行查看。

>[AZURE.TIP] 如果你熟悉 Visual Studio 和更改 C# 中的常量和变量值的基本概念，可从 Github 获取并使用[日志转换器工具](https://github.com/Azure-Samples/networking-dotnet-log-converter)。

## 后续步骤

- [Visualize your Azure Audit Logs with Power BI](http://blogs.msdn.com/b/powerbi/archive/2015/09/30/monitor-azure-audit-logs-with-power-bi.aspx)（使用 Power BI 对 Azure 审核日志进行可视化）博客文章。
- [View and analyze Azure Audit Logs in Power BI and more](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/)（在 Power BI 等工具中查看和分析 Azure 审核日志）

<!---HONumber=Mooncake_Quality_Review_1202_2016-->