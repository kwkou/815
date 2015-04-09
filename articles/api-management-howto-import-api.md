<properties pageTitle="API 管理关键概念" metaKeywords="" description="了解有关 API、产品、角色、组和其他 API 管理关键概念。" metaCanonical="" services="" documentationCenter="API Management" title="API 管理关键概念" authors="sdanie" solutions="" manager="" editor="" />

# 如何通过 Azure API 管理中的操作导入 API 的定义

在 API 管理（预览版）中，可以创建新的 API，并手动添加操作，或 API 可以随着一个步骤中的操作导入。

API 和它们的操作可以使用以下格式导入。

-   WADL
-   Swagger

本指南演示如何创建一个新的 API，并在一个步骤中导入它。

> 有关手动创建一个 API 和添加操作的信息，请参阅[如何创建 Api][如何创建 Api]和 [如何将操作添加到 API][如何将操作添加到 API]。

## 本主题内容

-   [导入一个 API][导入一个 API]
-   [导出一个 API][导出一个 API]
-   [后续步骤][后续步骤]

## <a name="import-api"> </a>导入一个 API

要创建和配置 API，请单击 API 管理系统服务的 Azure 门户中的**管理控制台**。这使您转到 API 管理管理门户。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![管理控制台][管理控制台]

单击左侧 **API 管理**菜单的 **API**，然后单击**导入 API**。

![导入 API][导入 API]

**导入 API**窗口具有对应于提供 API 规范的三种方法的三个选项卡。

-   **剪贴板**允许您将 API 规格粘贴到指定文本框。
-   **从文件**允许您浏览并选择包含 API 规范的文件。
-   **从 URL**允许您向 API 的规范提供 URL。

![导入 API 格式][导入 API 格式]

提供 API 规范后，使用右侧的单选按钮来指示的规范格式。支持以下格式任务：

-   WADL
-   Swagger

接下来，输入 **Web API URL suffix**。这附加到您的 API 管理服务的基础 URL。基础 URL 对 API 管理服务的每个实例上托管的所有 API 很常见。API 管理通过后缀区分 API，并且对于特定的 API 管理服务实例特定的每个 API，此后缀必须唯一。

所有值都输入后，单击**保存**创建 API 和关联的操作。

## <a name="export-api"> </a>导出一个 API

除了导入新的 API，您可以从管理控制台导出您的 API 的定义。若要这样做，请单击 **API** 的**摘要选项卡**的**导出 API**。

![导出 API][导出 API]

可以使用 WADL 或 Swagger 导出 API。选择所需的格式，请单击**保存**，然后选择要在其中保存该文件的位置。

![导出 API 格式][导出 API 格式]

## <a name="next-steps"> </a>后续步骤

API 创建并导入操作后，您可以查看和配置任何其他设置，将 API 添加到一种产品并将其发布，这样就可提供给开发人员。有关详细信息，请参阅下面的指南。

-   [如何配置 API 设置][如何配置 API 设置]
-   [如何创建和发布产品][如何创建和发布产品]

  [如何创建 Api]: ../api-management-howto-create-apis
  [如何将操作添加到 API]: ../api-management-howto-add-operations
  [导入一个 API]: #import-api
  [导出一个 API]: #export-api
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [管理控制台]: ./media/api-management-howto-import-api/api-management-management-console.png
  [导入 API]: ./media/api-management-howto-import-api/api-management-import-apis.png
  [导入 API 格式]: ./media/api-management-howto-import-api/api-management-import-api-clipboard.png
  [导出 API]: ./media/api-management-howto-import-api/api-management-export-api.png
  [导出 API 格式]: ./media/api-management-howto-import-api/api-management-export-api-format.png
  [如何配置 API 设置]: ../api-management-howto-create-apis/#configure-api-settings
  [如何创建和发布产品]: ../api-management-howto-add-products
