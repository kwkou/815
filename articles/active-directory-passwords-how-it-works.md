<properties 
	pageTitle="工作原理：Azure AD 密码管理 |Windows Azure" 
	description="了解 Azure AD 密码管理的不同组件，其中包括用户注册、重置和更改其密码的位置以及管理员配置、报表本地 Active Directory 密码并启用对其的管理的位置。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory" 
	ms.date="06/08/2015" 
	wacn.date="08/29/2015"/>

# 密码管理的工作原理
Azure Active Directory 中的密码管理由下述的几个逻辑组件构成。请单击每个链接以了解有关该组件的详细信息。

- [**密码管理配置门户**](#password-management-configuration-portal) – 管理员可以通过在 [Azure 管理门户](https://manage.windowsazure.cn)中导航至其目录的“配置”选项卡来控制其租户中密码管理方式的不同方面。
- [**用户注册门户**](#user-registration-portal) – 用户可以通过此 web 门户自行进行注册以重置密码。
- [**用户密码重置门户**](#user-password-reset-portal) – 用户可以根据由管理员控制的密码重置策略，使用多个不同的质询来重置自己的密码
- [**用户密码更改门户**](#user-password-change-portal) – 用户可以通过使用此 web 门户输入其旧密码，然后选择一个新的密码从而随时更改自己的密码
- [**密码管理报告**](#password-management-reports) – 管理员可以通过在 [Azure 管理门户](https://manage.windowsazure.cn)中导航至其目录的“报表”选项卡的“活动报表”部分来查看和分析其租户中的密码重置和注册活动
- [**Azure AD Connect 的密码写回组件**](#password-writeback-component-of-azure-ad-connect) - 管理员可以选择在安装 Azure AD Connect 时启用“密码写回”功能，以便从云中管理联合或密码同步用户的密码。

## <a name="password-management-configuration-portal">密码管理配置门户
可通过使用 [Azure 管理门户](https://manage.windowsazure.cn)导航至目录的**配置**选项卡中的**用户密码重置策略**为特定目录配置“密码管理”策略。可从该配置页面控制组织中密码管理方式的许多方面，其中包括：

- 为目录中的所有用户启用和禁用密码重置
- 设置用户重置其密码所必须通过的质询数量（一个或两个）
- 为组织中的用户设置你希望其通过的特定类型的质询，可选类型如下：
 - 移动电话（通过短信或语音呼叫接收验证码）
 - 办公室电话（语音呼叫）
 - 备用电子邮件（电子邮件验证码）
 - 安全问题（基于知识的身份验证）
- 设置用户为使用安全问题身份验证方法所必须注册的问题的数量（仅当启用了安全问题时才可见）
- 设置用户为使用安全问题身份验证方法在重置过程中必须提供的问题的数量（仅当启用了安全问题时才可见）
- 定义用户为使用安全问题身份验证方法可能选择注册使用的自定义安全问题（仅当启用了安全问题时才可见）
- 当用户转到 [http://myapps.microsoft.com](http://myapps.microsoft.com) 位置的应用程序“访问面板”时，要求用户注册以进行密码重置。
- 要求用户在经过一定天数（天数可配置）后再次确认其之前注册的数据（仅当启用了强制注册时才可见）
- 提供用户在重置密码过程中遇到问题时将要向其显示的自定义帮助服务台电子邮件或 URL
- 启用或禁用“密码写回”功能（如果已使用 AAD Connect 部署了“密码写回”）
- 查看“密码写回”代理的状态（如果已使用 AAD Connect 部署了“密码写回”）
- 用户完成密码重置时向其发送电子邮件通知（位于 [Azure 管理门户](https://manage.windowsazure.cn)的**通知**部分）
- 其他管理员完成密码重置时向管理员发送电子邮件通知（位于 [Azure 管理门户](https://manage.windowsazure.cn)的**通知**部分）
- 通过使用租户品牌自定义功能在用户密码重置门户和密码重置电子邮件中标注你所在组织的徽标和名称（在 [Azure 管理门户](https://manage.windowsazure.cn)的“目录属性”部分）

若要详细了解有关在你的组织中配置“密码管理”的详细信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started)。

##<a name="user-registration-portal"></a>用户注册门户
必须先使用正确的身份验证数据对用户的云用户账户进行了更新以确保用户可通过由其管理员定义的适当数量的密码重置质询之后，用户才能够使用密码重置。管理员还可以通过使用 Azure web 门户或 Office web 门户、DirSync / Azure AD Connect 或 Windows PowerShell 代表其用户定义此身份验证信息。

但是，如果你更希望让自己的用户注册其自己的数据，我们还提供一个网页，用户可转至该网页以提供此信息。此页将允许用户根据其组织中已启用的密码重置策略指定身份验证信息。此数据一旦通过验证，则将存储在用户的云用户帐户中，以便将来用于帐户恢复。以下是注册门户的外观：

  ![][001]

有关详细信息，请参阅[入门：Azure AD 密码管理](active-directory-passwords-getting-started)和[最佳实践：Azure AD 密码管理](/documentation/articles/active-directory-passwords-best-practices)。

##<a name="user-password-reset-portal"></a>用户密码重置门户
一旦启用了自助服务密码重置、设置了组织的自助服务密码重置策略并确保你的用户在目录中具有相应的联系人数据，组织中的用户便能够从任何使用“工作”或“学校”帐户登录的网页（如 [portal.partner.microsoftonline.cn](https://portal.partner.microsoftonline.cn)）自动重置其自己的密码。用户在这类网页上将看到一个**无法访问你的帐户？**链接。

  ![][002]

单击此链接将启动自助服务密码重置门户。

  ![][003]

若要了解有关用户可如何重置其自己的密码的详细信息，请参阅[入门：Azure AD 密码管理](active-directory-passwords-getting-started)。

##<a name="user-password-change-portal"></a>用户密码更改门户
如果用户想要更改其自己的密码，可通过在任何时候使用密码更改门户来实现这一操作。用户可以通过“访问面板”配置文件页面或通过从 Office 365 应用程序内单击“更改密码”链接来访问密码更改门户。当用户密码过期时，用户登录时也将被要求自动更改其密码。

  ![][004]

在这两种情况下，如果已启用“密码写回”功能且该用户已经过联合或密码同步，这些更改的密码将写回到你的本地 Active Directory。下面是密码更改门户的外观：

  ![][005]

若要了解有关用户可如何更改其自己的本地 Active Directory 密码的详细信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started)。

##<a name="password-writeback-component-of-azure-ad-connect"></a>Azure AD Connect 的“密码写回”组件
如果组织中用户的密码源自你的本地环境（通过联合或是密码同步），可以安装最新版本的 Azure AD Connect 以便可以从云端直接更新这些密码。这意味着当你的用户忘记或是想要修改其 AD 密码，他们可以直接通过 web 实现此操作。以下是 Azure AD Connect 安装向导中“密码写回”的位置：

  ![][007]

有关 Azure AD Connect 的详细信息，请参阅[入门：Azure AD Connect](active-directory-aadconnect)。有关“密码写回”功能的详细信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started)。


<br/> <br/> <br/>

**其他资源**


* [什么是密码管理](/documentation/articles/active-directory-passwords)
* [密码管理入门](/documentation/articles/active-directory-passwords-getting-started)
* [自定义密码管理](/documentation/articles/active-directory-passwords-customize)
* [密码管理最佳实践](/documentation/articles/active-directory-passwords-best-practices)
* [密码管理常见问题](/documentation/articles/active-directory-passwords-faq)
* [密码管理疑难解答](/documentation/articles/active-directory-passwords-troubleshoot)
* [了解详细信息](/documentation/articles/active-directory-passwords-learn-more)
* [MSDN 上的密码管理](https://msdn.microsoft.com/zh-cn/library/azure/dn510386.aspx)



[001]: ./media/active-directory-passwords-how-it-works/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-how-it-works/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-how-it-works/003.jpg "Image_003.jpg"
[004]: ./media/active-directory-passwords-how-it-works/004.jpg "Image_004.jpg"
[005]: ./media/active-directory-passwords-how-it-works/005.jpg "Image_005.jpg"
[006]: ./media/active-directory-passwords-how-it-works/006.jpg "Image_006.jpg"
[007]: ./media/active-directory-passwords-how-it-works/007.jpg "Image_007.jpg"
 

<!---HONumber=67-->