# Advanced Autoscale configuration using Resource Manager templates for VM Scale Sets（通过用于 VM 规模集的 Resource Manager 模板进行的高级自动缩放配置）

用户可以根据性能指标阈值，按定期计划或特定日期来扩大或缩小虚拟机规模集。还可以为缩放操作配置电子邮件和 webhook 通知。本文演示了在 VM 规模集上使用 Resource Manager 模板配置以上所有内容的示例。

>[AZURE.NOTE] 尽管本文演示的是适用于 VM 规模集的步骤，但也可以将这些相同的步骤用于自动缩放云服务和 Web 应用。有关根据简单的性能指标（如 CPU）在 VM 规模集上进行简单的缩小/扩大设置的相关信息，请参阅 [Linux](/documentation/articles/virtual-machine-scale-sets-linux-autoscale/) 和 [Windows](/documentation/articles/virtual-machine-scale-sets-windows-autoscale/) 文档



## 演练
在本演练中，将使用 [Azure 资源浏览器](https://resources.azure.cn/)来配置和更新规模集的自动缩放设置。Azure 资源浏览器是通过 Resource Manager 模板轻松管理 Azure 资源的一种方式。如果不熟悉 Azure 资源浏览器工具，请参阅[此简介](https://azure.microsoft.com/blog/azure-resource-explorer-a-new-tool-to-discover-the-azure-api/)。

1. 使用基本自动缩放设置部署新的规模集。本文使用的是 Azure 快速入门库中的一个示例，它拥有具有基本自动缩放模板的 Windows 规模集。Linux 规模集以相同的方式工作。

2. 创建规模集后，请从 Azure 资源浏览器导航到规模集资源。用户在 Microsoft.Insights 节点下将看到以下内容。

	![Azure 资源管理器](./media/insights-advanced-autoscale-vmss/azure_explorer_navigate.png)  


	模板执行已创建了名为 **'autoscalewad'** 的默认自动缩放设置。在右侧，可以查看此自动缩放设置的完整定义。在本例中，默认自动缩放设置附带了基于 CPU% 的扩大和缩小规则。

3. 现在，可根据计划或特定要求添加更多的配置文件和规则。创建具有三个配置文件的自动缩放设置。若要了解自动缩放中的配置文件和规则，请参阅[自动缩放最佳做法](/documentation/articles/insights-autoscale-best-practices/)。

    | 配置文件和规则 | 说明 |
	|---------|-------------------------------------|
	| **配置文件** | **基于性能/指标** |
	| 规则 | 服务总线队列消息计数 > x |
	| 规则 | 服务总线队列消息计数 < y |
	| 规则 | CPU% < n |
	| 规则 | CPU% < p |
	| **配置文件** | **工作日上午时间（没有规则）** |
	| **配置文件** | **产品上市日（没有规则）** |

4. 以下是用于进行此演练所假设的一个缩放方案。
	- _**基于负载** - 根据在规模集上托管的应用程序上的负载情况进行扩大或缩小。_
	- _**消息队列大小** - 使用服务总线队列为应用程序传入消息。如果消息计数或 CPU 达到阈值，则使用队列的消息计数和 CPU%，并配置默认配置文件来触发缩放操作。_
	- _**每周和每日时间** - 每周定期按基于“每日时间”的名为“工作日上午时间”的配置文件执行。根据历史数据，在此期间最好有一定数量的 VM 实例来处理应用程序的负载。_
	- _**特殊日期** - 添加了“产品上市日”配置文件。提前计划具体日期，以便应用程序做好准备以处理由市场通知以及将新产品放入到应用程序中时所产生的负载。_
	- _可以在最后两个配置文件中放置基于其他性能指标的规则。本例中决定不使用任何一个其他规则，而是依赖基于性能指标的默认规则。对于定期和基于日期的配置文件来说，规则是可选的。_

	[自动缩放最佳做法](/documentation/articles/insights-autoscale-best-practices/)一文中还捕获了配置文件和规则的自动缩放引擎的优先顺序。有关自动缩放的常用指标列表，请参阅[自动缩放的常用指标](/documentation/articles/insights-autoscale-common-metrics/)

5. 请确保在资源浏览器中处于**读/写**模式。

	![Autoscalewad，默认自动缩放设置](./media/insights-advanced-autoscale-vmss/autoscalewad.png)  


6. 单击“编辑”。将自动缩放设置中的“配置文件”元素**替换**为以下内容：

	![配置文件](./media/insights-advanced-autoscale-vmss/profiles.png)  


	```
	{
	        "name": "Perf_Based_Scale",
	        "capacity": {
	          "minimum": "2",
	          "maximum": "12",
	          "default": "2"
	        },
	        "rules": [
	          {
	            "metricTrigger": {
	              "metricName": "MessageCount",
	              "metricNamespace": "",
	              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
	              "timeGrain": "PT5M",
	              "statistic": "Average",
	              "timeWindow": "PT5M",
	              "timeAggregation": "Average",
	              "operator": "GreaterThan",
	              "threshold": 10
	            },
	            "scaleAction": {
	              "direction": "Increase",
	              "type": "ChangeCount",
	              "value": "1",
	              "cooldown": "PT5M"
	            }
	          },
	          {
	            "metricTrigger": {
	              "metricName": "MessageCount",
	              "metricNamespace": "",
	              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue",
	              "timeGrain": "PT5M",
	              "statistic": "Average",
	              "timeWindow": "PT5M",
	              "timeAggregation": "Average",
	              "operator": "LessThan",
	              "threshold": 3
	            },
	            "scaleAction": {
	              "direction": "Decrease",
	              "type": "ChangeCount",
	              "value": "1",
	              "cooldown": "PT5M"
	            }
	          },
	          {
	            "metricTrigger": {
	              "metricName": "\\Processor(_Total)\\% Processor Time",
	              "metricNamespace": "",
	              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
	              "timeGrain": "PT5M",
	              "statistic": "Average",
	              "timeWindow": "PT30M",
	              "timeAggregation": "Average",
	              "operator": "GreaterThan",
	              "threshold": 85
	            },
	            "scaleAction": {
	              "direction": "Increase",
	              "type": "ChangeCount",
	              "value": "1",
	              "cooldown": "PT5M"
	            }
	          },
	          {
	            "metricTrigger": {
	              "metricName": "\\Processor(_Total)\\% Processor Time",
	              "metricNamespace": "",
	              "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachineScaleSets/<this_vmss_name>",
	              "timeGrain": "PT5M",
	              "statistic": "Average",
	              "timeWindow": "PT30M",
	              "timeAggregation": "Average",
	              "operator": "LessThan",
	              "threshold": 60
	            },
	            "scaleAction": {
	              "direction": "Increase",
	              "type": "ChangeCount",
	              "value": "1",
	              "cooldown": "PT5M"
	            }
	          }
	        ]
	      },
	      {
	        "name": "Weekday_Morning_Hours_Scale",
	        "capacity": {
	          "minimum": "4",
	          "maximum": "12",
	          "default": "4"
	        },
	        "rules": [],
	        "recurrence": {
	          "frequency": "Week",
	          "schedule": {
	            "timeZone": "Pacific Standard Time",
	            "days": [
	              "Monday",
	              "Tuesday",
	              "Wednesday",
	              "Thursday",
	              "Friday"
	            ],
	            "hours": [
	              6
	            ],
	            "minutes": [
	              0
	            ]
	          }
	        }
	      },
	      {
	        "name": "Product_Launch_Day",
	        "capacity": {
	          "minimum": "6",
	          "maximum": "20",
	          "default": "6"
	        },
	        "rules": [],
	        "fixedDate": {
	          "timeZone": "Pacific Standard Time",
	          "start": "2016-06-20T00:06:00Z",
	          "end": "2016-06-21T23:59:00Z"
	        }
	      }
	```
	有关支持的字段及其值，请参阅 [Autoscale REST API documentation](https://msdn.microsoft.com/zh-cn/library/azure/dn931928.aspx)（自动缩放 REST API 文档）。

	现在自动缩放设置包含之前所述的三个配置文件。

7. 	最后，来看一下自动缩放**通知**部分。成功触发扩大或缩小操作后，自动缩放通知允许执行三个操作。

	1. 通知订阅的管理员和共同管理员

	2. 向一组用户发送电子邮件

	3. 触发 webhook 调用。当激发 webhook 后，它会发送关于自动缩放条件和规模集资源的元数据。若要了解自动缩放 webhook 的有效负载的详细信息，请参阅 [Configure Webhook & Email Notifications for Autoscale](/documentation/articles/insights-autoscale-to-webhook-email/)（为自动缩放配置 webhook 和电子邮件通知）。

	将以下内容添加到自动缩放设置，以替换其值为 null 的**通知**元素

	```
	"notifications": [
	      {
	        "operation": "Scale",
	        "email": {
	          "sendToSubscriptionAdministrator": true,
	          "sendToSubscriptionCoAdministrators": false,
	          "customEmails": [
	              "user1@mycompany.com",
	              "user2@mycompany.com"
	              ]
	        },
	        "webhooks": [
	          {
	            "serviceUri": "https://foo.webhook.example.com?token=abcd1234",
	            "properties": {
	              "optional_key1": "optional_value1",
	              "optional_key2": "optional_value2"
	            }
	          }
	        ]
	      }
	    ]

	```

	在资源浏览器中单击“Put”按钮以更新自动缩放设置。

已更新 VM 规模集上的自动缩放设置，以包含多个缩放配置文件和缩放通知。

## 后续步骤

请使用以下链接了解关于自动缩放的详细信息。

[自动缩放的常用指标](/documentation/articles/insights-autoscale-common-metrics/)

[Best Practices for Azure Autoscale（Azure自动缩放的最佳实践）](/documentation/articles/insights-autoscale-best-practices/)

[Manage Autoscale using PowerShell（使用 PowerShell 管理自动缩放）](/documentation/articles/insights-powershell-samples/#create-and-manage-autoscale-settings)

[Manage Autoscale using CLI（使用 CLI 管理自动缩放）](/documentation/articles/insights-cli-samples/#autoscale)

[Configure Webhook & Email Notifications for Autoscale（自动缩放配置 webhook 和电子邮件通知）](/documentation/articles/insights-autoscale-to-webhook-email/)

<!---HONumber=Mooncake_1107_2016-->