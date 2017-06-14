<properties
    pageTitle="Power BI Embedded 是什么？"
    description="借助 Power BI Embedded，可将 Power BI 报表集成到 Web 或移动应用程序中，因此无需生成自定义解决方案。"
    services="power-bi-embedded"
    documentationcenter=""
    author="guyinacube"
    manager="erikre"
    editor=""
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid="03649b72-b7d7-40ca-b077-12356d72d4f3"
    ms.service="power-bi-embedded"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="powerbi"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="asaxton"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="f89115098ec980236f7216095cbecc291a5af48b"
    ms.lasthandoff="04/14/2017" />

# <a name="what-is-power-bi-embedded"></a>Power BI Embedded 是什么？
通过 **Power BI Embedded**，可以将 Power BI 报表集成到 web 或移动应用程序。

![](./media/powerbi-embedded-whats-is/what-is.png)

Power BI Embedded 是一项 **Azure 服务** ，使 ISV 能够在其自己的应用程序中融入 Power BI 数据体验。 开发人员已经构建了应用程序，并且这些应用程序具有其自己的用户和不同的功能集。 这些应用也可能碰巧具有内置的数据元素（如图表和报表），这些元素现在可以由 Power BI Embedded 提供支持。 不需要 Power BI 帐户便可使用这些应用。 可继续像以前那样登录这些应用程序，查看并与 Power BI 报表体验进行交互，而无需任何额外的授权。

## <a name="licensing-for-power-bi-embedded"></a>Power BI Embedded 的授权
在 **Power BI Embedded** 使用模型中，Power BI 的授权不是由最终用户负责的。  而是应该由使用视觉对象的应用开发人员来购买 **会话** ，并且费用计入拥有这些资源的订阅中。 可在[定价页](/pricing/details/power-bi-embedded/)上找到其他信息。

## <a name="power-bi-embedded-conceptual-model"></a>Power BI Embedded 概念模型

![](./media/powerbi-embedded-whats-is/model.png)

与 Azure 中的任何其他服务一样，Power BI Embedded 的资源也是通过 [Azure资源管理器API](https://msdn.microsoft.com/zh-cn/library/mt712306.aspx) 进行预配的。 在此示例中，预配的资源是一个 **Power BI 工作区集合**。

## <a name="workspace-collection"></a>工作区集合
**工作区集合**是一个顶级 Azure 资源容器，包含 0 个或多个**工作区**。  **工作区** **集合** 具有所有标准 Azure 属性和下列内容：

- **访问密钥** - 安全地调用 Power BI API 时使用的密钥（后面的部分中进行了介绍）。
- **用户** - 具有管理员权限的 Azure Active Directory (AAD) 用户，通过 Azure 门户或 Azure资源管理器API 管理 Power BI 工作区集合。
- **区域** - 在预配**工作区集合**的过程中，可以选择要在其中预配的区域。 有关详细信息，请参阅 [Azure 区域](https://azure.microsoft.com/regions/)。

## <a name="workspace"></a>工作区
**工作区**是一个包含 Power BI 内容的容器，该内容可以包括数据集和报表。 **工作区**在首次创建后为空。 你将使用 Power BI Desktop 创作内容，然后使用 [Power BI Import API](https://msdn.microsoft.com/zh-cn/library/mt711504.aspx) 以编程方式将 PBIX 部署到工作区中。 还可以通过编程方式创建数据集，然后在你的应用程序内而非使用 Power BI Desktop 创建报表。

## <a name="using-workspace-collections-and-workspaces"></a>使用工作区集合和工作区
**工作区集合**和**工作区**是内容容器，它们按最适合所生成应用程序设计的方式进行使用和组织。 它们提供了许多不同的方式供用户在其中排列内容。 可以选择将所有内容都放在一个工作区中并在稍后使用应用令牌进一步细分客户之间的内容。 还可以选择将所有客户放在单独的工作区中，使他们分开一些。 另外，也可以选择按区域而不是按客户来组织用户。 这种设计十分灵活，可以选择最佳的方式来组织内容。

## <a name="cached-datasets"></a>缓存的数据集
可以使用缓存的数据集。  但是，将缓存的数据加载到 **Power BI Embedded**中后，将无法刷新该数据。 缓存的数据集意味着你已将数据导入到 Power BI Desktop 中而非使用 DirectQuery。

## <a name="authentication-and-authorization-with-app-tokens"></a>使用应用令牌进行身份验证和授权
**Power BI Embedded** 委托应用程序来执行所有必要的用户身份验证和授权。 没有明确要求最终用户必须是 Azure Active directory (Azure AD) 的客户。  相反，应用程序通过使用**应用程序身份验证令牌（应用令牌）**向 **Power BI Embedded** 表明自己获得了呈现 Power BI 报表的授权。  当应用想要呈现报表时，可以根据需要创建这些 **应用令牌** 。

![](./media/powerbi-embedded-whats-is/app-tokens.png)

**应用程序身份验证令牌（应用令牌）**用于向 **Power BI Embedded** 表明身份。  有三种类型的 **应用令牌**：

1. 预配令牌 - 在**工作区集合**中预配新的**工作区**时使用
2. 开发令牌 - 直接调用 **Power BI REST API** 时使用
3. 嵌入令牌 - 进行调用以在嵌入式 iframe 中呈现报表时使用

这些令牌适用于与 **Power BI Embedded**进行交互的各个阶段。  这些令牌经过专门设计，以便可以将应用中的权限委托给 Power BI。 有关详细信息，请参阅[应用令牌流](/documentation/articles/power-bi-embedded-app-token-flow/)。

## <a name="create-or-edit-reports-within-your-application"></a>在应用程序内创建或编辑报表

现在，你可以直接在应用程序中编辑现有报表或创建新报表，而非必须使用 Power BI Desktop。 这要求数据集存在于你的工作区中。

## <a name="see-also"></a>另请参阅

- [常见 Power BI Embedded 方案](/documentation/articles/power-bi-embedded-scenarios/)  
- [Power BI Embedded 入门](/documentation/articles/power-bi-embedded-get-started/)  
- [示例入门](/documentation/articles/power-bi-embedded-get-started-sample/)  
- [嵌入报表](/documentation/articles/power-bi-embedded-embed-report/)  
- [在 Power BI Embedded 中进行身份验证和授权](/documentation/articles/power-bi-embedded-app-token-flow/)  
- [JavaScript 嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
- [PowerBI-CSharp Git 存储库](https://github.com/Microsoft/PowerBI-CSharp)  
- [PowerBI-Node Git 存储库](https://github.com/Microsoft/PowerBI-Node)  
- 有更多问题？ [试用 Power BI 社区](http://community.powerbi.com/)

<!--Update_Description: wording update-->