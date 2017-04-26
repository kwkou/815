<properties
    pageTitle="Azure 多重身份验证常见问题解答 | Azure"
    description="与 Azure 多重身份验证相关的常见问题与解答。 Azure 多重身份验证是要求使用多种方式（而不仅仅是用户名和密码）来验证用户身份的一种方法。 它为用户登录和事务提供了额外的安全层。"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="yossib"
    translationtype="Human Translation" />
<tags
    ms.assetid="50bb8ac3-5559-4d8b-a96a-799a74978b14"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/26/2017"
    wacn.date="04/24/2017"
    ms.author="kgremban"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="557e858c49df437d661da7fa3c7ea44be48f4ae2"
    ms.lasthandoff="04/14/2017" />

# <a name="frequently-asked-questions-about-azure-multi-factor-authentication"></a>有关 Azure 多重身份验证的常见问题
本“常见问题”文章解答有关 Azure 多重身份验证和使用多重身份验证服务的常见问题，包括计费模式和可用性相关问题。

## <a name="general"></a>常规
**问：Azure 多重身份验证服务器如何处理用户数据？**

使用多重身份验证服务器时，用户数据只存储在本地服务器中。 云中不会持久存储任何用户数据。 当用户执行多重身份验证时，多重身份验证服务器会将数据发送到 Azure 多重身份验证云服务以进行身份验证。 多重身份验证服务器和多重身份验证云服务器之间的通信通过出站端口 443 使用安全套接字层 (SSL) 或传输层安全性 (TLS)。

当身份验证请求发送到云服务时，将收集数据用于身份验证和使用情况报告。 多重身份验证日志中包含的数据字段如下：

- **唯一 ID**（用户名或本地多重身份验证服务器 ID）
- **姓名**（可选）
- **电子邮件地址**（可选）
- **电话号码**（用于语音通话或短信身份验证）
- **设备令牌**（用于移动应用身份验证）
- **身份验证模式**
- **身份验证结果**
- **多重身份验证服务器名称**
- **多重身份验证服务器 IP**
- **客户端 IP**（如果可用）

可以在多重身份验证服务器中配置可选字段。

验证结果（成功或拒绝）和任何拒绝原因也与身份验证数据一起存储，可从身份验证和使用情况报告中获取。

## <a name="billing"></a>计费
可参阅 [多重身份验证定价页](/pricing/details/multi-factor-authentication/)获得大多数计费问题的答案。

**问：通过电话或短信对用户进行身份验证时，组织是否需要付费？**

对于通过 Azure 多重身份验证拨打的每个电话或者向用户发送的每条短信，组织不需要付费。 可能会向电话主人收取接听电话或接收短信的费用（根据其个人电话服务）。

**问：按用户计费模式的收费依据是配置为使用多重身份验证的用户数还是执行验证的用户数？**

计费依据是配置为使用多重身份验证的用户数。

**问：多重身份验证如何计费？**

使用“每用户”或“每身份验证”模型时，Azure MFA 是基于消耗的资源。 任何费用都将针对组织的 Azure 订阅计取，就和虚拟机、网站等一样。

使用授权模式时，需要购买 Azure 多重身份验证许可证并将其分配给用户，如同购买 Office 365 及其他订阅产品时一样。

在 [Azure 多重身份验证的工作原理](/documentation/articles/multi-factor-authentication-how-it-works/)中，了解各选项的详细信息

**问：是否有可供管理员使用的免费 Azure 多重身份验证版本？**

在某些情况下有。 可供 Azure 管理员使用的多重身份验证免费提供 Azure MFA 功能的子集。 此优惠适用于 Azure Active Directory 实例中的 Azure 全局管理员组的成员，这些实例未链接到基于使用的 Azure 多重身份验证提供程序。 使用多重身份验证提供程序，会将目录中已配置为使用多重身份验证的所有管理员和用户升级到完整版本的 Azure 多重身份验证。

**问：是否有可供 Office 365 用户使用的免费 Azure 多重身份验证版本？**

在某些情况下有。 适用于 Office 365 的多重身份验证免费提供 Azure MFA 功能的子集。 当基于消耗的 Azure 多重身份验证提供程序尚未链接到 Azure Active Directory 的相应实例时，将向分配有 Office 365 许可证的用户提供此优惠。 使用多重身份验证提供程序，会将目录中已配置为使用多重身份验证的所有管理员和用户升级到完整版本的 Azure 多重身份验证。

**问：组织是否可以随时在“按用户”和“按身份验证”消费量计费模式之间切换？**

组织在创建资源时可选择计费模式。 预配资源之后无法更改计费模式。 但是，可以创建另一个多重身份验证资源替换原始资源。 用户设置和配置选项不能转移到新资源。

**问：组织是否可以随时在消费量计费和授权模式之间切换？**

将许可证添加到已有按用户的 Azure 多重身份验证提供程序的目录时，会根据拥有的许可证数量减除基于消费量的计费。 如果配置为使用多重身份验证的所有用户都分配有许可证，则管理员可以删除 Azure 多重身份验证提供程序。

不能混用按身份验证的消费量计费与授权模式。 如果按身份验证的多重身份验证提供程序已链接到目录，则无论组织是否拥有许可证，都会对其收取所有多重身份验证验证请求的费用。

**问：组织是否必须使用并同步标识才能使用 Azure 多重身份验证？**

组织使用基于消费量的计费模式时，不需要 Azure Active Directory。 可以视需要将多重身份验证提供链接到目录。 如果组织未链接到目录，可以将多重身份验证服务器或多重身份验证 SDK 部署在本地。

授权模式需要 Azure Active Directory，因为在购买许可证并将其分配给目录中的用户时，许可证将添加到目录。

## <a name="usability"></a>可用性
**问：如果用户未收到电话答复或者无法使用电话，该怎么办？**

如果用户配置了备用电话，则应在登录页出现提示时重试，并选择该电话。 如果用户未配置其他方法，则组织管理员可以更新分配到用户主要电话的号码。

**问：如果用户因无法再访问某个帐户而联系管理员，管理员该如何处理？**

管理员可以要求用户再次完成注册过程来重置其帐户。 详细了解[管理云中 Azure 多重身份验证的用户和设备设置](/documentation/articles/multi-factor-authentication-manage-users-and-devices/)。

**问：如果包含应用密码的用户手机丢失或被盗，管理员该如何处理？**

管理员可以删除用户的所有应用密码，防止未经授权的访问。 用户购买替代设备后，即可重新创建密码。 详细了解[管理云中 Azure 多重身份验证的用户和设备设置](/documentation/articles/multi-factor-authentication-manage-users-and-devices/)。

**问：如果用户无法登录非浏览器应用，该怎么办？**

配置为使用多重身份验证的用户需要通过一个应用密码来登录某些非浏览器应用。 用户需要清除（删除）登录信息，重新启动该应用，然后使用其用户名和应用密码登录。

> [AZURE.NOTE]
> 适用于 Office 2013 客户端的新式验证
> 
> Office 2013 客户端（包括 Outlook）支持新式验证协议。 可以将 Office 2013 配置为支持多重身份验证。 配置 Office 2013 后，Office 2013 客户端不需要应用密码。 有关详细信息，请参阅 [Office 2013 modern authentication public preview announcement](https://blogs.office.com/2015/03/23/office-2013-modern-authentication-public-preview-announced/)（Office 2013 新式验证公共预览版发布声明）。
> 
> 

**问：如果用户未收到短信或者回复双向短信但验证超时，该怎么办？**

因为存在可能影响服务可靠性的不可控因素，所以无法保证双向 SMS 中的短信送达和对回复的接收。 这些因素包括目的地国家/地区、移动电话运营商和信号强度。

难以可靠接收短信的用户应改为选择移动应用或电话呼叫的方法。 移动应用可同时通过手机网络和 Wi-Fi 连接接收通知。 此外，即使设备根本没有信号，也可以生成验证码。 Microsoft Authenticator 应用可用于 [Windows Phone](http://go.microsoft.com/fwlink/?Linkid=825071)、[Android](http://go.microsoft.com/fwlink/?Linkid=825072) 和 [IOS](http://go.microsoft.com/fwlink/?Linkid=825073)。

如果必须使用短信，建议尽可能使用单向短信，而不要使用双向短信。 单向短信更加可靠，可以防止由于回复其他国家/地区发来的短信而给用户造成的全球短信费用。

**问：是否可以在 Azure 多重身份验证服务器上使用硬件令牌？**

如果使用的是 Azure 多重身份验证服务器，可以导入第三方基于时间的一次性密码 (TOTP) 开放式身份验证 (OATH) 令牌，然后将其用于多重身份验证。

如果将机密密钥文件放在 CSV 文件中并导入到 Azure 多重身份验证服务器中，则可以使用充当 OATH TOTP 令牌的 ActiveIdentity 令牌。 可将 OATH 令牌与以下服务配合使用：Active Directory 联合身份验证服务 (ADFS)；远程身份验证拨入用户服务 (RADIUS)（如果客户端系统可以处理访问质询响应）；基于 Internet Information Server (IIS) 窗体的身份验证。

可以使用以下格式导入第三方 OATH TOTP 令牌：  

- 可移植对称密钥容器 (PSKC)  
- CSV 如果文件包含序列号、Base 32 格式的密钥和时间间隔  

**问：是否可以使用 Azure 多重身份验证服务器保护终端服务？**

可以，但如果使用的是 Windows Server 2012 R2 或更高版本，则只能使用远程桌面网关（RD 网关）保护。

Windows Server 2012 R2 中的安全性更改，改变了 Azure 多重身份验证服务器连接到 Windows Server 2012 和更低版本中的本地安全机构 (LSA) 安全包的方式。  如果使用 Windows Server 2012 R2，则需要 RD 网关。

**问：为何设置来电显示后，用户会收到来自匿名呼叫方的多重身份验证电话？**

通过公共电话网络拨打多重身份验证电话时，有时会通过不支持来电显示的运营商路由电话。 因此，尽管多重身份验证系统始终会发送呼叫号码，但并不保证会显示该号码。

## <a name="errors"></a>错误
**问：如果用户使用移动应用通知时看到“身份验证请求不适用于已激活的帐户”错误消息，该怎么办？**

告诉他们按照此过程从移动应用中删除其帐户，然后重新添加：

1. 转到 Azure 门户预览配置文件，然后使用组织帐户登录。
2. 选择“其他安全性验证” 。
3. 从移动应用中删除现有帐户。
4. 单击“配置”，然后按照说明重新配置移动应用。

**问：如果用户在登录非浏览器应用程序时看到 0x800434D4L 错误消息，该怎么办？**

目前，用户只能在可以通过浏览器访问的应用程序和服务中使用其他安全性验证。 安装在本地计算机上的非浏览器应用程序（也称为*富客户端应用程序*，例如 Windows PowerShell）无法使用需要其他安全性验证的帐户。 在此情况下，用户可能会看到应用程序生成 0x800434D4L 错误。

此问题的解决方法是使用不同的用户帐户进行管理员相关操作和非管理员操作。 稍后，可以在管理员帐户与非管理员帐户之间链接邮箱，以便能够使用非管理员帐户登录到 Outlook。 了解如何 [使管理员能够打开和查看用户邮箱的内容](http://help.outlook.com/141/gg709759.aspx?sl=1)，获取有关此解决方法的更多详细信息。

## <a name="next-steps"></a>后续步骤
如果此处未解答你的问题，请将其留在页面底部的评论中。 或者，以下是一些用于获取帮助的其他选项：

- 在 [Microsoft 支持知识库](https://www.microsoft.com/en-us/Search/result.aspx?form=mssupport&q=phonefactor&form=mssupport) 中搜索常见技术问题的解决方法。
- 在 [Azure Active Directory 论坛](https://social.msdn.microsoft.com/Forums/azure/newthread?category=windowsazureplatform&forum=WindowsAzureAD&prof=required)中搜索和浏览来自社区的技术问题与解答，或者提出自己的问题。
- 对于旧版 PhoneFactor 客户，如果有疑问或需要重置密码方面的帮助，请使用[密码重置](mailto:phonefactorsupport@microsoft.com)链接建立支持案例。
- 通过 [Azure 多重身份验证服务器 (PhoneFactor) 支持](https://support.microsoft.com/zh-cn/oas/default.aspx?prid=14947)联系支持专业人员。 与我们联系时，尽可能包含有关问题的更多信息将很有帮助。 可以提供的信息包括出现错误的页面、特定错误代码、特定会话 ID 以及看到错误的用户的 ID。

<!--Update_Description: wording update-->