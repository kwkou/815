<properties
   pageTitle="常见问题"
   description="Power BI Embedded 常见问题解答"
   services="power-bi-embedded"
   documentationCenter=""
   authors="mgblythe"
   manager="NA"
   editor=""
   tags=""/>

<tags
   ms.service="power-bi-embedded"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="powerbi"
   ms.date="07/05/2016"
   ms.author="mblythe"
   wacn.date="12/29/2016"/>  


# Power BI Embedded 常见问题解答

## 有关 Power BI Embedded 的最新公告是什么？

Microsoft Power BI Embedded 服务现已公开发布了标准 GA SLA。若要了解有关此版本的详细信息，请参[新增功能](/documentation/articles/power-bi-embedded-whats-new/)。

## Microsoft Power BI Embedded 是什么？

Microsoft Power BI Embedded 现已公开发布。Power BI Embedded 是一项 Azure 服务，应用程序开发人员可以通过该服务将令人惊叹的完整交互式报表和可视化元素嵌入到面向客户的应用中，使客户无需花费时间和费用从头开始生成其自己的控件。现在，全球 9 个数据中心内都提供了包含 SLA 的 Power BI Embedded。此服务中还提供了增强的功能，例如，支持通过 Power BI Embedded 中的行级安全性 (RLS) 提供数据安全性以用于高级筛选。Power BI Embedded 定价模型也进行了简化和更新。若要了解详细信息，请参阅 [Power BI Embedded](/home/features/power-bi-embedded/)。

## 谁想要使用 Microsoft Power BI Embedded，为什么？

Microsoft Power BI Embedded 适用于想要向其用户的所有设备提供令人惊叹的交互式数据可视化元素体验的应用程序开发人员，使用户无需自行生成这些体验。通过 Power BI Embedded，开发人员可以利用直接查询提供始终保持最新的视图。开发人员还可以使用 Azure ARM API 和 Power BI API 以编程方式部署、管理和自动化 Power BI。与所有的 Power BI 服务一样，这种嵌入式服务也可以自动缩放以满足应用程序的使用情况和需求。Power BI Embedded 服务采取即用即付消费的定价模式。

## Power BI Embedded 与 Power BI 服务有什么关系？

Power BI Embedded 和 Power BI 服务是单独的产品/服务。Power BI Embedded 采取基于消费的计费模式，通过 Azure 门户部署并且旨在使 ISV 能够将数据可视化元素嵌入到应用程序中，供其客户使用。Power BI 服务通过 O365 门户进行计费和部署，是一个独立通用的 BI 产品/服务，主要供企业内部使用。若要了解 Power BI 服务的详细信息，请参阅 [www.powerbi.com](https://powerbi.microsoft.com)。

## Power BI Embedded 如何改进我的应用？

当可以利用优秀的交互式数据可视化功能直接在应用程序中通知用户的决策时，应用程序的功能将更为强大。Power BI Embedded 可通过丰富的、始终保持最新的交互式数据可视化元素增强应用，以便提高应用的实用性、用户的满意度和忠诚度，并可在任何设备上轻松提供语境分析。

## 关于如何在应用中使用 Power BI Embedded 是否存在任何规则或限制？

Power BI Embedded 仅用于为供第三方使用而提供的应用程序。如果想要使用 Power BI Embedded 服务创建内部业务应用程序，则每个内部用户都需要具有一个 Power BI Pro USL，并且你的组织除需要支付 Power BI Pro USL 费用外，还需要支付使用 Power BI Embedded 服务的费用。为避免产生 Power BI Pro USL 费和 Power BI Embedded 用于内部应用程序的消耗费，Power BI 服务在 Power BI Embedded 外提供了其自己的内容嵌入功能，而未额外增加 Power BI USL 持有者的成本 (dev.powerbi.com)。

## Power BI Embedded 可以用于创建内部应用程序吗？

不可以，Power BI Embedded 仅针对外部用户使用，不能在内部的业务应用程序中使用。为了嵌入用于内部业务应用程序的 Power BI 内容，应该使用 Power BI 服务，并且所有使用该内容的用户必须拥有一个有效的 Power BI 免费版或 Power BI Pro 的用户订阅许可证。有关将内部应用程序与 Power BI 服务进行集成的详细信息，请访问 [https://dev.powerbi.com](https://dev.powerbi.com)。

## 此服务是否在全球提供？

目前，大多数数据中心内都提供 Power BI Embedded 服务。始终可以[在此处](https://azure.microsoft.com/status/)检查最新可用性。

## 对于该服务，可用的 SLA 是什么？

Power BI Embedded 具有 Azure 标准 SLA。有关详细信息，请参阅[服务级别协议](/support/legal/sla/)。

## 此服务是如何定价的？

有关定价信息，请参阅 [Power BI Embedded 定价](/pricing/details/power-bi-embedded/)。

## 什么是呈现以及它是如何计费的？

>[AZURE.NOTE] Power BI Embedded 在预览阶段提供了按呈现的折扣定价，但根据用户反馈，以后将停止使用这种定价方式，而采用按会话定价。从按呈现定价转换为按会话定价将从 2016 年 9 月 1 日起生效。

呈现是显示给最终用户的可视元素，结果是产生对服务的查询。例如，如果一个用户查看包含 4 个视觉对象的报表，它将会产生 4 个呈现。如果用户刷新报表并且向服务发送更多的查询，它将会产生另外 4 个呈现。服务所有者将控制最终用户可以推动结果为付费呈现的新查询的程度，以限制成本风险并最小化静态数据方案中的成本。

呈现按每 1,000 个呈现计费。少于 1,000 个的呈现数量按比例计费。客户每月会接收到 1,000 个免费的呈现。通过批量许可协议购买的客户应该咨询他们的 Microsoft 合作伙伴或卖家有关定价的信息。

## 什么是报表会话，它是如何计费的？

会话是指最终用户和 Power BI Embedded 报表之间的一组交互活动。每次向用户显示 Power BI Embedded 报表时，就会启动一个会话，并向订阅持有者收取使用会话的费用。会话按统一收费率计费，独立于报表中的视觉对象元素数量或报表内容的刷新频率。当用户关闭报表时或者会话于一小时后超时时，会话结束。

## 是否提供了任何工具或指南来帮助用户估计应预估的呈现/会话数？ 如何才能知道已完成了多少呈现？

Azure 门户将提供关于针对订阅已执行的呈现/报表会话数的帐单详细信息。

## 使用 Power BI Embedded 开发应用程序是否需要 Power BI 订阅？ 如何开始？

作为应用程序开发人员，创建希望在应用程序中使用的报表和可视化元素无需具有 Power BI 订阅。但需要 Azure 订阅和免费的 Power BI Desktop 应用程序。

有关如何使用 Power BI Embedded 服务的详细信息，请参阅服务文档。

## 用户有一个 Azure 订阅。是否可以通过现有的订阅使用 Power BI Embedded？

是的。可以使用现有的 Azure 订阅预配和使用 Microsoft Power BI Embedded 服务。

## 应用程序的最终用户是否需要 Power BI 许可证？

否。应用程序的最终用户无需单独购买 Power BI 订阅便可访问应用内数据可视化元素。在 Power BI Embedded 模型中，将通过 Azure 消耗计量器针对服务向应用程序提供商计费。请参阅[定价和许可页](/pricing/details/power-bi-embedded/)。

## 如何对 Power BI Embedded 的用户身进行身份验证？

Power BI Embedded 服务使用“应用令牌”进行身份验证和授权，而不是使用显式的最终用户身份验证。在应用令牌模型中，应用程序管理最终用户的身份验证和授权。然后，若有必要，应用将创建

并发送应用令牌，以指示服务来呈现所请求的报表。此设计不要求应用使用 Azure AD 进行用户身份验证和授权，但仍然可以这样做。可以[在此处](/documentation/articles/power-bi-embedded-app-token-flow/)了解应用令牌的详细信息。Power BI Embedded 中还引入了行级安全性 (RLS) 功能以用于高级安全性筛选方案。

## Power BI Embedded 当前支持哪些数据源？

将支持通过直接查询对使用基本凭据的云数据源进行访问。这意味着目前支持的源有 Azure SQL DB 和 Azure SQL DW 等。在未来几个月中将添加对其他数据源和访问类型的支持。将在 Power BI 开发人员论坛 ([https://dev.powerbi.com](https://dev.powerbi.com/)) 上公布受支持的新数据源。

## Power BI Embedded 的租户模型如何工作？

在 Power BI Embedded 模型中，明确要求 Azure AD 租户中必须存在客户。可以为客户选择是否需要 Azure AD。这样，应用程序的体系结构和基础结构就可以用来确定 Power BI Embedded 要求的租户模型。

开发人员/员工操作或创建应用程序时将需要具有 AAD 用户帐户才能通过 Azure 门户管理 Azure 订阅和工作区集合。开发人员可以使用编程 API 导入报表、修改连接字符串、获取嵌入式 URL、改用应用令牌进行身份验证，因此无需使用 AAD。有关如何使用 API 和 Azure 门户的详细信息，请参阅 Azure.com 中的 [Power BI Embedded documentation](/documentation/services/power-bi-embedded/)（Power BI Embedded 文档）页。

## 可以从何处了解详细信息？

可以访问 [Power BI Embedded documentation](http://go.microsoft.com/fwlink/?LinkId=760526)（Power BI Embedded 文档）页。通过访问 [Power BI 开发人员博客](http://blogs.msdn.com/powerbidev)或通过访问 dev.powerbi.com 中的 Power BI 开发人员中心，可以了解该服务的最新信息。也可以在 [Stackoverflow](http://stackoverflow.com/questions/tagged/powerbi) 上提问。

## 如何开始？

可以立即开始体验免费版！ 如果拥有 Azure 订阅，现在就可以直接从 Azure 门户预配 Power BI Embedded。也可以创建自己的 [Azure 帐户](/pricing/1rmb-trial/)。一旦 Power BI Embedded 服务预配完毕，就可以直接轻松使用 Power BI REST API，或使用 [GitHub](https://www.nuget.org/profiles/powerbi) 上提供的开发人员 SDK。关于如何使用开发人员 SDK 提供的示例。

## 另请参阅

- [Microsoft Power BI Embedded 是什么](/documentation/articles/power-bi-embedded-what-is-power-bi-embedded/)
- [Microsoft Power BI Embedded 入门](/documentation/articles/power-bi-embedded-get-started/)

<!---HONumber=Mooncake_1010_2016-->