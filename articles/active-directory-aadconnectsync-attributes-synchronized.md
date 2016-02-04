<properties
	pageTitle="Azure AD Connect Sync：与 Azure Active Directory 同步的属性"
	description="列出同步到 Azure Active Directory 的属性。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="swadhwa"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/27/2015"
	wacn.date="01/29/2016"/>


# Azure AD Connect Sync：与 Azure Active Directory 同步的属性

本主题列出了通过 Azure AD Connect Sync 同步的属性。<br> 属性按照相关的 Azure AD 应用进行分组。
 




## Office 365 ProPlus

| 属性名称| 用户| 注释 |
| --- | :-: | --- |
| accountEnabled| X| 派生自 userAccountControl|
| cn| X| |
| displayName| X| |
| objectSID| X| |
| pwdLastSet| X| |
| sourceAnchor| X| 用于用户的属性在安装指南中进行配置。|
| usageLocation| X| AD DS 中的 msExchUsageLocation|
| userPrincipalName| X| |


## Exchange Online

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X| | | 派生自 userAccountControl|
| assistant| X| X| | |
| authOrig| X| X| X| |
| c| X| X| | |
| cn| X| | X| |
| co| X| X| | |
| company| X| X| | |
| countryCode| X| X| | |
| department| X| X| | |
| description| X| X| X| |
| displayName| X| X| X| |
| dLMemRejectPerms| X| X| X| |
| dLMemSubmitPerms| X| X| X| |
| extensionAttribute1| X| X| X| |
| extensionAttribute10| X| X| X| |
| extensionAttribute11| X| X| X| |
| extensionAttribute12| X| X| X| |
| extensionAttribute13| X| X| X| |
| extensionAttribute14| X| X| X| |
| extensionAttribute15| X| X| X| |
| extensionAttribute2| X| X| X| |
| extensionAttribute3| X| X| X| |
| extensionAttribute4| X| X| X| |
| extensionAttribute5| X| X| X| |
| extensionAttribute6| X| X| X| |
| extensionAttribute7| X| X| X| |
| extensionAttribute8| X| X| X| |
| extensionAttribute9| X| X| X| |
| facsimiletelephonenumber| X| X| | |
| givenName| X| X| | |
| homePhone| X| X| | |
| info| X| X| X| |
| Initials| X| X| | |
| l| X| X| | |
| legacyExchangeDN| X| X| X| |
| mailNickname| X| X| X| |
| mangedBy| | | X| |
| manager| X| X| | |
| member| | | X| |
| mobile| X| X| | |
| msDS-HABSeniorityIndex| X| X| X| |
| msDS-PhoneticDisplayName| X| X| X| |
| msExchArchiveGUID| X| | | |
| msExchArchiveName| X| | | |
| msExchAssistantName| X| X| | |
| msExchAuditAdmin| X| | | |
| msExchAuditDelegate| X| | | |
| msExchAuditDelegateAdmin| X| | | |
| msExchAuditOwner| X| | | |
| msExchBlockedSendersHash| X| X| | |
| msExchBypassAudit| X| | | |
| msExchCoManagedByLink| | | X| |
| msExchDelegateListLink| X| | | |
| msExchELCExpirySuspensionEnd| X| | | |
| msExchELCExpirySuspensionStart| X| | | |
| msExchELCMailboxFlags| X| | | |
| msExchEnableModeration| X| | X| |
| msExchExtensionCustomAttribute1| X| X| X| |
| msExchExtensionCustomAttribute2| X| X| X| |
| msExchExtensionCustomAttribute3| X| X| X| |
| msExchExtensionCustomAttribute4| X| X| X| |
| msExchExtensionCustomAttribute5| X| X| X| |
| msExchHideFromAddressLists| X| X| X| |
| msExchImmutableID| X| | | |
| msExchLitigationHoldDate| X| X| X| |
| msExchLitigationHoldOwner| X| X| X| |
| msExchMailboxAuditEnable| X| | | |
| msExchMailboxAuditLogAgeLimit| X| | | |
| msExchMailboxGuid| X| | | |
| msExchModeratedByLink| X| X| X| |
| msExchModerationFlags| X| X| X| |
| msExchRecipientDisplayType| X| X| X| |
| msExchRecipientTypeDetails| X| X| X| |
| msExchRemoteRecipientType| X| | | |
| msExchRequireAuthToSendTo| X| X| X| |
| msExchResourceCapacity| X| | | |
| msExchResourceDisplay| X| | | |
| msExchResourceMetaData| X| | | |
| msExchResourceSearchProperties| X| | | |
| msExchRetentionComment| X| X| X| |
| msExchRetentionURL| X| X| X| |
| msExchSafeRecipientsHash| X| X| | |
| msExchSafeSendersHash| X| X| | |
| msExchSenderHintTranslations| X| X| X| |
| msExchTeamMailboxExpiration| X| | | |
| msExchTeamMailboxOwners| X| | | |
| msExchTeamMailboxSharePointUrl| X| | | |
| msExchUserHoldPolicies| X| | | |
| msOrg-IsOrganizational| | | X| |
| objectSID| X| | X| |
| oOFReplyToOriginator| | | X| |
| otherFacsimileTelephone| X| X| | |
| otherHomePhone| X| X| | |
| otherTelephone| X| X| | |
| pager| X| X| | |
| physicalDeliveryOfficeName| X| X| | |
| postalCode| X| X| | |
| proxyAddresses| X| X| X| |
| publicDelegates| X| X| X| |
| pwdLastSet| X| | | |
| reportToOriginator| | | X| |
| reportToOwner| | | X| |
| securityEnabled| | | X| 派生自 groupType|
| sn| X| X| | |
| sourceAnchor| X| X| X| 用于用户的属性在安装指南中进行配置。|
| 号| X| X| | |
| streetAddress| X| X| | |
| targetAddress| X| X| | |
| telephoneAssistant| X| X| | |
| telephoneNumber| X| X| | |
| thumbnailphoto| X| X| | |
| title| X| X| | |
| unauthOrig| X| X| X| |
| usageLocation| X| | | AD DS 中的 msExchUsageLocation|
| userCertificate| X| X| | |
| userPrincipalName| X| | | |
| userSMIMECertificates| X| X| | |
| wWWHomePage| X| X| | |


## SharePoint Online

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X| | | 派生自 userAccountControl|
| authOrig| X| X| X| |
| c| X| X| | |
| cn| X| | X| |
| co| X| X| | |
| company| X| X| | |
| countryCode| X| X| | |
| department| X| X| | |
| description| X| X| X| |
| displayName| X| X| X| |
| dLMemRejectPerms| X| X| X| |
| dLMemSubmitPerms| X| X| X| |
| extensionAttribute1| X| X| X| |
| extensionAttribute10| X| X| X| |
| extensionAttribute11| X| X| X| |
| extensionAttribute12| X| X| X| |
| extensionAttribute13| X| X| X| |
| extensionAttribute14| X| X| X| |
| extensionAttribute15| X| X| X| |
| extensionAttribute2| X| X| X| |
| extensionAttribute3| X| X| X| |
| extensionAttribute4| X| X| X| |
| extensionAttribute5| X| X| X| |
| extensionAttribute6| X| X| X| |
| extensionAttribute7| X| X| X| |
| extensionAttribute8| X| X| X| |
| extensionAttribute9| X| X| X| |
| facsimiletelephonenumber| X| X| | |
| givenName| X| X| | |
| hideDLMembership| | | X| |
| homephone| X| X| | |
| info| X| X| X| |
| initials| X| X| | |
| ipPhone| X| X| | |
| l| X| X| | |
| mail| X| X| X| |
| mailnickname| X| X| X| |
| managedBy| | | X| |
| manager| X| X| | |
| member| | | X| |
| middleName| X| X| | |
| mobile| X| X| | |
| msExchTeamMailboxExpiration| X| | | |
| msExchTeamMailboxOwners| X| | | |
| msExchTeamMailboxSharePointLinkedBy| X| | | |
| msExchTeamMailboxSharePointUrl| X| | | |
| objectSID| X| | X| |
| oOFReplyToOriginator| | | X| |
| otherFacsimileTelephone| X| X| | |
| otherHomePhone| X| X| | |
| otherIpPhone| X| X| | |
| otherMobile| X| X| | |
| otherPager| X| X| | |
| otherTelephone| X| X| | |
| pager| X| X| | |
| physicalDeliveryOfficeName| X| X| | |
| postalCode| X| X| | |
| postOfficeBox| X| X| | |
| preferredLanguage| X| | | |
| proxyAddresses| X| X| X| |
| pwdLastSet| X| | | |
| reportToOriginator| | | X| |
| reportToOwner| | | X| |
| securityEnabled| | | X| 派生自 groupType|
| sn| X| X| | |
| sourceAnchor| X| X| X| 用于用户的属性在安装指南中进行配置。|
| 号| X| X| | |
| streetAddress| X| X| | |
| targetAddress| X| X| | |
| telephoneAssistant| X| X| | |
| telephoneNumber| X| X| | |
| thumbnailphoto| X| X| | |
| title| X| X| | |
| unauthOrig| X| X| X| |
| url| X| X| | |
| usageLocation| X| | | AD DS 中的 msExchUsageLocation|
| userPrincipalName| X| | | |
| wWWHomePage| X| X| | |

## Lync Online

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X| | | 派生自 userAccountControl|
| c| X| X| | |
| cn| X| | X| |
| co| X| X| | |
| company| X| X| | |
| department| X| X| | |
| description| X| X| X| |
| displayName| X| X| X| |
| facsimiletelephonenumber| X| X| X| |
| givenName| X| X| | |
| homephone| X| X| | |
| ipPhone| X| X| | |
| l| X| X| | |
| mail| X| X| X| |
| mailNickname| X| X| X| |
| managedBy| | | X| |
| manager| X| X| | |
| member| | | X| |
| mobile| X| X| | |
| msExchHideFromAddressLists| X| X| X| |
| msRTCSIP-ApplicationOptions| X| | | |
| msRTCSIP-DeploymentLocator| X| X| | |
| msRTCSIP-Line| X| X| | |
| msRTCSIP-OptionFlags| X| X| | |
| msRTCSIP-OwnerUrn| X| | | |
| msRTCSIP-PrimaryUserAddress| X| X| | |
| msRTCSIP-UserEnabled| X| X| | |
| objectSID| X| | X| |
| otherTelephone| X| X| | |
| physicalDeliveryOfficeName| X| X| | |
| postalCode| X| X| | |
| preferredLanguage| X| | | |
| proxyAddresses| X| X| X| |
| pwdLastSet| X| | | |
| securityEnabled| | | X| 派生自 groupType|
| sn| X| X| | |
| sourceAnchor| X| X| X| 用于用户的属性在安装指南中进行配置。|
| 号| X| X| | |
| streetAddress| X| X| | |
| telephoneNumber| X| X| | |
| thumbnailphoto| X| X| | |
| title| X| X| | |
| usageLocation| X| | | AD DS 中的 msExchUsageLocation|
| userPrincipalName| X| | | |
| wWWHomePage| X| X| | |


## Azure RMS

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X| | | 如果启用了帐户，则进行定义。|
| cn| X| | X| 公用名或别名。大多数情况下是 [mail] 值的前缀。|
| displayName| X| X| X| 表示通常显示为友好名称（名字姓氏）的名称的字符串。|
| mail| X| X| X| 完整的电子邮件地址。|
| member| | | X| |
| objectSID| X| | X| 机械属性。用于维护 Azure AD 和 AD 之间的同步的 AD 用户标识符。|
| proxyAddresses| X| X| X| 机械属性。通过 Azure AD 使用。包含用户的所有辅助电子邮件地址。|
| pwdLastSet| X| | | 机械属性。用于了解使已颁发令牌失效的时间。|
| securityEnabled| | | X| 派生自 groupType。|
| sourceAnchor| X| X| X| 机械属性。用于保持 ADDS 与 Azure AD 之间的关系的不可变标识符。|
| usageLocation| X| | | 机械属性。用户所在的国家/地区。用于进行许可证分配。|
| userPrincipalName| X| | | 此 UPN 是用户的登录 ID。大多数情况下与 [mail] 值相同。|


## Intune

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X| | | 派生自 userAccountControl|
| c| X| X| | |
| cn| X| | X| |
| description| X| X| X| |
| displayName| X| X| X| |
| mail| X| X| X| |
| mailnickname| X| X| X| |
| member| | | X| |
| objectSID| X| | X| |
| proxyAddresses| X| X| X| |
| pwdLastSet| X| | | |
| securityEnabled| | | X| 派生自 groupType|
| sourceAnchor| X| X| X| 用于用户的属性在安装指南中进行配置。|
| usageLocation| X| | | AD DS 中的 msExchUsageLocation|
| userPrincipalName| X| | | |


## Dynamics CRM

| 属性名称| 用户| 联系人| 组| 注释 |
| --- | :-: | :-: | :-: | --- |
| accountEnabled| X| | | 派生自 userAccountControl|
| c| X| X| | |
| cn| X| | X| |
| co| X| X| | |
| company| X| X| | |
| countryCode| X| X| | |
| description| X| X| X| |
| displayName| X| X| X| |
| facsimiletelephonenumber| X| X| | |
| givenName| X| X| | |
| l| X| X| | |
| managedBy| | | X| |
| manager| X| X| | |
| member| | | X| |
| mobile| X| X| | |
| objectSID| X| | X| |
| physicalDeliveryOfficeName| X| X| | |
| postalCode| X| X| | |
| preferredLanguage| X| | | |
| pwdLastSet| X| | | |
| securityEnabled| | | X| 派生自 groupType|
| sn| X| X| | |
| sourceAnchor| X| X| X| 用于用户的属性在安装指南中进行配置。|
| 号| X| X| | |
| streetAddress| X| X| | |
| telephoneNumber| X| X| | |
| title| X| X| | |
| usageLocation| X| | | AD DS 中的 msExchUsageLocation|
| userPrincipalName| X| | | |


## 其他资源

* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)
 
<!--Image references-->

<!---HONumber=71-->