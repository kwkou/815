<properties pageTitle="如何管理 Azure API 管理中的开发人员帐户" metaKeywords="" description="了解如何在 Azure API 管理中创建或邀请开发人员" metaCanonical="" services="" documentationCenter="API Management" title="如何管理 Azure API 管理中的开发人员帐户" authors="sdanie" solutions="" manager="" editor="" />

# 如何管理 Azure API 管理中的开发人员帐户

在 API 管理 （预览版）中，开发人员是公开使用 API 管理的 API 的用户。本指南将演示如何进行创建并邀请开发人员使用 API 和产品，您向 API 管理实例提供。

## 本主题内容

-   [创建新开发人员][创建新开发人员]
-   [邀请开发人员][邀请开发人员]
-   [停用或重新激活开发人员帐户][停用或重新激活开发人员帐户]
-   [后续步骤][后续步骤]

## <a name="create-developer"> </a>创建新开发人员

要创建新开发人员，请单击 API 管理服务的 Azure 门户中的**管理控制台**。这使您转到 API 管理管理门户。

> 如果尚未创建 API 管理系统服务实例，请参阅[开始使用 Azure API 管理][开始使用 Azure API 管理]教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。

![API 管理控制台][API 管理控制台]

单击左侧 **API 管理**菜单的**开发人员**，然后单击**添加用户**。

![创建开发人员][创建开发人员]

为新开发人员输入**电子邮件**、**密码**和**名称**，然后单击**保存**。

![创建开发人员][1]

默认情况下，新创建的开发人员帐户是**活动**，与**开发人员**组相关联。

![新的开发人员][新的开发人员]

**活动**状态中的开发人员帐户可用于访问他们具有订阅的所有 API。要将新创建的开发人员与其他组相关联，请参阅[如何将组与开发人员关联][如何将组与开发人员关联]。

## <a name="invite-developer"> </a>邀请开发人员

要邀请一名开发人员，请单击左侧 **API 管理**菜单的**开发人员**，然后单击**邀请用户**。

![邀请开发人员][2]

输入开发人员的姓名和电子邮件地址，然后单击**邀请**。

![邀请开发人员][3]

将显示一条确认消息，但新受邀请的开发人员之后不会出现在列表中，直到接受了邀请。

![邀请确认][邀请确认]

> 当邀请一名开发人员时，会将一封电子邮件发送给开发人员。此电子邮件使用一个模板生成并可自定义。有关详细信息，请参阅[配置电子邮件模板][配置电子邮件模板]。

一旦接受邀请，该帐户将变为活动状态。

## <a name="block-developer"> </a>停用或重新激活开发人员帐户

默认情况下，新创建的或受邀请的开发人员帐户是**活动**。要停用开发人员帐户，请单击**阻止**。要重新激活阻止的开发人员帐户，请单击**激活**。阻止的开发人员帐户不能访问开发人员门户或调用任何 API。

![阻止开发人员][新的开发人员]

## <a name="next-steps"> </a>后续步骤

创建开发人员帐户后，可以将其与角色相关联，并订阅产品和 API。有关详细信息，请参阅[如何创建和使用组][如何创建和使用组]。

  [创建新开发人员]: #create-developer
  [邀请开发人员]: #invite-developer
  [停用或重新激活开发人员帐户]: #block-developer
  [后续步骤]: #next-steps
  [开始使用 Azure API 管理]: ../api-management-get-started
  [创建 API 管理服务实例]: ../api-management-get-started/#create-service-instance
  [API 管理控制台]: ./media/api-management-howto-create-or-invite-developers/api-management-management-console.png
  [创建开发人员]: ./media/api-management-howto-create-or-invite-developers/api-management-create-developer.png
  [1]: ./media/api-management-howto-create-or-invite-developers/api-management-add-new-user.png
  [新的开发人员]: ./media/api-management-howto-create-or-invite-developers/api-management-new-developer.png
  [如何将组与开发人员关联]: ../api-management-howto-create-groups/#associate-group-developer
  [2]: ./media/api-management-howto-create-or-invite-developers/api-management-invite-developer.png
  [3]: ./media/api-management-howto-create-or-invite-developers/api-management-invite-developer-window.png
  [邀请确认]: ./media/api-management-howto-create-or-invite-developers/api-management-invite-developer-confirmation.png
  [配置电子邮件模板]: ../api-management-howto-configure-notifications/#email-templates
  [如何创建和使用组]: ../api-management-howto-create-groups
