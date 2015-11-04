<properties
   pageTitle="Azure AD Connect：版本发布历史记录 | Windows Azure"
   description="本主题列出 Azure AD Connect 和 Azure AD Sync 的所有版本"
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="08/24/2015"
   wacn.date="11/02/2015"/>

# Azure AD Connect：版本发布历史记录

Azure Active Directory 团队会定期更新 Azure AD Sync 的新特性和功能。并非所有的新增内容都适用于所有受众。

本文旨在帮助你跟踪已发布的版本，并了解你是否需要更新为最新版本。

## 1.0.8667.0
发布日期：2015 年 8 月

**新功能：**

- Azure AD Connect 安装向导现已本地化为所有 Windows Server 语言。
- 添加了在使用 Azure AD 密码管理时的帐户解锁支持。

**已解决的问题：**

- 如果另一位用户而不是第一位启动安装的人继续安装，则 Azure AD Connect 安装向导将会崩溃。
- 如果 Azure AD Connect 的先前卸载操作无法将 Azure AD Connect Sync 完全卸载，则无法重新安装。
- 如果用户不在林的根域中或使用了非英文版 Active Directory，则无法使用快速安装选项安装 Azure AD Connect。
- 如果无法解析 Active Directory 用户帐户的 FQDN，则会显示“无法提交架构”的误导性错误消息。
- 如果 Active Directory 连接器上使用的帐户已在向导外部更改，向导在进行后续操作时将会失败。
- Azure AD Connect 有时无法在域控制器上安装。
- 如果添加了扩展属性，则无法启用和禁用“暂存模式”。
- 由于 Active Directory 连接器上的密码不正确，某些配置中的密码写回失败。
- 如果属性筛选中使用 dn，则无法升级 DirSync。

## 1.0.8641.0
发布日期：2015 年 6 月

**Azure AD Connect 的初始版本。**

名称从 Azure AD Sync 更改为 Azure AD Connect。

## 1.0.494.0501
发布日期：2015 年 5 月

**新要求：**

- Azure AD Sync 现在要求安装 .Net framework 版本 4.5.1。

**已解决的问题：**

- 从 Azure AD 进行密码写回失败并出现 servicebus 连接错误。

## 1.0.491.0413
发布日期：2015 年 4 月

**已解决的问题和改进：**

- 如果启用回收站且林中存在多个域，Active Directory 连接器不会正确处理删除。
- 对 Azure Active Directory 连接器的导入操作性能有所改进。
- 当某个组超过成员资格限制（默认情况下，此限制设置为 5 万个对象）时，便会在 Azure Active Directory 中删除该组。新行为是将保留该组、引发错误且不导出任何新的成员资格更改。
- 如果连接器空间中已经存在 DN 相同的暂存删除，则无法设置新对象。
- 某些对象需在增量同步期间同步，尽管对象上并无暂存更改。
- 强制密码同步还会删除首选的 DC 列表。
- CSExportAnalyzer 的某些对象状态存在问题。

**新功能：**

- 联接现在可以连接到 MV 中的“任何”对象类型。

## 1.0.485.0222
发布日期：2015 年 2 月

**改进：**

- 改进了导入性能。

**已解决的问题：**

- 密码同步具有属性筛选所用的 cloudFiltered 属性。已筛选的对象将不再在密码同步范围中。
- 在拓扑中有非常多域控制器的极少数情况下，密码同步不起作用。
- 在设备管理后从 Azure AD 连接器导入时，已在 Azure AD/Intune 中启用了“Stopped-server”。
- 从同一林中的多个域联接外部安全主体 (FSP) 会导致模糊联接错误。

## 1.0.475.1202
发布日期：2014 年 12 月

**新功能：**

- 现在支持使用基于属性的筛选执行密码同步。有关详细信息，请参阅“使用筛选进行密码同步”。
- 属性 msDS-ExternalDirectoryObjectID 将写回 AD。这将添加对 Office 365 应用程序的支持，支持其使用 OAuth2 同时访问混合 Exchange 部署中的联机邮箱和本地邮箱。

**修复了升级问题：**

- 服务器上提供了登录助手的更新版本。
- 自定义安装路径用于安装 Azure AD Sync。
- 无效的自定义加入条件阻止了升级。

**其他修复：**

- 修复了 Office Pro Plus 的模板。
- 修复了以短划线开头的用户名导致的安装问题。
- 修复了第二次运行安装向导时丢失 sourceAnchor 设置的问题。
- 修复了用于密码同步的 ETW 跟踪

## 1.0.470.1023
发布日期：2014 年 10 月

**新功能：**

- 从多个本地 AD 到 Azure AD 的密码同步。
- 已将安装 UI 本地化为所有的 Windows Server 语言。

**从 AADSync 1.0 正式版升级**

如果你已经安装 Azure AD Sync，则还必须执行另外一个步骤（考虑到你可能已更改现成的同步规则）。在升级到 1.0.470.1023 版之后，已修改的同步规则将被复制。对于每个已修改的同步规则，请执行以下操作：

- 找到已修改的同步规则，并记下所做的更改。
- 删除同步规则。
- 找到由 Azure AD Sync 创建的新同步规则，然后重新应用所做的更改。

**AD 帐户的权限**

必须为 AD 帐户授予其他权限，才能从 AD 读取密码哈希。要授予的权限称为“复制目录更改”和“复制目录更改所有项”。需要这两个权限才能读取密码哈希。

## 1.0.419.0911
发布日期：2014 年 9 月

**Azure AD Sync 的初始版本。**

## 其他资源
<!-- [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis) -->

[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=76-->