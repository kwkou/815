<properties 
	pageTitle="什么是：Azure AD 密码管理 | Windows Azure"
	description="介绍 Azure AD 中的密码管理功能，包括密码重置、更改、密码管理报告，以及将密码写回到本地 Active Directory。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="06/08/2015" 
	wacn.date="08/29/2015"/>

# 从任意位置管理密码
利用自助服务降低成本和节省人力，一直以来都是世界各地 IT 部门追求的主要目标。事实上，市场中充斥着各种产品，让你能够从云或本地管理本地组、密码或用户配置文件。独树一帜的 Azure AD 提供一些现今市场上最容易使用且最强大的自助服务功能。

**Azure AD 密码管理**提供一组让用户随时随地从任何设备管理密码的功能，同时还能遵循你定义的安全策略。

## 概述
你在五分钟内即可开始体验 Azure AD 密码管理，而且在几小时内就能将它部署到整个组织。以下是一些有用资源，可帮助你使用该服务：

* [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works) - 了解六个不同的服务组件及其功能
* [**入门**](/documentation/articles/active-directory-passwords-getting-started) - 了解如何让用户重置及更改云密码或本地密码
* [**自定义**](/documentation/articles/active-directory-passwords-customize) - 了解如何根据组织的需求自定义服务的外观和行为
* [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices) - 了解如何快速部署且有效管理组织的密码
* [**深入分析**](/documentation/articles/active-directory-passwords-get-insights) - 了解集成式报告功能
* [**常见问题**](/documentation/articles/active-directory-passwords-faq) - 获取常见问题的解答
* [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot) - 了解如何快速排查服务的问题
* [**了解更多**](/documentation/articles/active-directory-passwords-learn-more) - 深入探索服务工作原理的技术细节


## Azure AD 密码管理有何用途？
以下是使用 Azure AD 密码管理功能可实现的某些目的。

- **自助密码更改**：可让最终用户或管理员就可以更改过期的或未过期的密码，而无需请求管理员或帮助台提供支持。
- **自助密码重置**：最终用户或管理员可以自行重置密码，而无需请求管理员或帮助台提供支持。自助密码重置功能需要 Azure AD 高级或基本版。有关详细信息，请参阅“Azure Active Directory 版本”。
- **管理员启动的密码重置**：管理员可以通过 [Azure 管理门户](https://manage.windowsazure.cn)重置某个最终用户的或其他管理员的密码。
- **密码管理活动报告**：管理员可以深入了解发生在其组织中的密码重置和注册活动。 
- **密码写回**：从云管理本地密码，因此，所有上述方案都可以由经过联合身份验证的或密码同步的用户本人或其代表来执行。密码写回功能需要 Azure AD Premium。有关详细信息，请参阅“Azure AD Premium 入门”。

## 为何使用 Azure AD 密码管理？
以下是应该使用 Azure AD 密码管理功能的某些原因

- **降低成本** - 支持人员辅助的密码重置通常占组织 IT 部门 20% 的费用
- **改善用户体验** - 用户不希望每次忘记密码时，都必须致电技术支持并花费大量时间在通话上
- **减少技术支持人力** - 密码管理是大部分组织设立技术支持的最重要因素
- **实现移动** - 用户可随时随地重置密码

## 最新更新
**带品牌的 SSPR 注册** - 2015 年 4 月

- 密码重置注册页面现在已加上你的公司徽标！

**安全问题** - 2015 年 3 月

- 我们已将安全问题发布到 GA！

**帐户解锁** - 2015 年 3 月

- 用户现在可以在重置密码后解锁帐户

<br/> <br/> <br/>

**其他资源**


* [密码管理的工作原理](/documentation/articles/active-directory-passwords-how-it-works)
* [密码管理入门](/documentation/articles/active-directory-passwords-getting-started)
* [自定义密码管理](/documentation/articles/active-directory-passwords-customize)
* [密码管理最佳实践](/documentation/articles/active-directory-passwords-best-practices)
* [如何通过密码管理报告获取操作见解](/documentation/articles/active-directory-passwords-get-insights)
* [密码管理常见问题](/documentation/articles/active-directory-passwords-faq)
* [密码管理疑难解答](/documentation/articles/active-directory-passwords-troubleshoot)
* [了解详细信息](/documentation/articles/active-directory-passwords-learn-more)
* [MSDN 上的密码管理](https://msdn.microsoft.com/zh-cn/library/azure/dn510386.aspx) 

<!---HONumber=67-->