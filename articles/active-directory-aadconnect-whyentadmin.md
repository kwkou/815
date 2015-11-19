<properties 
	pageTitle="需要企业管理员帐户的原因" 
	description="自定义设置描述。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>

# 设置 Azure AD Connect 时需要使用企业管理员帐户连接 AD DS 的原因

下表说明了设置 Azure AD Connect 时需要企业管理员帐户的原因。

在以下情况下 | 说明 
------------- | ------------- |
快速设置和 DirSync 升级 | <li>对于“快速设置”，我们将创建用于同步的本地 Active Directory 帐户（也称为 AD 连接器帐户），并分配用于同步和密码同步的适当权限</li><li>对于 DirSync 升级，我们将在当前配置的 AD Connector 帐户上重置密码，并将新的 Azure AD Connect 同步服务配置为使用此帐户。</li>



**其他资源**


* [有关 Azure AD Connect 帐户和权限的更多信息](/documentation/articles/active-directory-aadconnect-account-summary)
* [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)
* [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/zh-cn/library/azure/dn832695.aspx) 

<!---HONumber=67-->