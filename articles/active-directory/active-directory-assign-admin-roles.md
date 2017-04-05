<properties
    pageTitle="在 Azure Active Directory 中分配管理员角色 | Azure"
    description="介绍 Azure Active Directory 提供的管理员角色，以及如何分配这些角色。"
    services="active-directory"
    documentationcenter=""
    author="curtand"
    manager="femila"
    editor="" />
<tags
    ms.assetid="7fc27e8e-b55f-4194-9b8f-2e95705fb731"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/07/2017"
    wacn.date="04/05/2017"
    ms.author="curtand" />

# 在 Azure Active Directory 中分配管理员角色
使用 Azure Active Directory (Azure AD) 时，可以指定不同的管理员来执行不同的功能。这些管理员可以按角色访问 Azure 门户预览或 Azure 经典管理门户中的各种功能：创建或编辑用户、将管理角色分配给他人、重置用户密码、管理用户许可证以及管理域，等等。不论是通过 Office 365 门户、Azure 经典管理门户还是用于 Windows PowerShell 的 Azure AD 模块分配管理员角色，分配了该角色的用户在组织订阅的所有云服务中都拥有相同的权限。

提供以下管理员角色：

- **[全局管理员/公司管理员](#global-administrator)**：有权访问所有管理功能。注册 Azure 帐户的人员将成为全局管理员。只有全局管理员才能分配其他管理员角色。你的公司中可以有多个全局管理员。

  > [AZURE.NOTE]
  在 Microsoft 图形 API、Azure AD 图形 API 和 Azure AD PowerShell 中，此角色标识为“公司管理员”。它是 [Azure 门户预览](https://portal.azure.cn)中的“全局管理员”。
  >
  >

## 管理员权限

### 全局管理员 <a name="global-administrator"></a>
| 有权执行的操作 | 无权执行的操作 |
| --- | --- |
| <p>查看公司信息和用户信息</p><p>管理 Office 支持票证为 Office 产品</p><p>执行计费和采购操作</p><p>重置用户密码</p><p>创建和管理用户视图</p><p>创建、编辑和删除用户与组，以及管理用户许可证</p><p>管理域</p><p>管理公司信息</p><p>向其他人委派管理角色</p><p>使用目录同步</p><p>启用或禁用多重身份验证</p><p>查看报告</p> |不适用 |


## 有关全局管理员角色的详细信息
全局管理员有权访问所有管理功能。默认情况下，系统会将注册 Azure 订阅的人员指派为目录的全局管理员角色。只有全局管理员才能分配其他管理员角色。

## 分配或删除管理员角色
1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，单击“Active Directory”，然后单击所在组织的目录的名称。
2. 在“用户”页上，单击你想要编辑的用户的显示名称。
3. 在“组织角色”列表中，选择要分配给此用户的管理员角色，或者选择“用户”（如果要删除现有的管理员角色）。
4. 在“备用电子邮件地址”框中键入一个电子邮件地址。此电子邮件地址用于接收重要通知（包括有关密码自助重置的通知），因此，不管该用户是否能够访问 Azure，都必须能够访问其电子邮件帐户。
5. 选择“允许”或“阻止”以指定是否允许用户登录并访问服务。
6. 从“使用位置”下拉列表中指定位置。
7. 完成后，单击“保存”。

## 后续步骤
- 若要了解有关如何在 Azure 中控制资源访问的详细信息，请参阅 [Understanding resource access in Azure](/documentation/articles/active-directory-understanding-resource-access/)（了解 Azure 中的资源访问权限）
- 有关 Azure Active Directory 如何与 Azure 订阅相关联的详细信息，请参阅 [How Azure subscriptions are associated with Azure Active Directory](/documentation/articles/active-directory-how-subscriptions-associated-directory/)（Azure 订阅与 Azure Active Directory 的关联方式）
- [管理用户](/documentation/articles/active-directory-create-users/)
- [管理密码](/documentation/articles/active-directory-manage-passwords/)

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->