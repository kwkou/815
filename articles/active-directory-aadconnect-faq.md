<properties 
	pageTitle="Azure AD Connect：常见问题 | Azure"
	description="此页包含有关 Azure AD Connect 的常见问题。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="05/12/2016"
	wacn.date="06/14/2016"/>

# Azure AD Connect 常见问题

## 常规安装
**问：如果 Azure AD 全局管理员已启用 2FA，安装是否能够正常进行？**  
2016 年 2 月版本开始支持此功能。

**问：Azure AD Connect 是否提供无人值守安装方法？**

仅支持使用安装向导来安装 Azure AD Connect。不支持无人值守和静默安装。

**问：我有一个林，但无法连接到其中的某个域。如何安装 Azure AD Connect？**  
2016 年 2 月版本开始支持此功能。

## 网络
**问：我的防火墙、网络设备或其他软硬件会限制在网络上打开连接的最长时间。使用 Azure AD Connect 时，客户端超时阈值应设为多少？**

所有网络软件、物理设备或其他软硬件限制最长连接时间的阈值应该至少为 5 分钟 (300 秒)，使装有 Azure AD Connect 客户端的服务器能够与 Azure Active Directory 连接。这同样适用于以前发布的 Microsoft 标识同步工具。

**是否支持 SLD（单一标签域）？**  
Azure AD Connect 不支持使用 SLD 的本地林/域。

**问：是否支持包含句点的 NetBios 名称？**  
Azure AD Connect 不支持 NetBios 名称包含句点“.”的本地林/域。

## 联合
**问：如果我收到一封电子邮件，要求我续订 Office 365 证书，我该怎么办？**  
请根据[续订证书](/documentation/articles/active-directory-aadconnect-o365-certs/)主题中所述的指导来续订证书。

**问：我为 O365 信赖方设置了“自动更新信赖方”。当我的令牌签名证书自动滚动更新时，我是否需要采取任何措施？**  
请参考[续订证书](/documentation/articles/active-directory-aadconnect-o365-certs/)一文中所述的指导。

## 环境
**问：安装 Azure AD Connect 之后，是否支持重命名服务器？**  
不支持。更改服务器名称将导致同步引擎无法连接到 SQL 数据库，并且服务将无法启动。

## 标识数据
**问：Azure AD 中的 UPN (userPrincipalName) 属性与本地 UPN 不匹配，这是为什么？**  
请参阅以下文章：

- [Office 365、Azure 或 Intune 中的用户名与本地 UPN 或备用登录 ID 不匹配](https://support.microsoft.com/en-us/kb/2523192)
- [在将用户帐户的 UPN 更改为使用不同的联合域后，Azure Active Directory 同步工具未同步更改](https://support.microsoft.com/en-us/kb/2669550)

## 自定义配置
**问：在哪里可以找到 Azure AD Connect 的 PowerShell cmdlet 介绍？**  
仅支持客户使用本站点上介绍的 cmdlet，而不支持使用 Azure AD Connect 中的其他 PowerShell cmdlet。

**问：我是否可以使用同步服务管理器中的“服务器导出/服务器导入”在服务器之间移动配置？**  
不可以。此选项不会检索所有配置设置，因此不应使用。应该改用向导在第二台服务器上创建基础配置，并使用同步规则编辑器生成 PowerShell 脚本，如此即可在服务器之间移动任何自定义规则。

## 故障排除

**问：如何获取有关 Azure AD Connect 的帮助？**

[搜索 Microsoft 知识库 (KB)](https://www.microsoft.com/zh-cn/Search/result.aspx?q=azure%20active%20directory%20connect&form=mssupport)

- 在 Microsoft 知识库 (KB) 中搜索有关 Azure AD Connect 支持的常见故障维修服务问题的技术解决方案。

[Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=WindowsAzureAD)

- 单击[此处](https://social.msdn.microsoft.com/Forums/azure/zh-cn/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)，在社区中搜索和浏览技术问题与答案，或提出自己的问题。


[Azure AD Connect 客户支持](https://manage.windowsazure.cn/?getsupport=true)

- 使用此链接，以便通过 Azure 经典管理门户获取支持。 

<!---HONumber=Mooncake_0606_2016-->