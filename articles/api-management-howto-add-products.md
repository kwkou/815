<properties pageTitle="如何在 Azure API 管理中创建和发布产品" metaKeywords="" description="了解如何在 Azure API 管理中创建和发布产品。" metaCanonical="" services="" documentationCenter="API Management" title="如何在 Azure API 管理中创建和发布产品" authors="sdanie" solutions="" manager="" editor="" />

# 如何在 Azure API 管理中创建和发布产品

在 Azure API 管理（预览版）中，产品包含一个或多个 API 以及使用配额和使用条款。一旦产品发布，开发人员可以订阅该产品，并开始使用产品的 API。本主题为开发人员提供创建产品、添加一个 API 以及将其发布的指南。

## 本主题内容

-   [创建产品][创建产品]
-   [将 API 添加到产品][将 API 添加到产品]
-   [将描述性信息添加到产品][将描述性信息添加到产品]
-   [发布产品][发布产品]
-   [使产品对开发人员可见][使产品对开发人员可见]
-   [查看产品的订户][查看产品的订户]
-   [后续步骤][后续步骤]

## <a name="create-product"> </a>创建产品

添加并配置操作到管理控制台中的 API。要访问管理控制台，请单击 API 管理服务的 Azure 门户中的**管理控制台**。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![API 管理控制台][API 管理控制台]

单击左侧菜单中的**产品**显示**产品**页，然后单击**添加产品**。

![产品][产品]

![新产品][新产品]

在**名称**字段终输入产品的描述性名称，在**说明**字段中输入产品的说明。

如果您希望管理员能够审阅和接受或拒绝订阅此产品的尝试，请选中**需要订阅审批**。如果未选中此框，则将自动批准订阅尝试。有关订阅的详细信息，请参阅[查看到产品的订户][查看产品的订户]。

![产品][1]

> 默认情况下新产品未发布，并仅对**管理员**组可见。

要配置产品，请单击**产品**选项卡中的产品名称。

## <a name="add-apis"> </a>将 API 添加到产品

**产品**页包含用于配置的四个链接：**摘要**、**设置**、**可见性**、和**开发人员**。**摘要**选项卡是您可在其中 API 和发布或取消发布产品的位置。

![摘要][摘要]

在发布您的产品前，您需要添加一个或多个 API。要执行此操作，请单击**将 API 添加到产品**。

![添加 API][添加 API]

选择所需的 API，然后单击**保存**。

## <a name="add-description"> </a>将描述性信息添加到产品

**设置**选项卡允许您提供有关该产品的详细的信息，如其用途，它提供访问权的 API 和其他有用的信息。内容针对调用 API 的开发人员，并可采用纯文本或 HTML 标记编写。

![产品设置][产品设置]

如果您想手动批准所有产品的订阅请求，请选择**需要订阅审批**。默认情况下会自动授予所有产品订阅。

选择填写**使用条款**字段描述必须接受哪些订户才能使用该产品的使用条款。

## <a name="publish-product"> </a>发布产品

在产品中的 API 可调用前，必须先发布该产品。在产品的**摘要**选项卡上，单击**发布**，然后单击**是，发布**以确认。要将以前发布的产品专用，请单击**取消发布**。

![发布产品][2]

## <a name="make-visible"> </a>使产品对开发人员可见

**可见性**选项卡允许您选择可在开发人员门户网站上查看该产品并订阅该产品的角色。

![产品可见性][产品可见性]

要启用或禁用组中对开发人员的产品的可见性，请选中或取消选中组旁边的复选框，然后单击**保存**。

> 有关详细信息，请参阅[如何创建和使用组以管理 Azure API 管理中的开发人员帐户][如何创建和使用组以管理 Azure API 管理中的开发人员帐户]。

## <a name="view-subscribers"> </a>查看产品的订户

**开发人员**选项卡列出订阅到产品的开发人员。通过单击开发人员的名称，可以查看每个开发人员的详细信息和设置。此示例中还没有开发人员订阅该产品。

![开发人员][开发人员]

## <a name="next-steps"> </a>后续步骤

一旦添加所需的 API 且产品发布，开发人员可以订阅该产品并开始调用 API。有关演示这些项目以及高级产品配置的教程，请参阅[如何在 Azure API 管理中创建和配置高级产品设置][如何在 Azure API 管理中创建和配置高级产品设置]。

  [创建产品]: #create-product
  [将 API 添加到产品]: #add-apis
  [将描述性信息添加到产品]: #add-description
  [发布产品]: #publish-product
  [使产品对开发人员可见]: #make-visible
  [查看产品的订户]: #view-subscribers
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [API 管理控制台]: ./media/api-management-howto-add-products/api-management-management-console.png
  [产品]: ./media/api-management-howto-add-products/api-management-products.png
  [新产品]: ./media/api-management-howto-add-products/api-management-add-new-product.png
  [1]: ./media/api-management-howto-add-products/api-management-products-page.png
  [摘要]: ./media/api-management-howto-add-products/api-management-new-product-summary.png
  [添加 API]: ./media/api-management-howto-add-products/api-management-add-apis-to-product.png
  [产品设置]: ./media/api-management-howto-add-products/api-management-product-settings.png
  [2]: ./media/api-management-howto-add-products/api-management-publish-product.png
  [产品可见性]: ./media/api-management-howto-add-products/api-management-product-visibility.png
  [如何创建和使用组以管理 Azure API 管理中的开发人员帐户]: ../api-management-howto-create-groups
  [开发人员]: ./media/api-management-howto-add-products/api-management-developer-list.png
  [如何在 Azure API 管理中创建和配置高级产品设置]: ../api-management-howto-product-with-rules
