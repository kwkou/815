<properties
   pageTitle="在 Azure 中管理 Office 365 订阅的目录"
   description="使用 Azure Active Directory 和 Azure 经典管理门户来管理 Office 365 订阅帐户目录"
   services="active-directory"
   documentationCenter=""
   authors="curtand"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="05/16/2016"
   wacn.date="06/23/2016"/>

#在 Azure 中管理 Office 365 订阅的目录

本文介绍如何在 Azure 经典管理门户中管理为 Office 365 订阅创建的目录。完成此任务的步骤因你是否拥有 Azure 订阅而异。必须是 Azure 订阅的服务管理员或协同管理员才能登录到 Azure 经典管理门户。

如果你还没有 Azure 订阅，只需使用用于登录 Office 365 的工作或学校帐户注册即可。

![](./media/active-directory-manage-o365-subscription/AAD_O365_01.png)

将不会显示相应的 Azure 订阅，但你可以单击“注册 Azure”，此时 Office 365 帐户中的相关信息将预填充到注册表单中。默认情况下，将为同一帐户分配服务管理员角色。

![](./media/active-directory-manage-o365-subscription/AAD_O365_02.png)

完成 Azure 订阅后，你便可以登录到 Azure 经典管理门户并访问 Azure 服务。若要管理对 Office 365 用户进行身份验证所用的同一目录，请单击 Active Directory 扩展。

如果你已有 Azure 订阅，则管理其他目录的过程也很简单明了。下图可帮助你了解该过程。

在此示例中，Michael Smith 有一个用于 Contoso.com 的 Office 365 订阅。另外，他还有一个使用其 Microsoft 帐户 msmith@hotmail.com 注册的 Azure 订阅。在这种情况下，他管理着两个目录。

| 订阅 | Office 365 | Azure |
|  -------------- | ------------- | ------------------------------- |
| 显示名称 | Contoso | 默认目录 |
| 域名 | contoso.com | msmithhotmail.onmicrosoft.com |

他希望在使用 Microsoft 帐户登录到 Azure 时管理 Contoso 目录中的用户身份，以便启用 Azure AD 功能，如 Multifactor Authentication。

![](./media/active-directory-manage-o365-subscription/AAD_O365_03.png)

在这种情况下，这两个目录是相互独立的。

##管理两个独立的目录
若要在以 msmith@hotmail.com 身份登录到 Azure 时管理这两个目录，Michael Smith 需要完成以下步骤：

> [AZURE.NOTE]
仅当用户使用 Microsoft 帐户登录时才能完成这些步骤。如果用户是使用工作或学校帐户登录的，则“使用现有目录”选项不可用，因为工作或学校帐户只能通过其主目录（也就是存储着工作或学校帐户的目录，该目录归工作单位或学校拥有）进行身份验证。

1.	以 msmith@hotmail.com 身份登录到 Azure 经典管理门户。
2.	单击“新建”>“应用程序服务”>“Active Directory”>“目录”>“自定义创建”。
3.	单击“使用现有目录”，然后选中“已准备好立即注销”。
4.	以 Contoso.onmicrosoft.com 的全局管理员身份（例如 msmith@contoso.com)）登录到 Azure 经典管理门户。
5.	当系统提示“是否要将 Contoso 目录用于 Azure?”时，单击“继续”。
6.	单击“立即注销”。
7.	以 msmith@hotmail.com 身份登录到 Azure 经典管理门户。Contoso 目录和默认目录将显示在 Active Directory 扩展中。

完成这些步骤之后，msmith@hotmail.com 将成为 Contoso 目录的全局管理员。

##以全局管理员身份管理资源
现在，让我们假设 John Doe 需要登录到 Azure 经典管理门户并管理与 msmith@hotmail.com 的 Azure 订阅关联的网站和数据库资源。为此，Michael Smith 需要完成以下附加步骤：

1.	使用 Azure 订阅的服务管理员帐户（在本示例中为 msmith@hotmail.com)）登录到 Azure 经典管理门户。
2.	将订阅传输到 Contoso 目录：单击“设置”>“订阅”> 选择订阅 >“编辑目录”> 选择“Contoso (Contoso.com)”。在传输过程中，将删除是订阅的协同管理员的任何工作或学校帐户。
3.	添加 John Doe 作为订阅的协同管理员：单击“设置”>“管理员”> 选择订阅 >“添加”> 键入 **JohnDoe@Contoso.com**。

##后续步骤
有关订阅与目录之间关系的详细信息，请参阅[订阅如何与目录关联](/documentation/articles/active-directory-how-subscriptions-associated-directory/)。

<!---HONumber=Mooncake_0613_2016-->