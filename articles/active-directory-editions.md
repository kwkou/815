<properties
	pageTitle="Azure Active Directory 版本 | Azure"
	description="一个介绍 Azure Active Directory 的免费版和付费版选项的主题。Azure Active Directory Basic 是免费版，而 Azure Active Directory Premium 是付费版。"
	services="active-directory" 
	documentationCenter="" 
	authors="MarkusVi"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="05/12/2016"
	wacn.date="06/21/2016" />

# Azure Active Directory 版本

所有 Microsoft Online 业务服务都依赖于 Azure Active Directory 进行登录和满足其他身份识别需求。如果你订阅了任何 Microsoft Online 业务服务（例如 Office 365、Azure 等），则表示已获得了 Azure Active Directory (Azure AD)，你可以用它来访问下述所有免费功能。

Azure Active Directory 是在云中为员工、合作伙伴和客户提供全面标识和访问管理功能的服务。它为开发人员整合了目录服务、高级标识管理和一个基于标准的多功能平台，并可让你针对自己的应用程序或数千个预先集成应用程序中的任何一个进行访问管理。使用 Azure Active Directory 免费版，你可以管理用户和组、与本地目录同步，以及在 Azure、Office 365 和数千种主流 SaaS 应用程序（如 Salesforce、Workday、Concur、DocuSign、Google Apps、Box、ServiceNow、Dropbox 等）上单一登录。若要了解有关 Azure Active Directory 的详细信息，请阅读[什么是 Azure AD](/documentation/articles/active-directory-whatis/)。


若要增强你的 Azure Active Directory，可以使用 Azure Active Directory 基本和高级版添加付费功能。Azure Active Directory 付费版建立在现有免费目录基础之上，提供企业级功能，包括自助服务、增强型监视、安全报告、Multi-Factor Authentication (MFA) 和移动工作者安全访问。

Office 365 订阅包括以下比较表中所述的其他 Azure Active Directory 功能。


> [AZURE.NOTE]有关这些版本的定价选项，请参阅 [Azure Active Directory 定价](/pricing/details/active-directory/)。<br>中国地区目前不支持 Azure Active Directory Premium 和 Azure Active Directory 基本版。有关详细信息，请通过 Azure Active Directory 论坛与我们联系。

> [AZURE.NOTE]
> 许多 Azure Active Directory 功能以“即付即用”版本的形式提供：
> 可以通过基于用户或基于身份验证的提供程序使用 Azure Multi-Factor Authentication。有关详细信息，请参阅[什么是 Azure Multi-Factor Authentication？](/documentation/articles/multi-factor-authentication/)


##比较正式推出的功能

> [AZURE.NOTE] 有关此数据的不同视图，请参阅 [Azure Active Directory 功能](https://www.microsoft.com/en/server-cloud/products/azure-active-directory/features.aspx)。

| | Azure AD Free | Azure AD Basic | Azure AD Premium |
| ---                      | :-:           | :-:            | :-:              |
| 常用功能 | ![勾选标记][12] | ![勾选标记][12] | ![勾选标记][12] |
| Basic 功能 | | ![勾选标记][12] | ![勾选标记][12] |
| Premium 功能 | | | ![勾选标记][12] |



<br>

**常用功能**

- [目录对象](#directory-objects) 

- [用户/组管理（添加/更新/删除）/基于用户的预配、设备注册](#usergroup-management-addupdatedelete-user-based-provisioning-device-registration)

- [单一登录 (SSO)](#single-sign-on-sso)

- [云用户的自助密码更改](#self-service-password-change-for-cloud-users)

- [Connect（将本地目录扩展到 Azure Active Directory 的同步引擎）](#connect-sync-engine-that-extends-on-premises-directories-to-azure-active-directory)

- [安全性/使用情况报告](#securityusage-reports)



**Basic 功能**

- [基于组的访问管理/预配](#group-based-access-managementprovisioning)

- [云用户的自助密码重置](#self-service-password-reset-for-cloud-users)

- [公司品牌（登录页/访问面板自定义）](#company-branding-logon-pagesaccess-panel-customization)

- [应用程序代理](#application-proxy)

- [SLA 99.9%](#sla-999)


**Premium 功能**

- [自助组和应用管理/自助应用程序添加/动态组](#self-service-group-and-app-managementself-service-application-additions-dynamic-groups)

- [通过本地写回实现自助密码重置/更改/解锁](#self-service-password-resetchangeunlock-with-on-premises-write-back)

- [Multi-Factor Authentication（云和本地（MFA 服务器））](#multi-factor-authentication-cloud-and-on-premises-mfa-server)

- [MIM CAL + MIM 服务器](#mim-cal-mim-server)

- [Cloud App Discovery](#cloud-app-discovery)

- [Connect Health](#connect-health)

- [组帐户的自动密码滚动更新](#automatic-password-rollover-for-group-accounts)


**Azure Active Directory Join — 仅限 Windows 10 相关功能**

- [让设备加入 Azure AD、Desktop SSO、Microsoft Passport for Azure AD 和 Administrator Bitlocker 恢复](#join-a-device-to-azure-ad-desktop-sso-microsoft-passport-for-azure-ad-administrator-bitlocker-recovery)

- [MDM 自动注册、自助 Bitlocker 恢复、其他本地管理员通过 Azure AD Join 加入 Windows 10 设备](#mdm-auto-enrolment-self-service-bitlocker-recovery-additional-local-administrators-to-windows-10-devices-via-azure-ad-join)




<br>
<hr>
## 常用功能
#### 目录对象 

**类型：**常用功能

默认使用配额为 150,000 个对象。对象是目录服务中的一个实体，由其唯一可分辨名称表示。对象的一个例子就是用于身份验证目的的用户条目。如果你需要超过此默认使用量配额，请与支持人员联系。50 万个对象限制不适用于 Office 365、Microsoft Intune 或任何其他依赖 Azure Active Directory 提供目录服务的 Microsoft 付费联机服务。


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| 最多 500,000 个对象| 无对象限制| 无对象限制| Office 365 用户帐户没有对象限制|


<br>

#### 用户/组管理（添加/更新/删除）/基于用户的预配、设备注册

**类型：**常用功能

**可用性：**


| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|

**更多详细信息：**

- [管理 Azure AD 目录](/documentation/articles/active-directory-administer/)


<br>
<hr>
#### 单一登录 (SSO)

**类型：**常用功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| 每个用户 10 个应用 [1] <br>（预集成 SaaS 和开发人员集成应用）| 每个用户 10 个应用 [1] <br>（免费层 + 应用程序代理应用） | 无限制 [2] <br>（Free、Basic 层 + 自助应用集成模板）| 每个用户 10 个应用 [1] <br>（预集成 SaaS 和开发人员集成应用）|

[1] 借助 Azure AD Free 和 Azure AD Basic，已获权访问 SaaS 应用的最终用户可以在其访问面板中看到最多 10 个应用并获得对这些应用的 SSO 访问权限。管理员可配置 SSO，并为使用 Free 和 Basic 的用户分配访问所需数量的 SaaS 应用的权限，但最终用户在其访问面板中一次只能看到 10 个应用。

[2] 通过使用应用程序库菜单中提供的模板，自助集成支持 SAML、SCIM 或基于窗体的身份验证的任何应用程序。有关更多详细信息，请参阅[针对不在 Azure Active Directory 应用程序库中的应用程序配置单一登录](/documentation/articles/active-directory-saas-custom-apps/)。

**更多详细信息：**

- [使用 Azure Active Directory (AD) 管理应用程序](/documentation/articles/active-directory-enable-sso-scenario/)

<br>
<hr>
#### 云用户的自助密码更改

**类型：**常用功能

**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|

**更多详细信息：**

- [如何更新自己的密码](/documentation/articles/active-directory-passwords-update-your-own-password/)


<br>
<hr>
#### Connect（将本地目录扩展到 Azure Active Directory 的同步引擎） 

**类型：**常用功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|

**更多详细信息：**

- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

<br>
<hr>
#### 安全/使用情况报告

**类型：**常用功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| 3 个基本报告| 3 个基本报告| 高级报告| 3 个基本报告|

**更多详细信息：**

- [查看访问和使用情况报告](/documentation/articles/active-directory-view-access-usage-reports/)

<br>
<hr>

## Premium 和 Basic 功能
#### 基于组的访问管理/预配

**类型：**Basic 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | ![勾选标记][12]| ![勾选标记][12]| |

<br>
<hr>
#### 云用户的自助密码重置

**类型：**Basic 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|

**更多详细信息：**

- [用户和管理员的 Azure AD 密码重置](/documentation/articles/active-directory-passwords/)

<br>
<hr>
#### 公司品牌（登录页/访问面板自定义）

**类型：**Basic 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|

**更多详细信息：**

- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding/)

<br>
<hr>
#### 应用程序代理

**类型：**Basic 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | ![勾选标记][12]| ![勾选标记][12]| |
<br>
<hr>
#### SLA 99.9%

**类型：**Basic 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | ![勾选标记][12]| ![勾选标记][12]| ![勾选标记][12]|

**更多详细信息：**

- [服务级别协议](/support/legal/sla/)

<br>
<hr>

## Premium 功能
#### 自助组和应用管理/自助应用程序添加/动态组

**类型：**Premium 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | | ![勾选标记][12]| |



<br>
         
#### 通过本地写回实现自助密码重置/更改/解锁

**类型：**Premium 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | | ![勾选标记][12]| |



<br>

#### Multi-Factor Authentication（云和本地（MFA 服务器））

**类型：**Premium 功能


**可用性：**

| 免费版| 基本版| Premium Edition| 仅限 Office 365 应用 |
| :-: | :-: | :-: | :-: |
| | | ![勾选标记][12]| 对于 Office 365 应用限制为云|

**更多详细信息：**

- [什么是 Azure Multi-Factor Authentication？](/documentation/articles/multi-factor-authentication/multi-factor-authentication)

<br>
<hr>
#### MIM CAL + MIM 服务器 

Microsoft 标识管理器服务器软件权限随 Windows Server 许可证（任意版本）一起授予。由于 Microsoft 标识管理器在 Windows Server 操作系统上运行，只要服务器正在运行有效的、经过许可的 Windows Server 副本，就能在该服务器上安装和使用 Microsoft 标识管理器。Microsoft 标识管理器服务器不需要其他单独的许可证。




## 后续步骤

- [Azure Active Directory 高级版入门](/documentation/articles/active-directory-get-started-premium/)
- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding/)


<!--Image references-->
[12]: ./media/active-directory-editions/ic195031.png

<!---HONumber=Mooncake_0613_2016-->