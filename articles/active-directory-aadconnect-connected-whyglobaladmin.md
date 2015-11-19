<properties 
	pageTitle="设置 Azure AD Connect 时需要 Azure AD 全局管理员帐户的原因" 
	description="解释为何需要全局管理员帐户的自定义设置描述。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>

# 设置 Azure AD Connect 时需要 Azure AD 全局管理员帐户的原因

下表说明了设置 Azure AD Connect 时需要 Azure AD 全局管理员帐户的原因。

在以下情况下 | 说明 
------------- | ------------- |
快速设置和 DirSync 升级 | 我们将在 Azure AD 目录中启用同步（如果需要），并创建用于进行中同步操作的 Azure AD 帐户（Azure AD 连接器帐户）。 
对于自定义设置 | 我们将在 Azure AD 目录中启用同步，并创建用于进行中同步操作的 Azure AD 帐户（Azure AD 连接器帐户）。对于使用 AD FS 选项的单一登录，我们将在 Azure AD 中读取和配置联合属性。



**其他资源**


* [有关 Azure AD Connect 帐户和权限的更多信息](/documentation/articles/active-directory-aadconnect-account-summary)
* [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)
* [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/zh-cn/library/azure/dn832695.aspx) 

<!---HONumber=67-->