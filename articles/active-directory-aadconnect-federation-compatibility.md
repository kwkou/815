<properties
	pageTitle="Azure AD 联合身份验证兼容性列表"
	description="本页列出了可用于实现单一登录的非 Microsoft 标识提供者。"
	services="active-directory"
	documentationCenter=""
	authors="billmath"
	manager="stevenpo"
	editor="curtand"/>

<tags
	ms.service="active-directory"
	ms.date="05/12/2016"
	wacn.date="06/23/2016"/>

# Azure AD 联合身份验证兼容性列表
Azure Active Directory 为 Office 365 和其他 Microsoft Online 服务提供单一登录与增强的应用程序访问安全性，以便在不使用任何非 Microsoft 解决方案的情况下实施混合部署和仅限云的部署。与大多数 Microsoft Online 服务一样，Office 365 可与 Azure Active Directory 集成，以利用目录服务、身份验证和授权。Azure Active Directory 还为数千种 SaaS 应用程序与本地 Web 应用程序提供单一登录。有关支持的 SaaS 应用程序，请参阅 Azure Active Directory 应用程序库。

对于投资了非 Microsoft 联合解决方案的组织，本主题包含有关通过以下“Azure Active Directory 联合兼容性列表”中所列的非 Microsoft 标识提供者，为使用 Microsoft 联机服务的 Windows Server Active Directory 用户配置单一登录的指导。

Microsoft 已使用非 Microsoft 标识提供者针对 Azure Active Directory 的一组常见用例测试了这些单一登录体验。

>[AZURE.IMPORTANT] Microsoft 仅测试了这些单一登录方案的联合功能。Microsoft 未对这些单一登录方案的同步、双重身份验证等组件执行测试。

>按备用 ID 和 UPN 使用登录也未在此计划中测试。



- [Azure Active Directory](#azure-active-directory)
- [Optimal IDM Virtual Identity Server Federation Services](#optimal-idm-virtual-identity-server-federation-services) 
- [PingFederate 6.11](#pingfederate-611) 
- [PingFederate 7.2](#pingfederate-72) 
- [Centrify](#centrify) 
- [IBM Tivoli Federated Identity Manager 6.2.2](#ibm-tivoli-federated-identity-manager-622) 
- [SecureAuth IdP 7.2.0](#secureauth-idp-720) 
- [CA SiteMinder 12.52](#ca-siteminder-1252) 
- [RadiantOne CFS 3.0](#radiantone-cfs-30) 
- [Okta](#okta) 
- [OneLogin](#onelogin) 
- [NetIQ Access Manager 4.0.1](#netiq-access-manager-401) 
- [BIG-IP with Access Policy Manager BIG-IP ver.11.3x – 11.6x](#big-ip-with-access-policy-manager-big-ip-ver-113x-116x) 
- [VMware Workspace Portal version 2.1](#vmware-workspace-portal-version-21) 
- [Sign&go 5.3](#signampgo-53) 
- [IceWall Federation Version 3.0](#icewall-federation-version-30) 
- [CA Secure Cloud](#ca-secure-cloud) 
- [Dell One Identity Cloud Access Manager v7.1](#dell-one-identity-cloud-access-manager-v71) 
- [AuthAnvil Single Sign On 4.5](#authavil-single-sign-on-45) 

>[AZURE.IMPORTANT] 由于这些是第三方产品，Microsoft 不会对与这些标识提供者相关的问题和疑问提供支持，例如部署、配置、故障排除、最佳实践等方面的问题和疑问。如果需要获得支持或者存在有关这些标识提供者的疑问，请直接联系提供支持的第三方。

>仅使用 WS 联合和 WS 信任协议测试这些第三方标识提供者与 Microsoft 云服务的互操作性。测试不包括使用 SAML 协议。

## Azure Active Directory 
Azure Active Directory 可以通过与本地 Active-Directory 联合，或者在没有本地联合服务器的情况下通过使用密码同步，来对用户进行身份验证。

下面是此登录体验的方案支持对照表：


| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|
|使用 ADAL 的现代应用程序，例如 Office 2016| 支持|无|

有关将 Azure Active Directory 与 AD FS 配合使用的详细信息，请参阅 [Active Directory Federation Services (ADFS)（Active Directory 联合身份验证服务 (ADFS)）](/documentation/articles/active-directory-aadconnect-get-started-custom/#configuring-federation-with-ad-fs)

有关将 Azure Active Directory 与密码同步配合使用的详细信息，请参阅 [Azure AD Connect](/documentation/articles/active-directory-aadconnect/)。


## Optimal IDM Virtual Identity Server Federation Services 
Optimal IDM Virtual Identity Server Federation Services 可以对位于客户本地 Active Directory 的用户进行身份验证。

下面是此单一登录体验的方案支持对照表：


| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |有关客户端访问策略的更多信息，请参阅 [Limiting Access to Office 365 Services Based on the Location of the Client（基于客户端位置限制其对 Office 365 服务的访问权限）](https://technet.microsoft.com/library/hh526961.aspx)。|



## PingFederate 6.11 

PingFederate 6.11 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：


| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无（以前的版本必须升级到 6.11）|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关如何配置此 STS 从而为你的 Active Directory 用户提供单一登录体验的 PingFederate 说明，请从[此处](http://go.microsoft.com/fwlink/?LinkID=266321)下载 pdf 文件。

## PingFederate 7.2 
PingFederate 7.2 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：


| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关如何配置此 STS 从而为你的 Active Directory 用户提供单一登录体验的 PingFederate 说明，请参阅[此文](http://documentation.pingidentity.com/display/PF72/PingFederate+7.2)。

## Centrify 
Centrify 帮助提供针对 Office 365 的联合单一登录体验，而无需托管本地联合服务器。

下面是此单一登录体验的方案支持对照表：


| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |不支持客户端访问控制 

有关 Centrify 的详细信息，请参阅[此处](http://www.centrify.com/cloud/apps/single-sign-on-for-office-365.asp)。|

## IBM Tivoli Federated Identity Manager 6.2.2 
适用于 Microsoft Applications 1.4 的装有 IBM Security Access Manager 的 IBM Tivoli Federated Identity Manager 6.2.2 实施了广泛使用的 WS 联合/WS 信任标识标准，以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 IBM Tivoli Federated Identity Manager 的详细信息，请参阅 [IBM Security Access Manager for Microsoft Applications](http://www-01.ibm.com/support/docview.wss?uid=swg24029517)。

## SecureAuth IdP 7.2.0 
SecureAuth IdP 7.2.0 实施广泛使用的 WS 联合/WS 信任身份标准，以提供单一登录体验和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 SecureAuth 的详细信息，请参阅 [SecureAuth IdP](http://go.microsoft.com/?linkid=9845293)。

## CA SiteMinder 12.52 
CA SiteMinder Federation 12.52 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |Windows 集成身份验证|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 CA SiteMinder 详细信息，请参阅 [CA SiteMinder Federation](http://www.ca.com/us/products/ca-single-sign-on.html)。

## RadiantOne CFS 3.0 
RadiantOne Cloud Federation Service (CFS) 3.0 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 RadiantOne CFS 的详细信息，请参阅 [RadiantOne CFS](http://www.radiantlogic.com/products/radiantone-cfs/)。


## Okta 
Okta 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：


| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |Windows 集成身份验证需要增设 Web 服务器和 Okta 应用程序。|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 Okta 的详细信息，请参阅 [Okta](https://www.okta.com/)。
 
## OneLogin 
2014 年 5 月测试的 OneLogin 实施广泛使用的 WS 联合/WS 信任标识标准，以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |Windows 集成身份验证|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 OneLogin 的详细信息，请参阅 [OneLogin](https://www.onelogin.com/)。

## NetIQ Access Manager 4.0.1 
NetIQ Access Manager 4.0.1 实施广泛使用的 WS 联合/WS 信任标识标准，以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |**支持 Kerberos 约定|
| 富客户端应用程序（如 Lync、Office Subscription、CRM）| 支持 |不支持 Windows 集成身份验证| 
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync）| 支持 |无|

**NetIQ 支持通过配置 Kerberos 约定实现 Kerberos 身份验证。如需此配置的帮助，请联系 NetIQ 或查看设置指南。有关 NetIQ Access Manager 的详细信息，请参阅 [NetIQ Access Manager](https://www.netiq.com/documentation/netiqaccessmanager4/identityserverhelp/data/b12iqp0m.html)。

## BIG-IP with Access Policy Manager BIG-IP ver.11.3x – 11.6x 
BIG-IP with Access Policy Manager (APM) BIG-IP ver.11.3x – 11.6x 实施广泛使用的 SAML 标识标准以提供单一登录体验和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 不支持 |不支持|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 BIG-IP Access Policy Manager 的详细信息，请参阅 [BIG-IP Access Policy Manager](https://f5.com/products/modules/access-policy-manager)。

有关如何将 STS 配置为向 Active Directory 用户提供单一登录体验的说明，请从[此处](http://www.f5.com/pdf/deployment-guides/microsoft-office-365-idp-dg.pdf)下载 pdf 文件，以获取 BIG-IP Access Policy Manager 的说明。

## VMware Workspace Portal version 2.1 
VMware Workspace Portal version 2.1 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |不支持 Windows 集成身份验证|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |不支持 Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 VMware Workspace Portal version 2.1 的详细信息，请从[此处](http://pubs.vmware.com/workspace-portal-21/topic/com.vmware.ICbase/PDF/workspace-portal-21-resource.pdf)下载 pdf 文件。

## Sign&go 5.3 
Sign&go 5.3 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |支持 Kerberos 约定 |
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|


Sign&go 5.3 支持通过配置 Kerberos 约定实现 Kerberos 身份验证。如需此配置的帮助，请联系 Ilex 或在[此处](http://www.ilex-international.com/docs/sign&go_wsfederation_en.pdf)查看设置指南。


## IceWall Federation Version 3.0 
IceWall Federation Version 3.0 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |不支持 Windows 集成身份验证|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |不支持 Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 IceWall Federation 的详细信息，请参阅[此处](http://h50146.www5.hp.com/products/software/security/icewall/eng/federation/)和[此处](http://h50146.www5.hp.com/products/software/security/icewall/federation/office365.html)。

## CA Secure Cloud 

CA Secure Cloud 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |不支持 Windows 集成身份验证|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |不支持 Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 CA Secure Cloud 详细信息，请参阅 [CA Secure Cloud](http://www.ca.com/us/products/security-as-a-service.aspx)。

## Dell One Identity Cloud Access Manager v7.1 
Dell One Identity Cloud Access Manager 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |无|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |无|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|

有关 Dell One Identity Cloud Access Manager 的详细信息，请参阅 [Dell One Identity Cloud Access Manager](http://software.dell.com/products/cloud-access-manager)。

 有关如何配置此 STS 从而为你的 Office 365 用户提供单一登录体验的说明，请参阅 [Configure Office 365 Users（配置 Office 365 用户）](http://documents.software.dell.com/dell-one-identity-cloud-access-manager/7.1/how-to-configure-microsoft-office-365)。

## AuthAnvil Single Sign On 4.5 
AuthAnvil Single Sign On 4.5 实施广泛使用的 WS 联合标识标准以提供单一登录和属性交换框架。

下面是此单一登录体验的方案支持对照表：

| 客户端 |支持 |异常|
| --------- | --------- |--------- |
| 基于 Web 的客户端（如 Exchange Web Access 和 SharePoint Online） | 支持 |不支持 Windows 集成身份验证|
| 富客户端应用程序（如 Lync、Office Subscription、CRM） | 支持 |不支持 Windows 集成身份验证|
| 多重格式电子邮件客户端（如 Outlook 和 ActiveSync） | 支持 |无|


有关详细信息，请参阅 [AuthAnvil Single Sign On](https://help.scorpionsoft.com/entries/26538603-How-can-I-Configure-Single-Sign-On-for-Office-365-)（AuthAnvil 单一登录）。

<!---HONumber=Mooncake_0606_2016-->