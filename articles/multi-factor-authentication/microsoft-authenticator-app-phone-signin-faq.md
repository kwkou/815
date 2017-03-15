<properties
    pageTitle="Microsoft 验证器手机登录 - Azure 和 Microsoft 帐户 | Azure"
    description="无需键入密码，使用手机登录到 Microsoft 帐户。本文解答有关此功能的常见问题。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="librown" />
<tags
    ms.assetid=""
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/02/2017"
    wacn.date="03/14/2017"
    ms.author="kgremban" />  


# 使用手机而不是密码登录

Microsoft 验证器应用可在输入密码后执行多重身份验证，帮助确保帐户的安全性。但你是否知道它能完全取代 Microsoft 个人帐户密码？

iOS 和 Android 设备提供此功能，此功能适用于 Microsoft 个人帐户。

## 工作原理

在登录 Microsoft 帐户时，许多人会将 Microsoft 验证器应用用于多重身份验证。键入密码，然后转到应用以批准通知或获取验证码。使用手机登录时，会跳过密码，并在手机上完成所有身份验证。这与多重身份验证的原理相同，手机登录将要求你提供你知道的信息和拥有的物品。

## 如何入门

若要使用手机登录到 Microsoft 个人帐户，请执行以下步骤：

1. 为帐户启用手机登录。

  - 如果没有 Microsoft 验证器应用，请根据 [Microsoft 验证器页](/documentation/articles/microsoft-authenticator-app-how-to/)中步骤安装和添加 Microsoft 个人帐户。新添加的帐户将自动启用，因此可以顺利完成后续操作。

2. Microsoft 向手机发送通知。确认该通知可登录到你的帐户。

## 常见问题 

### 为何使用手机登录比键入密码更安全？  

当今，大部分人都是使用用户名和密码登录到网站或应用。遗憾的是，密码常常会丢失、被盗或被黑客猜出。当你设置用于登录的 Microsoft 验证器应用时，我们将在你的手机上生成一个可以解锁帐户的密钥。我们将通过已在手机上使用的 PIN 或生物识别来保护此密钥。使用手机登录时，此密钥通过以下两个因素安全地证明你的身份：手机本身和解锁手机的能力。

所用的密钥与在 Windows Hello 和 FIDO Alliance UAF 规范中使用的密钥类似。生物数据仅用于在本地保护密钥，绝不会发送或存储到云中。
 
### 在哪种场合下可以用手机取代密码，或者仍然需要输入密码？  

目前，手机登录功能仅适用于由 Microsoft 个人帐户提供支持的 Web 应用和服务、使用 Microsoft 个人帐户的 iOS 或 Android 应用以及使用 Microsoft 个人帐户的 Windows 10 应用。登录到这些网站或应用时，输入密码的页面上会显示带有“改用应用”字样的链接。

目前，无法使用手机登录解锁 Windows 电脑、XBOX 或任何 Microsoft 应用的桌面版（如 Office 应用）。
 
### 手机登录是否可以取代多重身份验证？ 是否应将其关闭？   

有时需要关闭。我们正努力扩大手机登录的适用范围，但目前在 Microsoft 生态系统中仍有不支持它的场合。在这些场合中，仍需要使用多重身份验证进行安全登录。因此，不应关闭帐户的多重身份验证。
 
### 如果启用帐户的多重身份验证，是否必须确认两条通知？

不需要。使用手机登录到 Microsoft 帐户被视为多重身份验证。可通过解锁手机再确认通知来证明身份，无需键入密码再确认通知。我们不会发送第二条要求确认的通知。

### 如果手机遗失或未随身携带，应如何访问我的帐户？  

你可以联系管理员，在 “Multi-factor authentication” 配置页面上，选中你的用户名，然后点击 “Manage user settings”， 在弹出的页面上勾选 “Require selected users to provide contact methods again”， 这样你就可以在登录时使用其他验证方式或更换手机号码。

### 是否可以使用该应用登录到我的所有 Microsoft 帐户？   
此功能目前仅适用于 Microsoft 个人帐户。
 
### 是否可以使用手机登录到我的电脑？  
对于电脑，建议使用 Windows 10 中的 Windows Hello，通过面部、指纹或 PIN 进行登录。
 
### 是否可以使用 Windows Phone 登录？  
目前，没有为 Windows Phone 上的 Microsoft 验证器开发此功能。

## 后续步骤
如果尚未下载 Microsoft 验证器应用，请查看相关文档。该应用适用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)。在适用于 [Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [iOS](http://go.microsoft.com/fwlink/?Linkid=825073) 的 Microsoft 验证器应用中可以使用手机登录。

如有关于该应用的一般性问题，请参阅 [Microsoft 验证器常见问题](/documentation/articles/microsoft-authenticator-app-faq/)

<!---HONumber=Mooncake_0306_2017-->
