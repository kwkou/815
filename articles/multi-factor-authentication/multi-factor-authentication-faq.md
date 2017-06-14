<properties
    pageTitle="Azure 多重身份验证常见问题解答 | Azure"
    description="与 Azure 多重身份验证相关的常见问题与解答。 Azure 多重身份验证是要求使用多种方式（而不仅仅是用户名和密码）来验证用户身份的一种方法。 它为用户登录和事务提供了额外的安全层。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="yossib" />
<tags
    ms.assetid="50bb8ac3-5559-4d8b-a96a-799a74978b14"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/26/2017"
    ms.author="kgremban"
    wacn.date="06/12/2017"
    ms.custom="H1Hack27Feb2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="dafbe5f4b9903b72dd03b042bcb17eb5d606d620"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="frequently-asked-questions-about-azure-multi-factor-authentication"></a>有关 Azure 多重身份验证的常见问题
本“常见问题解答”文章解答有关 Azure 多重身份验证和使用多重身份验证服务的常见问题。 


## <a name="manage-and-support-user-accounts"></a>管理和支持用户帐户

**问：如果用户的手机未收到响应，或者用户没带手机，我该怎么告诉他们？**

但愿你的用户已配置多种验证方法。 请告诉他们再次尝试登录，但需要在登录页上选择另一种验证方法。 

可以让用户转到[最终用户故障排除指南](/documentation/articles/multi-factor-authentication-end-user-troubleshoot/)。


**问：如果某个用户无法进入其帐户，我该办什么？**

可以要求用户再次完成注册过程来重置其帐户。 详细了解[管理云中 Azure 多重身份验证的用户和设备设置](/documentation/articles/multi-factor-authentication-manage-users-and-devices/)。

**问：我的用户指出，有时他们收不到短信，或者回复了双向短信但验证超时。**

发送短信和接收双向短信回复无法得到保障，因为它们属于可能影响服务可靠性的不可控因素。 这些因素包括目标国家/地区、移动电话运营商和信号强度。

如果用户经常无法可靠地接收短信，请告诉他们改用移动应用或电话验证方法。 移动应用可以同时通过手机网络和 Wi-Fi 连接接收通知。 此外，即使设备根本没有信号，也可以生成验证码。 Microsoft 验证器应用适用于 [Android](http://go.microsoft.com/fwlink/?Linkid=825072)、[iOS](http://go.microsoft.com/fwlink/?Linkid=825073) 和 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)。

如果必须使用短信，建议尽可能使用单向短信，而不要使用双向短信。 单向短信更加可靠，可以防止由于回复其他国家/地区发来的短信而给用户造成的全球短信费用。

**问：是否可以更改在系统超时之前，用户必须输入短信中验证代码的时限？**

在某些情况下可以。 可以在 Azure MFA 服务器 7.0 和更高版本中配置双向短信的超时设置。 

默认情况下，Azure MFA 服务器会存储一次性密码 300 秒（5 分钟）。 如果用户在 300 秒后输入其代码，其身份验证将被拒绝。 可以通过设置注册表项来调整超时。 

1. 转到 HKLM\Software\Wow6432Node\Positive Networks\PhoneFactor。
2. 创建名为 **pfsvc_pendingSmsTimeoutSeconds** 的 DWORD 注册表项，并设置希望 Azure MFA 服务器存储一次性密码的时间长短（以秒为单位）。

对于单向短信，MFA 服务器将存储一次性密码 300 秒，而 Azure AD 中基于云的 MFA 将存储一次性密码 180 秒。 此设置不可配置。 

**问：我在 MFA 服务器中配置了来电显示，但用户仍然接到匿名呼叫者的多重身份验证电话。**

通过公共电话网络拨打多重身份验证电话时，有时会通过不支持来电显示的运营商路由电话。 因此，尽管多重身份验证系统始终会发送呼叫号码，但并不保证会显示该号码。

**问：为何系统提示用户注册其安全信息？**
有多种原因会导致系统提示用户注册其安全信息：

- 该用户的管理员已在 Azure AD 中为其启用 MFA，但没有为其帐户注册安全信息。 
- 该用户已在 Azure AD 中启用自助密码重置。 以后如果用户忘记了密码，安全信息可帮助他们重置密码。
- 该用户访问的应用程序配置了一个要求使用 MFA 的条件访问策略，但该应用程序以前未注册 MFA。
- 该用户正在将某个设备注册到 Azure AD（包括 Azure AD Join），并且你的组织要求使用 MFA 进行设备注册，但该用户以前未注册 MFA。
- 该用户正在 Windows 10 中生成 Windows Hello for Business（需要 MFA），但以前未注册 MFA。
- 组织已创建并启用一个 MFA 注册策略，该策略已应用到该用户。
- 该用户以前已注册 MFA，但选择的验证方法后来被管理员禁用。 因此，该用户必须再次完成 MFA 注册，以选择新的默认验证方法。


## <a name="errors"></a>错误
**问：如果用户使用移动应用通知时看到“身份验证请求不适用于已激活的帐户”错误消息，该怎么办？**

告诉他们按照此过程从移动应用中删除其帐户，然后重新添加：

1. 转到 Azure 门户预览配置文件，然后使用组织帐户登录。
2. 选择“其他安全性验证” 。
3. 从移动应用中删除现有帐户。
4. 单击“配置”，然后按照说明重新配置移动应用。

**问：如果用户在登录非浏览器应用程序时看到 0x800434D4L 错误消息，该怎么办？**

尝试登录到安装在本地计算机上的、无法使用需要多重身份验证的帐户的非浏览器应用程序时，将发生 0x800434D4L 错误。 

此问题的解决方法是使用不同的用户帐户进行管理员相关操作和非管理员操作。 稍后，可以在管理员帐户与非管理员帐户之间链接邮箱，以便能够使用非管理员帐户登录到 Outlook。 了解如何 [使管理员能够打开和查看用户邮箱的内容](http://help.outlook.com/141/gg709759.aspx?sl=1)，获取有关此解决方法的更多详细信息。

## <a name="next-steps"></a>后续步骤
如果此处未解答你的问题，请将其留在页面底部的评论中。 或者，以下是一些用于获取帮助的其他选项：

- 在 [Microsoft 支持知识库](https://www.microsoft.com/en-us/Search/result.aspx?form=mssupport&q=phonefactor&form=mssupport) 中搜索常见技术问题的解决方法。
- 在 [Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)中搜索和浏览来自社区的技术问题与解答，或者提出自己的问题。
- 对于旧版 PhoneFactor 客户，如果有疑问或需要重置密码方面的帮助，请使用[密码重置](mailto:phonefactorsupport@microsoft.com)链接建立支持案例。
- 通过 [Azure 多重身份验证服务器 (PhoneFactor) 支持](https://support.microsoft.com/zh-cn/oas/default.aspx?prid=14947)联系支持专业人员。 与我们联系时，尽可能包含有关问题的更多信息将很有帮助。 可以提供的信息包括出现错误的页面、特定错误代码、特定会话 ID 以及看到错误的用户的 ID。

<!--Update_Description: wording update-->