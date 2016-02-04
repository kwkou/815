<properties 
	pageTitle="Azure AD Connect 工作原理" 
	description="了解 Azure AD Connect 的工作原理。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/24/2015" 
	wacn.date=""/>

# Azure AD Connect 工作原理



Azure Active Directory Connect 由三个主要部分组成，分别是同步服务、可选的 Active Directory 联合身份验证服务功能，以及使用 [Azure AD Connect Health](https://msdn.microsoft.com/library/azure/dn906722.aspx) 实现的监视功能。


<center>![Azure AD Connect 堆栈](./media/active-directory-aadconnect-how-it-works/AADConnectStack2.png)
</center>

- 同步 - 此部分由以前包含以前作为 DirSync 和 AAD Sync 发布的组件和功能组成。此部分负责创建用户和组。它还负责确保本地环境中有关用户和组的信息与云匹配。
- AD FS - 这是 Azure AD Connect 的可选部分，可用于使用本地 AD FS 基础结构设置混合环境。组织可以使用此部分来解决复杂的部署，包括域加入 SSO、实施 AD 登录策略和智能卡或第三方 MFA 等方案。有关配置 SSO 的更多信息，请参阅[使用单一登录的 DirSync](https://msdn.microsoft.com/library/azure/dn441213.aspx)。
- 运行状况监视 - 对于使用 AD FS 的复杂部署，Azure AD Connect Health 能够可靠监视联合服务器，并在 Azure 门户中提供一个中心位置用于查看此活动。有关更多信息，请参阅 [Azure Active Directory Connect Health](https://msdn.microsoft.com/library/azure/dn906722.aspx)。


## Azure AD Connect 支持组件

下面列出了 Azure AD Connect 将在设置 Azure AD Connect 的服务器上安装的必备组件支持组件。此列表针对基本快速安装。如果你在“安装同步服务”页上选择使用其他 SQL Server，则不会安装下面列出的 SQL Server 2012 组件。

- Azure AD Connect Azure AD 连接器
- Microsoft SQL Server 2012 命令行实用工具
- Microsoft SQL Server 2012 本机客户端
- Microsoft SQL Server 2012 Express LocalDB
- 用于 Windows PowerShell 的 Azure Active Directory 模块
- 面向 IT 专业人员的 Microsoft Online Services 登录助手
- Microsoft Visual C++ 2013 Redistribution Package




 

<!---HONumber=76-->