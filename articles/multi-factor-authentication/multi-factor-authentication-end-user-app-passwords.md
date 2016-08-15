<properties 
	pageTitle="Azure MFA 中的应用密码是什么？" 
	description="此页面将帮助用户了解什么是应用密码，以及在 Azure MFA 中，应用密码有什么作用。" 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtland"/>

<tags 
	ms.service="multi-factor-authentication" 
	ms.date="05/12/2016" 
	wacn.date="04/13/2016"/>




# Azure 多重身份验证中的应用密码是什么？

某些非浏览器应用（例如使用 Exchange Active Sync 的 Apple 本机电子邮件客户端）目前不支持多重身份验证。多重身份验证 是按用户启用的。这意味着，如果为某个用户启用了多重身份验证，而该用户尝试使用非浏览器应用将会失败。使用应用密码可以避免这种情况。

>[AZURE.NOTE]适用于 Office 2013 客户端的现代身份验证
>
> Office 2013 客户端（包括 Outlook）现在支持新的身份验证协议，并且可以启用对多重身份验证的支持。这意味着一旦启用后，就不需要对 Office 2013 客户端使用应用密码。有关详细信息，请参阅 [Office 2013 现代身份验证公共预览版发布声明](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)。
 
## 如何使用应用密码

以下是有关如何使用应用密码的一些要点。

- 实际的密码将自动生成，并非由用户提供。这是因为自动生成的密码会使攻击者更难进行猜测，因而更安全。
- 目前，每个用户有 40 个密码的限制。如果在达到该限制后尝试创建密码，系统会提示你删除一个现有的应用密码，以便创建新密码。
- 建议按设备而不是按应用程序创建应用密码。例如，可以为笔记本电脑创建一个应用密码，并将该应用密码用于该笔记本电脑上的所有应用程序。
- 当你首次登录时，系统将为你提供一个应用密码。如果需要更多的应用密码，你可以创建。
 
<center>![Setup](./media/multi-factor-authentication-end-user-app-passwords/app.png)</center>

创建一个应用密码后，你可使用此密码来取代这些非浏览器应用中的原始密码。例如，如果你在手机上使用多重身份验证和 Apple 本机电子邮件客户端。你可使用应用密码，以便绕过多重身份验证并继续工作。

### 创建应用密码
在你首次登录期间，系统将为你提供一个可用的应用密码。另外，以后你还可以创建应用密码。具体的操作方式取决于你使用多重身份验证的方式。请选择最适用你自己的方式。

如何使用 Multi-Factor Authentication|说明
:------------- | :------------- | 
[我要将它用于 Office 365](#creating-and-deleting-app-passwords-with-office-365)| 这意味着你要通过 Office 365 门户创建应用密码。
[我不知道](#creating-and-deleting-app-passwords-with-myapps-portal)|这意味着你要通过 [https://myapps.microsoft.com](https://myapps.microsoft.com) 创建应用密码
[我要将它用于 Azure](#create-app-passwords-in-the-azure-portal)| 这意味着你要通过 Azure 门户创建应用密码。




 

<!---HONumber=69-->