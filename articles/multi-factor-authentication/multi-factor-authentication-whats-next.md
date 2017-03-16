<properties
    pageTitle="配置 Azure MFA | Azure"
    description="Azure 多重身份验证页介绍了使用 MFA 可以执行的后续步骤。其中包括报告、欺诈警报、一次性跳过、自定义语音消息、缓存，受信任的 IP 和应用密码。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="yossib" />
<tags
    ms.assetid="75af734e-4b12-40de-aba4-b68d91064ae8"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/15/2017"
    wacn.date="03/14/2017"
    ms.author="kgremban" />  

# 配置 Azure 多重身份验证

在启动并运行 Azure 多重身份验证后，可以参考以下文章来对其进行管理。本文涵盖各种主题，足以让用户充分利用 Azure 多重身份验证。请注意，并非所有版本的 Azure 多重身份验证都提供这些功能。

可以通过 “Manage service settings” 页访问多重身份验证。 请以管理员身份登录 Azure 管理门户，然后选择 “Active Directory” 选项。单击你的目录，然后单击“配置”选项卡。在“多重身份验证”部分下，单击 “Manage service settings”。

| 功能 | 说明 |
| :---- | :---- |
| [可选择验证方法](#selectable-verification-methods) | 允许你选择可供用户使用的身份验证方法。 |

## 可选择验证方法  <a name="selectable-verification-methods"></a>
现在，可以选择当用户使用 Azure 多重身份验证时供用户使用的身份验证方法。此功能以前只提供了本地服务器版本。下表提供了可为用户启用或禁用的各种身份验证方法的简要概述。

方法|说明
:------------- | :------------- | 
[拨打电话](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-phone/)| 向身份验证电话拨打自动语音呼叫。用户接听电话，并按电话键盘上的 # 进行身份验证。此电话号码将不会同步到本地 Active Directory。
[向手机发送短信](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-phone/)|向用户发送包含验证码的短信。系统会提示用户使用验证码回复短信或在登录界面中输入验证码。
[通过移动应用发送通知](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-app/)|在此模式下，Microsoft Authenticator App 可防止对帐户进行未经授权的访问并停止欺诈性事务。此功能是使用推送到你的手机或已注册设备上的推送通知来完成的。你可以直接查看通知，如果该通知是合法的，请点击“验证”。否则，你可以选择“拒绝”。</br></br>Microsoft Authenticator App 可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。|
[通过移动应用发送验证码](/documentation/articles/multi-factor-authentication-end-user-first-time-mobile-app/)|在此模式下，Microsoft Authenticator App 可用作生成 OATH 验证码所需的软件令牌。然后可以将此验证码与用户名和密码一起输入，进行第二种形式的身份验证。</li><p> Microsoft Authenticator App 可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

### 如何启用/禁用身份验证方法

1. 登录到 Azure 经典管理门户。
2. 在左侧单击“Active Directory”。
3. 在 Active Directory 中，单击要对其启用或禁用身份验证方法的目录。
4. 在选择的目录上，单击“配置”。
5. 在“多重身份验证”部分中，单击“管理服务设置”。
6. 在“服务设置”页上的验证选项下，选择/取消选择你要使用的选项。</br></br>![验证选项](./media/multi-factor-authentication-whats-next/authmethods.png)
9. 单击“保存”。
10. 单击“关闭”。

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->