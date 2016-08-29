<properties 
   pageTitle="监视 Load Balancer 的操作、事件和计数器 | Azure"
   description="了解如何为 Azure Load Balancer 启用警报事件以及探测运行状况日志记录"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager"
/>
<tags 
   ms.service="load-balancer"
   ms.date="04/05/2016"
   wacn.date="08/29/2016" />

# 用于 Azure Load Balancer 的 Log Analytics（预览版）
可以在 Azure 中使用不同类型的日志对负载平衡器进行管理和故障排除。这些日志中有些可以通过门户访问，并且所有日志都可以从 Azure blob 存储中提取并在不同工具（如 Excel 和 PowerBI）中查看。你可以了解有关下面的列表中不同类型日志的详细信息。


- **审核日志：**可以使用 [Azure 审核日志](/documentation/articles/insights-debugging-with-events)（以前称为操作日志）查看提交到你的 Azure 订阅的所有操作及其状态。审核日志默认情况下启用，并且可以在 Azure 门户中查看。
- **警报事件日志：**可以使用此日志来查看针对负载平衡器发出的警报。每隔五分钟收集一次负载平衡器的状态。仅在引发了负载平衡器警报事件的情况下，才会向此日志写入相关内容。
- **运行状况探测日志：**可以使用此日志来查看探测运行状况时的检查状态、负载平衡器后端处于联机状态的实例的数目，以及从负载平衡器接收网络流量的虚拟机的百分比。探测状态事件变化时，将会向此日志写入相应内容。

>[AZURE.WARNING] 日志仅适用于在资源管理器部署模型中部署的资源。不能将日志用于经典部署模型中的资源。要更好地了解两种模型，可参考[了解资源管理器部署和典型部署](/documentation/articles/resource-manager-deployment-model/)一文。<BR> 
Log Analytics 当前仅适用于面向 Internet 的负载平衡器。该限制是暂时性的，以后随时可能更改。请务必重新访问此页，以了解将来发生的更改。

## 启用日志记录
所有 Resource Manager 资源都会始终自动启用审核日志记录。需启用事件和运行状况探测日志记录才能开始收集通过这些日志提供的数据。若要启用日志记录，请按以下步骤操作。

登录到 [Azure 门户](http://portal.azure.cn)。如果你还没有负载平衡器，请先[创建负载平衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)，然后继续。

在门户中，单击“浏览”>>“负载平衡器”。

![门户 - load-balancer](./media/load-balancer-monitor-log/load-balancer-browse.png)

选择现有的负载平衡器，然后单击“所有设置”。

![门户 - load-balancer-settings](./media/load-balancer-monitor-log/load-balancer-settings.png) 
<BR>

在“设置”边栏选项卡中，单击“诊断”，然后在“诊断”窗格中，单击“状态”旁边的“启用”。
在“设置”边栏选项卡中，单击“存储帐户”，然后选择现有的存储帐户，或者新建一个。

在“存储帐户”正下方的下拉列表中，选择是否需要记录警报事件和/或探测运行状况，然后单击“保存”。

![预览门户 - 诊断日志](./media/load-balancer-monitor-log/load-balancer-diagnostics.png)  


>[AZURE.INFORMATION] 审核日志不需要单独的存储帐户。使用存储来记录事件和运行状况探测需支付服务费用。

## 审核日志
默认情况下由 Azure 生成此日志（以前称为“操作日志”）。日志在 Azure 的事件日志存储区中保留 90 天。通过阅读[查看事件和审核日志](/documentation/articles/insights-debugging-with-events/)一文可了解有关这些日志的详细信息。

## 警报事件日志
只有你按照上述详细步骤基于每个负载平衡器启用了该日志，才会生成该日志。数据存储在你启用日志记录时指定的存储帐户中。如下所示，将以 JSON 格式记录信息。

	
	{
    "time": "2016-01-26T10:37:46.6024215Z",
	"systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
    "category": "LoadBalancerAlertEvent",
    "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
    "operationName": "LoadBalancerProbeHealthStatus",
    "properties": {
        "eventName": "Resource Limits Hit",
        "eventDescription": "Ports exhausted",
        "eventProperties": {
            "public ip address": "40.117.227.32"
        }
    }
	

JSON 输出显示的 *eventname* 属性将说明负载平衡器创建警报的原因。在本示例中，生成警报是因为源 IP NAT 限制 (SNAT) 导致 TCP 端口耗竭。

## 运行状况探测日志
只有你按照上述详细步骤基于每个负载平衡器启用了该日志，才会生成该日志。数据存储在你启用日志记录时指定的存储帐户中。创建了名为“insights-logs-loadbalancerprobehealthstatus”的容器并记录了以下数据：

		{
	    "records":

	    {
	   		"time": "2016-01-26T10:37:46.6024215Z",
	        "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
	        "category": "LoadBalancerProbeHealthStatus",
	        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
	        "operationName": "LoadBalancerProbeHealthStatus",
	        "properties": {
	            "publicIpAddress": "40.83.190.158",
	            "port": "81",
	            "totalDipCount": 2,
	            "dipDownCount": 1,
	            "healthPercentage": 50.000000
	        }
	    },
	    {
	        "time": "2016-01-26T10:37:46.6024215Z",
			"systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
	        "category": "LoadBalancerProbeHealthStatus",
	        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
	        "operationName": "LoadBalancerProbeHealthStatus",
	        "properties": {
	            "publicIpAddress": "40.83.190.158",
	            "port": "81",
	            "totalDipCount": 2,
	            "dipDownCount": 0,
	            "healthPercentage": 100.000000
	        }
	    }

	]
	}

JSON 输出在属性字段显示了探测运行状况的基本信息。*dipDownCount* 属性显示在后端因探测响应失败而收不到网络流量的实例的总数。

## 查看和分析审核日志
你可以使用任何以下方法查看和分析审核日志数据：

- **Azure 工具：**通过 Azure PowerShell、Azure 命令行界面 (CLI)、Azure REST API 或 Azure 预览门户检索审计日志中的信息。[使用资源管理器审核操作](/documentation/articles/resource-group-audit/)一文中详细介绍了每种方法的分步说明。
- **Power BI：**如果还没有 [Power BI](https://powerbi.microsoft.com/pricing) 帐户，你可以免费试用。使用[适用于 Power BI 的 Azure 审核日志内容包](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-audit-logs)，你可以借助预配置的仪表板（可直接使用或进行自定义）分析你的数据。

## 查看和分析运行状况探测和事件日志 
你需要连接到你的存储帐户并检索事件和运行状况探测日志的 JSON 日志项。下载 JSON 文件后，你可以将它们转换为 CSV 并在 Excel、PowerBI 或任何其他数据可视化工具中查看。

>[AZURE.TIP] 如果你熟悉 Visual Studio 和更改 C# 中的常量和变量值的基本概念，则可以使用 Github 提供的[日志转换器工具](https://github.com/Azure-Samples/networking-dotnet-log-converter)。

## 其他资源

- [使用 Power BI 直观显示你的 Azure 审核日志](http://blogs.msdn.com/b/powerbi/archive/2015/09/30/monitor-azure-audit-logs-with-power-bi.aspx)博客文章。
- [查看和分析 Power BI 中的 Azure 审核日志及更多内容](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/)博客文章。

<!---HONumber=Mooncake_0822_2016-->