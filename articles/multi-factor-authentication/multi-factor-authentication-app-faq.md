<properties
    pageTitle="Microsoft Authenticator 应用常见问题解答"
    description="提供与 Microsoft Authenticator 应用和 Azure 多重身份验证相关的常见问题与解答列表。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="pblachar, librown" />  

<tags
    ms.assetid="f04d5bce-e99e-4f75-82d1-ef6369be3402"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/02/2016"
    wacn.date="12/22/2016"
    ms.author="kgremban" />

# Microsoft Authenticator 应用程序常见问题解答
Microsoft Authenticator 应用替代了 Azure Authenticator 应用，建议使用 Azure 多重身份验证时使用该应用。此应用可用于 Windows Phone、Android 和 iOS。

## 常见问题
### Azure Authenticator、多重身份验证和 Microsoft 帐户应用发生了什么情况？
Microsoft Authenticator 应用将替换这些应用。Azure Authenticator 已升级为 Microsoft Authenticator。如果使用多重身份验证或 Microsoft 帐户应用，请安装 Microsoft Authenticator，并再次添加帐户。确保先将帐户添加到新应用中，再删除旧应用。

### 我已使用 Microsoft Authenticator 应用程序发送验证码。如何切换到一键式推送通知？
通过推送通知批准登录仅适用于 Microsoft 帐户，而不适用于 Google 或 Facebook 等第三方帐户。但对于工作或学校 Microsoft 帐户，组织可选择禁用此选项。

如果使用 Microsoft 帐户作为个人帐户，并且想要切换到推送通知，需要再次添加帐户。向您的帐户重新注册该设备，并设置推送通知。

如果对工作或学校帐户使用 Microsoft Authenticator，那么组织可决定是否允许使用一键式通知。

### 我什么时候能在 iPhone 或 iPad 上使用一键式推送通知？
此功能到 8 月底之前都处于测试阶段，之后将会广泛可用于 Microsoft 帐户。如果想加入我们的测试计划，请发送电子邮件至 msauthenticator@microsoft.com。邮件中需包括您的名字、姓氏和 Apple ID。

### 一键式推送通知是否适用于非 Microsoft 帐户？
不适用，推送通知只适用于 Microsoft 帐户和 Azure Active Directory 帐户。如果您的工作单位或学校使用的是 Azure AD 帐户，则可能会禁用此功能。

### 从备份还原设备后，帐户代码丢失或失效。发生了什么情况？
出于安全考虑，我们不会从应用备份还原帐户。如果从备份还原 iOS 应用，仍会显示帐户，但不可将其用于接收登录验证或生成安全代码。还原应用后，请删除并再次添加帐户。

### 我有了一台新设备。如何从旧设备删除 Microsoft Authenticator 应用并将其迁移到新设备？
将 Microsoft Authenticator 应用添加到新设备不会从任何其他设备将其自动删除。若要管理为帐户配置的设备，请访问用于管理双重验证的网站，选择删除旧应用。

对于个人 Microsoft 帐户，此网站即[帐户安全](https://account.microsoft.com/security)页面。对于工作或学校 Microsoft 帐户，此网站可能是 [MyApps](https://myapps.microsoft.com) 或组织已设置的自定义门户。

### 如何从应用删除帐户？
- iOS：从主屏幕中，在帐户磁贴上向左轻扫。选择“删除”。
- Windows Phone：从主屏幕中，选择“菜单”按钮，然后选择“编辑帐户”。点击帐户名称旁边的 **X**。
- Android：从主屏幕中，选择“菜单”按钮，然后选择“编辑帐户”。点击帐户名称旁边的 **X**。

如果拥有已注册到组织的 Android 设备，可能需要完成一个额外步骤才能删除帐户。在这些设备上，以设备管理员自动注册了 Microsoft Authenticator 应用。如果想要彻底卸载应用，需受先在应用设置中取消注册应用。

### 应用为什么要请求如此多的权限？
我们的应用将请求的权限以及应用中如何使用这些权限的完整列表如下：

- **相机**：用户添加工作、学校或非 Microsoft 帐户时，应用使用相机来扫描 QR 码。
- **联系人和手机**：用户使用个人 Microsoft 帐户登录时，应用将尝试通过查找手机上使用的现有帐户来简化过程。
- **SMS**：用户首次使用个人 Microsoft 帐户登录时，应用必须确保手机号码与已记录的号码相匹配。我们将向用于下载应用的手机发送短信。该短信包含 6-8 位的验证码。应用不会要求用户查找并在应用中输入此验证码，而是在短信中自动查找它。
- **在其他应用上方显示**：收到通知以验证身份时，在任何可能正在运行的其他应用上显示该通知。
- **从 Internet 接收数据**：需要此权限才能发送通知。
- **阻止手机进入睡眠状态**：如果用户将设备注册到组织，则组织可以更改其手机上的这一策略。
- **控制振动**：用户可以选择收到验证身份的通知时是否振动。
- **使用指纹硬件**：某些工作和学校帐户在验证身份时需要其他 PIN。为简化此过程，我们允许使用指纹而不输入 PIN。
- **查看网络连接**：应用需要网络/Internet 连接才能添加 Microsoft 帐户。
- **读取存储的内容**：仅当通过应用设置报告技术问题时，才会使用此权限。将从存储中收集某些信息来诊断问题。
- **完全网络访问**：此权限为发送验证身份的通知所需。
- **在启动时运行**：如果用户重启手机，此权限可确保继续接收验证身份的通知。

## 联系我们
如果此处未解答你的问题，请在页面底部留下评论。或[联系支持人员](https://support.microsoft.com/zh-cn/contactus)，我们会尽快回应你的问题。

如果联系支持人员，请尽量提供以下信息：

- **用户 ID** - 你尝试登录时使用的是哪个电子邮件地址？
- **错误的一般描述** - 你看到的确切错误消息是什么？ 如果没有任何错误消息，请详细描述你所发现的意外行为。
- **页面** - 你看到错误时位于哪个页面（包括 URL）？
- **ErrorCode** - 你收到的具体错误代码。
- **SessionId** - 你收到的具体会话 ID。
- **相关性 ID** - 用户看到错误时所生成的相关性 ID 代码。
- **时间戳** - 你看到错误时的精确日期和时间（包括时区）是多少？

其中许多信息可以在登录页上找到。当未及时验证你的登录时，请选择“查看详细信息”。

![登录错误详细信息](./media/multi-factor-authentication-end-user-troubleshoot/view_details.png)  


提供这些信息将有助于我们尽快为你解决问题。

## 相关主题
- [Azure 多重身份验证常见问题](/documentation/articles/multi-factor-authentication-faq/)
- 关于 Microsoft 帐户的[双重验证](https://support.microsoft.com/zh-cn/help/12408/microsoft-account-about-two-step-verification)
- [执行双重验证时遇到问题？](/documentation/articles/multi-factor-authentication-end-user-troubleshoot/)

<!---HONumber=Mooncake_1212_2016-->
