<properties pageTitle="如何在 Azure API 管理中创建 API" metaKeywords="" description="了解如何在 Azure API 管理中创建和配置 API。" metaCanonical="" services="" documentationCenter="API Management" title="如何在 Azure API 管理中创建 API" authors="sdanie" solutions="" manager="" editor="" />

# 如何在 Azure API 管理中创建 API

API 管理（预览版）中的 API 表示一组可由客户端应用程序调用的操作。在管理控制台中创建新的 API，然后添加所需的操作。一旦添加操作，该 API 添加到某一产品并可以发布。一旦发布一个 API，它可通过订阅并由开发人员使用。

本指南演示过程中的第一步：如何在 API 管理中创建和配置新的 API。有关添加操作和发布产品的详细信息，请参阅[如何将操作添加到 API][如何将操作添加到 API] 以及[如何创建和发布产品][如何创建和发布产品]。

## 本主题内容

-   [创建新的 API][创建新的 API]
-   [配置 API 设置][配置 API 设置]
-   [后续步骤][后续步骤]

## <a name="create-new-api"> </a>创建新的 API

要创建和配置 API，请单击 API 管理系统服务实例的 Azure 门户中的**管理控制台**。这使您转到 API 管理管理门户。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![管理控制台][管理控制台]

单击左侧 **API 管理**菜单的 **API**，然后单击**添加 API**。

![创建 API][创建 API]

使用**添加新的 API** 窗口以配置新的 API。

![添加新的 API][添加新的 API]

以下三个字段用于配置新的 API。

-   **Web API 标题**为 API 提供唯一并且描述性的名称。它显示在开发人员和管理门户中。
-   **Web 服务 URL** 引用执行该 API 的 HTTP 服务。API 管理将请求转发到此地址。
-   **Web API URL 后缀**附加到 API 管理服务的基础 URL。基础 URL 是常见的由 API 管理服务实例托管的所有 API。API 管理通过其后缀区分 API，因此后缀对给定发布者上的每个 API 必须唯一。

一旦配置三个值，则单击**保存**。一旦创建新的 API，该 API 的摘要页显示在管理门户中。

![API 摘要][API 摘要]

## <a name="configure-api-settings"> </a>配置 API 设置

您可以使用**设置**选项卡来验证并编辑一个 API 的配置。**Web API 标题**、**Web 服务 URL** 和**Web API URL 后缀**最初在创建 API 时设置，并且可在此处修改。**说明**提供可选的描述，**使用凭据**允许您配置基础 HTTP 身份验证。

![API 设置][API 设置]

要为执行 API 的 web 服务配置 HTTP 基本身份验证，请从**使用凭据**下拉列表选择**基本**，然后输入所需的凭据。

![基本身份验证设置][基本身份验证设置]

单击**保存**保存对 API 设置进行的任何更改。

## <a name="next-steps"> </a>后续步骤

一旦创建 API 并配置设置，接下来的步骤是将操作添加到该 API，将 API 添加到产品并将其发布，这样就可用于开发人员。有关详细信息，请参阅下面的两个指南。

-   [如何将操作添加到 API][如何将操作添加到 API]
-   [如何创建和发布产品][如何创建和发布产品]

  [如何将操作添加到 API]: ../api-management-howto-add-operations
  [如何创建和发布产品]: ../api-management-howto-add-products
  [创建新的 API]: #create-new-api
  [配置 API 设置]: #configure-api-settings
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [管理控制台]: ./media/api-management-howto-create-apis/api-management-management-console.png
  [创建 API]: ./media/api-management-howto-create-apis/api-management-create-api.png
  [添加新的 API]: ./media/api-management-howto-create-apis/api-management-add-new-api.png
  [API 摘要]: ./media/api-management-howto-create-apis/api-management-api-summary.png
  [API 设置]: ./media/api-management-howto-create-apis/api-management-api-settings.png
  [基本身份验证设置]: ./media/api-management-howto-create-apis/api-management-api-settings-credentials.png
