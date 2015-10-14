<properties
   pageTitle="Microsoft Azure 使用情况 API 和 RateCard 使 Cloudyn 能够为客户提供 ITFM"
   description="Microsoft Azure 计费合作伙伴 Cloudyn 将 Azure 计费 API 集成到了其产品中，从而能够根据亲身体验提供独特的观点。对于有兴趣使用/试用 Cloudyn for Azure 服务的 Azure 和 Cloudyn 客户而言，这是非常有用的。"
   services="billing"
   documentationCenter=""
   authors="BryanLa"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="billing"
   ms.date="06/14/2015"
   wacn.date="10/3/2015"/>

# Microsoft Azure 使用情况 API 和 RateCard 使 Cloudyn 能够为客户提供 ITFM 

Cloudyn 是 Microsoft 的开发合作伙伴，也是云管理功能领域的领先提供商，它试用了新的 Microsoft Azure 资源使用情况 API 和 RateCard API 的专用预览版。使用情况 API 提供了有关订阅的估计 Azure 消耗数据。RateCard API 提供了适用于非企业协议 (EA) 客户的所有 Azure 服务的完整定价信息。这些 API 集成在一起即能为 IT 财务管理 (ITFM) 工具（如 Cloudyn 所提供的工具）输入完整的基础信息。

## 介绍 

将来自使用情况 API 的数据与来自 RateCard API 的数据“相乘”（使用量 [单位] 价格 [$单位] = 详细的使用量和成本），即能创建 Azure 目前所能提供的最精确可靠的计费信息。

![ITFM 概述][1]

使用这些 API 能提供有关客户使用量和成本的关键信息，从而使得 Cloudyn 能够以简单的编程方式分析客户帐户，并为客户执行各种 ITFM 任务。

## 将 Cloudyn 与 RateCard 和使用情况 API 相集成
RateCard API 需要多个输入参数（如区域信息、货币和区域设置），但最重要的一个是 OfferDurableID。OfferDurableID 指定客户所使用的 Azure 产品/服务类型（即用即付、6 到 12 个月的旧承诺计划、MSDN 产品/服务、MPN 产品/服务、促销产品/服务和其他）。在指定订阅的“产品/服务 ID”下的 [Azure 使用情况和计费门户](https://account.windowsazure.cn/Subscriptions)下，可以找到 OfferDurableID。

在注册 [Cloudyn for Azure](https://www.cloudyn.com/microsoft-azure/) 服务时，客户可以添加其 OfferDurableID 代码，这样 Cloudyn 就能通过 RateCard API 提取相关的定价信息。有关不同类型产品/服务的信息，请参阅 [Microsoft Azure 产品/服务详细信息](http://www.windowsazure.cn/en-gb/support/legal/offer-details/)页面。

![Cloudyn ITFM 引擎概述][2]

除了 Azure 性能 API，Cloudyn 还使用使用情况 API 和 RateCard API 来创建可视化、分析、警报、报告、成本管理和可操作建议等其他层级，从而为 Azure 客户提供可靠的企业云 ITFM 工具。

## 使用情况 API 和 RateCard API 集成所能启用的 Cloudyn ITFM 用例 
使用情况 API 和 RateCard API 集成所能启用的一般 Cloudyn ITFM 用例

+ **成本分析** - 允许将云成本细分为任何本机标识的维度（提供程序、服务、帐户、区域等）。Azure 使用情况 API 和 RateCard API 使复杂的细分任务变得简单容易：按帐户对使用量和成本数据进行最细致的细分，然后由 Cloudyn 进行筛选和组合，并以图形或表格形式呈现给用户。

![成本分析饼形图][3]

+ **成本分配 360** - 使财务和 IT 经理们能够发现云部署的实际成本细分、驱动因素和趋势。它还能使管理人员轻松地在不同业务部门、部门、区域等之间分配部署费用，使管理人员前所未有地清楚了解有关云成本的信息，以及为企业 Chargeback 和 Showback 提供便利。Azure 使用情况 API 和 RateCard API 是 Cloudyn 成本分配引擎的输入，而 Cloudyn 成本分配引擎是这些 API 的补充，它为分配无标记或不可标记的资源定义了方法和业务逻辑。

![成本分摊 360 图表][4]

+ **经济高效的大小调整** - 对于未得到充分使用的虚拟机提供大小调整建议，从而减少客户花费在过大或超配置计算机上的费用。通过检查虚拟机 CPU 和 RAM 度量值（通过性能 API）、运行时的小时数（通过使用情况 API）和成本（通过 RateCard API），就可以实现这一点。Cloudyn 然后为未充分利用的 CPU 或 RAM 资源（性能）提供大小调整建议，然后估算节约的费用，方法是：将 VM 间的价格增量 (RateCard) 乘以未充分利用计算机的实际使用时间（使用情况）。 

![经济高效的大小调整][5]

+ **云迁移建议** - 对进行云迁移提供财务上的建议。将用户当前在其主要云供应商上部署的云资源成本与其在 Azure 上进行等效部署的成本进行比较。然后从财务角度提供按资源迁移到 Azure 的详尽细致建议。对在 Azure 上进行的等效部署需求进行评估后（基于性能度量和用户首选项），Cloudyn 使用 RateCard API 来估算在 Azure 上进行等效部署的成本。

+ **性能报表** - 由 Azure 性能 API 提供。这些报表提供了一系列功能，包括 CPU 和 RAM 使用率，以及优化建议。以下是一个实例的使用率报表示例，按平均 CPU 使用率给出了该实例的细分数据。

![性能报表][6]

+ **类别管理器** - Cloudyn 的一项强大功能，能对散乱的云资源进行组织。它使用户能够自由地创建自己的唯一类别（标记）以对业务活动进行有效的测量和报告。此外，用户可以轻松地管理和分类不一致的标记（例如，错别字和其他差异），并自动检测无标记的资源以进行准确的成本分类。

![类别管理器][7]

## 视频 

该简短视频演示了 Azure 客户如何使用 Cloudyn for Azure 和 Azure 计费 API 来深入了解其 Azure 消耗数据。
 
> [AZURE.VIDEO cloudyn-provides-cloud-itfm-tools-via-microsoft-azure-apis]


## 后续步骤

+ 开始免费试用 [Cloudyn for Azure](https://www.cloudyn.com/microsoft-azure/)，了解如何通过 Microsoft Azure 云部署的自定义报告和分析功能实现成本透明化。
+ 有关 Azure 资源使用情况 API 和 RateCard API 的概述，请参阅[深入了解 Microsoft Azure 资源消耗](/documentation/articles/billing-usage-rate-card-overview)。 
+ 请查看 [Azure 计费 REST API 参考](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)了解有关这两种 API（属于 Azure 资源管理器提供的 API 集）的更多信息。
+ 如果您想直接深入了解示例代码，请查看我们在 [Github 上的 Microsoft Azure 计费 API 代码示例](https://github.com/Azure/BillingCodeSamples)。

## 了解详细信息
+ 若要了解有关 Microsoft Azure 企业协议 (EA) 产品/服务的详细信息，请访问[许可适用于企业的 Azure](/pricing/enterprise-agreement/)
+ 若要了解有关 Azure 资源管理器的详细信息，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview)。
+ 有关有助于了解云费用所需工具套件的其他信息，请参阅 Gartner 文章 [IT 财务管理 (ITFM) 工具的市场指南](http://www.gartner.com/technology/reprints.do?id=1-212F7AL&ct=140909&st=sb)。

<!--Image references-->
[1]: ./media/billing-usage-rate-card-partner-solution-cloudyn/Cloudyn-ITFM-Overview.png
[2]: ./media/billing-usage-rate-card-partner-solution-cloudyn/Cloudyn-ITFM-Engine-Overview.png
[3]: ./media/billing-usage-rate-card-partner-solution-cloudyn/Cloudyn-Cost-Analysis-Pie-Chart.png
[4]: ./media/billing-usage-rate-card-partner-solution-cloudyn/Cloudyn-Cost-Allocation-360-Chart.png
[5]: ./media/billing-usage-rate-card-partner-solution-cloudyn/Cloudyn-Cost-Effective-Sizing.png
[6]: ./media/billing-usage-rate-card-partner-solution-cloudyn/Cloudyn-Performance-Reports.png
[7]: ./media/billing-usage-rate-card-partner-solution-cloudyn/Cloudyn-Category-Manager.png

<!---HONumber=71-->