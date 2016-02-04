<properties
	pageTitle="Azure AD Connect Sync - 实现密码同步"
	description="为你提供需要了解密码同步的工作原理以及如何在你的环境中启用它的信息。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="swadhwa"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/27/2015"
	wacn.date="01/29/2016"/>


# Azure AD Connect Sync：实现密码同步 

在实现密码同步的情况下，你能够使你的用户使用他们用于登录到你的本地 Active Directory 的相同密码登录到 Azure Active Directory。

本主题旨在为你提供需要了解密码同步的工作原理以及如何在你的环境中启用它的信息。

## 什么是密码同步

密码同步是 Azure Active Directory Connect Synchronization Services (Azure AD Connect Sync) 的一项功能，使用此功能可以将本地 Active Directory 的用户密码同步到 Azure Active Directory (Azure AD)。利用此功能，你的用户通过使用他们用于登录到你的本地网络的相同密码，能够登录到他们的 Azure Active Directory 服务（如 Office 365、Microsoft Intune、CRM Online 等）。请务必注意，此功能不提供单一登录 (SSO) 解决方案，因为在基于密码同步的过程中，不会发生令牌共享/交换。

> [AZURE.NOTE]有关为 FIPS 和密码同步配置的 Active Directory 域服务的更多详细信息，请参阅与 FIPS 兼容的系统中未能实现的密码同步。
 
## 密码同步的可用性

Azure Active Directory 的任何客户都有资格运行密码同步。有关密码同步和其他功能（例如联合身份验证）的兼容性的信息，请参阅以下内容。

## 密码同步的工作原理

密码同步是由 Azure AD Connect Sync 实现的目录同步功能的扩展。因此，此功能要求你在本地和 Azure Active Directory 之间配置目录同步。

Active Directory 域服务以实际用户密码的哈希值表示形式存储密码。密码哈希不能用于登录到你的本地网络。还对其进行了设计，以便不能为了获得用户的纯文本密码访问权限而将其撤销。若要同步密码，Azure AD connect Sync 则提取本地 Active Directory 中的用户密码哈希。同步到 Azure Active Directory 身份验证服务之前，已对密码哈希应用其他安全处理。密码同步过程的实际数据流等同于用户数据（例如 DisplayName 或电子邮件地址）的同步。

密码的同步频率高于其他属性的标准目录同步窗口。密码基于每个用户进行同步，并且通常按时间顺序同步。当用户的密码从本地 AD 同步到云时，则会覆盖现有的云密码。

当首次启用密码同步功能时，它会将范围内的所有用户的密码从本地 Active Directory 初始同步到 Azure Active Directory。不能显式定义将其密码同步到云的一组用户。随后，当本地用户已更改密码时，大多数情况下在几分钟内，密码同步功能会检测并同步已更改的密码。密码同步功能会自动重试失败的用户密码同步。如果尝试同步密码期间出现错误，该错误会被记录在事件查看器中。

同步密码对当前登录的用户没有任何影响。如果登录到云服务的用户也更改本地密码，云服务会话则会不间断的继续。但是，只要该云服务需要用户重新进行身份验证，就需要提供新密码。此时，要求用户提供新密码，即最近已从本地 Active Directory 同步到云的密码。

## 安全注意事项

同步密码时，用户的纯文本版本的密码既不能向密码同步功能公开，也不能向 Azure AD 或任何相关联的服务公开。

此外，对于本地 Active Directory 以可撤消的加密格式存储密码没有任何要求。Windows Active Directory 密码哈希摘要用于本地 AD 和 Azure Active Directory 之间的传输。密码哈希摘要不能用于访问客户的本地环境中的资源。

## 密码策略注意事项

有两种类型的密码策略受启用密码同步的影响：

1. 密码复杂性策略
2. 密码过期策略

### 密码复杂性策略

当启用密码同步时，本地 Active Directory 中配置的密码复杂性策略会覆盖云中可能为同步的用户定义的任何复杂性策略。这意味着客户的本地 Active Directory 环境中的任何有效密码都可用于访问 Azure AD 服务。


> [AZURE.NOTE]直接在云中创建的用户的密码仍受到云中定义的密码策略的约束。
 
### 密码过期策略

如果用户属于密码同步的范围，云帐户密码则设置为“永不过期”。这意味着用户的密码可能在本地环境中过期，但他们可以使用这个过期的密码继续登录到云服务。

云密码将在下次用户在本地环境中更改密码时更新。


## 覆盖已同步密码

管理员可以使用 Azure Active Directory PowerShell 手动重置用户的密码。

在这种情况下，新密码会覆盖用户的已同步密码，并且在云中定义的所有密码策略都会应用于新的密码。

如果用户再次更改本地密码，新密码则会同步到云，并会手动覆盖更新的密码。


## 准备密码同步

必须启用 Azure Active Directory 租户进行目录同步，然后才能启用该租户进行密码同步。


## 启用密码同步

在运行 Azure AD Connect 配置向导时启用密码同步。

在“可选功能”对话框页上，选择“**密码同步**”。
 
![可选功能][1]


> [AZURE.NOTE]此过程会触发完全同步。完成完全同步周期通常比完成其他同步周期花费更长的时间。
 

## 管理密码同步

你可以通过运行 Azure AD Connect 的计算机的事件日志监视密码同步的进度。


### 确定密码同步状态

你可以通过查看符合以下条件的事件确定已经成功同步其密码的用户：

| 源| 事件 ID |
| --- | --- |
| 目录同步| 656|
| 目录同步| 657|
 
具有事件 ID 656 的事件提供已处理密码更改请求的报告：

![事件 ID 656][2]

具有 ID 657 的相应事件提供这些请求的结果：

![事件 ID 657][3]

在事件中，受影响的对象由其定位点和 DN 值进行标识。定位点值与通过 Get-MsoUser cmdlet 针对某个用户返回的 **ImmutableId** 值对应。

除了对象标识符之外，**事件 ID 656** 提供用户的密码在本地 Active Directory 中进行更改的日期：

![密码更改请求][4]

事件 ID 657 具有除了源对象标识符之外的结果字段，以指示该用户对象的同步状态。

成功同步的密码在事件 ID 657 通过 Result 属性的成功值表示的事件中。当密码同步尝试失败时，Result 属性的值为 Failure：

![密码更改结果][5]

 
## 禁用密码同步

通过重新运行 Azure AD Connect 配置向导禁用密码同步。当向导出现提示时，取消选中“密码同步”复选框。


> [AZURE.NOTE]此过程会触发完全同步。完成完全同步周期通常比完成其他同步周期花费更长的时间。
 
运行配置向导之后，你的租户不再同步密码。新密码更改不会同步到云。以前同步其密码的用户必须将能够继续使用这些密码登录，直到他们在云中手动更改其密码。



## 其他资源

* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)
 
<!--Image references-->
[1]: ./media/active-directory-aadsync-implement-password-synchronization/IC759788.png
[2]: ./media/active-directory-aadsync-implement-password-synchronization/IC662504.png
[3]: ./media/active-directory-aadsync-implement-password-synchronization/IC662505.png
[4]: ./media/active-directory-aadsync-implement-password-synchronization/IC662506.png
[5]: ./media/active-directory-aadsync-implement-password-synchronization/IC662507.png

<!---HONumber=71-->