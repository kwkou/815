<properties 
	pageTitle="使用 Azure AD Connect 连接你的目录" 
	description="Azure AD Connect 连接目录的自定义设置描述。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>



# 使用 Azure AD Connect 连接你的目录

对于“自定义设置”，不需要使用企业管理员帐户连接到 Active Directory 林。向导将接受域或本地用户帐户。但是，该帐户将用作本地 AD 连接器帐户，也就是说，正是该帐户读取和写入用于同步的目录信息。

这意味着，你必须分配其他权限以实现以下方案：

方案 |权限
------------- | ------------- |
密码同步| <li>复制目录更改。</li> <li>复制所有目录更改。</li>
Exchange 混合部署|请参阅 [Office 365 Exchange 混合 AAD 同步写回属性和权限](https://msdn.microsoft.com/zh-cn/library/azure/dn757602.aspx#exchange)。
密码写回 | <li>更改密码</li><li>重置密码</li>
用户、组和设备写回|对你想要写回的目录对象和属性的写入权限
单一登录和 AD FS| 联合服务器所在的域中的域管理员权限。





**其他资源**

* [有关 Azure AD Connect 帐户和权限的更多信息](/documentation/articles/active-directory-aadconnect-account-summary)
* [用于密码同步的权限](https://msdn.microsoft.com/zh-cn/library/azure/dn757602.aspx#psynch)
* [用于 Exchange 混合部署的权限](https://msdn.microsoft.com/zh-cn/library/azure/dn757602.aspx#exchange)
* [用于密码写回的权限](https://msdn.microsoft.com/zh-cn/library/azure/dn757602.aspx#pwriteback)
* [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)
* [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/zh-cn/library/azure/dn832695.aspx)
 

<!---HONumber=67-->