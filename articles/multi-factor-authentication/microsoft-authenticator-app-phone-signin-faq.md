<properties
    pageTitle="Microsoft Authenticator 手机登录 - Azure 和 Microsoft 帐户 | Azure"
    description="使用手机登录 Microsoft 帐户，而不是键入密码。 本文解答有关此功能的常见问题。"
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
    ms.date="04/02/2017"
    wacn.date="05/15/2017"
    ms.author="kgremban"
    ms.custom="end-user"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="3ff18e6f95d8bbc27348658bc5fce50c3320cf0a"
    ms.openlocfilehash="7248c690f466e1c13bd4d60003627c165c92f91e"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/15/2017" />

# <a name="sign-in-with-your-phone-not-your-password"></a>使用手机而不是密码登录

Microsoft 验证器应用可在输入密码后执行多重身份验证，帮助确保帐户的安全性。 但你是否知道它能完全取代 Microsoft 个人帐户密码？ 

iOS 和 Android 设备提供此功能，此功能适用于 Microsoft 个人帐户。 

## <a name="how-it-works"></a>工作原理

在登录 Microsoft 帐户时，许多人会将 Microsoft 验证器应用用于多重身份验证。 键入密码，然后转到应用以批准通知或获取验证码。 使用手机登录时，会跳过密码，并在手机上完成所有身份验证。 这与多重身份验证的原理相同，手机登录将要求你提供你知道的信息和拥有的物品。 拥有的物品仍是手机，但此时我们会要求输入手机的 PIN 或生物识别密钥作为知道的信息。 

## <a name="how-to-get-started"></a>如何入门

若要使用手机登录到 Microsoft 个人帐户，请执行以下步骤： 

1. 为帐户启用手机登录。 

  - 如果还没有 Microsoft Authenticator 应用，请按照 [Microsoft Authenticator 页](/documentation/articles/microsoft-authenticator-app-how-to/)中步骤安装和添加 Microsoft 个人帐户。 新添加的帐户将自动启用，因此请放心执行后续操作。

2. Microsoft 向手机发送通知。确认该通知可登录到你的帐户。

## <a name="faq"></a>常见问题 

### <a name="how-is-signing-in-with-my-phone-more-secure-than-typing-a-password"></a>为何使用手机登录比键入密码更安全？  

当今，大部分人都是使用用户名和密码登录到网站或应用。  遗憾的是，密码常常会丢失、被盗或被黑客猜出。 当你设置用于登录的 Microsoft 验证器应用时，我们将在你的手机上生成一个可以解锁帐户的密钥。 我们将通过已在手机上使用的 PIN 或生物识别来保护此密钥。  使用手机登录时，此密钥通过以下两个因素安全地证明你的身份：手机本身和解锁手机的能力。 

所用的密钥与在 Windows Hello 和 FIDO Alliance UAF 规范中使用的密钥类似。 生物数据仅用于在本地保护密钥，绝不会发送或存储到云中。 
 
### <a name="where-can-i-use-my-phone-to-replace-my-password-and-where-would-i-still-need-the-password"></a>在哪种场合下可以用手机取代密码，或者仍然需要输入密码？  

目前手机登录功能仅适用于由 Microsoft 个人帐户提供支持的 Web 应用和服务、使用 Microsoft 个人帐户的 iOS 或 Android 应用以及使用 Microsoft 个人帐户的 Windows 10 应用。 登录这些网站或应用时，通常输入密码的页面上会有显示为“改用应用”的链接。 

目前，无法使用手机登录解锁 Windows 电脑、XBOX 或任何 Microsoft 应用的桌面版（如 Office 应用）。 
 
### <a name="does-this-replace-two-step-verification-should-i-turn-it-off"></a>手机登录是否可以取代多重身份验证？ 是否应将其关闭？   

有时需要关闭。 我们正努力扩大手机登录的适用范围，但目前在 Microsoft 生态系统中仍有不支持它的场合。 在这些场合中，仍需要使用多重身份验证进行安全登录。 因此，不应关闭帐户的多重身份验证。 
 
### <a name="okay-if-i-keep-two-step-verification-turned-on-for-my-account-will-i-have-to-approve-two-notifications"></a>如果启用帐户的多重身份验证，是否必须确认两条通知？

不需要。 使用手机登录到 Microsoft 帐户被视为多重身份验证。 可通过解锁手机再确认通知来证明身份，无需键入密码再确认通知。 我们不会发送第二条要求确认的通知。

### <a name="what-if-i-lose-my-phone-or-dont-have-it-with-me-how-can-i-access-my-account"></a>如果手机遗失或未随身携带，应如何访问我的帐户？  

你可以联系管理员，在 “Multi-factor authentication” 配置页面上，选中你的用户名，然后点击 “Manage user settings”， 在弹出的页面上勾选 “Require selected users to provide contact methods again”， 这样你就可以在登录时使用其他验证方式或更换手机号码。

### <a name="can-i-use-the-app-to-sign-in-to-all-my-accounts-with-microsoft"></a>是否可用此应用登录我所有的 Microsoft 帐户？   
此功能目前仅适用于 Microsoft 个人帐户。 
 
### <a name="can-i-sign-into-my-pc-with-my-phone"></a>是否可以使用手机登录到我的电脑？  
对于电脑，建议使用 Windows 10 中的 Windows Hello，通过面部、指纹或 PIN 进行登录。   
 
### <a name="can-i-sign-in-with-my-windows-phone"></a>是否可以使用 Windows Phone 登录？  
目前，没有为 Windows Phone 上的 Microsoft 验证器开发此功能。 

## <a name="next-steps"></a>后续步骤
如果尚未下载 Microsoft Authenticator 应用，请查看。 此应用适用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)，用于 [Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073) 的 Microsoft Authenticator 应用提供手机登录功能。

如果你有关于此应用的常见问题，请参阅 [Microsoft Authenticator 常见问题解答](/documentation/articles/microsoft-authenticator-app-faq/)

<!---Update_Description: wording update -->