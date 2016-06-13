<properties
   pageTitle="常见问题"
   description="Power BI Embedded 常见问题解答"
   services="power-bi-embedded"
   documentationCenter=""
   authors="dvana"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="power-bi-embedded"
   ms.date="03/29/2016"
   wacn.date="06/13/2016"/>

# Power BI Embedded 常见问题解答

1. **关于 Power BI 的内部版本是如何宣布的？**

    Microsoft 宣布现已推出 Microsoft Power BI Embedded 服务公共预览版。

2.	**Microsoft Power BI Embedded 是什么？**

    Microsoft Power BI Embedded 是一项 Azure 服务，应用程序开发人员可以通过该服务将令人惊叹的完整交互式报表和可视化元素嵌入到面向客户的应用中，使客户无需花费时间和费用从头开始生成其自己的控件。你可以从广泛的现有数据可视化元素中进行选择，或轻松创建自定义可视化元素以满足你的应用程序的独特需求。Power BI Embedded 适用于旨在向客户提供 BI 作为其大型应用程序的一部分的 ISV/应用程序开发人员。

3.	**谁想要使用 Microsoft Power BI Embedded，以及为什么？**

    Microsoft Power BI Embedded 适用于想要向其用户的所有设备提供令人惊叹的交互式数据可视化元素体验的应用程序开发人员，使用户无需自行生成这些体验。通过 Power BI Embedded，开发人员可以利用**直接查询**提供始终保持最新的视图。开发人员还可以使用 Azure ARM API 和 Power BI API 以编程方式部署、管理和自动化 Power BI。与所有的 Power BI 服务一样，这种嵌入式服务也可以自动缩放以满足应用程序的使用情况和需求。Power BI Embedded 服务采取即用即付消费的定价模式。有关定价，请参阅[此服务是如何定价的？](#price)。

4.	**Power BI Embedded 与 Power BI 服务有什么关系？**

    Power BI Embedded 和 Power BI 服务是单独的产品/服务。Power BI Embedded 采取基于消费的计费模式，通过 Azure 门户部署并且旨在使 ISV 能够将数据可视化元素嵌入到应用程序中，供其客户使用。Power BI 服务通过 O365 门户进行计费和部署，是一个独立通用的 BI 产品/服务，主要供企业内部使用。有关 Power BI 服务的详细信息，请访问 [www.powerbi.com](www.powerbi.com)。

5.	**Power BI Embedded 如果改进我的应用？**

    当利用数据可视化直接在应用程序中通知用户的决策时，应用程序的功能显得更为强大。Power BI Embedded 可通过丰富的、始终保持最新的交互式数据可视化元素增强你的应用，以便提高你的应用的实用性、用户的满意度和忠诚度，并可在任何设备上轻松提供语境分析。

6.	**关于如何在应用中使用 Power BI Embedded 是否存在任何规则或限制？**

    仅当你的应用程序：(1) 将主要和重要的功能添加到 Power BI Embedded 服务中并且从根本上该应用程序不是任何 Power BI 服务的替代品，以及当 (2) 单独提供给你公司的外部用户时，你可以在你开发的应用程序中使用 Power BI Embedded 服务。不得在内部业务应用程序中使用 Power BI Embedded 服务。

7.	**Power BI Embedded 可以用于创建内部应用程序吗？**

    不可以，Power BI Embedded 仅针对外部用户使用，不能在内部的业务应用程序中使用。为了嵌入用于内部业务应用程序的 Power BI 内容，你应该使用 Power BI 服务，并且所有使用该内容的用户必须拥有一个有效的 Power BI 免费版或 Power BI Pro 的用户订阅许可证。有关使用 Power BI 服务进行内部嵌入的详细信息，请访问 [https://dev.powerbi.com](https://dev.powerbi.com)。

8.	**此服务是否在全球提供？**

    Power BI Embedded 服务自内部版本 2016 公告起在北美提供（在美国中南部中央数据中心）。我们将在不久之后在其余 Azure 数据中心推出此服务，尽请期待。

9.	**对于该服务，可用的 SLA 是什么？**

    Power BI Embedded 作为 Azure 服务目前处于预览阶段，没有正式的 SLA。当该服务从预览版移动到正式版后将提供 SLA。


10.	<a name="price"/></a>**此服务是如何定价的？**

    Power BI Embedded 当前为预览版，并且在 2016 年 5 月 1 日前免费提供。从 2016 年 5 月 1 日开始，该服务将按渲染定价。客户可以通过两个主要许可载体购买该服务：Microsoft 在线订阅计划 (MOSP) 或企业 VL 计划。

    仅当你的应用程序：(1) 将主要和重要的功能添加到我们的服务中并且从根本上该应用程序不是任何 Power BI 服务的替代品，以及当 (2) 单独提供给外部用户时，你可以在你开发的应用程序中使用 Power BI Embedded 服务。不得在内部业务应用程序中使用 Power BI Embedded 服务。

    ![](./media/power-bi-embedded-faq/price.png)

11.	**什么是渲染以及它是如何计费的？**

    当向需要对该服务进行查询的最终用户显示可视化元素时发生渲染。Power BI 服务会尝试缓存渲染的内容（如可能），以便减少将针对你的应用程序收费的渲染数。但是，用户操作（例如筛选）可能会导致对我们的服务进行查询，这又会继续产生新的渲染。

    例如，当用户查看包含四个视觉效果的报表时，此操作可能会导致出现四个渲染。如果用户刷新报表并且向该服务发送更多查询，它将会产生另外四个渲染。服务所有者将控制最终用户可以推动结果为付费渲染的新查询的程度，以限制成本风险并最小化静态数据方案中的成本。

    渲染按每 1,000 个渲染计费。少于 1,000 个的渲染数量按比例计费。客户每月会接收到 1,000 个免费的渲染。通过批量许可协议购买的客户应该咨询他们的 Microsoft 合作伙伴或卖家有关定价的信息。

12.	**是否提供有任何工具或指南以帮助我估计我应预估的渲染数？ 如何才能知道我已完成了多少渲染？**

    Azure 门户将提供关于针对你的订阅已执行的渲染数的帐单详细信息。

13.	**使用 Power BI Embedded 开发应用程序是否需要 Power BI 订阅？ 如何开始？**

    作为应用程序开发人员，创建你希望在应用程序中使用的报表和可视化元素无需具有 Power BI 订阅。但需要 Azure 订阅和免费的 Power BI Desktop 应用程序。

    有关如何使用 Power BI Embedded 服务的详细信息，请参阅服务文档。

14.	**我拥有一个 Azure 订阅。是否可以通过现有的订阅使用 Power BI Embedded？**

    是的。你可以使用现有的 Azure 订阅预配和使用 Microsoft Power BI Embedded 服务。

15.	**应用程序的最终用户是否需要 Power BI 许可证？**

    否。应用程序的最终用户无需购买单独的 Power BI 订阅便可访问应用内数据可视化元素。在 Power BI Embedded 模型中，应用程序提供程序将通过 Azure 消耗计量器对服务的应用程序提供程序进行计费。请参阅[定价和许可页](/home/features//power-bi-embedded/)。

16.	**如何对 Power BI Embedded 的用户身进行身份验证？**

    Power BI Embedded 服务使用应用程序令牌进行身份验证和授权，而非显式最终用户身份验证。在应用程序令牌模型中，应用程序为你的最终用户管理身份验证和授权。然后，如有必要，你的应用将创建并发送应用程序令牌，告诉我们的服务渲染请求的报表。尽管可以这样做，但这种设计其实并不要求应用使用 Azure Active Directory 进行用户身份验证和授权。有关应用程序令牌的详细信息，请参阅 [应用程序令牌](/documentation/articles/power-bi-embedded-get-started-sample/#key-flow)文档页。

17.	**Power BI Embedded 当前支持哪些数据源？**

    在该服务的公共预览版期间，我们将支持通过直接查询对使用基本凭据的云数据源进行访问。这意味着目前支持的源有 Azure SQL DB、HDInsight Spark 和 Azure SQL DW 等。我们将在未来几个月中添加对其他数据源和访问类型的支持。我们将在 Power BI 开发人员网站 ([http://dev.powerbi.com](http://dev.powerbi.com/)) 上公布受支持的新数据源。

18.	**Power BI Embedded 的租户模型如何工作？**

    在 Power BI Embedded 模型中，有一条对 Azure Active Directory (Azure AD) 租户的显式要求，即要求该租户中必须存在客户。你可以为你的客户选择是否需要 Azure AD。这样，应用程序的体系结构和基础结构就可以用来确定 Power BI Embedded 要求的租户模型。

    开发人员/员工操作或创建你的应用程序时将需要具有 Azure AD 用户帐户才能通过 Azure 门户管理你的 Azure 订阅和工作区集合。开发人员可以使用编程 API 导入报表、修改连接字符串、获取嵌入式 URL、改用应用程序令牌进行身份验证，因此无需使用 Azure AD。有关如何使用我们的 API 和 Azure 门户的详细信息，请参阅 Azure.com 中的[Power BI Embedded documentation（Power BI Embedded 文档）](/documentation/services/power-bi-embedded/)页。

19.	**可以从何处了解详细信息？**

    你可以访问 [Power BI Embedded documentation](/services/power-bi-embedded/)（Power BI Embedded 文档）页。通过访问 [Power BI 开发人员博客](http://blogs.msdn.com/powerbidev)或通过访问 dev.powerbi.com 中的 Power BI 开发人员中心，你可以了解该服务的最新信息。也可以在 [Stackoverflow](http://stackoverflow.com/questions/tagged/powerbi) 上提问。

20.	**如何开始？**

    你可以立即开始体验免费版！ 如果拥有 Azure 订阅，现在就可以直接从 Azure 门户预配 Power BI Embedded。也可以创建[免费 Azure 帐户](/pricing/1rmb-trial)。一旦 Power BI Embedded 服务预配完毕，你就可以直接轻松使用 Power BI REST API，或使用 [GitHub](http://go.microsoft.com/fwlink/?LinkID=746472) 上提供的开发人员 SDK。关于如何使用开发人员 SDK 提供的示例。

## 另请参阅

- [Microsoft Power BI Embedded 是什么](/documentation/articles/power-bi-embedded-what-is-power-bi-embedded)
- [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started)

<!---HONumber=Mooncake_0523_2016-->