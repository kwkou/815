<properties
	pageTitle="将自定义域名添加到 Azure Active Directory | Azure"
	description="如何将公司域名添加到 Azure Active Directory，以及如何验证域名。"
	services="active-directory"
	documentationCenter=""
	authors="jeffsta"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/20/2016"
	wacn.date="07/04/2016"/>

# 将自定义域名添加到 Azure Active Directory

你的组织购买了一个或多个域名来开展业务，而你的用户将使用企业域名来登录你的企业网络。使用 Azure Active Directory (Azure AD) 时，也可以将企业域名添加到 Azure AD。这样，你便可以在目录中分配用户熟悉的用户名，例如“alice@contoso.com”。 过程很简单：

- 在 Azure 经典管理门户的“添加域”向导中添加域名

- 在 Azure AD 经典管理门户或 Azure AD Connect 工具中获取 DNS 条目

- 在 DNS 注册机构的网站上，将域名的 DNS 条目添加到 DNS 区域文件中

- 在 Azure AD 经典管理门户或 Azure AD Connect 工具中验证域名


在验证自定义域名之前，用户必须使用类似于“alice@contoso.onmicrosoft.com”的用户名登录，此类名称使用目录的初始域名。如果需要多个自定义域名，例如“contoso.com”和“contosobank.com”，最多可以添加 900 个域名。请使用本文中的相同步骤来添加每个域名。

## 将自定义域名添加到目录

1. 使用充当 Azure AD 目录全局管理员的用户帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn/)。

2. 在“Active Directory”中打开你的目录，然后选择“域”选项卡。

3. 在命令栏上选择“添加”，然后输入自定义域的名称，例如“contoso.com”。请务必包含 .com、.net 或其他顶级扩展名。

4. 如果打算对使用本地 Active Directory 的[联合登录名](https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Configuring-AD-FS-for-user-sign-in-with-Azure-AD-Connect)配置此域，请选中该复选框。

5. 选择“添加”。

添加域名后，Azure AD 必须验证你的组织是否拥有该域名。必须先在 DNS 区域文件中添加该域名的 DNS 条目，然后 Azure AD 才能执行此验证。此任务是在域名注册机构网站上执行的。

## 获取域名的 DNS 条目

如果未与本地 Windows Server Active Directory 联合，DNS 条目位于“添加域”向导的第二页。

如果你要配置用于联合的域，将定向到下载 Azure AD Connect 工具的页。请运行 Azure AD Connect 工具来[获取需要在域名注册机构添加的 DNS 条目](/documentation/articles/active-directory-aadconnect-get-started-custom/#verify-the-azure-ad-domain-selected-for-federation)。Azure AD Connect 工具也会验证 Azure AD 的域名。

## 将 DNS 条目添加到 DNS 区域文件

1.  登录到域的域名注册机构。如果你的权限不足以更新 DNS 条目，请求助具有此访问权限的人员或团队添加 DNS 条目。

2.  通过添加 Azure AD 提供给你的 DNS 条目来更新域的 DNS 区域文件。此 DNS 条目可让 Azure AD 验证你是否拥有该域。DNS 条目不会更改任何行为，例如邮件路由或 Web 托管。传播 DNS 记录最多可能需要一个小时。

[有关在常见 DNS 注册机构中添加 DNS 条目的说明](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)

## 使用 Azure AD 验证域名

添加 DNS 条目后，必须确保 Azure AD 验证域名。这是成功之前的最后一个步骤。

如果“添加域”向导仍保持打开，请在向导的第三页上选择“验证”。在验证之前，请等待一段时间（最长一个小时）来传播 DNS 条目。

如果“添加域”向导未保持打开，可以在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中验证域。

1.  使用充当 Azure AD 目录全局管理员的用户帐户登录。

2.  打开你的目录，然后选择“域”选项卡。

3.  选择要验证的域。

4.  在命令栏上选择“验证”，然后在对话框中选择“验证”。

祝贺你操作成功！ 现在，你可以[分配包含自定义域名的用户名](/documentation/articles/active-directory-add-domain-add-users/)。如果在验证域名时遇到任何问题，请参阅[故障排除](#troubleshooting)部分。

## 故障排除
无法验证自定义域名的原因有多种。我们从最常见到最不常见的原因逐一分析。

- 在传播 DNS 条目之前尝试验证域名。请稍候片刻，然后重试。

- 根本没有输入 DNS 记录。请验证 DNS 条目并等待它传播完成，然后重试。

- 域名已在其他目录中验证。请找到该域名并将它从其他目录中删除，然后重试。

- DNS 记录包含错误。请更正错误，然后重试。

- 你的权限不足以更新 DNS 记录。与组织中具有此访问权限的人员或团队共享 DNS 条目，并请求他们添加 DNS 条目。


## 后续步骤

-   [分配包含自定义域名的用户名](/documentation/articles/active-directory-add-domain-add-users/)
-   [管理自定义域名](/documentation/articles/active-directory-add-manage-domain-names/)
-   [了解 Azure AD 中的域管理概念](/documentation/articles/active-directory-add-domain-concepts/)
-   [在用户登录时显示公司的品牌](/documentation/articles/active-directory-add-company-branding/)
-   [使用 PowerShell 管理 Azure AD 中的域名](https://msdn.microsoft.com/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains)

<!---HONumber=Mooncake_0606_2016-->