<properties
	pageTitle="添加自定义域名并设置 Azure Active Directory 的联合登录 | Azure"
	description="如何将公司域名添加到 Azure Active Directory，以及如何在 Azure Active Directory 与本地联合身份验证解决方案之间设置联合登录。"
	services="active-directory"
	documentationCenter=""
	authors="jeffsta"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/18/2016"
	wacn.date="08/01/2016"/>

# 将自定义域名添加到 Azure Active Directory

你可以配置一个自定义域名（例如“contoso.com”），以便 contoso.com 中的用户可以从你的企业网络进行联合单一登录。如果你已获得 Active Directory 联合身份验证服务 (AD FS) 或者在企业网络上运行不同的联合服务器，可以使用 Azure AD Connect 工具将 Azure AD 配置为使用自定义域名。你还可以使用 Azure AD Connect 部署新的 AD FS 环境并对其进行配置，以便在 Azure AD 上进行联合单一登录。

如果你不需要且并不打算部署 AD FS 或另一个联合服务器，请遵循以下说明：[将自定义域名添加到 Azure Active Directory](/documentation/articles/active-directory-add-domain/)。

## 将自定义域名添加到目录

1. 使用充当 Azure AD 目录全局管理员的用户帐户登录到 [Azure 经典门户](https://manage.windowsazure.cn/)。

2. 在“Active Directory”中打开你的目录，然后选择“域”选项卡。

3. 在命令栏上选择“添加”，然后输入自定义域的名称，例如“contoso.com”。请务必包含 .com、.net 或其他顶级扩展名。

4. 选中“我计划配置此域为使用本地 Active Directory 进行单点登录”复选框。

5. 选择“添加”。

运行 Azure AD Connect 工具以获取 Azure AD 用来验证域的 DNS 条目。可以在向导中的“Azure AD 域”步骤内看到该 DNS 条目。[这些说明](/documentation/articles/active-directory-aadconnect-get-started-custom/#verify-the-azure-ad-domain-selected-for-federation)中示范了向导中该步骤的大致情形。如果你没有 Azure AD Connect 工具，可以[在此处下载](http://go.microsoft.com/fwlink/?LinkId=615771)。

## 在域名注册机构中为域添加 DNS 条目

在 Azure AD 中使用自定义域名的下一个步骤就是更新域的 DNS 区域文件。这可让 Azure AD 确认组织拥有该自定义域名。

1. 登录到域名注册机构网站。如果你无权执行此操作，请让组织中具有此访问权限的个人或团队完成步骤 2 并在完成时通知你。

2. 通过添加 Azure AD 提供给你的 DNS 条目来更新域的 DNS 区域文件。此 DNS 条目可让 Azure AD 验证你是否拥有该域。DNS 条目不会更改任何行为，例如邮件路由或 Web 托管。

有关此步骤的帮助，请阅读 [Instructions for adding a DNS entry at popular DNS registrars](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)（有关在常见 DNS 注册机构中添加 DNS 条目的说明）

## 使用 Azure AD 验证域名

一旦添加了 DNS 条目，就可以使用 Azure AD 验证域名。

若要验证该域，请在 Azure AD Connect 向导的“Azure AD 域”步骤中选择“下一步”。Azure AD 将在 DNS 区域文件中查找域的 DNS 条目。只有在 DNS 记录传播完成之后，Azure AD 才能验证域名。传播通常只需要几秒钟，但有时可能需要一小时以上。如果第一次验证不成功，请稍后重试。

然后，继续执行 Azure AD Connect 向导的剩余步骤。这会将用户从 Windows Server AD 同步到 Azure AD。为联合身份验证配置的域中的已同步用户可以体验从企业网络到 Azure AD 的联合单一登录。

## 故障排除

如果无法验证自定义域名，请尝试以下方法。我们从最常见到最不常见的原因逐一分析。

1.	**等待一小时**。必须先传播 DNS 记录，Azure AD 才能验证域。这可能需要一小时以上。

2.	**确保已输入正确的 DNS 记录**。请在该域的域名注册机构网站上完成此步骤。如果 DNS 条目不在 DNS 区域文件中，或者与 Azure AD 提供给你的 DNS 条目不完全匹配，则 Azure AD 无法验证域名。如果你无权访问域名注册机构以更新域的 DNS 记录，请与组织内具有此访问权限的个人或团队共享 DNS 条目，并请他们添加 DNS 条目。

3.	**从 Azure AD 的另一个目录删除域名**。域名只能在单个目录中验证。如果域名先前在另一个目录中验证过，则必须先在那里将其删除后，才可在新的目录中验证。若要了解如何删除域名，请阅读 [Manage custom domain names（管理自定义域名）](/documentation/articles/active-directory-add-manage-domain-names/)。

## 添加更多自定义域名

如果你的组织使用多个自定义域名，例如“contoso.com”和“contosobank.com”，你最多可以添加 900 个域名。请使用本文中的相同步骤来添加每个域名。

## 后续步骤

-   [管理自定义域名](/documentation/articles/active-directory-add-manage-domain-names/)
-   [了解 Azure AD 中的域管理概念](/documentation/articles/active-directory-add-domain-concepts/)
-   [在用户登录时显示公司的品牌](/documentation/articles/active-directory-add-company-branding/)
-   [使用 PowerShell 管理 Azure AD 中的域名](https://msdn.microsoft.com/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains/)

<!---HONumber=Mooncake_0808_2016-->