<properties
	pageTitle="Azure AD Connect 同步：同步到 Azure Active Directory 的属性 | Azure"
	description="列出同步到 Azure Active Directory 的属性。"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="03/16/2016"
	wacn.date="04/14/2016"/>


# Azure AD Connect 同步：与 Azure Active Directory 同步的属性

本主题列出了通过 Azure AD Connect 同步进行同步的属性。  
属性按照相关的 Azure AD 应用进行分组。

## 要同步的属性
常见的问题是最少要同步的属性列表是什么。默认的（也是建议的）方法是保留默认属性，以便可以在云中构造完整的 GAL（全局地址列表），并获取 Office 365 工作负荷中的所有功能。在某些情况下，组织并不想要将某些属性同步到云中，因为它们包含敏感数据或 PII（个人身份信息），如以下示例中所示。

![错误的属性](./media/active-directory-aadconnectsync-attributes-synchronized/badextensionattribute.png)

在此情况下，请从以下属性列表着手，并标识包含敏感数据或 PII 数据、因而不能同步的属性。然后在安装期间使用 [Azure AD 应用程序和属性筛选](/documentation/articles/active-directory-aadconnect-get-started-custom/#azure-ad-app-and-attribute-filtering)将它们取消选择。

>[AZURE.WARNING] 取消选择属性时，应该小心，只取消选择那些绝对不能同步的属性。取消选择其他属性可能会对功能造成负面影响。

## Office 365 ProPlus

| 属性名称| 用户| 注释 |
| --- | :-: | --- |
| accountEnabled| X| 如果启用了帐户，则进行定义。|
| cn| X|  |
| displayName| X|  |
| objectSID| X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| pwdLastSet| X| 机械属性。用于了解使已颁发令牌失效的时间。由密码同步和联合使用。|
| sourceAnchor| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| usageLocation| X| 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X| UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|



## <a name="exchange-hybrid-writeback"></a>Exchange Online

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X| |  | 如果启用了帐户，则进行定义。|
| assistant| X| X|  |  |
| authOrig| X| X| X|  |
| c| X| X|  |  |
| cn| X|  | X|  |
| co| X| X|  |  |
| company| X| X|  |  |
| countryCode| X| X|  |  |
| department| X| X|  |  |
| description| X| X| X|  |
| displayName| X| X| X|  |
| dLMemRejectPerms| X| X| X|  |
| dLMemSubmitPerms| X| X| X|  |
| extensionAttribute1| X| X| X|  |
| extensionAttribute10| X| X| X|  |
| extensionAttribute11| X| X| X|  |
| extensionAttribute12| X| X| X|  |
| extensionAttribute13| X| X| X|  |
| extensionAttribute14| X| X| X|  |
| extensionAttribute15| X| X| X|  |
| extensionAttribute2| X| X| X|  |
| extensionAttribute3| X| X| X|  |
| extensionAttribute4| X| X| X|  |
| extensionAttribute5| X| X| X|  |
| extensionAttribute6| X| X| X|  |
| extensionAttribute7| X| X| X|  |
| extensionAttribute8| X| X| X|  |
| extensionAttribute9| X| X| X|  |
| facsimiletelephonenumber| X| X|  |  |
| givenName| X| X|  |  |
| homePhone| X| X|  |  |
| info| X| X| X| 组当前不使用此属性。|
| Initials| X| X|  |  |
| l| X| X|  |  |
| legacyExchangeDN| X| X| X|  |
| mailNickname| X| X| X|  |
| mangedBy|  |  | X|  |
| manager| X| X|  |  |
| member|  |  | X|  |
| mobile| X| X|  |  |
| msDS-HABSeniorityIndex| X| X| X|  |
| msDS-PhoneticDisplayName| X| X| X|  |
| msExchArchiveGUID| X|  |  |  |
| msExchArchiveName| X|  |  |  |
| msExchAssistantName| X| X|  |  |
| msExchAuditAdmin| X|  |  |  |
| msExchAuditDelegate| X|  |  |  |
| msExchAuditDelegateAdmin| X|  |  |  |
| msExchAuditOwner| X|  |  |  |
| msExchBlockedSendersHash| X| X|  |  |
| msExchBypassAudit| X|  |  |  |
| msExchCoManagedByLink|  |  | X|  |
| msExchDelegateListLink| X|  |  |  |
| msExchELCExpirySuspensionEnd| X|  |  |  |
| msExchELCExpirySuspensionStart| X|  |  |  |
| msExchELCMailboxFlags| X|  |  |  |
| msExchEnableModeration| X|  | X|  |
| msExchExtensionCustomAttribute1| X| X| X| Exchange Online 当前不使用此属性。 |
| msExchExtensionCustomAttribute2| X| X| X| Exchange Online 当前不使用此属性。 |
| msExchExtensionCustomAttribute3| X| X| X| Exchange Online 当前不使用此属性。 |
| msExchExtensionCustomAttribute4| X| X| X| Exchange Online 当前不使用此属性。 |
| msExchExtensionCustomAttribute5| X| X| X| Exchange Online 当前不使用此属性。 |
| msExchHideFromAddressLists| X| X| X|  |
| msExchImmutableID| X|  |  |  |
| msExchLitigationHoldDate| X| X| X|  |
| msExchLitigationHoldOwner| X| X| X|  |
| msExchMailboxAuditEnable| X|  |  |  |
| msExchMailboxAuditLogAgeLimit| X|  |  |  |
| msExchMailboxGuid| X|  |  |  |
| msExchModeratedByLink| X| X| X|  |
| msExchModerationFlags| X| X| X|  |
| msExchRecipientDisplayType| X| X| X|  |
| msExchRecipientTypeDetails| X| X| X|  |
| msExchRemoteRecipientType| X|  |  |  |
| msExchRequireAuthToSendTo| X| X| X|  |
| msExchResourceCapacity| X|  |  |  |
| msExchResourceDisplay| X|  |  |  |
| msExchResourceMetaData| X|  |  |  |
| msExchResourceSearchProperties| X|  |  |  |
| msExchRetentionComment| X| X| X|  |
| msExchRetentionURL| X| X| X|  |
| msExchSafeRecipientsHash| X| X|  |  |
| msExchSafeSendersHash| X| X|  |  |
| msExchSenderHintTranslations| X| X| X|  |
| msExchTeamMailboxExpiration| X|  |  |  |
| msExchTeamMailboxOwners| X|  |  |  |
| msExchTeamMailboxSharePointUrl| X|  |  |  |
| msExchUserHoldPolicies| X|  |  |  |
| msOrg-IsOrganizational|  |  | X|  |
| objectSID| X| | X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| oOFReplyToOriginator|  |  | X|  |
| otherFacsimileTelephone| X| X|  |  |
| otherHomePhone| X| X|  |  |
| otherTelephone| X| X|  |  |
| pager| X| X|  |  |
| physicalDeliveryOfficeName| X| X|  |  |
| postalCode| X| X|  |  |
| proxyAddresses| X| X| X|  |
| publicDelegates| X| X| X|  |
| pwdLastSet| X|  |  | 机械属性。用于了解使已颁发令牌失效的时间。由密码同步和联合使用。|
| reportToOriginator|  |  | X|  |
| reportToOwner|  |  | X|  |
| securityEnabled|  |  | X| 派生自 groupType|
| sn| X| X|  |  |
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| 号| X| X|  |  |
| streetAddress| X| X|  |  |
| targetAddress| X| X|  |  |
| telephoneAssistant| X| X|  |  |
| telephoneNumber| X| X|  |  |
| thumbnailphoto| X| X|  |  |
| title| X| X|  |  |
| unauthOrig| X| X| X|  |
| usageLocation| X|  |  | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userCertificate| X| X|  |  |
| userPrincipalName| X| | | UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|
| userSMIMECertificates| X| X|  |  |
| wWWHomePage| X| X|  |  |


## SharePoint Online

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X|  |  | 如果启用了帐户，则进行定义。|
| authOrig| X| X| X|  |
| c| X| X|  |  |
| cn| X|  | X|  |
| co| X| X|  |  |
| company| X| X|  |  |
| countryCode| X| X|  |  |
| department| X| X|  |  |
| description| X| X| X|  |
| displayName| X| X| X|  |
| dLMemRejectPerms| X| X| X|  |
| dLMemSubmitPerms| X| X| X|  |
| extensionAttribute1| X| X| X|  |
| extensionAttribute10| X| X| X|  |
| extensionAttribute11| X| X| X|  |
| extensionAttribute12| X| X| X|  |
| extensionAttribute13| X| X| X|  |
| extensionAttribute14| X| X| X|  |
| extensionAttribute15| X| X| X|  |
| extensionAttribute2| X| X| X|  |
| extensionAttribute3| X| X| X|  |
| extensionAttribute4| X| X| X|  |
| extensionAttribute5| X| X| X|  |
| extensionAttribute6| X| X| X|  |
| extensionAttribute7| X| X| X|  |
| extensionAttribute8| X| X| X|  |
| extensionAttribute9| X| X| X|  |
| facsimiletelephonenumber| X| X|  |  |
| givenName| X| X|  |  |
| hideDLMembership|  |  | X|  |
| homephone| X| X|  |  |
| info| X| X| X|  |
| initials| X| X|  |  |
| ipPhone| X| X|  |  |
| l| X| X|  |  |
| mail| X| X| X|  |
| mailnickname| X| X| X|  |
| managedBy|  |  | X|  |
| manager| X| X|  |  |
| member|  |  | X|  |
| middleName| X| X|  |  |
| mobile| X| X|  |  |
| msExchTeamMailboxExpiration| X|  |  |  |
| msExchTeamMailboxOwners| X|  |  |  |
| msExchTeamMailboxSharePointLinkedBy| X|  |  |  |
| msExchTeamMailboxSharePointUrl| X|  |  |  |
| objectSID| X|  | X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| oOFReplyToOriginator|  |  | X|  |
| otherFacsimileTelephone| X| X|  |  |
| otherHomePhone| X| X|  |  |
| otherIpPhone| X| X|  |  |
| otherMobile| X| X|  |  |
| otherPager| X| X|  |  |
| otherTelephone| X| X|  |  |
| pager| X| X|  |  |
| physicalDeliveryOfficeName| X| X|  |  |
| postalCode| X| X|  |  |
| postOfficeBox| X| X|  |  |
| preferredLanguage| X|  |  |  |
| proxyAddresses| X| X| X|  |
| pwdLastSet| X|  |  | 机械属性。用于了解使已颁发令牌失效的时间。由密码同步和联合使用。|
| reportToOriginator|  |  | X|  |
| reportToOwner|  |  | X|  |
| securityEnabled|  |  | X| 派生自 groupType|
| sn| X| X|  |  |
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| 号| X| X|  |  |
| streetAddress| X| X|  |  |
| targetAddress| X| X|  |  |
| telephoneAssistant| X| X|  |  |
| telephoneNumber| X| X|  |  |
| thumbnailphoto| X| X|  |  |
| title| X| X|  |  |
| unauthOrig| X| X| X|  |
| url| X| X|  |  |
| usageLocation| X|  |  | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X|  |  | UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|
| wWWHomePage| X| X|  |  |

## Lync Online

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X|  |  | 如果启用了帐户，则进行定义。|
| c| X| X|  |  |
| cn| X|  | X|  |
| co| X| X|  |  |
| company| X| X|  |  |
| department| X| X|  |  |
| description| X| X| X|  |
| displayName| X| X| X|  |
| facsimiletelephonenumber| X| X| X|  |
| givenName| X| X|  |  |
| homephone| X| X|  |  |
| ipPhone| X| X|  |  |
| l| X| X|  |  |
| mail| X| X| X|  |
| mailNickname| X| X| X|  |
| managedBy|  |  | X|  |
| manager| X| X|  |  |
| member|  |  | X|  |
| mobile| X| X|  |  |
| msExchHideFromAddressLists| X| X| X|  |
| msRTCSIP-ApplicationOptions| X|  |  |  |
| msRTCSIP-DeploymentLocator| X| X|  |  |
| msRTCSIP-Line| X| X|  |  |
| msRTCSIP-OptionFlags| X| X|  |  |
| msRTCSIP-OwnerUrn| X|  |  |  |
| msRTCSIP-PrimaryUserAddress| X| X|  |  |
| msRTCSIP-UserEnabled| X| X|  |  |
| objectSID| X|  | X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| otherTelephone| X| X|  |  |
| physicalDeliveryOfficeName| X| X|  |  |
| postalCode| X| X|  |  |
| preferredLanguage| X|  |  |  |
| proxyAddresses| X| X| X|  |
| pwdLastSet| X|  |  | 机械属性。用于了解使已颁发令牌失效的时间。由密码同步和联合使用。|
| securityEnabled|  |  | X| 派生自 groupType|
| sn| X| X|  |  |
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| 号| X| X|  |  |
| streetAddress| X| X|  |  |
| telephoneNumber| X| X|  |  |
| thumbnailphoto| X| X|  |  |
| title| X| X|  |  |
| usageLocation| X|  |  | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X|  |  | UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|
| wWWHomePage| X| X|  |  |


## Azure RMS

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X|  |  | 如果启用了帐户，则进行定义。|
| cn| X|  | X| 公用名或别名。大多数情况下是 [mail] 值的前缀。|
| displayName| X| X| X| 表示通常显示为友好名称（名字姓氏）的名称的字符串。|
| mail| X| X| X| 完整的电子邮件地址。|
| member|  |  | X|  |
| objectSID| X|  | X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| proxyAddresses| X| X| X| 机械属性。通过 Azure AD 使用。包含用户的所有辅助电子邮件地址。|
| pwdLastSet| X|  |  | 机械属性。用于了解使已颁发令牌失效的时间。|
| securityEnabled|  |  | X| 派生自 groupType。|
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| usageLocation| X|  |  | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X|  |  | 此 UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|


## Intune

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X|  |  | 如果启用了帐户，则进行定义。|
| c| X| X|  |  |
| cn| X|  | X|  |
| description| X| X| X|  |
| displayName| X| X| X|  |
| mail| X| X| X|  |
| mailnickname| X| X| X|  |
| member|  |  | X|  |
| objectSID| X|  | X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| proxyAddresses| X| X| X|  |
| pwdLastSet| X| | | 机械属性。用于了解使已颁发令牌失效的时间。由密码同步和联合使用。|
| securityEnabled|  |  | X| 派生自 groupType|
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| usageLocation| X|  |  | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X|  |  | UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|



## Dynamics CRM

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X|  |  | 如果启用了帐户，则进行定义。|
| c| X| X|  |  |
| cn| X|  | X|  |
| co| X| X|  |  |
| company| X| X|  |  |
| countryCode| X| X|  |  |
| description| X| X| X|  |
| displayName| X| X| X|  |
| facsimiletelephonenumber| X| X|  |  |
| givenName| X| X|  |  |
| l| X| X|  |  |
| managedBy|  |  | X|  |
| manager| X| X|  |  |
| member|  |  | X|  |
| mobile| X| X|  |  |
| objectSID| X|  | X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| physicalDeliveryOfficeName| X| X|  |  |
| postalCode| X| X|  |  |
| preferredLanguage| X|  |  |  |
| pwdLastSet| X|  |  | 机械属性。用于了解使已颁发令牌失效的时间。由密码同步和联合使用。|
| securityEnabled|  |  | X| 派生自 groupType|
| sn| X| X|  |  |
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| 号| X| X|  |  |
| streetAddress| X| X|  |  |
| telephoneNumber| X| X|  |  |
| title| X| X|  |  |
| usageLocation| X|  |  | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X|  |  | UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|

## 第三方应用程序
这一组属性用作常规工作负荷或应用程序所需的最低属性。它可以用于上面未列出的工作负荷或非 Microsoft 应用程序。它显式用于以下目的：

- Yammer（实际上只使用 User）
- [SharePoint 等资源提供的混合企业到企业 (B2B) 跨组织协作方案](http://go.microsoft.com/fwlink/?LinkId=747036)

如果不使用 Azure AD 目录来支持 Office 365、Dynamics 或 Intune，则可以使用这一组属性。它包含一小部分核心属性。

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X|  |  | 如果启用了帐户，则进行定义。|
| cn| X|  | X|  |
| displayName| X| X| X|  |
| givenName| X| X|  |  |
| mail| X|  | X|  |
| managedBy|  |  | X|  |
| mailNickName| X| X| X|  |
| member|  |  | X|  |
| objectSID| X|  |  | 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| proxyAddresses| X| X| x|  |
| pwdLastSet| X|  |  | 机械属性。用于了解使已颁发令牌失效的时间。由密码同步和联合使用。|
| sn| X| X|  |  |
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| usageLocation| X|  |  | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X|  |  | UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|

## Windows 10
已加入 Windows 10 域的计算机（设备）会将某些属性同步到 Azure AD。这些属性始终同步，Windows 10 不会显示为可以取消选择的应用。通过填充 userCertificate 属性来标识已加入 Windows 10 域的计算机。

| 属性名称| 设备| 注释 |
| --- | :-: | --- |
| accountEnabled| X| |
| deviceTrustType| X| 已加入域的计算机的硬编码值。 |
| displayName | X| |
| ms-DS-CreatorSID | X| 也称为 registeredOwnerReference。|
| objectGUID | X| 也称为 deviceID。|
| objectSID | X| 也称为 onPremisesSecurityIdentifier。|
| operatingSystem | X| 也称为 deviceOSType。|
| operatingSystemVersion | X| 也称为 deviceOSVersion。|
| userCertificate | X| |

用户的这些属性是所选其他应用的补充。

| 属性名称| 用户| 注释 |
| --- | :-: | --- |
| domainFQDN| X| 也称为 dnsDomainName。例如 contoso.com。|
| domainNetBios| X| 也称为 netBiosName。例如CONTOSO。|

## Exchange 混合写回
当你选择启用 Exchange 混合部署时，这些属性将从 Azure AD 写回到本地 Active Directory。根据你的 Exchange 版本，可能会同步更少的属性。

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| msDS-ExternalDirectoryObjectID| X| | | 派生自 Azure AD 中的 cloudAnchor。这是 Exchange 2016 中的新属性。|
| msExchArchiveStatus| X|  |  | 联机存档：使客户能够存档邮件。|
| msExchBlockedSendersHash| X|  |  | 筛选：从客户端写回本地筛选及在线安全和已阻止的发件人数据。|
| msExchSafeRecipientsHash| X|  |  | 筛选：从客户端写回本地筛选及在线安全和已阻止的发件人数据。|
| msExchSafeSendersHash| X|  |  | 筛选：从客户端写回本地筛选及在线安全和已阻止的发件人数据。|
| msExchUCVoiceMailSettings| X|  |  | 启用统一消息传送 (UM) - 在线语音邮件：供 Microsoft Lync Server 集成用于向 Lync Server 本地表示用户在在线服务中有语音邮件。|
| msExchUserHoldPolicies| X|  |  | 诉讼数据保留：启用云服务来标识哪些用户正处于诉讼数据保留状态。|
| proxyAddresses| X| X| X| 只插入 Exchange Online 中的 x500 地址。|

## 设备写回
设备对象是在 Active Directory 中创建的。这些对象可以是加入 Azure AD 的设备，或加入域的 Windows 10 计算机。

| 属性名称| 设备| 注释 |
| --- | :-: | --- |
| altSecurityIdentities | X| |
| displayName | X| |
| dn | X| |
| msDS-CloudAnchor | X| |
| msDS-DeviceID | X| |
| msDS-DeviceObjectVersion | X| |
| msDS-DeviceOSType | X| |
| msDS-DeviceOSVersion | X| |
| msDS-DevicePhysicalIDs | X| |
| msDS-KeyCredentialLink | X| 只能在 Windows Server 2016 AD 架构上使用 |
| msDS-IsCompliant | X| |
| msDS-IsEnabled | X| |
| msDS-IsManaged | X| |
| msDS-RegisteredOwner | X| |


## 说明
- 使用替代 ID 时，本地属性 userPrincipalName 将与 Azure AD 属性 onPremisesUserPrincipalName 同步。替代 ID 属性（例如 mail）将与 Azure AD 属性 userPrincipalName 同步。
- 在上述列表中，对象类型 **User** 也适用于对象类型 **iNetOrgPerson**。

## 后续步骤
了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0509_2016-->