<properties 
	pageTitle="Azure 多重身份验证 - 后续步骤" 
	description="这个有关 Azure 多重身份验证的页面介绍了使用 MFA 可以执行的后续步骤。其中包括报告、欺诈警报、一次性跳过、自定义语音消息、缓存，受信任的 IP 和应用密码。" 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>  

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/23/2016"
	wacn.date="11/16/2016"
	ms.author="kgremban"/>

# 配置 Azure 多重身份验证

在启动并运行 Azure 多重身份验证后，可以参考以下文章来对其进行管理。本文涵盖各种主题，足以让用户充分利用 Azure 多重身份验证。请注意，并非所有版本的 Azure 多重身份验证都提供这些功能。

以下某些功能的配置可在 Azure 多重身份验证管理门户中找到。可以通过两种不同的方式访问此门户，这两种方式都是通过 Azure 门户预览进行的。第一种方式是管理 Multi-Factor Auth 提供程序（如果使用基于使用量的 MFA）。第二种方式是通过 MFA 服务设置。第二个选项需要 Multi-Factor Auth 提供程序或 Azure MFA、Azure AD Premium 或 Enterprise Mobility Suite 许可证。

若要通过 Azure Multi-Factor Auth 提供程序访问 MFA 管理门户，请以管理员身份登录 Azure 门户预览，然后选择“Active Directory”选项。单击“Multi-Factor Auth 提供程序”选项卡，然后单击底部的“管理”按钮。

若要通过“MFA 服务设置”页访问 MFA 管理门户，请以管理员身份登录 Azure 门户预览，然后选择“Active Directory”选项。单击你的目录，然后单击“配置”选项卡。在“多重身份验证”部分下，单击“管理服务设置”。在“MFA 服务设置”页底部，单击“转到门户”链接。


功能| 说明| 涵盖的内容
:------------- | :------------- | :------------- | 
[应用密码](#app-passwords)|应用密码允许非 MFA 感知应用程序跳过多重身份验证并继续工作。|有关应用密码的信息。
[可选择验证方法](#selectable-verification-methods)|允许你选择可供用户使用的身份验证方法。|有关启用或禁用特定身份验证方法（如呼叫或短信）的信息。


 
## 应用密码

在 Office 2010 或更低版本和 Apple Mail 等某些应用中，无法使用多重身份验证。若要使用这些应用，你需要使用“应用密码”来替换传统密码。应用密码可让应用程序跳过多重身份验证并继续工作。

>[AZURE.NOTE] 适用于 Office 2013 客户端的现代身份验证
>
> Office 2013 客户端（包括 Outlook）现在支持新的身份验证协议，并且可以启用对多重身份验证的支持。这意味着一旦启用后，就不需要对 Office 2013 客户端使用应用密码。有关详细信息，请参阅 [Office 2013 现代身份验证公共预览版发布声明](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)。



### 有关应用密码的重要事项

下面是你应该知道的有关应用密码的重要事项列表。

身份验证体验|对于基于浏览器的应用|对于非基于浏览器的应用
:------------- | :------------- | :------------- 
|<ul><li>身份验证的第一个因素在本地执行</li><li>第二个因素是由云标识执行的基于电话的方法。</li>|<ul><li>管理员和用户可以使用应用密码来登录。

- 用户可能有多个应用密码，从而增加了密码被盗的可能性。由于应用密码很难记住，因此用户可能会将密码写下来。不建议这么做并应阻止这么做，因为使用应用密码登录只需进行单重身份验证。
- 缓存密码并在本地方案中使用该密码的应用可能会启动失败，因为超出组织 ID 范畴的应用密码是不可识别的。一个示例是 Exchange 电子邮件，这些电子邮件位于本地但存档邮件位于云中。同一密码将无法正常工作。
- 实际的密码将自动生成，并非由用户提供。这是因为自动生成的密码会使攻击者更难进行猜测，因而更安全。
- 目前，每个用户有 40 个密码的限制。系统将提示你删除旧应用密码（现有的应用密码之一），以便创建新的应用密码。
- 对用户的帐户启用多重身份验证后，可以在大多数非浏览器客户端（例如 Outlook 和 Lync）上使用应用密码，但无法通过 Windows PowerShell 等非浏览器应用程序使用应用密码执行管理操作，即使该用户具有管理帐户。请确保使用强密码创建服务帐户以运行 PowerShell 脚本，而不要为该帐户启用多重身份验证。

>[AZURE.WARNING]  在客户端与同时与本地和云自动发现终结点通信的混合环境中，应用密码不起作用。
>
>请注意，在客户端与同时与本地和云自动发现终结点通信的混合环境中，应用密码不起作用，因为需要使用域密码在本地进行身份验证，使用应用密码在云中进行身份验证。


### 应用密码命名指南
建议应用密码名称应能反映使用应用密码的设备。例如，如果你有一台安装了非浏览器应用（如 Outlook、Word 和 Excel）的便携式计算机，则只需创建一个名为 Laptop 的应用密码，即可在所有这些应用程序中使用该应用密码。虽然你可以为所有这些应用程序创建单独的密码，但不建议这样做。建议的方式是每个设备使用一个应用密码。


<center>![Cloud](./media/multi-factor-authentication-whats-next/naming.png)</center>


### 联合 (SSO) 应用密码
Azure AD 支持与本地 Windows Server Active Directory 域服务 (AD DS) 联合。如果你的组织与 Azure AD 联合 (SSO) 并且你要使用 Azure 多重身份验证，则以下是你在使用应用密码时应了解的重要信息。这仅适用于联合 (SSO) 客户。

- 应用密码由 Azure AD 进行验证，从而绕过了联合。仅在设置应用密码时，才会主动使用联合。
- 与被动流程不同，对于联合 (SSO) 用户，我们从不转到标识提供者 (IdP)。密码将存储在组织 ID 中。如果用户离开公司，则该信息必须通过 DirSync 实时流到组织 ID 中。帐户禁用/删除可能需要长达 3 小时才能同步，从而延迟了 Azure AD 中应用密码的禁用/删除。
- 应用密码不遵循“本地客户端访问控制”设置
- 本地身份验证日志记录/审核功能不可用于应用密码
- 对于 Microsoft Lync 2013 客户端，更多最终用户培训是必需的。有关必需的步骤，请参阅如何将电子邮件中的密码更改为应用密码。
- 在对客户端使用多重身份验证时，某些先进的体系结构设计可能需要将组织用户名和密码与应用密码结合使用，具体取决于进行身份验证的位置。对于针对本地基础结构进行身份验证的客户端，你会使用组织用户名和密码。对于针对 Azure AD 进行身份验证的客户端，你会使用应用密码。

例如，假设你有一个由下列内容组成的体系结构：

- 你要将 Active Directory 的本地实例与 Azure AD 联合
- 你要使用 Exchange Online
- 你要使用本地专用的 Lync
- 你要使用 Azure 多重身份验证


![验证](./media/multi-factor-authentication-whats-next/federated.png)

 在这些情况下，你必须执行以下操作：

- 登录到 Lync 时，使用你组织的用户名和密码。
- 当尝试通过连接到 Exchange Online 的 Outlook 客户端访问通讯簿时，请使用应用密码。

### 允许用户创建应用密码
默认情况下，用户无法创建应用密码。为此，必须启用此功能。若要允许用户创建应用密码，请使用以下过程。

#### 允许用户创建应用密码



1. 登录到 Azure 经典管理门户。
2. 在左侧单击“Active Directory”。
3. 在“目录”下单击要为其启用此功能的用户的目录。
4. 在顶部单击“用户”。
5. 在页面底部，单击“管理 Multi-Factor Auth”。此时将打开“多重身份验证”页。
6. 在“多重身份验证”页的顶部，单击“服务设置”。
7. 确保选中了“允许用户创建用于登录到非浏览器应用程序的应用密码”旁边的单选按钮。


![创建应用密码](./media/multi-factor-authentication-whats-next/trustedips3.png)

### 创建应用密码
用户可以在最初注册时创建应用密码。注册过程结束时，系统将提供一个选项用于创建应用密码。

此外，用户日后还可以通过在 Azure 门户预览和 Office 365 门户中更改设置来创建应用密码。

### 在 Office 365 门户中创建应用密码
--------------------------------------------------------------------------------


1. 登录到 Office 365 门户
2. 在右上角选择设置小组件
3. 在左侧选择“其他安全性验证”
4. 在右侧，选择“更新用于帐户安全性的电话号码”
5. 在验证页的顶部选择应用密码
6. 单击“创建”
7. 输入应用密码的名称，然后单击“下一步”
8. 将应用密码复制到剪贴板，然后将它粘贴到你的应用。

<center>![Cloud](./media/multi-factor-authentication-whats-next/security.png)</center>



### 在没有 Office 365 或 Azure 订阅的情况下创建应用密码
--------------------------------------------------------------------------------
1. 登录到 [https://myapps.microsoft.com](https://myapps.microsoft.com)
2. 在顶部选择配置文件。
3. 单击你的用户名并选择“其他安全性验证”。
5. 在验证页的顶部选择应用密码
6. 单击“创建”
7. 输入应用密码的名称，然后单击“下一步”
8. 将应用密码复制到剪贴板，然后将它粘贴到你的应用。

![应用密码](./media/multi-factor-authentication-whats-next/myapp.png)

## 可选择验证方法
现在，可以选择当用户使用 Azure 多重身份验证时供用户使用的身份验证方法。此功能以前只提供了本地服务器版本。下表提供了可为用户启用或禁用的各种身份验证方法的简要概述。

方法|说明
:------------- | :------------- | 
[拨打电话](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-phone/)| 向身份验证电话拨打自动语音呼叫。用户接听电话，并按电话键盘上的 # 进行身份验证。此电话号码将不会同步到本地 Active Directory。
[向手机发送短信](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-phone/)|向用户发送包含验证码的短信。系统会提示用户使用验证码回复短信或在登录界面中输入验证码。
[通过移动应用发送通知](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-app/)|在此模式下，Microsoft Authenticator 应用可防止对帐户进行未经授权的访问并停止欺诈性事务。此功能是使用推送到你的手机或已注册设备上的推送通知来完成的。你可以直接查看通知，如果该通知是合法的，请点击“验证”。否则，你可以选择“拒绝”或选择拒绝并报告欺诈性通知。有关报告欺诈性通知的信息，请参阅“如何针对多重身份验证使用拒绝并报告欺诈功能”。</br></br>Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。|
[通过移动应用发送验证码](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-app/)|在此模式下，Microsoft Authenticator 应用可用作生成 OATH 验证码所需的软件令牌。然后可以将此验证码与用户名和密码一起输入，进行第二种形式的身份验证。</li><p> Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

### 如何启用/禁用身份验证方法

1. 登录到 Azure 经典管理门户。
2. 在左侧单击“Active Directory”。
3. 在 Active Directory 中，单击要对其启用或禁用身份验证方法的目录。
4. 在选择的目录上，单击“配置”。
5. 在“多重身份验证”部分中，单击“管理服务设置”。
6. 在“服务设置”页上的验证选项下，选择/取消选择你要使用的选项。</br></br>![验证选项](./media/multi-factor-authentication-whats-next/authmethods.png)
9. 单击“保存”。
10. 单击“关闭”。

<!---HONumber=Mooncake_1107_2016-->
