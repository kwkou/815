<properties 
	pageTitle="在 Azure 应用中使用本地 Active Directory 进行身份验证 | Azure" 
	description="了解 Azure 应用服务中的业务线应用在本地 Active Directory 上进行身份验证时可用的不同选项" 
	services="app-service" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.tgt_pltfrm="na" 
	ms.workload="web" 
	ms.date="08/31/2016" 
	wacn.date="10/25/2016"/>
	ms.author="cephalin"/>

# 在 Azure 应用中使用本地 Active Directory 进行身份验证 #

本文说明如何在 [Azure 应用服务](/documentation/articles/app-service-value-prop-what-is/)中使用本地 Active Directory (AD) 进行身份验证。Azure 应用托管在云中，但可以通过多种方法安全地对本地 AD 用户进行身份验证。

## 通过 Azure Active Directory 进行身份验证
Azure Active Directory 租户的目录可与本地 AD 同步。此方法可让 AD 用户从 Internet 访问应用，使用其本地凭据进行身份验证。此外，Azure 应用服务[为此方法提供周全的解决方案](/documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/)。只需按几下鼠标，就能为 Azure 应用启用目录同步的租户的身份验证。此方法具有以下优点：

-	应用中不需要任何身份验证代码。让应用服务自动执行身份验证，腾出时间专注于如何在应用中提供功能。
-	使用 [Azure AD 图形 API](http://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx) 可从 Azure 应用访问目录数据。
-	将 SSO 提供给 [Azure Active Directory 支持的所有应用程序](/home/features/identity/)，包括 Office 365、Dynamics CRM Online、Microsoft Intune 和数千个非 Microsoft 云应用程序。
-	Azure Active Directory 支持基于角色的访问控制。可以使用 [Authorize(Roles="X")] 模式，并且只需对代码进行最少量的更改。

若要了解如何编写使用 Azure Active Directory 进行身份验证的业务线 Azure 应用，请参阅 [Create a line-of-business Azure app with Azure Active Directory authentication](/documentation/articles/web-sites-dotnet-lob-application-azure-ad/)（使用 Azure Active Directory 身份验证创建业务线 Azure 应用）。

## 通过本地 STS 进行身份验证
如果已安装本地安全令牌服务 (STS)（例如 Active Directory 联合身份验证服务 (AD FS)），可以使用该服务来联合 Azure 应用的身份验证。当公司策略禁止将 AD 数据存储在 Azure 中时，这是最合适的方法。但请注意以下事项：

-	STS 拓扑必须在本地部署，这会产生成本和管理开销。
-	只有 AD FS 管理员可以配置[信赖方信任和声明规则](http://technet.microsoft.com/zh-cn/library/dd807108.aspx)，这可能会限制开发人员的选项。另一方面，可以根据应用程序管理和自定义[声明](http://technet.microsoft.com/zh-cn/library/ee913571.aspx)。
-	需要提供一个单独的解决方案通过公司防火墙访问本地 AD 数据。

若要了解如何编写使用本地 STS 进行身份验证的业务线 Azure 应用，请参阅 [Create a line-of-business Azure app with AD FS authentication](/documentation/articles/web-sites-dotnet-lob-application-adfs/)（使用 AD FS 身份验证创建业务线 Azure 应用）。
 

<!---HONumber=Mooncake_0926_2016-->