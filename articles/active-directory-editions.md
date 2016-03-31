<properties 
	pageTitle="Azure Active Directory 版本" 
	description="本主题介绍 Azure Active Directory 的免费版和付费版选项。" 
	services="active-directory" 
	documentationCenter="" 
	authors="MarkusVi"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="12/04/2015"
	wacn.date="01/29/2016" />

# Azure Active Directory 版本

所有 Microsoft Online 业务服务都依赖于 Azure Active Directory 进行登录和满足其他身份识别需求。如果你订阅了任何 Microsoft Online 业务服务（例如 Office 365、Azure 等），则表示已获得了 Azure Active Directory (Azure AD)，你可以用它来访问下述所有免费功能。

Azure Active Directory 是在云中为员工、合作伙伴和客户提供全面标识和访问管理功能的服务。它为开发人员整合了目录服务、高级标识管理和一个基于标准的多功能平台，并可让你针对自己的应用程序或数千个预先集成应用程序中的任何一个进行访问管理。使用 Azure Active Directory 免费版，你可以管理用户和组、与本地目录同步，以及在 Azure、Office 365 和数千种主流 SaaS 应用程序（如 Salesforce、Workday、Concur、DocuSign、Google Apps、Box、ServiceNow、Dropbox 等）上单一登录。若要了解有关 Azure Active Directory 的详细信息，请阅读[什么是 Azure AD](/documentation/articles/active-directory-whatis)。


若要增强你的 Azure Active Directory，可以使用 Azure Active Directory 基本和高级版添加付费功能。Azure Active Directory 付费版建立在现有免费目录基础之上，提供企业级功能，包括自助服务、增强型监视、安全报告、Multi-Factor Authentication (MFA) 和移动工作者安全访问。

Office 365 订阅包括以下比较表中所述的其他 Azure Active Directory 功能。


> [AZURE.NOTE]有关这些版本的定价选项，请参阅 [Azure Active Directory 定价](https://azure.microsoft.com/zh-CN/pricing/details/active-directory/)。<br>中国地区目前不支持 Azure Active Directory Premium 和 Azure Active Directory 基本版。有关详细信息，请通过 Azure Active Directory 论坛与我们联系。

> [AZURE.NOTE]
> 许多 Azure Active Directory 功能以“即付即用”版本的形式提供：
>
><!--  Active Directory B2C 是适用于面向消费者应用程序的标识和访问管理解决方案。有关详细信息，请参阅 [Azure Active Directory B2C](/documentation/services/active-directory-b2c/)-->
 
>-	可以通过基于用户或基于身份验证的提供程序使用 Azure Multi-Factor Authentication。有关详细信息，请参阅[什么是 Azure Multi-Factor Authentication？](/documentation/articles/multi-factor-authentication)


<br>

| 功能类型| 功能| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| --- | --- | --- | --- | --- | --- |
| **常用功能**| 目录对象 [1]| 最多 500,000 个对象| 无对象限制| 无对象限制| Office 365 用户帐户没有对象限制|
| | [用户和组管理（添加/更新/删除），基于用户的预配](/documentation/articles/active-directory-administer)，| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| | | 每个用户 10 个应用 [2]| 每个用户 10 个应用 [2]| 无限制| 每个用户 10 个应用 [2]|
| | [云用户的自助密码更改](/documentation/articles/active-directory-passwords-update-your-own-password)| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| | [Connect - 用于在本地目录与 Azure Active Directory 之间同步](/documentation/articles/active-directory-aadconnect)| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| | | ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| | [安全性/使用情况报告](/documentation/articles/active-directory-view-access-usage-reports)| 基本报告| 基本报告| 高级报告| 基本报告|
| **高级和基本版功能**| 
| | [云用户的自助密码重置](/documentation/articles/active-directory-passwords)| | ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| | [公司品牌（登录页和访问面板自定义）](/documentation/articles/active-directory-add-company-branding)| | ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| | [高可用性 SLA 运行时间 (99.9%)](https://azure.microsoft.com/support/legal/sla/)| | ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| **仅限高级版的功能**| 自助组管理/自助添加应用程序/动态组成员资格| | | ![勾选标记][12]| |
| | [通过本地写回实现自助密码重置、更改、解锁](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords)| | | ![勾选标记][12]| |
| | [Multi-Factor Authentication（云和本地）](/documentation/articles/multi-factor-authentication)| | | ![勾选标记][12]| 对于 Office 365 应用限制为云|
| | [Microsoft 标识管理器 (MIM) 用户许可证和 MIM 服务器 [3]](http://www.microsoft.com/server-cloud/products/microsoft-identity-manager/default.aspx)| | | ![勾选标记][12]| |
| | [Azure Active Directory Connect Health](/documentation/articles/active-directory-aadconnect-health)| | | ![勾选标记][12]| |
| | 组帐户的自动密码滚动更新| | | ![勾选标记][12]| |
| | **预览版**：条件性访问| | | ![勾选标记][12]| |
| | **预览版**：Privileged Identity Management| | | ![勾选标记][12]| |
| **Windows 10 和 Azure AD Join 相关的功能**| 将 Windows 10 设备加入 Azure AD、Desktop SSO、Microsoft Passport for Azure AD 和 Administrator Bitlocker 恢复| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|
| | MDM 自动注册、自助 Bitlocker 恢复、其他本地管理员通过 Azure AD Join 加入 Windows 10 设备| | | ![勾选标记][12]| |





[1] 默认使用配额为 150,000 个对象。对象是目录服务中的一个实体，由其唯一可分辨名称表示。对象的一个例子就是用于身份验证目的的用户条目。如果你需要超过此默认使用量配额，请与支持人员联系。500,000 个对象限制不适用于 Office 365、Microsoft Intune 或任何其他依赖 Azure Active Directory 提供目录服务的付费型 Microsoft 联机服务。

[2] 借助 Azure AD 免费版和 Azure AD 基本版，已获权访问 SaaS 应用的最终用户可以在其“访问面板”中看到最多 10 个应用并获得对这些应用的 SSO 访问权限。管理员可配置 SSO，并为使用免费和基本版的用户分配访问所需数量的 SaaS 应用的权限，但最终用户在其访问面板中一次只能看到 10 个应用。

[3] Microsoft 标识管理器服务器软件版权随 Windows Server 许可证（任意版本）一起授予。由于 Microsoft 标识管理器在 Windows Server 操作系统上运行，只要服务器正在运行有效的、经过许可的 Windows Server 副本，就能在该服务器上安装和使用 Microsoft 标识管理器。Microsoft 标识管理器服务器不需要其他单独的许可证。




## 后续步骤

- [Azure Active Directory 高级版入门](/documentation/articles/active-directory-get-started-premium)
- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding)
- [查看访问和使用情况报告](/documentation/articles/active-directory-view-access-usage-reports)


<!--Image references-->
[12]: ./media/active-directory-editions/ic195031.png

<!---HONumber=Mooncake_0118_2016-->