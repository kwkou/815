<properties pageTitle="如何将操作添加到 Azure API 管理中的 API" metaKeywords="" description="了解如何将操作添加到 Azure API 管理中的 API" metaCanonical="" services="" documentationCenter="API Management" title="如何将操作添加到 Azure API 管理中的 API" authors="sdanie" solutions="" manager="" editor="" />

# 如何将操作添加到 Azure API 管理中的 API

在可使用 API 管理（预览版）中的 API 之前，必须添加操作。本指南演示如何添加和配置不同类型的 API 操作到 API 管理。

## 本主题内容

-   [添加操作][添加操作]
-   [操作缓存][操作缓存]
-   [请求参数][请求参数]
-   [请求正文][请求正文]
-   [响应][响应]
-   [后续步骤][后续步骤]

## <a name="add-operation"> </a>添加操作

添加并配置操作到管理控制台中的 API。要访问管理控制台，请单击 API 管理服务的 Azure 门户中的**管理控制台**。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![API 管理控制台][API 管理控制台]

在 API 管理门户中选择所需的 API，然后选择**操作**选项卡。

![操作][操作]

单击**添加操作**添加新操作。将显示**新操作**，并且在默认情况下将选择**签名**选项卡。

![添加操作][1]

通过从下拉列表中选择指定 **HTTP 谓词**。

![HTTP 方法][HTTP 方法]

通过键入一个或多个 URL 路径段和零个或多个查询字符串参数组成的 URL 片段定义 URL 目标。URL 模板附加到 API 的基础 URL，标识单个 HTTP 操作。它可能包含一个或多个由大括号标识的命名变量部分。这些变量部分称为模板参数，并在管理平台处理请求时是从请求的 URL 中提取的动态分配值。

![URL 模板][URL 模板]

如需要，请指定**重写 URL 模板**。这允许您在前端使用标准的 URL 模板处理传入的请求，同时通过根据重写模板转换的 URL 调用后端。来自 URL 模板的模板参数应用在重写模板中。下面的示例演示内容类型如何编码为上一示例 web 服务中的路径段，它可以提供作为通过使用 URL 模板的 API 管理平台发布的 API 中的查询参数。

![URL 模板重写][URL 模板重写]

操作的调用方将使用格式 `/customers?customerid=ALFKI`，调用后端服务时这将映射到 `/Customers('ALFKI')`。

**显示**名称和**说明**提供操作的说明，并用于向使用开发人员门户中此 API 的开发人员提供文档。

![说明][说明]

操作说明可指定为**说明**文本框中的纯文本或 HTML。

## <a name="operation-caching"> </a>操作缓存

响应缓存可减少 API 使用者的延迟，降低带宽消耗并减少执行 API 的 HTTP web 服务上的负载。

要轻松快速地启用操作缓存，请选择**培训**选项卡并选中**启用**复选框。

![Caching][Caching]

**持续时间**指定在此期间操作响应保留在缓存中的时间段。默认值为 3600 秒或 1 小时。

缓存键用于区分这两个响应，以便与每个不同缓存键对应的响应将获得自己单独的缓存值。（可选）分别输入特定的查询字符串参数，和/或要在**通过查询字符串参数变化**和**按标头变化**文本框中的计算缓存键值的 HTTP 标头。如果指定无，完整请求 URL 和以下 HTTP 标头值用在缓存密钥生成过程中：**接受**和**接受字符集**。

> 有关缓存和缓存策略的详细信息，请参阅[如何在 Azure API 管理终缓存操作结果][如何在 Azure API 管理终缓存操作结果]。

## <a name="request-parameters"> </a>请求参数

在“参数”选项卡上管理操作参数。**签名**选项卡上的 **URL 模板**中指定的参数会自动添加，并且只能在编辑 URL 模板时更改。可以手动输入其他参数。

要添加新的查询参数，请单击**添加查询参数**并输入以下信息：

-   **名称** - 参数名称。
-   **说明** - 参数的简要说明（可选）。
-   **类型**-参数类型，在下拉菜单中选择。
-   **值** - 可以分配给此参数的值。其中一个值可被标记为默认（可选）。
-   **所需** - 选中复选框让参数为必须。

![请求参数][2]

## <a name="request-body"> </a>请求正文

如果允许该操作（例如 PUT、 POST）并且要求一个实例的正文，可采用受支持的表示形式格式(例如 json、 XML）提供一个示例。

> 请求正文仅用于文档目的而并非验证。

要输入请求的正文，请切换到**正文**选项卡。

单击**添加表示**，开始键入所需的内容类型名称（例如应用程序/json），在下拉列表中选择它，并按所选格式将所需的请求正文示例粘贴到文本框。

![请求正文][3]

除了表示形式，您还可在**说明**文本框中指定一个可选文本说明。

## <a name="responses"> </a>响应

为操作可能生成的所有状态代码提供相应示例是不错的做法。每个状态代码可能有多个响应正文示例，分别针对每种支持的内容类型。

要添加一个响应，请单击添加响应并开始键入所需的状态代码。此示例中状态代码是 **200 OK**。代码显示在下拉列表中后，选择它，创建响应代码并将其添加到您的操作。

![响应代码][响应代码]

单击添加表示形式，开始键入所需的内容类型名称（例如 application/json），然后在下拉菜单中选择它。

![正文内容类型][正文内容类型]

将请求正文示例按所选格式粘贴到文本框中。

![响应正文][响应正文]

如需要，请将可选说明添加到**说明**文本框。

配置该操作后，单击**保存**。

## <a name="next-steps"> </a>后续步骤

将操作添加到一个 API 后,下一步是将该 API 与产品相关联并将其发布，以便开发人员可调用其操作。

-   [如何创建和发布产品][如何创建和发布产品]

  [添加操作]: #add-operation
  [操作缓存]: #operation-caching
  [请求参数]: #request-parameters
  [请求正文]: #request-body
  [响应]: #responses
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [API 管理控制台]: ./media/api-management-howto-add-operations/api-management-management-console.png
  [操作]: ./media/api-management-howto-add-operations/api-management-operations.png
  [1]: ./media/api-management-howto-add-operations/api-management-add-operation.png
  [HTTP 方法]: ./media/api-management-howto-add-operations/api-management-http-method.png
  [URL 模板]: ./media/api-management-howto-add-operations/api-management-url-template.png
  [URL 模板重写]: ./media/api-management-howto-add-operations/api-management-url-template-rewrite.png
  [说明]: ./media/api-management-howto-add-operations/api-management-description.png
  [Caching]: ./media/api-management-howto-add-operations/api-management-caching-tab.png
  [如何在 Azure API 管理终缓存操作结果]: ../api-management-howto-cache
  [2]: ./media/api-management-howto-add-operations/api-management-request-parameters.png
  [3]: ./media/api-management-howto-add-operations/api-management-request-body.png
  [响应代码]: ./media/api-management-howto-add-operations/api-management-response-code.png
  [正文内容类型]: ./media/api-management-howto-add-operations/api-management-response-body-content-type.png
  [响应正文]: ./media/api-management-howto-add-operations/api-management-response-body.png
  [如何创建和发布产品]: ../api-management-howto-add-products
