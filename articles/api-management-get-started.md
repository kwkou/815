<properties pageTitle="开始使用 Azure API 管理" metaKeywords="" description="了解如何创建 API、操作并开始使用 API 管理。" metaCanonical="" services="" documentationCenter="API Management" title="开始使用 Azure API 管理" authors="sdanie" solutions="" manager="" editor="" />

# 开始使用 Azure API 管理

本指南演示如何快速开始使用 API 管理，并进行您的首次 API 调用。

## 本主题内容

-   [创建 API 管理实例][创建 API 管理实例]
-   [创建 API][创建 API]
-   [添加操作][添加操作]
-   [将新的 API 添加到产品][将新的 API 添加到产品]
-   [订阅包含 API 的产品][订阅包含 API 的产品]
-   [从开发人员门户调用操作][从开发人员门户调用操作]
-   [查看分析][查看分析]
-   [后续步骤][后续步骤]

## <a name="create-service-instance"> </a>创建 API 管理实例

> 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][Azure 免费试用]。

使用 API 管理的第一步是创建一个服务实例。登录[管理门户][管理门户]，然后单击**新建**、**应用程序服务**、**API 管理**、**创建**。

![API 管理新实例][API 管理新实例]

对于 **URL**，指定一个唯一的子域名用于服务 URL。

为您的服务实例选择所需的**定价层**、**订阅**和**区域**。所有定价层可用于本教程。做出选择后，单击下一步按钮。

![新的 API 管理服务][新的 API 管理服务]

为**组织名称**输入 **Contoso Ltd.**，然后在管理员电子邮件字段中输入电子邮件地址。

> 此电子邮件地址用于来自 API 管理系统的通知。有关详细信息，请参阅[配置通知][配置通知]。

单击复选框创建您的服务实例。

![新的 API 管理服务][1]

![新的 API 管理服务][2]

创建服务实例后,下一步是创建 API。

## <a name="create-api"> </a>创建 API

API 包含一组可以从客户端应用程序调用的操作。API 操作代理到现有的 web 服务。

每个 API 管理服务实例都通过示例 Echo API 预先配置，它返回发送给它的输入。要使用它，您可以调用任何 HTTP 谓词，并且返回值将等于为标头和您发送的正文。

本教程使用 http://echoapi.cloudapp.net/api web 服务，在调用 **My Echo Service** 的 API 管理中创建新的 API。

从 API 管理控制台创建和配置 API，通过 Azure 管理门户访问。要访问 API 管理控制台，请为 API 管理服务单击 Azure 门户中的**管理控制台**。

![新的 API 管理控制台][新的 API 管理控制台]

要创建 **My Echo API**，请单击左侧 **API 管理**菜单上的 **API**，然后单击**添加 API**。

![创建 API][3]

![添加新的 API][添加新的 API]

以下三个字段用于配置新的 API。

-   将 **My Echo API** 键入 **Web API 标题**文本框。**Web API 标题**为 API 提供唯一并且描述性的名称。它显示在开发人员和管理门户中。
-   将 **http://echoapi.cloudapp.net/api** 键入 **Web 服务 URL**。**Web 服务 URL** 引用执行该 API 的 HTTP 服务。API 管理将请求转发到此地址。
-   将 **myecho** 键入 **Web API URL 后缀**。**Web API URL 后缀**附加到 API 管理服务的基础 URL。您的 API 将共享一个公共基础 URL，并通过基础后面附加的独特后缀进行区分。

单击**保存**创建 API。一旦创建新的 API，该 API 的摘要页显示在管理门户中。

![API 摘要][API 摘要]

API 部分有四个选项卡。**摘要**选项卡显示有关 API 的基本度量和信息。**设置**选项卡用于查看和编辑 API 的配置，包括后端服务的身份验证凭据。**操作**选项卡用于管理 API 的操作，并用在教程中下面的步骤，而**问题**选项卡可以用于查看使用您的 API 的开发人员报告的问题。

> 回声 API 示例不使用身份验证，但有关配置身份验证的详细信息，请参阅[配置 API 设置][配置 API 设置]。

一旦创建 API 和配置设置，下一步是将操作添加到 API。操作定义用于验证传入的请求并自动生成文档。

## <a name="add-operation"> </a>添加操作

单击**操作**显示 API 的操作窗格。因为我们未添加任何操作，所以没有显示。

![操作][操作]

单击**添加操作**添加新操作。将显示**新操作**窗口，并且在默认情况下将选择**签名**选项卡。

![操作签名][操作签名]

此示例中，我们将指定回声服务上的 GET 操作。在**签名**选项卡上的字段中输入以下值。

-   将 **GET** 键入 **HTTP 谓词**文本框。开始键入后，可以从显示的 http 谓词的列表选择 **GET**。
-   将 **/resource** 键入 **URL 模板**文本框。
-   将 **GET resource** 键入**显示**名称文本框。
-   将 **A demonstration of a GET call on a sample resource.It is handled by an "echo" backend which returns a response equal to the request (the supplied headers and body are being returned as received).** 键入**说明**文本框。开发人员使用此 API 时，此描述用于生成此操作的文档。

单击**参数**为此操作配置查询字符串参数。此示例中有两个查询字符串参数。要添加查询参数，请单击**添加查询参数**并指定以下值。

对于第一个查询参数，配置下面的值。

-   将 **param1** 键入**名称**文本框。
-   将 **A sample parameter that is required.** 键入**说明**文本框。
-   单击**类型**字段，然后从列表选择**字符串**。支持的类型为**字符串**、**数字**、**布尔值**和 **dateTime**。
-   单击**值**字段，将**示例**键入文本框，然后单击加号将默认值文本添加到参数。添加默认文本后，单击**值**字段外的任意位置以关闭添加值窗口。
-   选中**需要**复选框。

对于第二个查询参数，输入下面的值。

-   “名称”：**param2**
-   **描述**：**另一个示例参数，设置为不需要。**
-   **类型**：**数字**

为操作可能生成的所有状态代码提供相应示例是不错的做法。每个状态代码可能有多个响应正文示例，分别针对每种支持的内容类型。本教程中，我们将添加一个 **200 OK** 响应代码。

单击“响应”部分中的**添加**，开始键入 **200** 到文本框，然后再从下拉列表选择 **200 OK**。

![添加响应][添加响应]

一旦选择 **200 OK**，一个新的响应代码添加到操作，响应窗口随即显示。将 **Returned in all cases.** 键入**说明**文本框。

![添加响应][4]

> **添加表示**用于在多种表示形式中配置响应。有关详细信息，请参阅[响应][响应]。

单击**保存**将新配置的操作添加到 API。

## <a name="add-api-to-product"> </a>将新的 API 添加到产品

开发人员必须首先订阅产品，然后才能进行 API 调用。一个产品提供对一个或多个 API 的访问，并且可以包含使用配额和速率限制等访问限制。在教程的此步骤，您将 My Echo API 添加到现有产品。

单击左侧 **API 管理**菜单的**产品**，查看和配置此 API 实例中提供的产品。

![产品][产品]

默认情况下，每个 API 管理实例附带两个示例产品：

-   **受限制**
-   **不受限制**

本教程中我们将使用**受限制**产品。单击**受限制**的**配置**查看设置，包括和该产品关联的 API。

![添加 API][添加 API]

单击**将 API 添加到产品**。

![添加 API][5]

为 **My Echo API** 选中此框，然后单击**保存**。

![添加的 API][添加的 API]

既然 **My Echo API** 与产品关联，开发人员可以订阅它并开始使用 API。

> 此教程步骤使用受限制的产品，它预先配置并可供使用。有关创建和发布新产品的分步指南，请参阅[如何创建和发布产品][如何创建和发布产品]。

## <a name="subscribe"> </a>订阅包含 API 的产品

为了对 API 调用，开发人员必须首先订阅一种产品，赋予它们访问权。开发人员可以在开发人员门户中订阅产品，管理员可以在管理控制台中将开发人员订阅到产品。您是默认管理员，因为您在教程前面的步骤中创建 API 管理实例，所以您将帐户订阅到“受限制”产品。

单击左侧 **API 管理**菜单的**开发人员**，查看和配置此服务实例中的开发人员。

![开发人员][开发人员]

单击**管理**用户右侧的**详细信息**为用户配置设置，包括订阅。

![添加订阅][添加订阅]

单击**添加订阅**。

![添加订阅][6]

为**有限制**选中此框并单击**订阅**。

![添加的订阅][添加的订阅]

订阅开发人员帐户后，您可以调用该产品的 API。

## <a name="call-operation"> </a>从开发人员门户调用操作

可以直接从开发人员门户调用操作，这样可以方便地查看和测试 API 的操作。在此教程步骤中，您将调用已添加到 **My Echo API** 的 Get 方法。单击管理门户右上角菜单中的**开发人员门户**。

![开发人员门户][开发人员门户]

单击顶部菜单的 **API**，然后单击 **My Echo API** 查看可用的操作。

![开发人员门户][7]

请注意将显示创建操作时添加的说明和参数，提供将使用此操作的开发人员的文档。

单击 **GET 资源**，然后单击**打开控制台**。

![操作控制台][操作控制台]

为参数输入一些值并指定您的开发人员密钥，然后单击 **HTTP Get**。

![HTTP Get][HTTP Get]

调用操作后，开发人员门户将从后端服务显示**请求的 URL**、**响应状态**、**响应标头**以及任何**响应内容**。

![响应][8]

## <a name="view-analytics"> </a>查看分析

要查看 **My Echo API** 的分析，请选择开发人员门户右上方的用户菜单的**管理**切换回管理门户。

![管理][管理]

管理门户的默认视图是仪表板，它提供 API 管理实例的概述。

![仪表板][仪表板]

将鼠标悬停在 My Echo API 的图表上，以查看给定时间段内 API 的使用情况的特定度量。

> 如果没有在图表上看到任何行，切换回开发人员门户并对 API 进行一些调用，稍等片刻再返回仪表板。

![分析][分析]

单击**查看详细信息**以查看 API 的摘要页，包括所显示度量的更大版本。

![摘要][摘要]

有关详细的度量和报告，请单击左侧 **API 管理**菜单的**分析**。

![概述][概述]

**分析**部分包含以下四个选项卡。

-   **快览**提供总体使用情况和运行状况指标，以及顶级开发人员、顶级产品、顶级 API 和顶级运算。
-   **使用情况**提供对 API 调用和带宽的深入观察，包含地理位置的表示形式。
-   **运行状况**侧重状态代码、缓存成功率、响应时间和 API 以及服务响应时间。
-   **活动**提供报表，向下钻取有关开发人员、产品、API 和操作的特定活动。

## <a name="next-steps"> </a>后续步骤

-   查看[开始使用高级 API 配置][开始使用高级 API 配置]教程中的其他主题。

  [创建 API 管理实例]: #create-service-instance
  [创建 API]: #create-api
  [添加操作]: #add-operation
  [将新的 API 添加到产品]: #add-api-to-product
  [订阅包含 API 的产品]: #subscribe
  [从开发人员门户调用操作]: #call-operation
  [查看分析]: #view-analytics
  [后续步骤]: #next-steps
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/
  [管理门户]: https://manage.windowsazure.cn/
  [API 管理新实例]: ./media/api-management-get-started/api-management-create-instance-menu.png
  [新的 API 管理服务]: ./media/api-management-get-started/api-management-create-instance-step1.png
  [配置通知]: ../api-management-howto-configure-notifications
  [1]: ./media/api-management-get-started/api-management-create-instance-step2.png
  [2]: ./media/api-management-get-started/api-management-instance-created.png
  [新的 API 管理控制台]: ./media/api-management-get-started/api-management-management-console.png
  [3]: ./media/api-management-get-started/api-management-create-api.png
  [添加新的 API]: ./media/api-management-get-started/api-management-add-new-api.png
  [API 摘要]: ./media/api-management-get-started/api-management-new-api-summary.png
  [配置 API 设置]: ../api-management-howto-create-apis/#configure-api-settings
  [操作]: ./media/api-management-get-started/api-management-myecho-operations.png
  [操作签名]: ./media/api-management-get-started/api-management-operation-signature.png
  [添加响应]: ./media/api-management-get-started/api-management-add-response.png
  [4]: ./media/api-management-get-started/api-management-add-response-window.png
  [响应]: ../api-management-howto-add-operations/#responses
  [产品]: ./media/api-management-get-started/api-management-list-products.png
  [添加 API]: ./media/api-management-get-started/api-management-add-api-to-product.png
  [5]: ./media/api-management-get-started/api-management-add-myechoapi-to-product.png
  [添加的 API]: ./media/api-management-get-started/api-management-api-added-to-product.png
  [如何创建和发布产品]: ../api-management-howto-add-products
  [开发人员]: ./media/api-management-get-started/api-management-developers.png
  [添加订阅]: ./media/api-management-get-started/api-management-add-subscription.png
  [6]: ./media/api-management-get-started/api-management-add-subscription-window.png
  [添加的订阅]: ./media/api-management-get-started/api-management-subscription-added.png
  [开发人员门户]: ./media/api-management-get-started/api-management-developer-portal-menu.png
  [7]: ./media/api-management-get-started/api-management-developer-portal-myecho-api.png
  [操作控制台]: ./media/api-management-get-started/api-management-developer-portal-myecho-api-console.png
  [HTTP Get]: ./media/api-management-get-started/api-management-invoke-get.png
  [8]: ./media/api-management-get-started/api-management-invoke-get-response.png
  [管理]: ./media/api-management-get-started/api-management-manage-menu.png
  [仪表板]: ./media/api-management-get-started/api-management-dashboard.png
  [分析]: ./media/api-management-get-started/api-management-mouse-over.png
  [摘要]: ./media/api-management-get-started/api-management-api-summary-metrics.png
  [概述]: ./media/api-management-get-started/api-management-analytics-overview.png
  [开始使用高级 API 配置]: ../api-management-get-started-advanced
