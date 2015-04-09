<properties pageTitle="如何在 Azure API 管理中创建和配置高级产品设置" metaKeywords="" description="了解如何通过配额和速率限制策略配置产品。" metaCanonical="" services="" documentationCenter="API Management" title="如何在 Azure API 管理中创建和配置高级产品设置" authors="sdanie" solutions="" manager="" editor="" />

# 如何在 Azure API 管理中创建和配置高级产品设置

在 Azure API 管理（预览版）中，产品可高度配置以满足 API 发布者的要求。本主题演示如何配置某些高级产品设置，包括速率限制和配额策略。

在本教程中，您将创建一种免费试用版产品，每分钟最多 10 次调用，每周最多允许 200 次调用，发布它并测试速率限制策略。

## 本主题内容

-   [创建产品][创建产品]
-   [将 API 添加到产品][将 API 添加到产品]
-   [配置调用速率限制和配额策略][配置调用速率限制和配额策略]
-   [发布产品][发布产品]
-   [对产品订阅开发人员帐户][对产品订阅开发人员帐户]
-   [调用操作并测试速率限制][调用操作并测试速率限制]
-   [后续步骤][后续步骤]

## <a name="create-product"> </a>创建产品

在此步骤中，您将创建一种免费试用版产品，不需要订阅审批。

要开始，请单击您的 API 管理服务的 Azure 门户中的**管理控制台**。这使您转到 API 管理管理门户。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![API 管理控制台][API 管理控制台]

单击左侧 **API 管理**菜单中的**产品**以显示**产品**页。

![添加产品][添加产品]

单击**添加产品**以显示**添加新产品**弹出窗口。

![添加新产品][添加新产品]

将 **Free Trial** 键入**标题**文本框。

将 **Subscribers will be able to run 10 calls/minute up to a maximum of 200 calls/week after which access is denied.** 键入**说明**文本框。

如果您希望管理员审阅和接受或拒绝尝试此产品的订阅，请选中**需要订阅审批**。如果未选中此框，则将自动批准订阅尝试。在此示例中自动批准订阅，因此不选中该复选框。

输入所有值后，单击**保存**创建产品。

![添加的产品][添加的产品]

默认情况下，新产品对**管理员**组中的用户可见。我们要添加**开发人员**组。**单击免费试用版**，然后选择**可见性**选项卡。

> 在 API 管理中，使用组来管理产品对开发人员的可见性。产品向组授予可见性，并且开发人员可以查看和订阅对他们所属的组可见的产品。有关详细信息，请参阅[如何创建和使用 Azure API 管理中的组][如何创建和使用 Azure API 管理中的组]。

![添加开发人员组][添加开发人员组]

检查**开发人员**组并单击**保存**。

## <a name="add-api"> </a>将 API 添加到产品

在教程的此步骤中，我们将 Echo API 添加到新的免费试用版产品。

> 每个预先配置 Echo API 的 API 管理服务实例，都可用于试验和了解 API 管理。有关详细信息，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]。

单击左侧 **API 管理**菜单的**产品**从，然后单击**免费试用版**配置该产品。

![配置产品][配置产品]

单击**将 API 添加到产品**。

![将 API 添加到产品][1]

选中 **Echo API** 旁边的方框，然后单击**保存**。

![添加 Echo API][添加 Echo API]

## <a name="policies"> </a>配置调用速率限制和配额策略

在策略编辑器中配置速率限制和配额。单击左侧 **API 管理**菜单下的**策略**，然后从**策略范围产品**下拉列表选择**免费试用版**。

![产品策略][产品策略]

单击**添加策略**导入策略模板，并开始创建速率限制和配额策略。

![添加策略][添加策略]

要插入策略，请将光标放置到策略模板的**入站**或**出站**部分。速率限制和配额策略是入站的策略，因此将光标定位在入站元素中。

![策略编辑器][策略编辑器]

我们将在本教程中添加的两个策略是**限制调用速率**和**设置使用配额**策略。

![策略语句][策略语句]

一旦将光标定位在**入站**策略元素中，请单击**限制调用速率**旁边的箭头插入其策略模板。

    <rate-limit calls="number" renewal-period="seconds">
    <api name="name" calls="number">
    <operation name="name" calls="number" />
    </api>
    </rate-limit>

**限制调用速率**可用在产品级别，也可用在 API 和单个操作名称级别。本教程中仅使用产品级别策略，因此请从**速率限制**元素删除 **api** 和**操作**元素，如下例中所示。

    <rate-limit calls="number" renewal-period="seconds">
    </rate-limit>

在**免费试用版**产品中，最大允许调用速率是每分钟 10 次，因此请键入 **10** 作为调用属性的值，键入 **60** 作为**续订期**属性。

    <rate-limit calls="10" renewal-period="60">
    </rate-limit>

要配置**设置使用配额**策略，直接在**入站**元素中新添加的**速率限制**元素下面定位光标，然后单击**设置使用配额**左侧的箭头。

    <quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    <api name="name" calls="number" bandwidth="kilobytes">
    <operation name="name" calls="number" bandwidth="kilobytes" />
    </api>
    </quota>

由于此策略还设计用在产品级别，因此请删除 **api** 和**操作**名称元素，如下例中所示。

    <quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    </quota>

配额可基于每个间隔、带宽或这两者的调用次数。本教程中我们不限制基于带宽，因此请删除**带宽**属性。

    <quota calls="number" renewal-period="seconds">
    </quota>

在**免费试用版**产品中，该配额是每周调用 200 次。指定 **200** 作为调用属性的值，并指定 **604800** 作为续订期的值。

    <quota calls="200" bandwidth="kilobytes" renewal-period="604800">
    </quota>

> 以秒为单位指定策略间隔。要计算一周的时间间隔，可将天数 (7) 乘以一天的小时数 (24) 乘以每小时的分钟数 (60) 再乘以一分钟的秒数 (60)。 7 \* 24 \* 60 \* 60 = 604800.

完成配置策略后，它应与下面的示例匹配。

    <policies>
        <inbound>
            <rate-limit calls="10" renewal-period="60">
            </rate-limit>
            <quota calls="200" renewal-period="604800">
            </quota>
            <base />
        
    </inbound>
    <outbound>
        
        <base />
        
        </outbound>
    </policies>

配置所需的策略后，请单击**保存**。

![保存策略][保存策略]

## <a name="publish-product"> </a>发布产品

现在，已添加 API 且配置策略，该产品准备由开发人员使用。开发人员可使用该产品之前，必须先发布产品。单击左侧 **API 管理**菜单的**产品**从，然后单击**免费试用版**配置该产品。

![配置产品][配置产品]

单击**发布**，然后单击**是，发布**确认。

![发布产品][2]

## <a name="subscribe-account"> </a>对产品订阅开发人员帐户

现在产品已发布，它可用于开发人员订阅和使用。

> API 管理实例的管理员自动订阅到每个产品。在此教程步骤中，我们将订阅免费试用版产品的非管理员开发人员帐户之一。如果您的开发人员帐户是管理员角色的一部分，则可以遵循此步骤，即使您已经订阅。

单击左侧 **API 管理**菜单上的**开发人员**，然后单击您的开发人员帐户的名称。在此示例中，我们将使用 **Clayton Gragg** 开发人员帐户。

![配置开发人员][配置开发人员]

单击**添加订阅**。

![添加订阅][添加订阅]

选中**免费试用版**旁边的框并单击**订阅**。

![添加订阅][3]

## <a name="test-rate-limit"> </a>调用操作并测试速率限制

现在已配置和发布免费试用版产品，我们可以调用某些操作并测试速率限制策略。通过单击右上方菜单中的**开发人员门户**切换到开发人员门户。

![开发人员门户][开发人员门户]

单击顶部菜单中的 **API** 并选择 **Echo API**。

![开发人员门户][4]

> 如果必须只有一个 API 得到配置或对您的帐户可见，然后单击 API 使您直接进入该 API 的操作。

选择 **GET 资源**操作，单击**打开控制台**。

![打开控制台][打开控制台]

保持默认参数值，然后为**免费试用版**产品选择您的订阅密钥。

![订阅密钥][订阅密钥]

> 如有多个订阅，请务必为**免费试用版**选择密钥，否则前面的步骤中配置的策略不会生效。

单击 **HTTP Get** 并查看响应。请注意 **200 OK** 的**响应状态**。

![操作结果][操作结果]

在大于每分钟 10 次调用的速率限制策略单击 **HTTP Get**。一旦超出速率限制策略，则返回响应状态 **429 太多请求**。

![操作结果][5]

**响应标头**和**响应内容**指示重试次数成功前的剩余时间间隔。

每分钟 10 次调用的速率限制策略生效时，后续调用将无法在超出速率限制的首次 10 个成功的调用前经过 60 秒之后进行调用。此示例中的剩余间隔为 43 秒。

## <a name="next-steps"> </a>后续步骤

-   查看[开始使用高级 API 配置][开始使用高级 API 配置]教程中的其他主题。

  [创建产品]: #create-product
  [将 API 添加到产品]: #add-api
  [配置调用速率限制和配额策略]: #policies
  [发布产品]: #publish-product
  [对产品订阅开发人员帐户]: #subscribe-account
  [调用操作并测试速率限制]: #test-rate-limit
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [API 管理控制台]: ./media/api-management-howto-product-with-rules/api-management-management-console.png
  [添加产品]: ./media/api-management-howto-product-with-rules/api-management-add-product.png
  [添加新产品]: ./media/api-management-howto-product-with-rules/api-management-new-product-window.png
  [添加的产品]: ./media/api-management-howto-product-with-rules/api-management-product-added.png
  [如何创建和使用 Azure API 管理中的组]: ../api-management-howto-create-groups
  [添加开发人员组]: ./media/api-management-howto-product-with-rules/api-management-add-developers-group.png
  [配置产品]: ./media/api-management-howto-product-with-rules/api-management-configure-product.png
  [1]: ./media/api-management-howto-product-with-rules/api-management-add-api.png
  [添加 Echo API]: ./media/api-management-howto-product-with-rules/api-management-add-echo-api.png
  [产品策略]: ./media/api-management-howto-product-with-rules/api-management-product-policy.png
  [添加策略]: ./media/api-management-howto-product-with-rules/api-management-add-policy.png
  [策略编辑器]: ./media/api-management-howto-product-with-rules/api-management-policy-editor-inbound.png
  [策略语句]: ./media/api-management-howto-product-with-rules/api-management-limit-policies.png
  [保存策略]: ./media/api-management-howto-product-with-rules/api-management-policy-save.png
  [2]: ./media/api-management-howto-product-with-rules/api-management-publish-product.png
  [配置开发人员]: ./media/api-management-howto-product-with-rules/api-management-configure-developer.png
  [添加订阅]: ./media/api-management-howto-product-with-rules/api-management-add-subscription-menu.png
  [3]: ./media/api-management-howto-product-with-rules/api-management-add-subscription.png
  [开发人员门户]: ./media/api-management-howto-product-with-rules/api-management-developer-portal-menu.png
  [4]: ./media/api-management-howto-product-with-rules/api-management-developer-portal-api-menu.png
  [打开控制台]: ./media/api-management-howto-product-with-rules/api-management-open-console.png
  [订阅密钥]: ./media/api-management-howto-product-with-rules/api-management-select-key.png
  [操作结果]: ./media/api-management-howto-product-with-rules/api-management-http-get-results.png
  [5]: ./media/api-management-howto-product-with-rules/api-management-http-get-429.png
  [开始使用高级 API 配置]: ../api-management-get-started-advanced
