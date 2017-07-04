<properties
    pageTitle="多重身份验证疑难解答 |Microsoft 文档"
    description="本文档向用户提供有关如何解决 Azure 多重身份验证问题的信息。"
    services="multi-factor-authentication"
    keywords="multifactor authentication 客户端, 身份验证问题, 相关性 ID"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="yossib" />
<tags
    ms.assetid="8f3aef42-7f66-4656-a7cd-d25a971cb9eb"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/15/2017"
    wacn.date="05/15/2017"
    ms.author="kgremban"
    ms.custom="end-user"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="c265a7a743f1d2b3182eadb34649a16f712df295"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="having-trouble-with-two-step-verification"></a>执行多重身份验证时遇到问题
本文讨论了执行多重身份验证时可能会遇到的一些问题。 如果此处没有包含你遇到的问题，请在注释部分中提供详细的反馈以便我们可以改进。

## <a name="i-lost-my-phone-or-it-was-stolen"></a>我的手机丢失或被盗

如果你的手机已丢失或被盗，也建议你让管理员重置你的应用密码并清除所有已记住的设备。 如果管理员不确定如何实现此目的，请他们参阅此文：[管理用户和设备](/documentation/articles/multi-factor-authentication-manage-users-and-devices/)。

## <a name="i-am-not-receiving-a-text-or-call-on-my-phone"></a>我的手机上未收到短信或呼叫
如果你可以尝试登录但没有收到短信或电话呼叫，可能有多个原因。 如果你的手机过去成功收到过短信或电话呼叫，则可能是手机提供商有问题，而不是你的帐户有问题。 请确保你的手机信号良好，并且如果尝试接收短信，请确保你的手机和服务计划支持短信。

如果在等待几分钟后仍未收到短信或来电，进入帐户的最快方式是尝试其他选项。

1. 在等待验证的页面上选择“使用其他验证方法”
   
    ![其他验证方法](./media/multi-factor-authentication-end-user-troubleshoot/diff_option.png)
2. 选择要使用的电话号码或发送方式。
   
    如果你收到了多个验证码，则只有最新的一个验证码才起作用。

如果你没有配置其他方法，请联系管理员并要求他们清除你的设置。 下次登录时，系统将重新提示[设置多重身份验证](/documentation/articles/multi-factor-authentication-end-user-first-time/)。

如果经常由于手机信号不佳导致延迟，建议在智能手机上使用 [Microsoft Authenticator 应用](/documentation/articles/microsoft-authenticator-app-how-to/)。 该应用可以生成用来登录的随机安全代码，这些代码不需要手机信号或 Internet 连接。

## <a name="i-didnt-find-an-answer-to-my-problem"></a>我找不到我的问题的解答。
如果你已尝试了这些故障排除步骤但仍然存在问题，请联系管理员或者联系为你设置多重身份验证的人员。 它们应该能够提供帮助。

此外，还可以在 [Azure AD 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=WindowsAzureAD)上发布问题，或[联系支持人员](https://support.microsoft.com/zh-cn/contactus)，我们将尽快答复。

如果你联系支持人员，请提供以下信息：

- **用户 ID** - 你尝试登录时使用的是哪个电子邮件地址？
- **错误的一般描述** - 你看到的确切错误消息是什么？  如果没有任何错误消息，请详细描述你所发现的意外行为。
- **页面** - 看到错误时所在的页面（包括 URL）。
- **ErrorCode** - 你收到的具体错误代码。
- **SessionId** - 你收到的具体会话 ID。
- **相关性 ID** - 用户看到错误时所生成的相关性 ID 代码。
- **时间戳** - 看到错误时的精确日期和时间（包括时区）。

其中许多信息可以在登录页上找到。 如果未及时验证登录，请选择“查看详细信息”。

![登录错误详细信息](./media/multi-factor-authentication-end-user-troubleshoot/view_details.png)

提供这些信息将有助于我们尽快为你解决问题。

## <a name="related-topics"></a>相关主题
- [管理多重身份验证设置](/documentation/articles/multi-factor-authentication-end-user-manage-settings/)  
- [Microsoft Authenticator 应用程序常见问题](/documentation/articles/microsoft-authenticator-app-faq/)

<!--Update_Description: wording update-->