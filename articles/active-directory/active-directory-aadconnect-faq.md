<properties
    pageTitle="Azure Active Directory Connect：常见问题 - Azure | Azure"
    description="此页包含有关 Azure AD Connect 的常见问题。"
    services="active-directory"
    documentationcenter=""
    author="billmath"
    manager="femila" />

<tags
    ms.assetid="4e47a087-ebcd-4b63-9574-0c31907a39a3"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/22/2017"
    ms.author="billmath" 
    wacn.date="04/05/2017"/>  


# 有关 Azure Active Directory Connect 的常见问题

## 常规安装
**问：如果 Azure AD 全局管理员已启用 2FA，安装是否能够正常进行？** 2016 年 2 月版本开始支持此功能。

**问：Azure AD Connect 是否提供无人值守安装方法？** 仅支持使用安装向导来安装 Azure AD Connect。不支持无人值守和静默安装。

**问：我有一个林，但无法连接到其中的某个域。我应该如何安装 Azure AD Connect？** 2016 年 2 月版本开始支持此功能。

**问：AD DS Health 代理是否可以在 Server Core 上运行？** 是的。安装代理后，可以使用以下 PowerShell cmdlet 完成注册过程：

`Register-AzureADConnectHealthADDSAgent -Credentials $cred`

## 网络
**问：我的防火墙、网络设备或其他软硬件会限制在网络上打开连接的最长时间。使用 Azure AD Connect 时，客户端超时阈值应设为多少？** 所有网络软件、物理设备或其他软硬件限制最长连接时间保持打开状态的阈值应该至少为 5 分钟（300 秒），使安装了 Azure AD Connect 客户端的服务器能够与 Azure Active Directory 连接。这同样适用于以前发布的 Microsoft 标识同步工具。

**是否支持 SLD（单一标签域）？** Azure AD Connect 不支持使用 SLD 的本地林/域。

**问：是否支持包含句点的 NetBios 名称？** Azure AD Connect 不支持 NetBios 名称包含句点“.”的本地林/域。

## 联合
**问：如果我收到一封电子邮件，要求我续订 Office 365 证书，我该怎么办？**请根据[续订证书](/documentation/articles/active-directory-aadconnect-o365-certs/)主题中所述的指导来续订证书。

**问：我为 O365 信赖方设置了“自动更新信赖方”。当我的令牌签名证书自动滚动更新时，我是否需要采取任何措施？** 请参考[续订证书](/documentation/articles/active-directory-aadconnect-o365-certs/)一文中所述的指导。

## 环境
**问：安装 Azure AD Connect 之后，是否支持重命名服务器？** 不支持。更改服务器名称将导致同步引擎无法连接到 SQL 数据库，并且服务将无法启动。

## 标识数据
**问：Azure AD 中的 UPN (userPrincipalName) 属性与本地 UPN 不匹配，这是为什么？** 请参阅以下文章：

- [Office 365、Azure 或 Intune 中的用户名与本地 UPN 或备用登录 ID 不匹配](https://support.microsoft.com/zh-cn/kb/2523192)
- [在将用户帐户的 UPN 更改为使用不同的联合域后，Azure Active Directory 同步工具未同步更改](https://support.microsoft.com/zh-cn/kb/2669550)

你还可以根据 [Azure AD Connect 同步服务功能](/documentation/articles/active-directory-aadconnectsyncservice-features/)中所述配置 Azure AD，以允许同步引擎更新 userPrincipalName。

**问：是否支持将本地 AD 组/联系人对象与现有的 Azure AD 组/联系人对象进行软匹配？** 否，目前不支持。

**问：是否支持在现有的 Azure AD 组/联系人对象上手动设置 ImmutableId 属性，以便将其与本地 AD 组/联系人对象进行硬匹配？** 否，目前不支持。

## “安全”
**问：是将在失败尝试次数达到特定数字后锁定帐户，还是会使用更复杂的策略？**</br> 我们使用更复杂的策略锁定帐户。此策略基于请求的 IP 和输入的密码。根据存在攻击的可能性，锁定持续时间也会增加。

**问：某些（常用）密码被拒绝并显示消息“此密码已使用多次”，这是指当前 Active Directory 中使用的密码吗？**</br> 这是指全局通用的密码，例如“Password”和“123456”的任何变体。

**问：来自可疑来源（僵尸网络、tor 终结点）的登录请求在 B2C 租户中会被阻止吗？还是说需要成为基本版或高级版租户才会如此？**</br> 我们确实有一个网关，该网关筛选请求且提供一些保护免受僵尸网络的危害，并应用于所有 B2C 租户。

## 自定义配置
**问：在哪里可以找到 Azure AD Connect 的 PowerShell cmdlet 介绍？** 仅支持客户使用本站点上介绍的 cmdlet，而不支持使用 Azure AD Connect 中的其他 PowerShell cmdlet。

**问：我是否可以使用 *Synchronization Service Manager* 中的“服务器导出/服务器导入”在服务器之间移动配置？** 不可以。此选项不会检索所有配置设置，因此不应使用。应该改用向导在第二台服务器上创建基础配置，并使用同步规则编辑器生成 PowerShell 脚本，如此即可在服务器之间移动任何自定义规则。请参阅[交叉迁移](/documentation/articles/active-directory-aadconnect-upgrade-previous-version/#swing-migration/)。

**问：是否可以为 Azure 登录页缓存密码？是否可以因为该页包含具有 autocomplete = "false" 属性的密码输入元素而阻止此功能？**</br> 我们目前不支持修改密码输入字段的 HTML 属性，包括 autocomplete 标记。我们当前正在开发一种功能，该功能允许使用自定义 Javascript，以便将任何属性添加到密码字段。该功能应该在 2017 年下半年可用。

**问：在 Azure 登录页上，将显示以前已成功登录的用户的用户名。此行为是否可关闭？**</br> 我们目前不支持修改登录页的 HTML 属性。我们当前正在开发一种功能，该功能允许使用自定义 Javascript，以便将任何属性添加到密码字段。该功能应该在 2017 年下半年可用。

**问：是否有方法阻止并发会话？**</br> 没有。



## 故障排除
**问：如何获取有关 Azure AD Connect 的帮助？**

- 在 Microsoft 知识库 (KB) 中搜索有关 Azure AD Connect 支持的常见故障维修服务问题的技术解决方案。

[Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=WindowsAzureAD)

- 单击[此处](https://social.msdn.microsoft.com/Forums/azure/zh-cn/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)，在社区中搜索和浏览技术问题与答案，或提出自己的问题。

[Azure AD Connect 客户支持](https://manage.windowsazure.cn/?getsupport=true)

- 使用此链接通过 Azure 门户获取支持。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->