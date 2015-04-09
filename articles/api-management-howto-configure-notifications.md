<properties pageTitle="如何在 Azure API 管理中配置通知和电子邮件模板" metaKeywords="" description="了解如何在 Azure API 管理中配置通知和电子邮件模板。" metaCanonical="" services="" documentationCenter="API Management" title="如何在 Azure API 管理中配置通知和电子邮件模板" authors="sdanie" solutions="" manager="" editor="" />

# 如何在 Azure API 管理中配置通知和电子邮件模板

API 管理（预览版）提供的功能为特定事件配置通知，以及配置用于和 API 管理实例的管理员及开发人员通信的电子邮件模板。本主题演示如何为可用事件配置通知，并提供配置用于这些事件的电子邮件模板的概述。

## 本主题内容

-   [配置发布者通知][配置发布者通知]
-   [配置电子邮件模板][配置电子邮件模板]

## <a name="publisher-notifications"> </a>配置发布者通知

要配置通知，请单击您的 API 管理服务的 Azure 门户中的**管理控制台**。这使您转到 API 管理管理门户。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![API 管理控制台][API 管理控制台]

单击左侧 **API 管理**菜单的**通知**，以查看可用的通知。

![发布者通知][发布者通知]

为通知可以配置以下事件的列表。

-   **订阅请求（需要批准）** - 指定电子邮件收件人和用户将收到有关需要批准的 API 产品的订阅请求的电子邮件通知。
-   **新订阅** - 指定电子邮件收件人和用户将收到有关新的 API 产品订阅的电子邮件通知。
-   **应用程序库请求** - 指定电子邮件收件人和用户将在新的应用程序提交到应用程序库时收到电子邮件通知。
-   **密件抄送** - 指定电子邮件收件人和用户将收到发送给开发人员的所有电子邮件的电子邮件密件副本。
-   **新的问题或评论** - 在开发人员门户上提交新问题或注释时，指定电子邮件收件人和用户将收到电子邮件通知。
-   **关闭帐户消息** - 指定电子邮件收件人和用户将在关闭帐户时收到电子邮件通知。
-   **接近订阅配额限制** - 以下电子邮件收件人和用户将在订阅使用情况接近使用配额时收到电子邮件通知。

对于每个事件，您可以指定电子邮件收件人使用电子邮件地址文本框，或从列表中选择用户。

要指定被通知的电子邮件地址，请在电子邮件地址文本框中输入。如果有多个电子邮件地址，使用逗号分隔它们。

![通知收件人][通知收件人]

要指定被通知的用户，请单击**添加收件人**，选中要得到通知的用户旁边的框，然后单击**确定**。

> 请注意只有管理员才会显示在列表中。

配置通知收件人后, 单击**保存**应用更新的通知收件人。

> 如果导航离开**发布者通知**选项卡，API 管理门户将询问您那里是否有未保存的更改。

## <a name="email-templates"> </a>配置电子邮件模板

API 管理提供了在管理和使用服务的过程中发送的电子邮件的电子邮件模板。提供以下电子邮件模板。

-   批准的应用程序库提交
-   开发人员告别字母
-   开发人员配额限制接近通知
-   邀请用户
-   添加到问题的新注释
-   接收到的新问题
-   激活的新订阅
-   订阅续订确认
-   订阅请求拒绝
-   接收的订阅请求

可按需修改这些模板。

要查看和配置 API 管理实例的电子邮件模板，请单击左侧 **API 管理**菜单的**通知**，然后选择**电子邮件模板**选项卡。

![电子邮件模板][电子邮件模板]

要查看或修改特定的模板，可以从**模板**下拉列表选择它。

![电子邮件模板列表][电子邮件模板列表]

每个电子邮件模板都有纯文本格式的主题，和 HTML 格式的正文定义。可按需自定义每一项。

![电子邮件模板编辑器][电子邮件模板编辑器]

**参数**列表包含参数参数，插入到主题或正文时，将在发送电子邮件时替换为指定的值。要插入一个参数，将光标置于要存放参数的位置，然后单击参数名称左侧的箭头。

单击**预览**或**发送测试**，了解电子邮件的外观或发送测试电子邮件。

> 请注意在预览或发送测试时这些参数不会替换为实际值。
>
> 要将更改保存到电子邮件模板，请单击**保存**，或取消更改时单击**取消**。

  [配置发布者通知]: #publisher-notifications
  [配置电子邮件模板]: #email-templates
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [API 管理控制台]: ./media/api-management-howto-configure-notifications/api-management-management-console.png
  [发布者通知]: ./media/api-management-howto-configure-notifications/api-management-publisher-notifications.png
  [通知收件人]: ./media/api-management-howto-configure-notifications/api-management-email-addresses.png
  [电子邮件模板]: ./media/api-management-howto-configure-notifications/api-management-email-templates.png
  [电子邮件模板列表]: ./media/api-management-howto-configure-notifications/api-management-email-templates-list.png
  [电子邮件模板编辑器]: ./media/api-management-howto-configure-notifications/api-management-email-template.png
