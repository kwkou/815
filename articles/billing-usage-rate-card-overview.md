<properties
   pageTitle="深入了解 Microsoft Azure 资源消耗"
   description="从概念上概述了 Azure 帐单使用状况和价目表 API，可帮助您深入了解 Azure 资源耗用量和趋势。"
   services="billing"
   documentationCenter=""
   authors="BryanLa"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="billing"
   ms.date="07/07/2015"
   wacn.date="10/3/2015"/>

# 深入了解 Microsoft Azure 资源消耗 

客户和合作伙伴需要能够准确地预测和管理其 Azure 费用。随着从 Capex 迁移到 Opex 模型，他们还需要能够执行 Showback 与 Chargeback 对比分析，并提供估计和计费的模式保真度（尤其是对于大型云部署）。

本文介绍的 Azure 资源使用状况和价目表 API 可以满足这些需求，帮助您获得对 Azure 资源耗用的全新进一步了解。

## Azure 资源使用状况和价目表 API 简介 

Azure 资源使用状况和价目表 API 作为资源提供单元进行实现，属于 Azure 资源管理器公开的 API 系列。

### Azure 资源使用状况 API（预览）
客户和合作伙伴可以使用 Azure 资源使用状况 API 来估计 Azure 耗用数据。具体功能包括：
	
- **Azure 基于角色的访问控制** - 客户和合作伙伴可以在 [Azure 预览门户](https://portal.azure.com)上或通过 [Azure PowerShell cmdlet](powershell-install-configure) 配置其访问策略，以指定哪些用户或应用程序可以有权访问订阅使用状况数据。调用方必须使用标准 Azure Active Directory 令牌进行身份验证。此外，调用方必须添加至读者、所有者或参与者角色，才能有权访问特定 Azure 订阅的使用状况数据。

- **每小时或每日聚合** - 调用方可以指定 Azure 使用状况数据是以每小时存储桶为单位，还是以每日存储桶为单位。默认值为每日聚合。

- **提供的实例元数据（包括资源标记）**- 响应中会提供实例级详细信息，如完全限定的资源 URI (/subscriptions/{subscription-id}/..)，以及资源组信息和资源标记。对于交叉收费等用例，这将帮助客户明确以编程方式按标记分配使用状况。

- **提供的资源元数据** - 资源详细信息（如测定仪名称、测定仪类别、测定仪子类别、计价单位和区域）也会传入响应，以便调用方能够更好地了解耗用量。我们还致力于跨 Azure 门户、Azure 使用状况 CSV、EA 计费 CSV 和其他面向公众的体验保持资源元数据术语的一致性，以使客户能够跨体验关联数据。

- **所有类型产品/服务的使用状况** - 可以访问所有类型产品/服务的使用状况数据，包括即用即付、MSDN、货币承诺、货币信用额和 EA 等。

### Azure 资源价目表 API（预览）
客户和合作伙伴可以使用 Azure 资源价目表 API 获取可用 Azure 资源的列表，以及每个资源的估计定价信息。具体功能包括：

- **Azure 基于角色的访问控制** - 客户和合作伙伴可以在 [Azure 预览门户](https://portal.azure.com)上或通过 [Azure PowerShell cmdlet](powershell-install-configure) 配置其访问策略，以指定哪些用户或应用程序可以有权访问价目表数据。调用方必须使用标准 Azure Active Directory 令牌进行身份验证。此外，调用方必须添加至读者、所有者或参与者角色，才能有权访问特定 Azure 订阅的使用状况数据。
	
- **支持即用即付、MSDN、货币承诺和货币信用额产品/服务（不支持 EA）**- 此 API 提供了 Azure 产品/服务级与订阅级费率信息。此 API 的调用方必须传入产品/服务信息，才能获取资源详细信息和费率。由于 EA 产品/服务按注册自定义费率，因此我们暂时无法提供 EA 费率。

## 方案

使用状况和价目表 API 组合可以实现下面一些方案：

- **Azure 月支出** - 客户可以将使用状况和价目表 API 结合使用，通过分析使用状况和估计费用的每小时和每日存储桶，可以更深入地了解云的月支出。 

- **设置警报** - 客户和合作伙伴可以将使用状况和价目表 API 结合使用，估计耗用量和费用，从而设置基于资源或货币的云耗用警报。

- **预测帐单** - 客户和合作伙伴可以估计耗用量和云支出，并应用机器学习算法来预测计费周期结束时的帐单。

- **耗用费用预分析** - 如果要将工作负荷转移到 Azure 中，则客户还可以使用价目表 API 来预测帐单（通过提供的所需使用状况数据）。如果客户在其他云或私有云中拥有现有的工作负荷，则还可以将使用状况与 Azure 费率进行映射，以便更好地估计 Azure 支出。这就改进了通过 [Azure 定价计算器](/pricing/calculator/)可以获取的视图，因为（例如）我们的计费合作伙伴能够透视产品/服务和不同产品/服务类型（不仅仅局限于即用即付，包括货币承诺和货币信用额）之间的对比/对照。此外，API 还能够按区域更改估计费用，同时启用制定部署决策所需的模拟分析类型，因为在全球不同的 DC 中部署资源会直接影响总费用。

- **模拟分析** -

	- 客户和合作伙伴可以确定在其他区域或 Azure 资源的其他配置上运行其工作负荷是否会更具成本效益。Azure 资源费用可能因其 Azure 区域而异，这使得客户和合作伙伴可以优化费用。

	- 客户和合作伙伴还可以确定其他 Azure 产品/服务类型是否提供更优惠的 Azure 资源费率。

## 合作伙伴解决方案

[Microsoft Azure 使用状况和价目表 API 启用 Cloudyn 为客户提供 ITFM](/documentation/articles/billing-usage-rate-card-partner-solution-cloudyn) 介绍了 Azure 计费 API 合作伙伴 [Cloudyn](https://www.cloudyn.com/microsoft-azure/) 提供的集成体验。本文详细介绍了体验范围，通过一个简短的视频展示了 Azure 客户如何使用 Cloudyn 和 Azure 计费 API 深入了解 Azure 耗用数据。

[Cloud Cruiser 和 Microsoft Azure 计费 API 集成](/documentation/articles/billing-usage-rate-card-partner-solution-cloudcruiser)介绍了 [Cloud Cruiser 的 Express for Azure Pack](http://www.cloudcruiser.com/partners/microsoft/) 如何直接从 WAP 门户运行，让客户能够顺畅地在一个用户界面中管理 Microsoft Azure 私有云或托管公有云的运营和财务方面。

## 后续步骤
+ 查看 [Azure 计费 REST API 参考](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)，详细了解这两个 API（属于 Azure 资源管理器提供的 API 集）。
+ 如果您想直接深入了解示例代码，请查看我们在 [Github 上的 Microsoft Azure 计费 API 代码示例](https://github.com/Azure/BillingCodeSamples)。

## 了解详细信息
+ 若要了解有关 Azure 资源管理器的详细信息，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview)。
+ 有关有助于了解云费用所需工具套件的其他信息，请参阅 Gartner 文章 [IT 财务管理 (ITFM) 工具的市场指南](http://www.gartner.com/technology/reprints.do?id=1-212F7AL&ct=140909&st=sb)。

<!---HONumber=71-->