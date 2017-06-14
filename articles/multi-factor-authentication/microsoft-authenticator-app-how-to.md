<properties
    pageTitle="Microsoft Authenticator 应用手机版 | Microsoft 文档"
    description="了解如何升级到最新版本的 Azure Authenticator。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="curtland" />
<tags
    ms.assetid="3065a1ee-f253-41f0-a68d-2bd84af5ffba"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/25/2017"
    ms.author="kgremban"
    wacn.date="06/12/2017"
    ms.custom="H1Hack27Feb2017, end-user"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="6f4bef4424b0540925a45bad4a773df040dc0b39"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="get-started-with-the-microsoft-authenticator-app"></a>Microsoft 验证器应用入门
Microsoft Authenticator 应用在工作或学校帐户（例如，bsimon@contoso.com）或 Microsoft 帐户（例如，bsimon@outlook.com）中提供了附加安全级别。

该应用以下面两种方式之一工作：

- **通知**。 该应用通过将通知推送到智能手机或平板电脑，可帮助防止对帐户进行未经授权的访问，以及停止欺诈性交易。 可以直接查看通知，如果该通知是合法的，则选择“验证” 。 否则，可以选择“拒绝” 。 有关拒绝通知的信息，请参阅“如何针对多重身份验证使用拒绝并报告欺诈功能”。
- **验证码**。 该应用可用作生成 OAuth 验证码所需的软件令牌。 输入用户名和密码后，在登录屏幕中输入该应用提供的代码。 验证码提供了第二种形式的身份验证。

Microsoft 验证器应用将替换 Azure 验证器应用。 可以继续使用 Azure Authenticator 应用，不过，如果决定过渡到新的 Microsoft Authenticator 应用，本文可提供帮助。  

## <a name="opt-in-for-two-step-verification"></a>针对多重身份验证选择加入

Microsoft Authenticator 应用不能单独发挥作用。 你需要将帐户配置为在你通过用户名和密码登录后提示你输入第二种验证方法。 

对于工作或学校帐户，通常不需要自己进行此选择。 相反，安全管理员将代表你选择加入，然后向你发送通知，告诉你需要为你的帐户注册验证方法。 如果是这种情况，请在 [Azure 多重身份验证对我意味着什么](/documentation/articles/multi-factor-authentication-end-user/)中了解详细信息。

对于个人帐户，你需要自己设置多重身份验证。 如果你有 Microsoft 帐户，[关于多重身份验证](https://support.microsoft.com/zh-cn/help/12408/microsoft-account-about-two-step-verification)中提供了那些步骤。 

还可以将 Microsoft Authenticator 与非 Microsoft 帐户一起使用。 它们可能不将此功能称为多重身份验证，但你应该能在安全性或登录设置下找到此功能。 

## <a name="install-the-app"></a>安装应用
Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [iOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

## <a name="add-accounts-to-the-app"></a>将帐户添加到应用
对于要添加到 Microsoft Authenticator 应用的每个帐户，使用以下过程之一。

### <a name="add-a-personal-microsoft-account-to-the-app"></a>将个人 Microsoft 帐户添加到应用

对于个人 Microsoft 帐户（用于登录到 Outlook.com、Xbox、Skype 等的帐户），只需在 Microsoft 验证器应用中登录到你的帐户。

### <a name="add-a-work-or-school-account-to-the-app-using-the-qr-code-scanner"></a>使用 QR 码扫描仪将工作或学校帐户添加到应用
1. 转到安全验证设置屏幕。  有关如何访问此屏幕的信息，请参阅[更改安全设置](/documentation/articles/multi-factor-authentication-end-user-manage-settings/#where-to-find-the-settings-page/)。
2. 选中“验证器应用”旁边的复选框，然后选择 “Set up”。

    ![安全验证设置屏幕上的“配置”按钮](./media/authenticator-app-how-to/azureauthe.png)

    这将显示一个有 QR 码的屏幕。

    ![提供 QR 码的屏幕](./media/authenticator-app-how-to/barcode2.png)
3. 打开 Microsoft Authenticator 应用。 在“帐户”屏幕上，选择 **+**，然后指定要添加工作帐户或学校帐户。
4. 使用相机扫描二维码，然后选择“完成”关闭二维码屏幕。

    如果相机不能正常工作，可以[手动输入 QR 码和 URL](#add-an-account-to-the-app-manually)。

5. 当该应用显示你的帐户名称和其下的六位代码时，你已完成。 

    ![帐户屏幕](./media/authenticator-app-how-to/accounts.png)

### <a name="add-an-account-to-the-app-manually"></a>手动将帐户添加到应用
1. 转到安全验证设置屏幕。  有关如何访问此屏幕的信息，请参阅[更改安全设置](/documentation/articles/multi-factor-authentication-end-user-manage-settings/)。
2. 选择 “Set up”。

    ![安全验证设置屏幕上的“配置”按钮](./media/authenticator-app-how-to/azureauthe.png)

    这将显示一个有 QR 码的屏幕。  记下该代码和 URL。

    ![提供 QR 码和 URL 的屏幕](./media/authenticator-app-how-to/barcode2.png)
3. 打开 Microsoft Authenticator 应用。 在“帐户”屏幕上，选择 **+**，然后指定要添加工作帐户或学校帐户。

4. 在扫描仪中，选择“手动输入代码”。

    ![用于扫描 QR 码的屏幕](./media/multi-factor-authentication-end-user-first-time/scan2.png)
5. 在应用的相应框中输入该代码和 URL，然后选择“完成”。

    ![用于输入代码和 URL 的屏幕](./media/authenticator-app-how-to/manual.png)

6. 当该应用显示你的帐户名称和其下的六位代码时，你已完成。

    ![帐户屏幕](./media/authenticator-app-how-to/accounts.png)

### <a name="add-an-account-to-the-app-using-touch-id"></a>使用 Touch ID 将帐户添加到应用
iOS 上的 Microsoft Authenticator 应用支持 Touch ID。  Azure 多重身份验证允许组织对设备要求提供 PIN。 使用 Touch ID，iOS 用户无需输入 PIN。 而是可以扫描其指纹，然后选择“批准” 。

使用 Microsoft Authenticator 设置 Touch ID 的过程很简单。 已完成使用 PIN 的普通验证问题。 如果设备支持 Touch ID，Microsoft Authenticator 将自动为该帐户设置它。

![验证 Touch ID 设置](./media/authenticator-app-how-to/touchid1.png)

随后，需要验证登录时，请选择收到的推送通知，并扫描指纹而不是输入 PIN。

![推送通知](./media/authenticator-app-how-to/touchid2.png)

## <a name="use-the-app-when-you-sign-in"></a>在登录时使用应用

将你的帐户添加到应用后，可能会提示你执行测试验证来确保所有事项都已正确配置。 之后，大功告成！ 在下次登录之前，不需要执行任何其他操作。

如果你选择在应用中使用验证码，则会在主页上看到它们。 它们每 30 秒更改一次，以便你在需要验证码时始终有新的验证码可用。 但是，在你登录且系统提示你输入验证码之前，不需要对它们执行任何操作。

<!---Update_Description: wording update -->