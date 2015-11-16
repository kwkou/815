<properties 
   pageTitle="监视 NSG 的操作、事件和计数器 | Windows Azure"
   description="了解如何对 NSG 启用计数器、事件和操作日志记录"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor="tysonn"
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="10/02/2015"
	wacn.date="11/12/2015"/>

#网络安全组 (NSG) 的日志分析

可以在 Azure 中使用不同类型的日志对 NSG 进行管理和故障排除。这些日志中有些可以通过门户访问，并且所有日志都可以从 Azure blob 存储中提取并在不同工具（如 Excel 和 PowerBI）中查看。你可以了解有关下面的列表中不同类型日志的详细信息。

- **审核日志：**可以使用 Azure 审核日志)（以前称为操作日志）查看提交到你的 Azure 订阅的所有操作及其状态。审核日志默认启用，并且可以在 Azure 管理门户中查看。
- **事件日志：**可以使用此日志查看有哪些 NSG 规则已基于 MAC 地址应用于虚拟机和实例角色。每隔 60 秒收集一次这些规则的状态。 
- **计数器日志：**可以使用此日志查看应用每个 NSG 规则拒绝或允许流量的次数。

>[AZURE.WARNING]日志仅适用于在资源管理器部署模型中部署的资源。不能将日志用于经典部署模型中的资源。要更好地了解两种模型，可参考[了解资源管理器部署和典型部署](/documentation/articles/resource-manager-deployment-model)一文。


## 审核日志
默认情况下由 Azure 生成此日志（以前称为“操作日志”）。日志在 Azure 的事件日志存储区中保留 90 天。

## 计数器日志
只有你按照上述步骤基于每个 NSG 启用了该日志，才会生成该日志。数据存储在你启用日志记录时指定的存储帐户中。应用于资源的每个规则以 JSON 格式记录，如下所示。

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
只有你按照上述步骤基于每个 NSG 启用了该日志，才会生成该日志。数据存储在你启用日志记录时指定的存储帐户中。将记录以下数据：

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

##查看和分析审核日志
你可以使用任何以下方法查看和分析审核日志数据：

- **Azure tools：**通过 Azure PowerShell、Azure 命令行界面 (CLI)、Azure REST API 或 Azure 管理门户检索审计日志中的信息。
- **Power BI：**如果还没有 [Power BI](https://powerbi.microsoft.com/pricing) 帐户，你可以免费试用。使用[适用于 Power BI 的 Azure 审核日志内容包](https://support.powerbi.com/knowledgebase/articles/742695)，你可以借助预配置的仪表板（可直接使用或进行自定义）分析你的数据。

##查看和分析计数器和事件日志 
你需要连接到你的存储帐户并检索事件和计数器日志的 JSON 日志项。下载 JSON 文件后，你可以将它们转换为 CSV 并在 Excel、PowerBI 或任何其他数据可视化工具中查看。

>[AZURE.TIP]如果你熟悉 Visual Studio 和更改 C# 中的常量和变量值的基本概念，则可以使用 Github 提供的[日志转换器工具](https://github.com/Azure-Samples/networking-dotnet-log-converter)。

##其他资源

- [使用 Power BI 直观显示你的 Azure 审核日志](http://blogs.msdn.com/b/powerbi/archive/2015/09/30/monitor-azure-audit-logs-with-power-bi.aspx)博客文章。
- [查看和分析 Power BI 中的 Azure 审核日志及更多内容](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/)博客文章。

<!---HONumber=79-->