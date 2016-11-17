<properties
	pageTitle="将自定义域名添加到 Azure Active Directory | Azure"
	description="如何将公司域名添加到 Azure Active Directory，以及如何验证域名。"
	services="active-directory"
	documentationCenter=""
	authors="jeffsta"
	manager="femila"
	editor=""/>  


<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/30/2016"
	wacn.date="10/31/2016"
	ms.author="curtand;jeffsta"/>  


# 将自定义域名添加到 Azure Active Directory

你的组织购买了一个或多个域名来开展业务，而你的用户将使用企业域名来登录你的企业网络。使用 Azure Active Directory (Azure AD) 时，也可以将企业域名添加到 Azure AD。这样，你便可以在目录中分配用户熟悉的用户名，例如“alice@contoso.com”。 过程很简单：

1. 将自定义域名添加到目录
2. 在域名注册机构中为域名添加 DNS 条目
3. 在 Azure AD 中验证自定义域名

> [AZURE.NOTE] 如果打算配置自定义域名以搭配 Active Directory 联合身份验证服务 (AD FS) 或公司网络上的其他安全令牌服务 (STS) 使用，请按照[添加和配置域以便与 Azure Active Directory 联盟](/documentation/articles/active-directory-add-domain-federated/)中的说明操作。如果打算将公司目录中的用户同步到 Azure AD，但[密码哈希同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/)不符合要求，这样做将非常有用。

## 将自定义域名添加到目录

1. 使用充当 Azure AD 目录全局管理员的用户帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn/)。

2. 在“Active Directory”中打开你的目录，然后选择“域”选项卡。

3. 在命令栏上，选择“添加”。输入自定义域的名称，例如“contoso.com”。请务必包含 .com、.net 或其他顶级扩展名，并清除“单一登录”（联盟）复选框。

4. 选择“添加”。

5. 在“添加域”向导的第二页中，获取 Azure AD 将用于验证组织是否拥有该自定义域名的 DNS 条目。

添加域名后，Azure AD 必须验证你的组织是否拥有该域名。必须先在 DNS 区域文件中添加该域名的 DNS 条目，然后 Azure AD 才能执行此验证。此任务是在域名注册机构网站上执行的。

## 在域名注册机构中为域添加 DNS 条目

在 Azure AD 中使用自定义域名的下一个步骤就是更新域的 DNS 区域文件。这可让 Azure AD 确认组织拥有该自定义域名。

1.  登录到域的域名注册机构。如果你无权访问域名注册机构以更新 DNS 条目，请让具有此访问权限的个人或团队完成步骤 2 并在完成时通知你。

2.  通过添加 Azure AD 提供给你的 DNS 条目来更新域的 DNS 区域文件。此 DNS 条目可让 Azure AD 验证你是否拥有该域。DNS 条目不会更改任何行为，例如邮件路由或 Web 托管。

有关添加 DNS 条目的帮助，请参阅[有关在常见 DNS 注册机构中添加 DNS 条目的说明](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)

## 使用 Azure AD 验证域名

一旦添加了 DNS 条目，就可以使用 Azure AD 验证域名。

如果“添加域”向导仍保持打开，请在向导的第三页上选择“验证”。当你选择“验证”时，Azure AD 会在 DNS 区域文件中查找域的 DNS 条目。只有在 DNS 记录传播完成之后，Azure AD 才能验证域名。此传播通常只需要几秒钟，但有时可能需要一小时以上。如果第一次验证不成功，请稍后重试。

如果“添加域”向导未保持打开，可以在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中验证域：

1.  使用充当 Azure AD 目录全局管理员的用户帐户登录。

2.  打开目录，然后选择“域”选项卡。

3.  选择要验证的域名，然后在命令栏上选择“验证”。

4. 在对话框中选择“验证”以完成验证。

现在，可以[分配包含自定义域名的用户名](/documentation/articles/active-directory-add-domain-add-users/)。

## 故障排除

如果无法验证自定义域名，请尝试以下方法。我们从最常见到最不常见的原因逐一分析。

1.	**等候一小时**。必须先传播 DNS 记录，Azure AD 才能验证域。这可能需要一小时以上。

2.	**确保已输入正确的 DNS 记录**。请在该域的域名注册机构网站上完成此步骤。如果 DNS 条目不在 DNS 区域文件中，或者与 Azure AD 提供给你的 DNS 条目不完全匹配，则 Azure AD 无法验证域名。如果你无权访问域名注册机构以更新域的 DNS 记录，请与组织内具有此访问权限的个人或团队共享 DNS 条目，并请他们添加 DNS 条目。

3.	**从 Azure AD 的另一个目录删除域名**。域名只能在单个目录中验证。如果域名先前在另一个目录中验证过，则必须先在那里将其删除后，才可在新的目录中验证。若要了解如何删除域名，请参阅[管理自定义域名](/documentation/articles/active-directory-add-manage-domain-names/)。


## 添加更多自定义域名

如果你的组织使用多个自定义域名，例如“contoso.com”和“contosobank.com”，你最多可以添加 900 个域名。请使用本文中的相同步骤来添加每个域名。

## 后续步骤

-   [分配包含自定义域名的用户名](/documentation/articles/active-directory-add-domain-add-users/)
-   [管理自定义域名](/documentation/articles/active-directory-add-manage-domain-names/)
-   [了解 Azure AD 中的域管理概念](/documentation/articles/active-directory-add-domain-concepts/)
-   [在用户登录时显示公司的品牌](/documentation/articles/active-directory-add-company-branding/)
-   [使用 PowerShell 管理 Azure AD 中的域名](https://msdn.microsoft.com/zh-cn/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains)

<!---HONumber=Mooncake_1024_2016-->
