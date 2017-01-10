<properties
    pageTitle="Azure MFA 中的应用密码是什么？"
    description="此页面将帮助用户了解什么是应用密码，以及在 Azure MFA 中，应用密码有什么作用。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="pblachar" />
<tags
    ms.assetid="345b757b-5a2b-48eb-953f-d363313be9e5"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/23/2016"
    wacn.date="01/10/2017"
    ms.author="kgremban" />  


# Azure 多重身份验证中的应用密码是什么？
某些非浏览器应用（例如使用 Exchange Active Sync 的 Apple 本机电子邮件客户端）目前不支持多重身份验证。多重身份验证是按用户启用的。这意味着，如果为某个用户启用了多重身份验证，而该用户尝试使用非浏览器应用，则会失败。使用应用密码可以避免这种情况。

> [AZURE.NOTE]
适用于 Office 2013 客户端的现代身份验证
>
> Office 2013 客户端（包括 Outlook）现在支持新的身份验证协议，并且可以启用对多重身份验证的支持。这意味着一旦启用后，就不需要对 Office 2013 客户端使用应用密码。有关详细信息，请参阅 [Office 2013 现代身份验证公共预览版发布声明](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)。
>
>

## 如何使用应用密码
以下是有关如何使用应用密码的一些要点。

- 实际的密码将自动生成，并非由用户提供。这是因为自动生成的密码会使攻击者更难进行猜测，因而更安全。
- 目前，每个用户有 40 个密码的限制。如果在达到该限制后尝试创建密码，系统会提示你删除一个现有的应用密码，以便创建新密码。
- 建议按设备而不是按应用程序创建应用密码。例如，可以为笔记本电脑创建一个应用密码，并将该应用密码用于该笔记本电脑上的所有应用程序。
- 首次登录时，系统将为你提供一个应用密码。如果需要更多的应用密码，你可以创建。

![设置](./media/multi-factor-authentication-end-user-app-passwords/app.png)

创建一个应用密码后，你可使用此密码来取代这些非浏览器应用中的原始密码。例如，如果你在手机上使用多重身份验证和 Apple 本机电子邮件客户端。你可使用应用密码，以便绕过多重身份验证并继续工作。

## 创建和删除应用密码
在你首次登录期间，系统将为你提供一个可用的应用密码。另外，以后你还可以创建和删除应用密码。具体的操作方式取决于你使用多重身份验证的方式。请选择最适用你自己的方式。

| 如何使用多重身份验证 | 说明 |
|:--- |:--- |
| [我要将它用于 Office 365](#creating-and-deleting-app-passwords-with-office-365) |这意味着你要通过 Office 365 门户创建应用密码。 |
| [我不知道](#creating-and-deleting-app-passwords-with-myapps-portal) |这意味着你要通过 [https://myapps.microsoft.com](https://myapps.microsoft.com) 创建应用密码 |


## 使用 Office 365 创建和删除应用密码
如果你在 Office 365 上使用多重身份验证，则需要通过 Office 365 门户创建和删除应用密码。

### 在 Office 365 门户中创建应用密码
- - -
1. 登录 [Office 365 门户](https://login.microsoftonline.com/)。
2. 在右上角选择小组件并选择“Office 365 设置”。
3. 单击“其他安全性验证”。
4. 在右侧，单击“更新用于帐户安全性的电话号码”链接。
   ![设置](./media/multi-factor-authentication-end-user-manage/o365a.png)
5. 随后你将转到可以更改设置的页面。
   ![云](./media/multi-factor-authentication-end-user-manage/o365b.png)
6. 在顶部的“其他安全性验证”旁边，单击“应用密码”。
7. 单击“创建”。
   ![云](./media/multi-factor-authentication-end-user-app-passwords/apppass.png)
8. 输入应用密码的名称，然后单击“下一步”。
   ![创建应用密码](./media/multi-factor-authentication-end-user-app-passwords/create1.png)
9. 将应用密码复制到剪贴板，然后将它粘贴到你的应用。
   ![创建应用密码](./media/multi-factor-authentication-end-user-app-passwords/create2.png)

### 使用 Office 365 门户删除应用密码
- - -
1. 登录 [Office 365 门户](https://login.microsoftonline.com/)。
2. 在右上角选择小组件并选择“Office 365 设置”。
3. 单击“其他安全性验证”。
4. 在右侧，单击“更新用于帐户安全性的电话号码”链接。
   ![设置](./media/multi-factor-authentication-end-user-manage/o365a.png)
5. 随后你将转到可以更改设置的页面。
   ![云](./media/multi-factor-authentication-end-user-manage/o365b.png)
6. 在顶部的“其他安全性验证”旁边，单击“应用密码”。
7. 在要删除的应用密码旁边，单击“删除”。![删除应用密码](./media/multi-factor-authentication-end-user-app-passwords/delete1.png)
8. 单击“是”确认删除。
   ![确认删除](./media/multi-factor-authentication-end-user-app-passwords/delete2.png)
9. 删除应用密码后，可以单击“关闭”。
   ![关闭](./media/multi-factor-authentication-end-user-app-passwords/delete3.png)

## 使用 Myapps 门户创建和删除应用密码。
如果你不确定多重身份验证的使用方式，你始终可以通过 myapps 门户创建和删除应用密码。

### 使用 Myapps 门户创建应用密码
1. 登录 [https://myapps.microsoft.com](https://myapps.microsoft.com)
2. 在顶部选择配置文件。
3. 选择“其他安全性验证”。
   ![云](./media/multi-factor-authentication-end-user-manage/myapps1.png)
4. 随后你将转到可以更改设置的页面。
   ![设置](./media/multi-factor-authentication-end-user-manage/proofup.png)
5. 在顶部的“其他安全性验证”旁边，单击“应用密码”。
6. 单击“创建”。
   ![创建应用密码](./media/multi-factor-authentication-end-user-app-passwords/create3.png)
7. 输入应用密码的名称，然后单击“下一步”。
   ![创建应用密码](./media/multi-factor-authentication-end-user-app-passwords/create1.png)
8. 将应用密码复制到剪贴板，然后将它粘贴到你的应用。
   ![创建应用密码](./media/multi-factor-authentication-end-user-app-passwords/create2.png)

### 使用 Myapps 门户删除应用密码
1. 登录 [https://myapps.microsoft.com](https://myapps.microsoft.com)
2. 在顶部选择配置文件。
3. 选择“其他安全性验证”。
   ![云](./media/multi-factor-authentication-end-user-manage/myapps1.png)
4. 随后你将转到可以更改设置的页面。
   ![设置](./media/multi-factor-authentication-end-user-manage/proofup.png)
5. 在顶部的“其他安全性验证”旁边，单击“应用密码”。
6. 在要删除的应用密码旁边，单击“删除”。
   ![删除应用密码](./media/multi-factor-authentication-end-user-app-passwords/delete1.png)
7. 单击“是”确认删除。
   ![确认删除](./media/multi-factor-authentication-end-user-app-passwords/delete2.png)
8. 删除应用密码后，可以单击“关闭”。
   ![关闭](./media/multi-factor-authentication-end-user-app-passwords/delete3.png)

<!---HONumber=Mooncake_0103_2017-->
