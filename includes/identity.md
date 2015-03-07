# Azure 标识

在公有云中与在本地管理标识一样重要。为此，Azure 支持多种不同的云标识技术。它们包括：

- 你可以使用通过 Azure 虚拟机技术创建的虚拟机在云中运行 Windows Server Active Directory（通常简称为 AD）。此方法在你使用 Azure 将本地数据中心扩展到云中时非常有用。

- 你可以使用 Azure Active Directory 让你的用户通过单一登录访问"软件即服务"(SaaS) 应用程序。例如，Microsoft 的 Office 365 使用此技术，而在 Azure 或其他云平台上运行的应用程序也可以使用此技术。

- 在云中或本地运行的应用程序可以使用 Azure Active Directory Access Control 让用户使用 Facebook、Google、Microsoft 和其他标识提供者的标识进行登录。


本文将一一介绍这三种方式。

## 目录

- [在 VM 中运行 Windows Server Active Directory](#adinvm)

- [使用 Azure Active Directory](#ad)

- [使用 Azure Active Directory Access Control](#ac)


## <a name="adinvm"></a>在 VM 中运行 Windows Server Active Directory

在 Azure VM 中运行 Windows Server AD 与在本地运行它非常类似。[图 1](#fig1) 显示了这种情况的典型示例。

![Azure Active Directory in Virtual Machine](./media/identity/identity_01_ADinVM.png)


<a name="Fig1"></a>图 1：Windows Server Active Directory 可在使用 Azure 虚拟网络连接到组织的本地数据中心的 Azure VM 中运行。

在此处显示的示例中，Windows Server AD 在使用 Azure 虚拟机（平台的 IaaS 技术）创建的 VM 中运行。这些 VM 和其他一些 VM 分组到使用 Azure 虚拟网络技术连接到本地数据中心的虚拟网络 (VNET)。该 VNET 将创建一个包含云 VM 的组，这些云 VM 通过虚拟专用网络 (VPN) 连接与本地网络交互。这样做可让这些 Azure VM 看起来就像是连接到本地数据中心的另一个子网一样。如图所示，其中两个 VM 在 Windows Server AD 域控制器中运行。VNET 中的其他 VM 可能运行应用程序（如 SharePoint），或者用于其他目的，例如用于开发和测试。本地数据中心也可以运行两个 Windows Server AD 域控制器。

可采用以下方法将云中的这些域控制器与在本地运行的域控制器相连接。它们包括：

- 让每个域控制器都成为某个 Active Directory 域的一部分。

- 在属于同一林的本地和云平台中分别创建单独的 AD 域。

- 在云中和本地创建单独的 AD 林，然后使用跨林信任或 Windows Server Active Directory 联合身份验证服务 (AD FS) 连接这些林，这些林也可在 Azure 上的 VM 中运行。

无论选择哪种方法，管理员都应确保来自本地用户的身份验证请求仅在必要时才会转至云域控制器，因为指向云的链接很可能比本地网络慢。考虑连接云和本地域控制器的另一个因素是复制产生的流量。云中的域控制器通常位于其自己的 AD 站点中，可允许管理员规划复制的频率。Azure 会对 Azure 数据中心发出的流量进行收费，虽然这不适用于发送的字节，但可能会影响管理员的复制选择。同样要指出的是，虽然 Azure 提供其自己的域名系统 (DNS) 支持，但此服务缺少 Active Directory 需要的功能（例如对动态 DNS 和 SRV 记录的支持）。因此，在 Azure 上运行 Windows Server AD 需要在云中设置你自己的 DNS 服务器。

在 Azure VM 上运行 Windows Server AD 在几种不同的情况下是很有意义的。下面是一些示例：

- 如果你将 Azure 虚拟机用作你自己数据中心的一个扩展，则你可以在云中运行需要本地域控制器来处理诸如 Windows 集成身份验证请求或 LDAP 查询事宜的应用程序。例如，SharePoint 频繁与 Active Directory 交互，因此，虽然你可能使用本地目录在 Azure 中运行 SharePoint 服务器场，但在云中设置域控制器可显著提高性能。（但这并不是必需的，了解这一点非常重要；很多应用程序仅使用本地域控制器即可在云中成功运行。）

- 假设遥远的分支办公室缺乏运行其自己的域控制器的资源。现今，其用户必须向在世界另一端的域控制器进行身份验证，从而导致登录速度缓慢。在较近的 Microsoft 数据中心中的 Azure 上运行 Active Directory 可加快此速度，并且无需在该分支办公室拥有更多服务器。

- 将 Azure 用于灾难恢复的组织可能在云中维护一小组活动 VM，包括域控制器。这样可根据需要随时准备扩展此站点来接管其他地方的故障。

也还有其他可能的情况。例如，你不需要将云中的 Windows Server AD 连接到本地数据中心。如果你要运行服务于特定用户组（如全部用户将使用基于云的标识单独登录）的 SharePoint 服务器场，则你可能要在 Azure 上创建一个独立林。使用此技术的方式取决于您的目标。（有关将 Windows Server AD 与 Azure 结合使用的更多详细指南，请 [参见此处](http://msdn.microsoft.com/zh-cn/library/azure/jj156090.aspx)。）

## <a name="ad"></a>使用 Azure Active Directory

随着 SaaS 应用程序越来越常见，它们产生了一个明显的问题：这些基于云的应用程序应当使用哪类目录服务？Microsoft 对该问题的回答是 Azure Active Directory。

在云中主要通过两种方式来使用此目录服务：

- 只使用 SaaS 应用程序的个人和组织可将 Azure Active Directory 作为其唯一的目录服务。

- 运行 Windows Server Active Directory 的组织可将其本地目录连接到 Azure Active Directory，然后使用此目录服务让用户通过单一登录访问 SaaS 应用程序。


[图 2](#fig2) 演示了这两种方式中的第一种，即，只使用 Azure Active Directory。

![Azure Active Directory in Virtual Machine](./media/identity/identity_02_AD.png)

<a name="fig2"></a>图 2：Azure Active Directory 可使组织的用户通过单一登录访问 SaaS 应用程序，包括 Office 365。

如图所示，Azure AD 是多租户服务。这意味着它能同时支持多个不同的组织，可存储每个组织中用户的目录信息。在此示例中，组织 A 中的用户正在尝试访问 SaaS 应用程序。此应用程序可能是 Office 365 的一部分（如 SharePoint Online），或者可能是其他应用程序，非 Microsoft 应用程序也可使用此技术。由于 Azure AD 支持 SAML 2.0 协议，因此应用程序所需的全部就是能够使用此行业标准进行交互。（事实上，使用 Azure AD 的应用程序可在任何数据中心运行，不仅仅是 Azure 数据中心。）

当用户访问 SaaS 应用程序时，将会开始此过程（步骤 1）。若要使用此应用程序，用户必须提供由 Azure AD 颁发的令牌。

此令牌包含标识用户的信息，由 Azure AD 进行数字签名。若要获得令牌，用户要通过提供用户名和密码向 Azure AD 验证自己的身份（步骤 2）。这样，Azure AD 即会返回用户所需的令牌（步骤 3）。

然后，此令牌将发送至 SaaS 应用程序（步骤 4），应用程序将验证该令牌的签名并使用其内容（步骤 5）。通常情况下，应用程序会使用令牌包含的标识信息来决定允许用户访问的信息内容，但也可能使用其他方式。

如果应用程序需要更多用户信息，而不仅仅是令牌中所含的信息，则应用程序可以使用 Azure AD 图形 API 直接从 Azure AD 请求相关信息（步骤 6）。在 Azure AD 的初始版本中，目录架构非常简单：它仅包含用户和组以及它们之间的关系。应用程序可使用此信息来了解用户之间的连接。例如，假设应用程序需要知道此用户的经理是谁，以确定是否允许他访问某块数据。应用程序可通过图形 API 查询 Azure AD 了解此信息。

图形 API 使用普通的 RESTful 协议，该协议可让大部分客户端（包括移动设备）轻松使用它。API 还支持由 OData 定义的扩展，以便通过添加查询语言等内容来让客户端以更多有效方式访问数据。（有关 OData 的详细信息，请参阅 [OData 简介](http://download.microsoft.com/download/E/5/A/E5A59052-EE48-4D64-897B-5F7C608165B8/IntroducingOData.pdf)。）由于图形 API 可用于了解用户之间的关系，因此，它能让应用程序了解嵌入特定组织的 Azure AD 架构中的社交图（这就是将其称为图形 API 的原因）。要针对图形 API 请求向 Azure AD 验证自己的身份，应用程序使用了 OAuth 2.0。

如果组织不使用 Windows Server Active Directory（即没有本地服务器或域）并且仅依赖使用 Azure AD 的云应用程序，则仅使用此云目录可让公司的用户通过单一登录访问所有这些应用程序。然而，尽管这种情况日益常见，但大部分组织仍然使用 Windows Server Active Directory 创建的本地域。Azure AD 在这里还扮演着有用的角色，如 [图 3](#fig3) 所示。

![Azure Active Directory in Virtual Machine](./media/identity/identity_03_AD.png)
<a id="fig3"></a>图 3：组织可将 Windows Server Active Directory 与 Azure Active Directory 联合，从而让其用户通过单一登录访问 SaaS 应用程序。

在这种情况下，组织 B 中的用户希望访问 SaaS 应用程序。在她执行此操作之前，组织的目录管理员必须使用 AD FS 与 Azure AD 建立联合关系，如图所示。这些管理员还必须在组织的本地 Windows Server AD 与 Azure AD 之间配置数据同步。这样即会自动将用户和组信息从本地目录复制到 Azure AD。请注意此操作的结果：实际上，组织正在将其本地目录扩展到云中。按照此方式将 Windows Server AD 与 Azure AD 相结合为组织提供了一种可作为单个实体管理的目录服务，同时使它们依旧在本地和云中发挥作用。

若要使用 Azure AD，用户首先需像往常一样登录其本地 Active Directory 域（步骤 1）。当她尝试访问 SaaS 应用程序（步骤 2）时，联合过程会导致 Azure AD 为她颁发此应用程序的令牌（步骤 3）。（有关联合工作原理的更多信息，请参见 [Windows 基于声明的标识：技术和方案](http://www.davidchappell.com/writing/white_papers/Claims-Based_Identity_for_Windows_v3.0--Chappell.docx)。）如前所述，此令牌包含标识用户的信息，并由 Azure AD 进行数字签名。然后，此令牌将发送至 SaaS 应用程序（步骤 4），应用程序将验证该令牌的签名并使用其内容（步骤 5）。在上一种情况下，SaaS 应用程序可使用图形 API 来了解有关用户的更多信息（如有需要）（步骤 6）。

现今，Azure AD 不能完全取代本地 Windows Server AD。正如已提到的那样，云目录拥有更加简单的架构，但同时也缺少一些功能，如组策略、存储有关计算机信息的能力和对 LDAP 的支持。（事实上，Windows 计算机不能配置为让用户只使用 Azure AD 进行登录，这是不受支持的方案。）相反，Azure AD 的初始目标包括让企业用户访问云中的应用程序，无需维护单独的登录，无需本地目录管理员手动将其本地目录与组织使用的每个 SaaS 应用程序进行同步。不过，随着时间的推移，期望此云目录服务能够处理更多的情况。

## <a name="ac"></a>使用 Azure Active Directory Access Control

基于云的标识技术可用于解决各种问题。例如，Azure Active Directory 可让组织的用户通过单一登录访问多个 SaaS 应用程序。但云中的标识技术还可用于其他方面。

例如，假设某个应用程序希望让用户使用由多个 *identity providers (IdPs)* 颁发的令牌进行登录。现今有很多不同的标识提供者（包括 Facebook、Google、Microsoft 和其他标识提供者），应用程序通常会让用户使用其中一种标识进行登录。为什么应用程序在能够依赖已有标识的情况下还要费力维护自己的用户和密码列表呢？接受现有标识可以让那些拥有很少要记住的用户名和密码的用户以及那些创建应用程序、不再需要维护自己的用户名和密码列表的人员的工作变得更简单。

但是，每个标识提供者所颁发的某些类型的令牌并不标准，因为每个 IdP 都采用其自己的格式。而且，这些令牌中的信息也不标准。希望接受由 Facebook、Google 和 Microsoft 等颁发的令牌的应用程序将面临需要编写唯一代码来处理这些不同格式的问题。

但为什么要这样做呢？为什么不改为创建一个中介，让它生成一种包含标识信息的通用表示方式的令牌格式呢？此方法可使创建应用程序的开发人员的工作变得更加简单，因为现在他们只需处理一种令牌。Azure Active Directory Access Control 正好可以实现此目的，它在云中提供了一个中介来处理各种令牌。[图 4](#fig4) 演示了 Access Control 的工作原理

![Azure Active Directory in Virtual Machine](./media/identity/identity_04_IdentityProviders.png) 
<a id="fig4"></a>图 4：Azure Active Directory Access Control 使应用程序接受不同标识提供者颁发的标识令牌变得更加容易。

当某用户尝试从浏览器访问此应用程序时，将会开始此过程。应用程序会将她重定向到她所选择且应用程序也信任的 IdP。她将向此 IdP 验证自己的身份，例如，通过输入用户名和密码（步骤 1），然后此 IdP 将返回包含有关她的信息的令牌（步骤 2）。

如图所示，Access Control 支持一组基于云的不同 IdP，包括由 Google、Yahoo、Facebook、Microsoft（以前称为 Windows Live ID）创建的帐户以及任何 OpenID 提供程序。它还支持使用 Azure Active Directory，以及通过与 AD FS、Windows Server Active Directory 进行联合创建的标识。其目标是覆盖目前最常用的标识，无论这些标识是由 IdP 在云中还是在本地颁发。

一旦用户的浏览器获得她所选择的 IdP 颁发的 IdP 令牌后，浏览器即会将此令牌发送至 Access Control（步骤 3）。Access Control 验证此令牌，确保它确实是由此 IdP 颁发，然后根据已为此应用程序定义的规则创建新的令牌。与 Azure Active Directory 一样，Access Control 是一种多租户服务，但租户是应用程序，而不是客户组织。每个应用程序均可获得自己的命名空间（如图所示），并可定义有关身份验证等内容的各种规则。

这些规则可使每个应用程序的管理员定义如何将各个 IdP 颁发的令牌转换为 Access Control 令牌。例如，如果不同的 IdP 使用不同的类型表示用户名，则 Access Control 规则可将所有这些类型转换为通用的用户名类型。Access Control 然后会将此新的标记发送回浏览器（步骤 4），浏览器再将其提交给应用程序（步骤 5）。应用程序获得 Access Control 令牌之后，将会确认此令牌确实是由 Access Control 颁发的，然后使用此令牌中包含的信息（步骤 6）。

此过程看起来可能有一点复杂，但实际上可以极大地简化应用程序创建者的工作。应用程序可接受多个标识提供者颁发的标识，同时仍然接收包含已知信息的单个令牌，而不必处理包含不同信息的各种令牌。此外，不需要将每个应用程序配置为信任各个 IdP，因为这些信任关系已改为由 Access Control 维护，应用程序只需信任 Access Control 即可。

需要指出的是，Access Control 的任何方面均不依赖于 Windows，它同样可由仅接受 Google 和 Facebook 标识的 Linux 应用程序使用。虽然 Access Control 属于 Azure Active Directory 系列，但你可以将其看作与上一节所介绍服务完全不同的服务。虽然两种技术均用于处理标识，但它们解决的问题截然不同（尽管 Microsoft 曾经表示希望在某些时刻将这两种技术集成）。

处理标识在几乎所有应用程序中都很重要。Access Control 的目的是使开发人员能够更轻松地创建接受来自不同标识提供者的标识的应用程序。通过在云中部署此服务，Microsoft 让运行在任何平台上的任何应用程序都可使用此服务。

##关于作者

David Chappell 是位于加利福尼亚州旧金山市的 Chappell & Associates [www.davidchappell.com](http://www.davidchappell.com) 的负责人。他通过演讲、写作和咨询，帮助全球的人们了解、使用新技术并围绕新技术做出更好的决策。
<!--HONumber=43--> 
