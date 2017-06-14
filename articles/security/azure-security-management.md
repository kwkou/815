<properties
   pageTitle="Azure 中的安全管理 | Azure"
   description="本文详细介绍在管理 Azure 环境（包括云服务、虚拟机和自定义应用程序）时增强远程管理安全的步骤。"
   services="azure-security, virtual-machines, cloud-services"
   documentationCenter="na"
   authors="TerryLanfear"
   manager="StevenPo"
   editor="TomSh"/>

<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="04/26/2016"
   wacn.date="05/30/2016"
   ms.author="terrylan"/>

# Azure 中的安全管理

Azure 订阅者可从多种设备管理其云环境，这些设备包括管理工作站、开发人员电脑，甚至是具有任务特定权限的特权最终用户设备。在某些情况下，可通过基于 Web 的控制台（例如 [Azure 门户](https://azure.microsoft.com/features/azure-portal/)）来执行管理功能。有其他情况下，可以从本地系统通过虚拟专用网络 (VPN)、终端服务、客户端应用程序协议或 Azure 服务管理 API (SMAPI)（以编程方式）直接连接到 Azure。此外，客户端终结点（例如平板电脑或智能手机）可以加入域或者受到隔离且不受管理。

尽管这么多种的访问和管理功能可提供丰富的选项，但选项太多也可能会让云部署承受巨大风险，因而难以管理、跟踪和审核管理操作。这种差异还可能由于用于管理云服务的客户端终结点进行的访问不受管制而带来安全威胁。使用普通工作站或专用工作站开发和管理基础结构将会打开不可预测的威胁媒介，例如 Web 浏览（例如水坑攻击）或电子邮件（例如社交工程和网络钓鱼）。

![][1]

在这种类型的环境中，发生攻击的可能性将会增加，因为难以构造安全策略和机制来适当管理各种终结点对 Azure 接口（例如 SMAPI）的访问。

### 远程管理威胁

攻击者通常将尝试通过入侵帐户凭据（例如，通过暴力破解密码、网络钓鱼和搜集凭据）或诱骗用户运行恶意代码（例如，从具有偷渡式下载的恶意网站或从恶意电子邮件的附件）来获取特殊访问权限。远程管理的云环境由于具有随时随地访问的特性，因此如果帐户遭到入侵，风险将会大增。

即使主要管理员帐户受到严格控制，攻击者仍可使用较低级别的用户帐户来利用安全策略中的弱点。缺乏适当的安全培训而使帐户信息意外泄漏或曝光，也可能导致数据泄露。

如果用户工作站也用于管理任务，可能会在许多不同的位置遭到入侵。无论用户是在浏览 Web、使用第三方工具和开源工具，还是打开包含特洛伊木马的恶意文档，都将面临入侵风险。

一般情况下，大多数导致数据泄漏的锁定式攻击，追根究底都是台式机上的浏览器入侵、外挂程序（例如 Flash、PDF、Java）和鱼叉式网络钓鱼（电子邮件）所造成的。这些计算机在用于开发或管理其他资产时，可能具有可供访问运行中服务器或网络设备以执行操作的管理级别权限或服务级别权限。

### 操作安全性基础知识

为了提升管理和操作的安全性，你可以减少可能的入口点数目以尽可能缩小客户端的受攻击面。这可以通过“职责分离”和“环境隔离”安全原则来实现。

让敏感功能彼此隔离可减少某个级别的错误导致另一个级别出现数据泄漏的可能性。示例:

- 管理任务不应该与可能将造成入侵的活动合并（例如，管理员的电子邮件中有恶意代码，从而感染基础结构服务器）。
- 用于高敏感性操作的工作站也不应该是用于高风险用途（例如浏览 Internet）的同一系统。

通过删除不必要的软件来减少系统的受攻击面。示例：

- 如果设备的主要用途是管理云服务，则标准的管理、支持或开发工作站都不应该请求安装电子邮件客户端或其他生产力应用程序。

对基础结构组件拥有管理员访问权限的客户端系统应该尽可能受到严格策略的限制，以降低安全风险。示例:

- 安全策略可以包含拒绝设备进行开放访问 Internet 和使用严格防火墙配置的组策略设置。
- 如果需要直接访问，请使用 Internet 协议安全性 (IPsec) VPN。
- 配置不同的管理和开发 Active Directory 域。
- 隔离和筛选管理工作站的网络流量。
- 使用反恶意软件的软件。
- 实施多重身份验证以降低凭据失窃的风险。

合并访问资源并消除不受管理的终结点也可以简化管理任务。


### 为 Azure 远程管理提供安全性

Azure 提供了安全机制来帮助管理员管理 Azure 云服务和虚拟机。这些机制包括：

- 监视、日志记录和审核。
- 证书和加密通信。
- Web 门户。
- 网络数据包筛选。

结合客户端安全配置和管理网关的数据中心部署，可以限制并监视管理员对于云应用程序和数据的访问。

> [AZURE.NOTE] 本文中的某些建议可能会导致数据、网络或计算资源使用量增加，从而增加许可或订阅成本。

## 强化的管理工作站

强化工作站的目标是要去除其他所有功能，只留下其运行所需的最重要功能，尽可能缩小潜在的受攻击面。系统强化包括安装最少量的服务和应用程序、限制应用程序执行、限制网络只能访问所需资源，以及让系统随时保持最新状态。此外，使用经过强化的管理工作站能够将管理工具和活动与其他最终用户任务隔离开来。

在本地企业环境中，可以通过专用的管理网络、必须用身份卡才能进入的服务器机房以及在受保护的网络区域上执行的工作站，限制物理基础结构的受攻击面。在云或混合 IT 模型中，由于无法实际接触到 IT 资源，因此想要让管理服务保持安全将是更复杂的任务。实现保护解决方案需要小心软件设置、安全为主的处理程序及完善的策略。

在锁定的工作站中使用仅需最低特权的最少软件使用量来进行云管理（以及应用程序开发），可以通过将远程管理和开发环境标准化来降低引发安全事件的风险。强化后的工作站配置可通过关闭恶意代码和入侵程序使用的许多常见手段，来帮助避免用于管理重要云资源的帐户遭到入侵。具体而言，可以使用 [Windows AppLocker](http://technet.microsoft.com/zh-cn/library/dd759117.aspx) 和 Hyper-V 技术来控制和隔离客户端系统行为并缓解威胁，包括电子邮件或 Internet 浏览。

在强化后的工作站上，管理员将运行标准用户帐户（它将阻止管理级别的执行），关联的应用程序将由允许列表进行控制。强化后的工作站的基本要素如下：

- 活动的扫描和修补。部署反恶意代码软件，定期执行漏洞扫描，及时使用最新的安全更新来更新所有工作站。
- 有限的功能。卸载任何不需要的应用程序，并禁用不必要的（启动）服务。
- 强化网络。使用 Windows 防火墙规则，仅允许与 Azure 管理相关的有效 IP 地址、端口和 URL。同时确保阻止工作站的入站远程连接。
- 执行限制。仅允许运行一组管理所需的预定义可执行文件（称为“默认拒绝”）。默认情况下，除非已在允许列表中明确定义，否则应该拒绝用户运行任何程序的权限。
- 最低特权。管理工作站的用户不应该拥有本地计算机本身的任何管理特权。这样，他们无法更改系统配置或系统文件（无论是有意或无意）。

通过在 Active Directory 域服务 (AD DS) 中使用组策略对象(GPO)，并通过（本地）管理域将其应用到所有管理帐户，可以强制实施上述所有要素。

### 管理服务、应用程序和数据

可以在 Azure 经典管理门户或 SMAPI 中通过 Windows PowerShell 命令行接口或利用这些 RESTful 接口的自建应用程序来执行 Azure 云服务配置。使用这些机制的服务包括 Azure Active Directory (Azure AD)、Azure 存储空间、Azure 网站和 Azure 虚拟网络，等等。

虚拟机部署的应用程序将根据需要提供自身的客户端工具和界面（例如 Microsoft Management Console (MMC)）、企业管理控制台（例如 Microsoft System Center 或 Windows Intune）或其他管理应用程序（例如 Microsoft SQL Server Management Studio）。这些工具通常驻留在企业环境或客户端网络中。它们可能依赖于需要直接有状态连接的特定网络协议，例如远程桌面协议 (RDP)。有些可能包含不应该通过 Internet 公开发布或访问的具有 Web 功能的接口。

你可以使用[多重身份验证](/documentation/articles/multi-factor-authentication/)、[X.509 管理证书](https://blogs.msdn.microsoft.com/azuresecurity/2015/07/13/certificate-management-in-azure-dos-and-donts/)和防火墙规则来限制访问 Azure 中的基础结构和平台服务管理。Azure 经典管理门户和 SMAPI 需要传输层安全性 (TLS)。但是，部署到 Azure 的服务和应用程序需要根据应用程序采取适当的保护措施。可以通过标准化的强化后工作站配置更轻松地经常启用这些机制。

### 管理网关

若要集中管理所有管理访问权限并简化监视与日志记录，可以在本地网络中部署连接到 Azure 环境的专用[远程桌面网关（RD 网关）](https://technet.microsoft.com/zh-cn/library/dd560672)服务器。

远程桌面网关是基于策略的 RDP 代理服务，可强制实施安全要求。同时实现 RD 网关与 Windows Server 网络访问保护 (NAP)，可帮助确保只有符合 Active Directory 域服务 (AD DS) 组策略对象 (GPO) 创建的特定安全运行状况条件的客户端可以连接。此外：

- 在 RD 网关上预配 [Azure 管理证书](http://msdn.microsoft.com/zh-cn/library/azure/gg551722.aspx)，使它成为可以访问 Azure 经典管理门户的唯一主机。
- 将 RD 网关加入管理员工作站所在的同一个[管理域](http://technet.microsoft.com/zh-cn/library/bb727085.aspx)。当你在具有对 Azure AD 的单向信任的域中使用站点到站点 IPsec VPN 或 ExpressRoute 时，或者要联合本地 AD DS 实例与 Azure AD 之间的凭据时，就必须这样做。
- 配置[客户端连接授权策略](http://technet.microsoft.com/zh-cn/library/cc753324.aspx)，让 RD 网关验证客户端计算机名称是否有效（已加入域）并可以访问 Azure 经典管理门户。
- 针对 [Azure VPN](/documentation/services/vpn-gateway/) 使用 IPsec 以进一步防止管理流量遭到窃听和令牌失窃，或考虑使用通过 [Azure ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) 建立隔离的 Internet 链路。
- 针对通过 RD 网关登录的管理员启用多重身份验证或智能卡身份验证。
- 在 Azure 中配置源 [IP 地址限制](http://azure.microsoft.com/blog/2013/08/27/confirming-dynamic-ip-address-restrictions-in-windows-azure-web-sites/)或[网络安全组](/documentation/articles/virtual-networks-nsg/)以将允许的管理终结点数目降到最低。

## 安全指导原则

一般情况下，帮助保护用于云的管理员工作站的做法，与用于任何本地工作站的做法非常类似 — 例如，最小化生成和限制权限。云管理的某些独特之处更类似于远程或带外企业管理。这些特点包括使用和审核凭据、增强安全的远程访问以及威胁检测和响应。

### 身份验证

可以使用 Azure 登录限制来限制用于访问管理工具的源 IP 地址和审核访问请求。若要帮助 Azure 识别管理客户端（工作站和/或应用程序），可以同时配置 SMAPI（通过客户开发的工具，例如 Windows PowerShell cmdlet）和 Azure 经典管理门户，来要求除了 SSL 证书外，还必须安装客户端管理证书。我们还建议管理员访问需要经过多重身份验证。

部署到 Azure 的某些应用程序或服务可能将针对用户和管理员访问拥有自己的身份验证机制，而其他应用程序或服务则将充分利用 Azure AD。根据是通过 Active Directory 联合身份验证服务 (AD FS)、使用目录同步还是仅在云中维护用户帐户来联合凭据，使用 [Microsoft 标识管理器](https://technet.microsoft.com/zh-cn/library/mt218776.aspx)（Azure AD 高级版中已随附）可帮助管理资源之间的标识生命周期。

### 连接

有多种机制可供帮助保护客户端与 Azure 虚拟网络的连接。在这些机制中，[站点到站点 VPN](https://channel9.msdn.com/series/Azure-Site-to-Site-VPN) (S2S) 和[点到站点 VPN](/documentation/articles/vpn-gateway-howto-point-to-site-classic-azure-portal/) (P2S) 支持使用行业标准 IPsec (S2S) 或[安全套接字隧道协议](https://technet.microsoft.com/magazine/2007.06.cableguy.aspx) (SSTP) (P2S) 来进行加密和隧道传输。当 Azure 连接到面向公众的 Azure 服务管理（例如 Azure 经典管理门户）时，Azure 需要超文本安全传输协议 (HTTPS)。

未通过 RD 网关连接到 Azure 的独立强化工作站应使用基于 SSTP 的点到站点 VPN 来与 Azure 虚拟网络建立初始连接，然后从 VPN 隧道与各个虚拟机建立 RDP 连接。

### 管理审核与策略强制实施

一般而言，有两种方法可用于帮助保护管理程序：审核和策略强制实施。同时采用这两种方法可进行全面控制，但并非所有情况下都能这样做。此外，每种方法在管理安全时都需要不同程度的风险、成本和精力，特别是当它涉及到对个人和系统体系结构给予的信任级别时。

监视、日志记录和审核可为跟踪和了解管理活动提供基础，但受限于所生成的数据量，它不一定都能巨细无遗地审核所有操作。但是，审核管理策略的效果是最佳做法。

包含严格访问控制的策略强制实施具有可控制管理员操作的编程机制，并可帮助确保使用所有可能的保护措施。日志记录可提供强制实施的证明，以及什么人在何时从什么地方执行了什么操作的日志。日志记录还可让你审核和交叉检查管理员如何遵循策略的相关信息，同时提供活动的证据。

## 客户端配置

对于强化的工作站，我们提供三种主要配置。这三者之间最大的差异在于成本、可用性和可访问性，但它们提供的所有选项都有类似的安全设置文件。下表简要分析了各种配置的优点与风险。（请注意，“企业电脑”指的是将为所有域用户（无论角色为何）部署的标准台式机配置）。

| 配置 | 优点 | 缺点 |
| ----- | ----- | ----- |
| 独立的强化工作站 | 受到严格控制的工作站 | 增加专用台式机的成本
| | 降低应用程序被利用的风险 | 增加管理工作 |
| | 明确的职责分离 | |
| 将企业电脑用作虚拟机 | 降低硬件成本 | |
| | 角色和应用程序隔离 | |
| Windows To Go 与 BitLocker 驱动器加密 | 与大部分电脑兼容 | 资产跟踪 |
| | 成本效益和可移植性 | |
| | 隔离的管理环境 | |

必须将强化的工作站用作主机而不是来宾，且主机操作系统与硬件之间没有任何组件。遵循“干净源原则”（也称为“安全来源”）意味着主机应该是最可靠的。否则，强化的工作站（来宾）在其托管所在的系统上容易受到攻击。

可以通过专用系统映像为每一台强化工作站进一步隔离管理功能，使其只具有管理选定的 Azure 和云应用程序所需的工具和权限，以及用于必要任务的特定本地 AD DS GPO。

对于没有任何本地基础结构的 IT 环境（例如，由于所有服务器都在云而不能访问 GPO 的本地 AD DS 实例），[Microsoft Intune](https://technet.microsoft.com/zh-cn/library/jj676587.aspx) 等服务可以简化工作站配置的部署和维护任务。

### 用于管理的独立强化工作站

使用独立的强化工作站时，管理员将使用一台电脑或膝上型计算机来执行管理任务，并使用另一台不同的电脑或膝上型计算机来执行非管理任务。专门负责管理 Azure 服务的工作站不需要安装其他应用程序。此外，使用的工作站如果支持[受信任的平台模块](https://technet.microsoft.com/zh-cn/library/cc766159) (TPM) 或类似的硬件级加密技术，将有助于进行设备身份验证和预防特定攻击。TPM 还可以使用 [BitLocker 驱动器加密](https://technet.microsoft.com/zh-cn/library/cc732774.aspx)来支持系统驱动器的整卷保护。

在独立的强化工作站方案中（如下所示），Windows 防火墙（或非 Microsoft 客户端防火墙）的本地实例将配置为阻止入站连接，例如 RDP。管理员可以登录到强化的工作站，并在与 Azure 虚拟网络建立 VPN 连接之后启动连接到 Azure 的 RDP 会话，但无法登录到企业电脑并使用 RDP 连接到强化的工作站本身。

![][2]

### 将企业电脑用作虚拟机

在部署单个独立强化工作站需要高昂成本或者不方便的情况下，强化的工作站可以托管用于执行非管理任务的虚拟机。

![][3]

若要避免使用单一工作站来进行管理和其他日常工作任务所可能引发的诸多安全风险，可以在强化的工作站部署 Windows Hyper-V 虚拟机。此虚拟机可用作企业电脑。企业电脑环境可以与主机保持隔离，以减少其受攻击面，并使得用户的日常活动（例如电子邮件）不会与机密的管理任务共存。

企业电脑虚拟机将在受保护的空间内运行，并提供用户应用程序。主机仍是“干净源”，并且将在根操作系统中强制实施严格的网络策略（例如，阻止来自虚拟机的 RDP 访问）。

### Windows To Go

需要独立的强化工作站的另一个替代方式是使用 [Windows To Go](https://technet.microsoft.com/zh-cn/library/hh831833.aspx) 驱动器，此功能支持客户端 USB 启动功能。Windows To Go 可让用户将兼容的电脑启动到从加密 USB 闪存驱动器运行的隔离系统映像。由于映像可以完全由企业 IT 团队负责管理、有严格的安全策略、最小的 OS 生成和 TPM 支持，因此 Windows To Go 可以提升对远程管理终结点的控制度。

在下图中，可移动映像是已加入域的系统，它已预配置为仅连接到 Azure、需要多重身份验证并阻止所有非管理流量。如果用户将同一台电脑启动到标准企业映像，并尝试访问 Azure 管理工具的 RD 网关，会话将被阻止。Windows To Go 将成为根级操作系统，并且不需要可能更容易遭受外部攻击的其他层（主机操作系统、虚拟机监控程序、虚拟机）。

![][4]

请务必注意，相比普通的台式机，USB 闪存驱动器更容易丢失。使用 BitLocker 加密整个卷时，如果配合强密码，攻击者就更不可能使用驱动器映像来展开恶意活动。此外，如果丢失 USB 闪存驱动器，则吊销和[颁发新管理证书](https://technet.microsoft.com/zh-cn/library/hh831574.aspx)以及快速重置密码可以降低风险。管理审核日志驻留在 Azure 而非客户端，进一步减少了丢失数据的可能性。

## 最佳实践

管理 Azure 中的应用程序和数据时，请注意以下附加指导原则。

### 准则

不要因为工作站已被锁定，就认为不需要满足其他常见安全要求。由于管理员帐户通常拥有提升权限的访问级别，因此潜在风险将提高。下表显示了风险及其替代安全做法的示例。

| 不要 | 要 |
| ----- | ----- |
| 不要通过电子邮件发送用于管理员访问权限或其他机密数据的凭据（例如 SSL 或管理证书） | 用声音提供帐户名称和密码（但不要将它们存储在语音邮件中）以保持机密性、远程安装客户端/服务器证书（通过加密会话）、从受保护的网络共享下载，或通过可移动媒体手动分发。 |
| | 主动管理你的管理证书生命周期。 |
| 不要在应用程序存储中存储未加密或未哈希处理的帐户密码（例如在电子表格、SharePoint 站点或文件共享中）。 | 创建安全管理策略和系统强化策略，并将它们应用到开发环境。 |
| | 使用 [Enhanced Mitigation Experience Toolkit 5.5](https://technet.microsoft.com/security/jj653751) 证书绑定规则，以确保能够正常访问 Azure SSL/TLS 站点。 |
| 不要在管理员之间共享帐户和密码，或在多个用户帐户或服务之间重复使用密码，特别是用于社交媒体或其他非管理活动的帐户或服务。 | 创建专用的 Microsoft 帐户来管理 Azure 订阅 — 此帐户不用于个人电子邮件。 |
| 不要通过电子邮件发送配置文件。 | 应该从受信任的源（例如，加密的 USB 闪存驱动器）而不是从可轻易入侵的机制（例如电子邮件）安装配置文件和档案。 |
| 不要使用弱密码或简单的登录密码。 | 强制实施强密码策略、过期周期（首次使用时更改）、控制台超时和自动帐户锁定。使用客户端密码管理系统配合多重身份验证来访问密码保管库。 |
| 不要在 Internet 上公开管理端口。 | 锁定 Azure 端口和 IP 地址以限制管理访问权限。有关详细信息，请参阅 [Azure Network Security](http://download.microsoft.com/download/4/3/9/43902EC9-410E-4875-8800-0788BE146A3D/Windows%20Azure%20Network%20Security%20Whitepaper%20-%20FINAL.docx)（Azure 网络安全性）白皮书。 |
| | 针对所有管理连接使用防火墙、VPN 和 NAP。 |

## Azure 操作

在 Microsoft 的 Azure 操作中，访问 Azure 的生产系统的操作工程师和支持人员将使用强化的工作站电脑与其中预配的 VM 来进行内部企业网络访问和运行应用程序（例如电子邮件、Intranet 等）。所有管理工作站计算机都装有 TPM，主机启动驱动器已使用 BitLocker 加密，并且已加入 Microsoft 主要企业域中的特殊组织单位 (OU)。

系统强化是通过组策略以集中式软件更新来强制实施的。为了审核和分析，将从管理工作站收集事件日志（例如安全性和 AppLocker）并将其保存到中心位置。

此外，将使用 Microsoft 网络上需要双重身份验证的专用 Jumpbox 来连接到 Azure 的生产网络。

## Azure 安全清单

将管理员在强化的工作站上可以执行的任务数降到最低，将有助于尽量降低开发和管理环境中的受攻击面。请使用以下技术来帮助保护强化的工作站：

- IE 强化。由于 Internet Explorer 浏览器（或任何类似的 Web 浏览器）将与外部服务器广泛交互，因此是恶意代码的主要入口点。请查看客户端策略并强制要求在保护模式下运行、禁用附加组件、禁用文件下载，并使用 [Microsoft SmartScreen](https://technet.microsoft.com/zh-cn/library/jj618329.aspx) 筛选。确保显示安全警告。利用 Internet 区域，并创建已为其配置合理强化的受信任站点列表。阻止其他所有站点和浏览器内代码，例如 ActiveX 和 Java。
- 标准用户。以标准用户的身份运行有许多好处，最重要的好处是通过恶意代码窃取管理员凭据将变得更困难。此外，标准用户帐户对根操作系统没有提升的权限，并且许多配置选项和 API 已按默认锁定。
- AppLocker。可以使用 [AppLocker](http://technet.microsoft.com/zh-cn/library/ee619725.aspx) 来限制用户可以运行的程序和脚本。可以在审核或强制模式下运行 AppLocker。默认情况下，AppLocker 的允许规则可让具有管理员令牌的用户运行客户端上的所有代码。设置此规则是为了避免管理员将自己锁定，并且只应用到提升权限的令牌。另请参阅 Windows Server [核心安全性](http://technet.microsoft.com/zh-cn/library/dd348705.aspx)中的“代码完整性”。
- 代码签名。为管理员使用的所有工具和脚本进行代码签名可提供方便管理的机制来部署应用程序锁定策略。哈希不会随着代码的快速更改而做出调整，并且文件路径不会提供高度安全性。你应该将 AppLocker 规则与 PowerShell [执行策略](http://technet.microsoft.com/zh-cn/library/ee176961.aspx)合并，此策略只允许[执行](http://technet.microsoft.com/zh-cn/library/hh849812.aspx)特定的已签名代码和脚本。
- 组策略。创建一个全局管理策略，该策略将应用到任何用于管理的域工作站（并阻止来自其他所有用途的访问），以及在这些工作站上进行身份验证的用户帐户。
- 安全性增强的预配。保护基线强化工作站映像以防遭到篡改。使用加密和隔离等安全措施来存储映像、虚拟机和脚本，并限制访问（也许可以使用可审核的签入/签出过程）。
- 修补。维护一致的生成（或针对开发、操作和其他管理任务使用不同的映像）、定期扫描更改和恶意代码、让生成保持最新状态，并且只在需要时才激活计算机。
- 加密。确保管理工作站装有 TPM 以便能够更安全地启用[加密文件系统](https://technet.microsoft.com/zh-cn/library/cc700811.aspx) (EFS) 和 BitLocker。如果使用 Windows To Go，请只配合 BitLocker 使用加密的 USB 密钥。
- 监管。使用 AD DS GPO 来控制所有管理员的 Windows 接口，例如文件共享。将管理工作站纳入审核、监视和日志记录过程中。跟踪所有管理员和开发人员的访问和使用活动。

## 摘要

使用强化的工作站配置来管理 Azure 云服务、虚拟机和应用程序，可帮助避免远程管理关键 IT 基础结构所造成的众多风险和威胁。Azure 和 Windows 提供了相关机制来帮助保护和控制通信、身份验证与客户端行为。

## 后续步骤
除了本文中所提到的特定项以外，以下资源也提供了有关 Azure 及相关 Microsoft 服务的更多常规信息：

- [Securing Privileged Access（保护特权访问）](https://technet.microsoft.com/zh-cn/library/mt631194.aspx)– 获取有关设计和构建安全管理工作站以管理 Azure 的技术详细信息
- [Microsoft 信任中心](https://www.microsoft.com/TrustCenter/Security/AzureSecurity) - 了解可保护 Azure 结构以及在 Azure 上运行的工作负荷的 Azure 平台功能
- [Microsoft 安全响应中心](http://www.microsoft.com/security/msrc/default.aspx) -- 可在其中报告或通过电子邮件发送到 [secure@microsoft.com](mailto:secure@microsoft.com) 的 Microsoft 安全漏洞（包括 Azure 问题）
- [Azure 安全博客](http://blogs.msdn.com/b/azuresecurity/) – 随时掌握 Azure 安全性的最新信息

<!--Image references-->
[1]: ./media/azure-security-management/typical-management-network-topology.png
[2]: ./media/azure-security-management/stand-alone-hardened-workstation-topology.png
[3]: ./media/azure-security-management/hardened-workstation-enabled-with-hyper-v.png
[4]: ./media/azure-security-management/hardened-workstation-using-windows-to-go-on-a-usb-flash-drive.png

<!---HONumber=Mooncake_0523_2016-->