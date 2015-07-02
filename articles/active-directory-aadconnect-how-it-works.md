<properties 
	pageTitle="Azure AD Connect 工作原理" 
	description="了解 Azure AD Connect 的工作原理。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="terrylan" 
	editor="lisatoft"/>

<tags 
	ms.service="active-directory"  
	ms.date="04/02/2015"
	wacn.date="06/16/2015"/>

# Azure AD Connect 工作原理 


<div>
<a href="/documentation/articles/active-directory-aadconnect/">简介</a> <a href="/documentation/articles/active-directory-aadconnect-how-it-works/">工作原理</a> <a href="/documentation/articles/active-directory-aadconnect-get-started/">入门</a> <a href="/documentation/articles/active-directory-aadconnect-whats-next/">后续步骤</a> <a href="/documentation/articles/active-directory-aadconnect-learn-more/">了解详细信息</a>
</div>

Azure Active Directory Connect 由三个主要部分组成，分别是同步服务、可选的 Active Directory 联合身份验证服务功能，以及使用 Azure AD Connect Health 实现的监视功能。 


![Azure AD Connect 堆栈](./media/active-directory-aadconnect-how-it-works/AADConnectStack.png)


- 同步 - 此部分由以前包含以前作为 DirSync 和 AAD Sync 发布的组件和功能组成。此部分负责创建用户和组。它还负责确保本地环境中有关用户和组的信息与云中的信息匹配。
- AD FS - 这是 Azure AD Connect 的可选部分，可用于使用本地 AD FS 基础结构设置混合环境。组织可以使用此部分来解决复杂的部署，包括域加入 SSO、实施 AD 登录策略和智能卡或第三方 MFA 等方案。有关配置 SSO 的更多信息，请参阅[使用单一登录的 DirSync](https://msdn.microsoft.com/zh-cn/library/azure/dn441213.aspx)。
- 运行状况监视 - 对于使用 AD FS 的复杂部署，Azure AD Connect Health 能够可靠监视联合服务器，并在 Azure 门户中提供一个中心位置用于查看此活动。有关更多信息，请参阅 [Azure Active Directory Connect Health](https://msdn.microsoft.com/zh-cn/library/azure/dn906722.aspx)。






**其他资源**

* [在云中使用本地标识基础结构](active-directory-aadconnect.md)
* [Azure AD Connect 入门](active-directory-aadconnect-get-started.md)
* [Azure AD Connect 后续步骤](active-directory-aadconnect-whats-next.md)
* [了解详细信息](active-directory-aadconnect-learn-more.md)
* [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/library/azure/dn832695.aspx)

<!---HONumber=60-->