<properties pageTitle="如何缓存 Azure API 管理中的操作结果" metaKeywords="" description="了解如何改善滞后时间、带宽消耗和 API 管理服务调用的 web 服务负载。" metaCanonical="" services="" documentationCenter="API Management" title="如何缓存 Azure API 管理中的操作结果" authors="sdanie" solutions="" manager="" editor="" />

# 如何缓存 Azure API 管理中的操作结果

API 管理（预览版）中的操作可以配置为响应缓存。响应缓存可以显著减少 API 延迟、带宽消耗和不经常更改数据的 web 服务负载。

本教程中将查看一个示例 Echo Api 操作的缓存设置和策略，并在开发人员门户中调用该操作以查看操作中的缓存。

## 本主题内容

-   [为缓存配置操作][为缓存配置操作]
-   [查看缓存策略][查看缓存策略]
-   [调用操作和测试缓存][调用操作和测试缓存]
-   [后续步骤][后续步骤]

## <a name="configure-caching"> </a>为缓存配置操作

在此步骤中，您将查看示例 Echo Api 的 **GET 资源（已缓存）**操作的的缓存设置。

> 每个预先配置 Echo API 的 API 管理服务实例，都可用于试验和了解 API 管理。有关详细信息，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]。

要开始，请单击您的 API 管理服务的 Azure 门户中的**管理控制台**。这使您转到 API 管理管理门户。

![API 管理控制台][API 管理控制台]

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

单击左侧 **API 管理**菜单的 **API**，然后单击 **Echo API**。

![Echo API][Echo API]

选择**操作**选项卡，然后单击**操作**列表的 **GET 资源（缓存）**操作。

![Echo API 操作][Echo API 操作]

选择**缓存**选项卡查看此操作的缓存设置。

![缓存选项卡][缓存选项卡]

要启用缓存的操作，请选中**启用**复选框。在此示例中启用缓存。

每个操作的响应基于**根据查询字符串参数变化**和**根据标头变化**字段中的值进行键控。如果您要缓存基于查询字符串参数或标头的多个响应，可以在这两个字段中对它们进行配置。

**持续时间**指定缓存响应的过期时间间隔。在此示例中时间间隔是 **3600** 秒，相当于一小时。

在此示例中使用缓存配置，对 **GET 资源（缓存）**操作的第一个请求将从服务返回响应。将缓存此响应，由指定的标头和查询字符串参数进行键控。采用匹配的参数，对操作的后续调用会返回缓存的响应，直到缓存时间间隔过期。

## <a name="caching-policies"> </a>查看缓存策略

为操作在**缓存**选项卡上配置缓存设置时，为操作添加缓存策略。可以在策略编辑器中查看并编辑这些策略。

单击左侧 **API 管理**菜单的**策略**，然后从**操作**下拉列表选择 **Echo API / GET 资源（缓存）**。

![策略范围操作][策略范围操作]

这将在策略编辑器中显示此操作的策略。

![API 管理策略编辑器][API 管理策略编辑器]

此操作的策略定义包括定义缓存配置的策略，使用上一步中**缓存**选项卡审核。

    <policies>
        <inbound>
            <base />
            <cache-lookup vary-by-developer="false" vary-by-developer-groups="false">
                <vary-by-header>Accept</vary-by-header>
                <vary-by-header>Accept-Charset</vary-by-header>
            </cache-lookup>
            <rewrite-uri template="/resource" />
        </inbound>
        <outbound>
            <base />
            <cache-store caching-mode="cache-on" duration="3600" />
        </outbound>
    </policies>

> 在策略编辑器中的缓存策略更改将反映在操作的**缓存**选项卡中，反之亦然。

## <a name="test-operation"> </a>调用操作和测试缓存

要查看作用的缓存，我们可以从开发人员门户调用操作。单击右上方菜单中的**开发人员门户**。

![开发人员门户][开发人员门户]

单击顶部菜单中的 **API** 并选择 **Echo API**。

![Echo API][1]

> 如果必须只有一个 API 得到配置或对您的帐户可见，然后单击 API 使您直接进入该 API 的操作。

选择 **GET 资源（缓存）**操作，单击**打开控制台**。

![打开控制台][打开控制台]

控制台允许直接从开发人员门户调用操作。

![控制台][控制台]

保留 **param1** 和 **param2** 的默认值。

从 **subscription-key** 下拉列表选择所需的密钥。如果您的帐户只有一个订阅，将已经选择它。

在**请求标头**文本框中输入 **sampleheader:value1**。

单击 **HTTP Get** 并记下响应标头。

在**请求标头**文本框中输入 **sampleheader:value2**，然后单击 **HTTP Get**。

请注意，**sampleheader** 的值仍是响应中的 **value1**。尝试一些不同的值，请注意返回来自第一次调用的缓存响应。

将 **25** 输入 **param2** 字段，然后单击 **HTTP Get**。

请注意，响应中 **sampleheader** 的值现在是 **value2**。因为操作结果都由查询字符串进行键控，所以没有返回以前缓存的响应。

## <a name="next-steps"> </a>后续步骤

-   查看[开始使用高级 API 配置][开始使用高级 API 配置]教程中的其他主题。
-   有关缓存策略的详细信息，请参阅 [API 管理策略参考][API 管理策略参考]中的[缓存策略][缓存策略]。

  [为缓存配置操作]: #configure-caching
  [查看缓存策略]: #caching-policies
  [调用操作和测试缓存]: #test-operation
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [API 管理控制台]: ./media/api-management-howto-cache/api-management-management-console.png
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [Echo API]: ./media/api-management-howto-cache/api-management-echo-api.png
  [Echo API 操作]: ./media/api-management-howto-cache/api-management-echo-api-operations.png
  [缓存选项卡]: ./media/api-management-howto-cache/api-management-caching-tab.png
  [策略范围操作]: ./media/api-management-howto-cache/api-management-operation-dropdown.png
  [API 管理策略编辑器]: ./media/api-management-howto-cache/api-management-policy-editor.png
  [开发人员门户]: ./media/api-management-howto-cache/api-management-developer-portal-menu.png
  [1]: ./media/api-management-howto-cache/api-management-apis-echo-api.png
  [打开控制台]: ./media/api-management-howto-cache/api-management-open-console.png
  [控制台]: ./media/api-management-howto-cache/api-management-console.png
  [开始使用高级 API 配置]: ../api-management-get-started-advanced
  [API 管理策略参考]: ../api-management-policy-reference
  [缓存策略]: ../api-management-policy-reference/#caching-policies
