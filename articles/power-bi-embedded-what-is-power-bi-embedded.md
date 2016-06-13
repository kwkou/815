<properties
   pageTitle="Microsoft Power BI Embedded 是什么"
   description="Power BI Embedded 使你可以将 Power BI 报表集成到你的 Web 或移动应用程序中，因此你无需为你的用户生成可视化数据的自定义解决方案"
   services="power-bi-embedded"
   documentationCenter=""
   authors="dvana"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="power-bi-embedded"
   ms.date="03/29/2016"
   wacn.date="05/30/2016"/>


# Microsoft Power BI Embedded 是什么

**Microsoft Power BI Embedded** 使你可以将 Power BI 报表集成到你的 Web 或移动应用程序中，因此你无需为你的用户生成可视化数据的自定义解决方案。

![](./media/powerbi-embedded-whats-is/what-is.png)

**Microsoft Power BI Embedded** 是一项 Azure 服务，使独立软件供应商 (ISV) 能够在其自己的应用程序中感受 Power BI 数据体验。作为 ISV，你一定已生成了应用程序。这些应用程序具有其自己的用户和一组不同的功能。这些应用也可能碰巧具有内置的数据元素（如图表和报表），这些元素现在可能由 **Microsoft Power BI Embedded** 提供支持。应用程序用户无需 Power BI 帐户便可使用你的应用。他们可以像以前那样继续登录到你的应用程序，查看并与 Power BI 报表和磁贴体验进行交互而无需任何额外的授权。

## Microsoft Power BI Embedded 的授权

在 **Microsoft Power BI Embedded** 使用模型中，Power BI 的授权不是由最终用户负责的。而是应该由使用视觉效果的应用程序开发人员来购买**渲染器**，费用计入拥有这些资源的订阅中。

## Microsoft Power BI Embedded 概念模型

![](./media/powerbi-embedded-whats-is/model.png)

如 Azure 中的其他服务一样，**Microsoft Power BI Embedded** 的资源也是通过 [Azure ARM API](https://msdn.microsoft.com/zh-cn/library/mt712306.aspx) 进行预配的。此示例中，你预配的资源是一个 **Power BI 工作区集合**。

## 工作区集合

**工作区集合**是一个顶级 Azure 资源容器，包含 0 个或多个**工作区**。**工作区****集合**具有所有标准的 Azure 属性和下列内容：

-	**访问密钥** – 安全调用 Power BI API（后面部分所述）时使用的密钥。
-	**用户** – 具有管理员权限的 Azure Active Directory (AAD) 用户，通过 Azure 门户或 ARM API 管理 Power BI 工作区集合。
-	**区域** – 预配**工作区集合**的一部分，你可以选择在其中进行预配的区域。有关详细信息，请参阅 [Azure 区域](https://azure.microsoft.com/regions/)。

## 工作区

**工作区** 是一个包含 Power BI 内容的容器，Power BI 内容可以包括数据集、报表和仪表板。**工作区**在首次创建时为空。在预览版期间，你可以使用 Power BI Desktop 创建所有内容，然后使用 [Power BI REST API](http://docs.powerbi.apiary.io/reference) 将其加载到你的其中一个工作区。

## 使用工作区集合和工作区
**工作区集合**和**工作区**是内容容器，无论哪种方式都适合用于对你创建的应用程序进行设计和整理。它们提供了许多不同的方式供你在其中排列内容。你可以选择将所有内容都放在一个工作区中并在稍后使用应用程序令牌进一步细分你的客户之间的内容。你还可以选择将所有客户放在单独的工作区中，使他们分开一些。或者，你也可能选择按区域而不是按客户来整理用户。这种设计十分灵活，你可以选择最佳的方式来整理内容。
## 预览版中的数据源

对于预览版，我们启用的数据源有限，如下所示：

### 直接查询

对于预览版中的云资源，我们将支持直接查询连接。这意味着你可以连接到其数据源，以显示最新的数据。这些数据源应能够从云获得并且应使用基本身份验证。关于数据源的部分理想候选项包括：

-	SQL Azure
-	SQL Azure DW
-	HD Insight Spark

## 缓存的数据集

预览版可以使用缓存的数据集。但是，一旦将缓存的数据加载到 **Microsoft Power BI Embedded** 之后便不可刷新该数据。

## 使用应用程序令牌进行身份验证和授权

**Microsoft Power BI Embedded** 将按照你的应用程序来执行所有必要的用户身份验证和授权。没有对最终用户必须是 Azure Active directory (Azure AD) 的客户这一条显式要求。相反，你的应用程序将明示授权：通过**应用程序身份验证令牌（应用程序令牌）**向 **Microsoft Power BI Embedded** 呈现 Power BI 报表。如果想要呈现一个报表，可以根据需要创建这些**应用程序令牌**。请参阅[应用程序令牌](/documentation/articles/power-bi-embedded-get-started-sample.com/zh-cn/library#key-flow)。

![](./media/powerbi-embedded-whats-is/app-tokens.png)

### 应用程序身份验证令牌

**应用程序身份验证令牌（应用程序的令牌）**用于对 **Microsoft Power BI Embedded** 进行身份验证。有三种类型的**应用程序令牌**：

1.	预配令牌 - 在**工作区集合**中预配新的**工作区**时使用
2.	开发令牌 - 直接对 **Power BI REST API** 进行调用时使用
3.	嵌入令牌 - 调用以便在嵌入式 iframe 中呈现报表时使用

这些令牌适用于与 **Microsoft Power BI Embedded** 进行交互的各种阶段。这些令牌经过专门设计，以便你可以将应用中的权限委托给 Power BI。

### 生成应用程序令牌

你可以使用预览版中提供的 SDK 生成令牌。首先，调用 Create/_/_/_Token() 方法之一。其次，调用 Generate() 方法，其中访问密钥可从**工作区集合**进行检索。令牌的基本创建方法请参见 Microsoft.PowerBI.Security.PowerBIToken 类中的定义，如下所示：

-	[CreateProvisionToken](https://msdn.microsoft.com/zh-cn/library/mt670218.aspx)
-	[CreateDevToken](https://msdn.microsoft.com/zh-cn/library/mt670215.aspx)
-	[CreateReportEmbedToken](https://msdn.microsoft.com/zh-cn/library/mt710366.aspx)

有关如何使用 [CreateProvisionToken](https://msdn.microsoft.com/zh-cn/library/mt670218.aspx) 和 [CreateDevToken](https://msdn.microsoft.com/zh-cn/library/mt670215.aspx) 的示例，请参阅 [Microsoft Power BI Embedded 代码示例入门](/documentation/articles/power-bi-embedded-get-started-sample.com/zh-cn/library)。


## 另请参阅
- [常见 Microsoft Power BI Embedded 方案](/documentation/articles/power-bi-embedded-scenarios.com/zh-cn/library)
- [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started.com/zh-cn/library)
- [应用程序令牌](/documentation/articles/power-bi-embedded-get-started-sample#key-flow)
- [Power BI REST API](http://docs.powerbi.apiary.io/reference)

<!---HONumber=Mooncake_0523_2016-->