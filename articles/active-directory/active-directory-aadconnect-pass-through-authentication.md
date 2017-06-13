<properties
    pageTitle="Azure AD Connect：直通身份验证 | Azure"
    description="本文介绍 Azure Active Directory (Azure AD) 直通身份验证，以及它如何针对本地 Active Directory 来验证用户密码，从而实现 Azure AD 登录。"
    services="active-directory"
    keywords="什么是 Azure AD Connect 直通身份验证, 安装 Active Directory, Azure AD 所需的组件, SSO, 单一登录"
    documentationcenter=""
    author="swkrish"
    manager="femila" />
<tags
    ms.assetid="9f994aca-6088-40f5-b2cc-c753a4f41da7"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/09/2017"
    ms.author="billmath" 
    wacn.date="06/12/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="9014b84b5f8008bd2761fa0da9e65abfb08529e4"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="configure-user-sign-in-with-azure-active-directory-pass-through-authentication"></a>将用户登录配置为使用 Azure Active Directory 直通身份验证

## <a name="what-is-azure-active-directory-pass-through-authentication"></a>什么是 Azure Active Directory 直通身份验证？

如果能够允许用户使用相同的凭据（密码）登录到本地资源和基于云的服务，则对用户和组织而言都非常有利。 这样可以让用户少记一个密码，从而提供更好的用户体验， 并且还可以减少用户忘记登录方式的可能性。 然后技术支持成本就会下降，因为密码相关问题通常占用大多数支持资源。

许多组织使用 [Azure AD 密码同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/)（一项用于将用户密码哈希从本地 Active Directory 同步到 Azure Active Directory (Azure AD) 的 [Azure AD Connect](/documentation/articles/active-directory-aadconnect/) 功能）向用户提供相同的凭据，让他们在各种本地资源和基于云的服务中使用。 但是，有些组织要求密码（即使采用哈希处理的形式）不得离开其内部组织边界。

Azure AD 直通身份验证功能为这些组织提供了一个简单的解决方案。 当用户登录到 Azure AD 时，此功能可以确保针对本地 Active Directory 直接验证用户的密码。 此功能还具有以下优势：

- 易于使用
  - 无需复杂的本地部署或网络配置即可执行密码验证。
  - 轻型本地连接器侦听和响应密码验证请求。
  - 本地连接器自动接收功能改进和 Bug 修复。
  - 它可以连同 [Azure AD Connect](/documentation/articles/active-directory-aadconnect/) 一起进行配置。 轻型本地连接器与 Azure AD Connect 安装在同一服务器上。
- 安全
  - 本地密码永远不会以任何形式存储在云中。
  - 轻型本地连接器只从网络内部建立出站连接。 因此，无需在外围网络（也称 DMZ）中安装连接器。
  - 直通身份验证与 Azure 多重身份验证无缝配合。
- 可靠且可缩放
  - 可在多台服务器上安装其他轻型本地连接器，实现登录请求的高可用性。

![Azure AD 直通身份验证](./media/active-directory-aadconnect-pass-through-authentication/pta1.png)

可以将直通身份验证与[无缝单一登录](/documentation/articles/active-directory-aadconnect-sso/) (SSO) 功能结合使用。 这样一来，如果用户位于你的企业网络中的企业计算机上，他们甚至不需要键入密码便可登录到 Azure AD — 真正的集成体验。

## <a name="whats-available-during-preview"></a>预览版提供哪些功能？

>[AZURE.NOTE]
>Azure AD 直通身份验证目前为预览版。 这是一项免费功能，不需要拥有任何付费版本的 Azure AD 即可使用此功能。

预览版完全支持以下方案：

- 所有基于 Web 浏览器的应用程序
- 支持[新式身份验证](https://aka.ms/modernauthga)的 Office 365 客户端应用程序

预览版_不_支持以下方案：

- 旧式 Office 客户端应用程序和 Exchange ActiveSync（即，移动设备上的本机电子邮件应用程序）。 <br>建议组织改用新式身份验证。

>[AZURE.IMPORTANT]
>启用直通身份验证时，默认还会启用密码同步，这样就可以实现直通身份验证功能目前尚不支持的方案（旧式 Office 客户端应用程序、Exchange ActiveSync 和适用于 Windows 10 设备的 Azure AD Join）。 密码同步只在这些特定的方案中充当回退机制。 如果不需要密码同步，可以在 Azure AD Connect 向导中的“可选功能”[](/documentation/articles/active-directory-aadconnect-get-started-custom/#optional-features)页上将它关闭。

### <a name="prerequisites"></a>先决条件

在启用 Azure AD 直通身份验证之前，需要事先满足以下先决条件：

- 具有 Azure AD 租户（你是该租户的全局管理员）。

  >[AZURE.NOTE]
  >强烈建议使用仅限云的帐户作为全局管理员帐户。 这样一来，就可以在本地服务出现故障或不可用时管理租户的配置。 

- Azure AD Connect 1.1.486.0 或更高版本。 建议使用[最新版的 Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594)。
- 运行 Windows Server 2012 R2 或更高版本的服务器，在其上可运行 Azure AD Connect。
  - 此服务器必须是需要验证其密码的用户所在的 AD 林的成员。
  - 请注意，直通身份验证连接器与 Azure AD Connect 安装在同一服务器上。 验证连接器版本是否为 1.5.58.0 或更高版本。

    >[AZURE.NOTE]
    >如果 AD 林之间存在信任关系并且正确配置了名称后缀路由，则支持多林环境。

- 若要获得高可用性，需在运行 Windows Server 2012 R2 或更高版本的其他服务器上安装独立的连接器。 （连接器版本需为 1.5.58.0 或更高版本。）
- 如果任何连接器与 Azure AD 之间存在防火墙：
    - 如果启用了 URL 筛选，请确保连接器能够与以下 URL 通信：
        -  \*.msappproxy.net
        -  \*servicebus.chinacloudapi.cn
    - 确保连接器还与 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=42064)建立直接 IP 连接。
    - 确保防火墙不执行 SSL 检查，因为连接器使用客户端证书与 Azure AD 通信。
    - 确保连接器可通过端口 80 和 443 向 Azure AD 发出出站请求。
      - 如果防火墙根据发起用户强制实施规则，请针对来自作为网络服务运行的 Windows 服务的流量打开这些端口。
      - 连接器通过端口 80 发出 HTTP 请求以下载 SSL 证书吊销列表。 自动更新功能的正常运行也需完成此操作。
      - 连接器通过端口 443 发出 HTTPS 请求以执行所有其他操作，例如启用和禁用功能、注册连接器、下载连接器更新和处理所有用户登录请求。

     >[AZURE.NOTE]
     >我们最近做了改进，减少了连接器与我们的服务通信时所需的端口数。 如果运行旧版 Azure AD Connect 和/或独立连接器，应继续让这些附加端口（5671、8080、9090、9091、9350、9352、10100-10120）保持打开状态。

### <a name="enable-azure-ad-pass-through-authentication"></a>启用 Azure AD 直通身份验证

可通过 Azure AD Connect 启用 Azure AD 直通身份验证。

如果你要执行 Azure AD Connect 全新安装，请选择“自定义安装路径”。[](/documentation/articles/active-directory-aadconnect-get-started-custom/) 在“用户登录”页上，选择“直通身份验证”。 成功完成上述步骤后，将在 Azure AD Connect 所在的服务器上安装直通身份验证连接器。 此外，还会在租户中启用直通身份验证功能。

![Azure AD Connect - 用户登录](./media/active-directory-aadconnect-sso/sso3.png)

如果已使用[快速安装](/documentation/articles/active-directory-aadconnect-get-started-express/)或[自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom/)路径安装了 Azure AD Connect，请在 Azure AD Connect 上选择“更改用户登录页”并单击“下一步”。 然后选择“直通身份验证”，在 Azure AD Connect 所在的服务器上安装直通身份验证连接器并在租户中启用该功能。

![Azure AD Connect - 更改用户登录](./media/active-directory-aadconnect-user-signin/changeusersignin.png)

>[AZURE.IMPORTANT]
>Azure AD 直通身份验证是租户级功能。 它会影响租户中所有托管域上的用户登录。 但是，联合域中的用户将继续使用 Active Directory 联合身份验证服务 (AD FS) 或你以前配置的任何其他联合身份验证提供程序登录。 如果将联合域转换为托管域，该域中的所有用户会自动开始使用直通身份验证登录。 仅限云的用户不受直通身份验证的影响。

如果打算在生产部署中使用直通身份验证，强烈建议在独立的服务器（不是运行 Azure AD Connect 和第一个连接器的服务器）上安装另一个连接器。 这样做可确保获得登录请求的高可用性。 可以安装任意数目的附加连接器。 具体的数目取决于租户处理的登录请求的峰值数目和平均数目。

遵照以下说明部署独立的连接器。

#### <a name="step-1-download-and-install-the-connector"></a>步骤 1：下载并安装连接器

此步骤在服务器中下载并安装连接器软件。

1.    [下载](https://go.microsoft.com/fwlink/?linkid=837580)最新的连接器。 验证连接器版本是否为 1.5.58.0 或更高版本。
2.    以管理员身份打开命令提示符。
3.    运行以下命令。 ****/q 选项表示“静默安装”- 安装程序不会提示你接受最终用户许可协议：`
AADApplicationProxyConnectorInstaller.exe REGISTERCONNECTOR="false" /q
`

>[AZURE.NOTE]
>只能在每台服务器上安装一个连接器。

#### <a name="step-2-register-the-connector-with-azure-ad-for-pass-through-authentication"></a>步骤 2：将连接器注册到 Azure AD 进行直通身份验证

此步骤将服务器上安装的连接器注册到我们的服务，以便可以使用它来接收登录请求。

1.    以管理员身份打开 PowerShell 窗口。
2.    导航到 C:\Program Files\Microsoft AAD App Proxy Connector，然后按如下所示运行脚本：`.\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Feature PassthroughAuthentication`
3.    出现提示时，请输入 Azure AD 租户中全局管理员帐户的凭据。

## <a name="how-does-azure-ad-pass-through-authentication-work"></a>Azure AD 直通身份验证的工作原理

当用户尝试登录到 Azure AD 时（如果已在租户中启用直通身份验证），会发生以下情况：

1. 用户在 Azure AD 登录页中输入其用户名和密码。 我们的服务将该用户名和密码（已使用公钥加密）排入队列等待验证。
2. 某个可用的本地连接器向该队列发出出站调用，并检索该用户名和密码。
3. 然后，连接器使用标准 Windows API（类似于 AD FS 使用的机制）针对 Active Directory 验证该用户名和密码。 请注意，用户名可以是本地默认用户名（通常为 `userPrincipalName`），也可以是 Azure AD Connect 中配置的另一个属性（称为 `Alternate ID`）。
4. 然后，本地域控制器评估请求并向连接器返回响应（成功或失败）。
5. 连接器随后将此响应返回给 Azure AD。
6. 然后，Azure AD 评估此响应，并向用户提供相应响应。 例如，向应用程序发回令牌或者要求执行多重身份验证。

下图演示了各个步骤。 所有请求和响应都是通过 HTTPS 通道发出的。

![直通身份验证](./media/active-directory-aadconnect-pass-through-authentication/pta2.png)

### <a name="password-writeback"></a>密码写回

如果你已在租户中针对特定的用户配置[密码写回](/documentation/articles/active-directory-passwords-update-your-own-password/)，则当用户使用直通身份验证登录时，他们可以像以往那样更改或重置其密码。 密码会按预期写回到本地 Active Directory。

但是，如果未在租户中配置密码写回，或者没有为用户分配有效的 Azure AD 许可证，则不允许用户在云中更新其密码。 密码过期时也是如此。 用户会看到消息：“你的组织不允许你更新此站点上的密码。 请根据组织建议的方法更新密码，或者请求管理员提供帮助。”

## <a name="next-steps"></a>后续步骤
- 请参阅如何在租户中启用 [Azure AD 无缝 SSO](/documentation/articles/active-directory-aadconnect-sso/) 功能。
- 请阅读[故障排除指南](/documentation/articles/active-directory-aadconnect-troubleshoot-pass-through-authentication/)，了解如何解决 Azure AD 直通身份验证的常见问题。

## <a name="feedback"></a>反馈
你的反馈对我们非常重要。 如有疑问，请在评论区中提出。 若要请求添加新功能，请在我们的 [UserVoice 论坛](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect)中发贴。

<!---Update_Description: wording update -->