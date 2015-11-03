<properties 
	pageTitle="有关 Azure AD Connect 凭据和权限的更多信息" 
	description="Azure AD Connect 凭据和权限的自定义设置描述。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>



# 有关 Azure AD Connect 凭据和权限的更多信息 


Azure AD Connect 向导提供两种不同方法，它们的权限要求不同：

* 在“快速设置”中，我们需要更多权限，以便轻松设置配置，而无需创建用户或单独配置权限。 

* 在“自定义设置”中，我们提供更多选择和选项，但在某些情况下，你将需要确保自己具有正确的权限。


## 收集的凭据，以及这些凭据在快速设置中的用途

向导页 | 收集的凭据 | 所需的权限| 用于 
------------- | ------------- |------------- |------------- |
连接到 Azure AD| Azure AD 目录凭据 | Azure AD 中的全局管理员角色 | <li>在 Azure AD 目录中启用同步。</li> <li>创建将在 Azure AD 中用于进行中同步操作的 Azure AD 帐户。</li>
连接到 AD DS | 本地 Active Directory 凭据 | Active Directory 中企业管理员 (EA) 组的成员| 将用作本地 AD 连接器帐户，也就是说，正是该帐户读取和写入用于同步的目录信息。
不可用|运行向导的用户的登录凭据| 本地服务器的管理员|向导将创建 AD 帐户，该帐户将用作本地计算机上的同步服务登录帐户。

<br> <br>


## 收集的凭据和它们在自定义安装程序中的用途

向导页 | 收集的凭据 | 所需的权限| 用于 
------------- | ------------- |------------- |------------- |
NA|运行向导的用户的登录凭据|本地服务器的管理员| <li>默认情况下，该向导创建将在本地计算机上用作同步服务登录帐户的 AD 帐户</li><li>如果管理员不指定特定帐户，我们仅创建同步服务登录帐户</li><li>该帐户是本地用户，除非在 DC 上该帐户才是域用户</li> 
安装“同步服务”页，“服务”帐户选项 | AD 或本地用户帐户凭据 | 本地用户|如果管理员指定了帐户，则此帐户用作同步服务的登录帐户。
连接到 Azure AD|Azure AD 目录凭据| Azure AD 中的全局管理员角色|该向导创建将在本地计算机上用作同步服务登录帐户的 AD 帐户。
连接你的目录|将连接到 Azure AD 的每个林的本地 Active Directory 凭据 |<li>向导所需的最低级别权限是域用户。</li> <li>但是，指定帐户必须具有预期方案要求的权限。</li><li>如果你想要为 Azure AD 配置密码同步，请确保为此帐户分配以下权限：-复制目录更改 -复制所有目录更改</li><li>如果你想要配置同步，以将信息从 Azure AD“写回”本地 AD，请确保该帐户对要写回的目录对象和属性拥有写入权限。</li> <li>如果你想要为登录配置 AD FS，请确保你为 AD FS 服务器所在的林提供的 AD 凭据拥有域管理员权限。</li><li>请参阅下表针对你的方案列出的其他要求。</li>|<li>创建本地 AD 管理代理 (MA) 帐户，该帐户将用于在本地 AD 中读取和写入对象和属性，以便持续同步操作。</li><li>将对所选同步选项的适当权限和访问控制设置分配到上述帐户和 AD 。</li>
AD FS 服务器|对于列表中的每个服务器，如果运行向导的用户的登录凭据权限不足，因而无法连接，则向导将会收集凭据|域管理员|安装和配置 AD FS 服务器角色。|
Web 应用程序代理服务器 |对于列表中的每个服务器，如果运行向导的用户的登录凭据权限不足，因而无法连接，则向导将会收集凭据|目标计算机上的本地管理员。|安装和配置 WAP 服务器角色。
代理信任凭据 |联合身份验证服务信任凭据（代理用来注册 FS 信任证书的凭据） |作为 AD FS 服务器本地管理员的域帐户|初始注册 FS WAP 信任证书
“AD FS 服务帐户”页上的“使用域用户帐户选项”|AD 用户帐户凭据|域用户|提供了其凭据的 AD 用户帐户将用作 AD FS 服务的登录帐户。



<br> <br>
## 特定方案所需权限

方案 |权限
------------- | ------------- |
密码同步| <li>复制目录更改。</li> <li>复制所有目录更改。</li>
Exchange 混合部署|请参阅 [Office 365 Exchange 混合 AAD 同步写回属性和权限](https://msdn.microsoft.com/library/azure/dn757602.aspx#exchange)。
密码写回 | <li>更改密码</li><li>重置密码</li>
用户、组和设备写回|对你想要写回的目录对象和属性的写入权限
单一登录和 AD FS| 联合服务器所在的域中的域管理员权限。

<br> <br>
## Azure AD Connect 创建的帐户的汇总。



创建的帐户 |分配的权限 | 用途
------------- | ------------- |------------- |
用于同步的 Azure AD 帐户|专用目录同步角色|进行中的同步操作（Azure AD MA 帐户）
快速设置：用于同步的 AD 帐户|同步和密码同步时对目录所需的读取/写入权限|进行中的同步操作（Azure AD MA 帐户）
快速设置：同步服务登录帐户 | 运行向导的用户的登录凭据|同步服务登录帐户
自定义设置：同步服务登录帐户 |不可用|同步服务登录帐户
AD FS：GMSA 帐户 (aadcsvc$)|域用户|FS 服务登录帐户


**其他资源**



* [用于密码同步的权限](https://msdn.microsoft.com/library/azure/dn757602.aspx#psynch)
* [用于 Exchange 混合部署的权限](https://msdn.microsoft.com/library/azure/dn757602.aspx#exchange)
* [用于密码写回的权限](https://msdn.microsoft.com/library/azure/dn757602.aspx#pwriteback)
* [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)
* [MSDN 上的 Azure AD Connect](active-directory-aadconnect)
 

<!---HONumber=67-->