#Azure 安全指南

##摘要

在开发 Azure 应用程序时，身份识别和访问控制是您必须关注的主要安全问题。
本主题说明与云中的身份识别和访问控制相关的重要安全问题，以及您如何可以更好地保护您的云应用程序。

##概述

应用程序的安全性是其一项外围应用功能。应用程序公开的外围应用越多，安全问题就越大。例如，从安全角度看，作为无人参与批处理过程运行的应用程序的公开面比对外发布的 Web 应用要少。

在移动到云时，您在一定程度上可以对基础结构和网络放心，因为我们采用世界一流的安全实践、工具和技术在数据中心中管理它们。另一方面，云平台本身会公开您的应用程序中可能会被攻击者利用的多个外围应用。这是因为在公开许多云技术和服务时采用了终结点与内存中组件的形式。Azure 存储、服务总线、SQL
数据库（以前称为 SQL Azure）和其他许多服务都可通过其终结点进行在线访问。

在云应用程序中，应用程序开发人员需要肩负起更多的责任，他们需要按照高安全性标准设计、开发和维护其云应用程序以阻止攻击者靠近。
请参考以下图表（摘自 J.D.Meier 的 [Azure 安全
说明 PDF](http://blogs.msdn.com/b/jmeier/archive/2010/08/03/now-available-azure-security-notes-pdf.aspx))：请注意云提供程序（在本例中为 Azure）如何处理基础结构部分，
从而将更多安全工作留给应用程序开发人员：

![Securing the application][01]

好消息是在开发云应用程序时，您已熟知的所有安全开发做法、原则和技术仍适用。请考虑以下必须处理的重要事项：

-   验证所有输入的类型、长度、范围和字符串模式以避免注入攻击，并且应适当净化应用回显的所有数据。
-   应以静态和在线方式保护敏感数据，以缓解信息泄露和数据篡改威胁。
-   必须适当进行审核和日志记录以缓解不可否认性威胁。
-   应使用平台提供的经验证机制实现身份验证和授权，以防止身份欺骗和特权提升威胁。

有关威胁、攻击、漏洞和对策的完整列表，请参考"模式和实践"中的[备忘单：Web
应用程序安全框架](http://msdn.microsoft.com/zh-cn/library/ff649461.aspx) and [Security Guidance for Applications Index](http://msdn.microsoft.com/zh-cn/library/ff650760.aspx).

在云中，身份验证和访问控制机制与可用于本地应用程序的这些机制大不相同。为云应用程序提供了更多身份验证和访问选项，这可能会导致混乱，进而导致实现中出现问题。在定义云应用的含义时，可能会引起更多混乱。例如，可以将应用程序部署到云，而其身份验证机制却由 Active Directory 提供。另一方面，应用程序可在本地进行部署，但方式是通过云中的身份验证机制（例如，通过 Azure
Active Directory 访问控制（以前称为访问控制
服务或 ACS））。

##威胁、漏洞和攻击

威胁是要避免的潜在坏结果，例如敏感信息泄露或服务变得不可用。
常见做法是使用首字母缩写词"STRIDE"对威胁进行分类：

-   **S**，表示欺骗或身份盗用
-   **T**，表示篡改数据
-   **R**，表示拒绝操作
-   **I**，表示信息泄露
-   **D**，表示拒绝服务
-   **E**，表示特权提升

漏洞是开发人员引入代码中的错误，使应用程序可被攻击者利用。例如，以明文形式发送敏感数据可能使攻击者发起流量探查攻击，从而导致信息泄露威胁。

攻击是指利用这些漏洞对应用程序造成损坏。例如，跨站点脚本（或 XSS）是一种利用未净化输出的攻击。另一个示例是通过在线窃听捕获以明文形式发送的凭据。这些攻击可能会导致身份欺骗威胁的产生。为简单起见，我们将威胁、漏洞和攻击视为坏事情。
若要大致了解与 Web 应用部署到 Azure 相关的坏事情，请参考以下图表（摘自 J.D.Meier 的
[Azure 安全说明 PDF](http://blogs.msdn.com/b/jmeier/archive/2010/08/03/now-available-azure-security-notes-pdf.aspx)):

![Threats Vulnerabilities and Attacks][02]

作为一名开发人员，您可以控制漏洞。您引入的漏洞越少，留给攻击者利用的选项就越少。

与身份识别和访问控制相关的漏洞使您易受到 STRIDE 模型中所有威胁的攻击。例如，执行不当的身份验证机制可能会导致身份欺骗，进而导致信息泄露、数据篡改、提升特权操作或甚至完全关闭服务。请考虑以下问题，它们可能指向您的云应用的身份识别和访问控制实现中的潜在漏洞：

-   您是否以明文形式在线将凭据发送到自己的 Azure 服务？
-   您是否以明文形式存储 Azure 服务凭据？
-   您的 Azure 服务凭据是否遵循强密码策略？
-   您是依赖 Azure 来验证凭据还是使用自定义验证机制？
-   您是否将 Azure 服务身份验证会话或令牌生存期限制为合理的时间范围？
-   您是否验证您的分布式云应用的每个 Azure 入口点的权限？
-   您的授权机制是否不安全，而您没有公开敏感信息或没有允许无限访问？

##对策

抵御攻击的最好方法是使用平台提供的身份识别和访问控制机制，而不要实施您自己的机制。可考虑采用以下出色的身份识别和访问控制技术：

**Windows Identity Foundation (WIF)。**WIF 是 .NET Framework 4.5 附带的 .NET 运行时库（在 .NET 3.5/4.0 中也可单独下载它）。WIF 在处理协议（例如 WS 联合身份验证和 WS-Trust）和处理令牌（例如安全声明标记语言 (SAML)）方面发挥着重要作用，
因此您无需在自己的应用程序中编写十分复杂的安全相关代码。以下资源提供了有关 WIF 的详细信息：

-   [Windows Identity Foundation 4.5 示例](http://code.msdn.microsoft.com/site/search?f%5B0%5D.Type=SearchText&f%5B0%5D.Value=wif&f%5B1%5D.Type=Topic&f%5B1%5D.Value=claims-based%20authentication) MSDN 代码库上的。
-   [适用于 Visual Studio 11 Beta 的 Windows Identity Foundation 4.5 工具](http://visualstudiogallery.msdn.microsoft.com/e21bf653-dfe1-4d81-b3d3-795cb104066e) on
    MSDN 代码库。
-   [Windows Identity Foundation 运行时 (.Net 3.5/4.0)](http://www.microsoft.com/zh-cn/download/details.aspx?id=17331) on MSDN.
-   [Windows Identity Foundation 3.5/4.0 示例和 Visual Studio 2008/2010 模板](http://www.microsoft.com/zh-cn/download/details.aspx?displaylang=en&id=4451) MSDN 上的。

**Azure AD 访问控制（以前称为 ACS）**。
Azure AD 访问控制是一项云服务，用于提供安全令牌
服务 (STS) 并允许与不同的标识提供程序 (IdP)（例如企业 Active Directory）或 Internet IdP（例如
Windows Live ID/Microsoft 帐户、Facebook、Google、Yahoo! 和 Open
ID 2.0 标识提供程序）进行联合。以下资源提供有关 Azure AD 访问控制的详细信息：

-   [访问控制服务 2.0](https://msdn.microsoft.com/zh-CN/library/gg429786.aspx) 
-   [使用 ACS 的方案和解决方案](http://msdn.microsoft.com/zh-cn/library/gg185920.aspx)
-   [ACS 操作指南](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg185939.aspx)
-   [基于声明的身份识别和访问控制指南](http://msdn.microsoft.com/zh-cn/library/ff423674.aspx)
-   [标识开发人员培训工具包](http://www.microsoft.com/zh-cn/download/details.aspx?id=14347)
-   [MSDN 托管的标识开发人员培训课程](http://msdn.microsoft.com/zh-cn/IdentityTrainingCourse)

**Active Directory 联合身份验证服务 (AD FS)。**Active Directory
联合身份验证服务 (AD FS) 2.0 为采用 Windows Server?? 和 Active Directory 技术的声明感知标识解决方案提供支持。AD FS 2.0 支持 WS-Trust、WS 联合身份验证和 SAML 协议。以下资源提供了有关 AD
FS 的详细信息：

-   [AD FS 2.0 内容映射](http://social.technet.microsoft.com/wiki/contents/articles/2735.ad-fs-2-0-content-map.aspx)
-   [Web SSO 设计][Web SSO 设计]
-   [联合 Web SSO 设计][联合 Web SSO 设计]

**Azure 共享访问签名。**使用共享访问签名，您可以微调对 Blob 或容器资源的访问权限。以下资源提供有关共享访问
签名的详细信息。

-   [管理对 Blob 和容器的访问权限](http://msdn.microsoft.com/zh-cn/library/ee393343.aspx)
-   [新存储功能：共享访问签名（可能为英文页面）](http://blog.smarx.com/posts/new-storage-feature-signed-access-signatures)
-   [共享访问签名现在很容易实现](http://blog.smarx.com/posts/shared-access-signatures-are-easy-these-days)

##方案地图

本节简要概述了该主题中涵盖的关键方案。
可将它用作指引来确定适合您的应用程序的标识解决方案。

-   **使用联合身份验证的 ASP.NET Web 表单应用程序。**在此方案中，可使用 Internet 标识（例如 Live ID/Microsoft 帐户）或 Windows Server Active Directory 中管理的企业标识来控制对您的 ASP.NET Web 表单应用的访问权限。
-   **使用联合身份验证的 WCF (SOAP) 服务。**在此方案中，可使用由 Azure AD 访问控制管理的服务标识来控制对您的 WCF (SOAP) 服务的访问权限。
-   **使用联合身份验证、Active Directory 中标识的 WCF (SOAP) 服务。**在此方案中，可使用由企业 Windows Server Active Directory 管理的标识来控制对您的 WCF (SOAP) Web 服务的访问权限。
-   **使用联合身份验证的 WCF (REST) 服务。**在此方案中，可使用由 Azure AD 访问控制管理的服务标识来控制对您的 WCF (REST) 服务的访问权限。
-   **使用 Live ID/Microsoft 帐户、Facebook、Google、Yahoo! 和 Open ID 的 WCF (REST) 服务。**在此方案中，可使用 Internet 标识（例如 Live ID/Microsoft 帐户）来控制对您的 WCF (REST) 服务的访问权限。
-   **在 ASP.NET Web 应用和 REST WCF 服务之间使用共享 SWT 令牌。**在此方案中，您具有包含前端 ASP.NET Web 应用和下游 REST 服务的分布式应用程序，并且您需要通过物理层传送最终用户的上下文。
-   **声明感知应用程序和服务中的基于角色的访问控制 (RBAC) 授权。**在此方案中，您需要在应用中实现基于角色的授权逻辑。
-   **声明感知应用程序和服务中的基于声明的授权。**在此方案中，您需要在应用中实现基于复杂授权规则的授权逻辑。
-   **Azure 存储服务身份识别和访问控制方案。**在此方案中，您需要安全地共享对 Azure 存储 Blob 和容器的访问权限。
-   **Azure SQL 数据库 身份识别和访问控制方案。**SQL 数据库 仅支持 SQL Server 身份验证。不支持 Windows 身份验证（集成安全性）。用户在每次连接到 SQL 数据库 时都必须提供凭据（登录名和密码）。
-   **Azure 服务总线身份识别和访问控制方案。**在此方案中，您需要安全地访问 Azure 服务总线队列。
-   **内存中的缓存身份识别和访问控制方案。**在此方案中，您需要安全地访问内存中缓存所管理的数据。
-   **Azure Marketplace 身份识别和访问控制方案。**在此方案中，您需要安全地访问 Azure Marketplace 数据集。

##Azure 身份识别和访问控制方案

本节概述了适用于不同应用程序体系结构的常见身份识别和访问控制方案。可使用方案地图进行快速定位。

###使用联合身份验证的 ASP.NET Web 表单应用程序

在此应用程序体系结构方案中，可以将您的 Web 应用部署到 Azure 或本地。该应用程序需要企业 Active Directory 或
Internet 标识提供程序对其用户进行身份验证。

若要实施此方案，可使用 Azure AD 访问控制和 Windows
Identity Foundation。

![Azure Active Directory Access control][03]

请参见以下资源来实施此方案：

-   [如何：使用 ACS 创建我的第一个声明感知 ASP.NET 应用程序](http://msdn.microsoft.com/zh-cn/library/gg429779.aspx)
-   [如何：在 ASP.NET Web 应用中托管登录页面](http://msdn.microsoft.com/zh-cn/library/gg185926.aspx)
-   [如何：使用 WIF 和 ACS 在声明感知 ASP.NET 应用程序中实现声明授权](http://msdn.microsoft.com/zh-cn/library/gg185907.aspx)    
-   [如何：使用 WIF 和 ACS 在声明感知 ASP.NET 应用程序中实现基于角色的访问控制 (RBAC)](http://msdn.microsoft.com/zh-cn/library/gg185914.aspx)
-   [如何：使用 X.509 证书配置 ACS 和 ASP.NET Web 应用之间的信任](http://msdn.microsoft.com/zh-cn/library/gg185947.aspx)
-   [代码示例：ASP.NET 简单表单](http://msdn.microsoft.com/zh-cn/library/gg185938.aspx)

###使用服务标识的 WCF (SOAP) 服务

在此应用程序体系结构方案中，可以将您的 WCF (SOAP) 服务部署到 Azure 或本地。该服务将作为下游服务供 Web 应用或其他 Web 服务访问。您需要使用应用程序特定标识来控制对它的访问权限。可根据您在 IIS 中使用的应用池帐户的类型来考虑要使用的标识 - 虽然技术不同，但方法类似，因为访问此服务时使用的是应用程序作用域帐户与最终用户帐户。

使用 Azure AD 访问控制中的服务标识功能。
这类似于您在将应用部署到 Windows Server 和 IIS 时所使用的 IIS 应用池帐户。配置 Azure
AD 访问控制以在 WCF (SOAP) 服务上颁发将由 WIF 处理的 SAML 令牌。

![WCF (SOAP) Service][04]


请参见以下资源来实施此方案：

-   [如何：添加具有 X.509 证书、密码或对称密钥的服务标识](http://msdn.microsoft.com/zh-cn/library/gg185924.aspx)
-   [如何：使用客户端证书进行身份验证以访问受 ACS 保护的 WCF 服务](http://msdn.microsoft.com/zh-cn/library/hh289316.aspx)
-   [如何：使用用户名和密码进行身份验证以访问受 ACS 保护的 WCF 服务](http://msdn.microsoft.com/zh-cn/library/gg185954.aspx)
-   [代码示例：WCF 证书身份验证](http://msdn.microsoft.com/zh-cn/library/gg185952.aspx)
-   [代码示例：WCF 用户名身份验证](http://msdn.microsoft.com/zh-cn/library/gg185927.aspx)

###使用联合身份验证、Active Directory 中标识的 WCF (SOAP) 服务

在此应用程序体系结构方案中，可以将您的 WCF (SOAP) 服务部署到 Azure 或本地。您需要使用由企业 Windows Server
Active Directory (AD) 管理的标识来控制对此服务的访问权限。

可使用配置为与
Windows Server AD FS 进行联合的 Azure AD 访问控制。在此例中，您无需使用
Azure AD 访问控制来配置服务标识。需要访问 WCF (SOAP) 服务的代理将凭据提供给 AD FS，在成功验证身份后，AD FS 会为此代理颁发令牌。然后，会将此令牌提交给 Azure AD 访问控制并将其重新颁发给代理。代理可使用令牌向 WCF (SOAP) 服务提交请求。

![WCF (SOAP) Service With AD][05]

请参见以下资源来实施此方案：

-   [如何：添加具有 X.509 证书、密码或对称密钥的服务标识](http://msdn.microsoft.com/zh-cn/library/gg185924.aspx)
-   [如何：将 AD FS 2.0 配置为标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185961.aspx)
-   [如何：使用管理服务将 AD FS 2.0 配置为企业标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185905.aspx)
-   [代码示例：带有 AD FS 2.0 的 WCF 联合身份验证
](http://msdn.microsoft.com/zh-cn/library/hh127796.aspx)

###使用服务标识的 WCF (REST) 服务

在此方案中，可以将您的 WCF (REST) 服务部署到 
Azure 或本地。该服务将作为下游服务供 Web 应用或其他 Web 服务访问。您需要使用特定于应用程序的标识来控制对它的访问权限。可根据您在 IIS 中使用的应用池帐户的类型来考虑要使用的标识 - 虽然技术不同，但方法类似，因为访问此服务时使用的是应用程序作用域帐户与最终用户帐户。

使用 Azure AD 访问控制中的服务标识功能。
配置 Azure AD 访问控制以颁发简单 Web 标记 (SWT) 令牌。若要在 REST 服务端上处理 SWT 令牌，您可以实施自定义令牌处理程序并将它插入 WIF 管道中，也可以"手动"分析它，而无需使用 WIF 基础结构。

请参考以下图表（WIF 是可选项）：

![REST Service][06]

请参见以下资源来实施此方案：

-   [如何：使用对称密钥配置 ACS 和 WCF 服务之间的信任](http://msdn.microsoft.com/zh-cn/library/gg185958.aspx)
-   [如何：对使用 ACS 部署到 Azure 的 REST WCF 服务进行身份验证](http://msdn.microsoft.com/zh-cn/library/hh289317.aspx)
-   [代码示例：ASP.NET Web 服务](http://msdn.microsoft.com/zh-cn/library/gg983271.aspx)
-   [代码示例：Windows Phone 7 应用程序](http://msdn.microsoft.com/zh-cn/library/gg983271.aspx)
-   [使用由 Azure 访问控制服务 (ACS) 颁发的 SWT 令牌保护 REST WCF](http://code.msdn.microsoft.com/REST-WCF-With-SWT-Token-123d93c0)

###使用 Live ID/Microsoft 帐户、Facebook、Google、Yahoo! 和 Open ID 的 WCF (REST) 服务

在此方案中，可以将您的 WCF (REST) 服务部署到 
Azure 或本地。您需要使用公共
Internet 标识（例如 Live ID/Microsoft 帐户或 Facebook）来控制对此服务的访问权限。

可使用 Azure AD 访问控制来颁发 SWT 令牌。若要在 REST 服务端上处理
SWT 令牌，您可以实施自定义令牌处理程序并将它插入 WIF 管道中，也可以"手动"分析它，而无需使用 WIF 基础结构。

应特别注意，若要实施此方案，应用程序需要使用 Web 浏览器控件来收集最终用户凭据。因此，如果从 ASP.NET Web 应用访问 REST 服务，则不适合采用此方案。此方案只适合通过用户的客户端应用程序（例如 Windows Phone 7 应用或丰富桌面客户端）访问 REST 服务的情况。弹出 Web 浏览器控件的关键原因是
Internet 标识本身不支持活动配置文件方案（Web 服务方案）。Internet 标识主要支持依赖浏览器重定向的被动配置文件方案（ Web 应用）：在此处使用 Web 浏览器控件会很方便。

请参考以下图表（WIF 是可选项，因而未显示）：

![WIF is optional][07]

请参见以下资源来实施此方案：

-   [如何：对使用 ACS 部署到 Azure 的 REST WCF 服务进行身份验证](http://msdn.microsoft.com/zh-cn/library/hh289317.aspx)
-   [如何：将 Google 配置为标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185976.aspx)
-   [如何：将 Facebook 配置为标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185919.aspx)
-   [如何：将 Yahoo! 配置为标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185977.aspx)
-  [代码示例：Windows Phone 7 应用程序](http://msdn.microsoft.com/zh-cn/library/gg983271.aspx)
-   [使用由 Azure 访问控制服务 (ACS) 颁发的 SWT 令牌保护 REST WCF](http://code.msdn.microsoft.com/REST-WCF-With-SWT-Token-123d93c0)


###在 ASP.NET Web 应用和 REST WCF 服务之间使用共享 SWT 令牌

在此方案中，您具有包含前端
ASP.NET Web 应用和下游 REST 服务的分布式应用程序，并且您需要跨物理层维护最终用户的上下文。在基于下游 REST 服务中的最终用户标识实施授权逻辑或日志记录时，需要这么做。

可配置 Azure AD 访问控制来颁发 SWT 令牌。SWT 令牌将颁发给前端 ASP.NET Web 应用，然后与下游 REST 服务共享。在此例中，只在 Azure AD 访问控制中配置了一个依赖方。但有几个问题需要注意：

-   由于 WIF 不提供现成的 SWT 令牌处理程序，您需要实施要与 ASP.NET Web 应用结合使用的自定义令牌处理程序。您应依赖 WIF 完成的众多工作来支持依赖浏览器重定向的 WS 联合身份验证协议，而不是自己实施该协议。
-   在实施 SWT 自定义令牌处理程序时，请确保将对启动令牌进行寻址以保证保留它。否则，您将无法共享它并将其发送到下游 REST 服务。
-   您不必在 REST 服务上使用 WIF；您可以"手动"分析令牌，因为在此例中无需处理重定向。

![ASP.NET Web 应用lication][08]

请参见以下资源来实施此方案：

-   [如何：将 Google 配置为标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185976.aspx)
-   [如何：将 Facebook 配置为标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185919.aspx)
-   [如何：将 Yahoo! 配置为标识提供程序](http://msdn.microsoft.com/zh-cn/library/gg185977.aspx)
-   [使用共享 SWT 令牌实现 ASP.NET Web 应用到 REST WCF 服务的委派](http://code.msdn.microsoft.com/ASPNET-Web-App-To-REST-WCF-b2b95f82)

###在声明感知应用程序和服务中实现基于角色的访问控制 (RBAC)

在此方案中，您需要在 Web 应用或服务中基于用户角色实施授权：具有所需角色的用户将获得访问权限，而那些没有所需角色的用户会被拒绝。简单地说，您的应用程序需要回答一个简单问题 - 是否为具有角色
X 的用户？

有多种方法可实施此方案。您可以使用 Azure
AD 访问控制、WIF 声明身份验证管理器、samlSecurityTokenRequirement 映射或客户角色管理器。

WIF 可在所有情况下使用。WIF 支持 IPrincipal.IsInRole（"MyRole"）方法。在大多数情况下，关键是确保令牌中有此 URI 的角色类型声明，
http://schemas.microsoft.com/ws/2008/06/identity/claims/role 以便在调用 IsInRole 方法时
WIF 可以成功验证角色的成员身份。

**Azure AD 访问控制**。在此实现中， 
使用 Azure AD 访问控制声明转换规则引擎。使用声明转换规则引擎规则时，您可以将任何传入声明转换为角色类型声明，以便在令牌到达应用程序或服务时，WIF 可以分析此角色声明以确保
成功调用 IsInRole 方法。

![][09]

**WIF ClaimsAuthenticationManager**。在此实现中，使用
ClaimsAuthenticationManager 作为 WIF 的扩展点。通过使用此方法，您可在应用程序中将任意传入声明转换为角色声明类型。转换的复杂度仅受您编写的代码限制。

![][10]

**samlSecurityTokenRequriement 映射**。在此实现中，可使用 web.config 中的 samlSecurityTokenRequirement 配置来告知 WIF 哪些声明类型的行为与角色声明类型类似。例如，如果令牌带有组类型的声明，您可以将其映射到角色声明类型。通过使用此方法，您只能实现简单的映射。

![][11]

**自定义 RoleManager。**在此实现中，可实施自定义
RoleManger。WIF 可用于在实施自定义 RoleManager 接口方法（例如 GetAllRoles()）时检查传入声明。

![][12]

请参见以下资源来实施此方案：

-   [如何：使用 WIF 和 ACS 在声明感知 ASP.NET 应用程序中实现基于角色的访问控制 (RBAC)](http://msdn.microsoft.com/zh-cn/library/gg185914.aspx)
-   [如何：使用规则实现令牌转换逻辑](http://msdn.microsoft.com/zh-cn/library/gg185955.aspx)
-   [使用 RoleManager 对声明感知 (WIF) ASP.NET Web 应用进行授权](http://blogs.msdn.com/b/alikl/archive/2010/11/18/authorization-with-rolemanager-for-claims-aware-wif-asp-net-web-applications.aspx)
-   代码示例：使用 [Windows Identity Foundation SDK](http://www.microsoft.com/downloads/details.aspx?FamilyID=c148b2df-c7af-46bb-9162-2c9422208504) 的 IsInRole 中的声明

###在声明感知应用程序和服务中实现基于声明的授权

在此方案中，您需要在 Web 应用或服务中实施复杂的授权逻辑，因为 IsInRole() 方法无法满足您的授权需要。如果您的授权方法依赖角色，请考虑实施上一节中概述的基于角色的访问控制。

可使用 ClaimsAuthorizationManager 作为 WIF 扩展点。ClaimsAuthorizationManager 允许进行外部访问权限检查调用，以便您的应用程序代码看起来比在应用程序的代码中实施访问权限检查时更整洁且更可维护。

![][13]

请参见以下资源来实施此方案：

-   [如何：使用规则实现令牌转换逻辑](http://msdn.microsoft.com/zh-cn/library/gg185955.aspx)
-   [如何：使用 WIF 和 ACS 在声明感知 ASP.NET 应用程序中实现声明授权](http://msdn.microsoft.com/zh-cn/library/gg185907.aspx)
-   代码示例：[Windows Identity Foundation SDK](http://www.microsoft.com/downloads/details.aspx?FamilyID=c148b2df-c7af-46bb-9162-2c9422208504) 中基于声明的授权


##Azure 存储服务身份识别和访问控制方案

在此方案中，您需要安全地共享对 Azure 存储 Blob 和容器的访问权限。

可使用共享访问签名。若要从您自己的应用程序访问您的存储服务帐户，请
在配置和管理您的存储服务帐户时使用通过 Azure 门户提供的共享哈希。当您希望为其他人提供对您的存储服务帐户中的 Blob 和容器的访问权限时，请使用共享访问签名
URL。

应特别注意安全管理信息以避免其公开；还应特别注意共享
访问签名的生存期。

![][14]

请参见以下资源来实施此方案

-   [管理对 Blob 和容器的访问权限](http://msdn.microsoft.com/zh-cn/library/ee393343.aspx)
-   [新存储功能：共享访问签名（可能为英文页面）](http://blog.smarx.com/posts/new-storage-feature-signed-access-signatures)
-   [共享访问签名现在很容易实现](http://blog.smarx.com/posts/shared-access-signatures-are-easy-these-days)


##Azure SQL 数据库 身份识别和访问控制方案

SQL 数据库 仅支持 SQL Server 身份验证。Windows
不支持身份验证（集成安全性）。用户在每次连接到
SQL 数据库 时都必须提供凭据（登录名和密码）。在管理您的用户名和密码时应特别注意避免信息泄露。

![][15]

请参见以下资源来实施此方案：

-   [安全指导原则和限制 (SQL 数据库)](http://msdn.microsoft.com/zh-cn/library/windowsazure/ff394108.aspx#authentication)
-   [如何：使用 sqlcmd 连接到 SQL 数据库](http://msdn.microsoft.com/zh-cn/library/windowsazure/ee336280.aspx)
-   [如何：使用 ADO.NET 连接到 SQL 数据库](http://msdn.microsoft.com/zh-cn/library/windowsazure/ee336243.aspx)
-   [如何：通过 ASP.NET 连接到 SQL 数据库](http://msdn.microsoft.com/zh-cn/library/windowsazure/ee621781.aspx)
-   [如何：通过 WCF Data Services 连接 SQL 数据库](http://msdn.microsoft.com/zh-cn/library/windowsazure/ee621789.aspx)
-  [如何：使用 PHP 连接到 SQL 数据库](http://msdn.microsoft.com/zh-cn/library/windowsazure/ff394110.aspx)
-   [如何：使用 JDBC 连接到 SQL 数据库](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg715284.aspx)
-   [如何：使用 ADO.NET 实体框架连接到 SQL 数据库](http://msdn.microsoft.com/zh-cn/library/windowsazure/ff951633.aspx)

##Azure 服务总线身份识别和访问控制方案

服务总线和 Azure AD 访问控制具有特殊关系，因为每个服务总线服务命名空间均可与同名且后缀为"-sb"的匹配访问控制服务命名空间相配对。存在此特殊关系的原因在于服务总线和访问控制管理它们之间的互相信任关系和相关加密密钥的方式。有关详细信息，请参见下面列出的资源。

![][16]

请参见以下资源来实施此方案：

-   [使用 ACS 保护服务总线](http://channel9.msdn.com/posts/Securing-Service-Bus-with-ACS) (视频)
-   [使用 ACS 保护服务总线](https://skydrive.live.com/view.aspx?cid=123CCD2A7AB10107&resid=123CCD2A7AB10107%211849) (幻灯片)
-   [使用访问控制服务进行服务总线身份验证和授权](http://msdn.microsoft.com/zh-cn/library/hh403962.aspx)

##内存中缓存身份识别和访问控制方案

内存中缓存（以前称为 Azure 缓存）依赖
Azure AD A访问控制进行身份验证。它可使用通过管理门户提供的共享密钥。在访问缓存时，可在您的代码或配置文件中使用密钥。确保安全存储密钥以避免信息泄露。

![][17]


请参见以下资源来实施此方案：

-   [如何：以编程方式为 Azure Caching 配置缓存客户端](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg618003.aspx)
-   [如何：使用应用程序配置文件为 Azure Caching 配置缓存客户端](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg278346.aspx)
-   [Azure 服务总线和 Caching 示例](http://msdn.microsoft.com/zh-cn/library/ee706741.aspx) (Caching 示例”一节）

##Azure Marketplace 身份识别和访问控制方案

在用户每次访问 Azure Marketplace 数据集时（无论是免费还是付费），都必须验证他的身份，然后才能授予其访问权限。在您创建应用程序时，您的代码中必须包括身份验证过程。请考虑以下常见方案：

###我访问我的数据集

在此方案中，您构建了一个使用您的 Marketplace 订阅中数据集的应用程序。您是该应用程序的用户。
可将该应用程序部署到 Azure、本地或 Marketplace。

使用通过您的 Marketplace 订阅提供的共享密钥。您可以通过使用 Marketplace 门户来获取该共享密钥。

![][18]

请参见以下资源来实施此方案：

-   [在 Marketplace 应用中使用 HTTP 基本身份验证](http://msdn.microsoft.com/zh-cn/library/gg193417.aspx)

###用户访问我的数据集

在此方案中，您构建了一个允许用户访问您的数据集的应用程序。可将该应用程序部署到 Azure、本地或 Marketplace。

若要实施此方案，请使用 OAuth 委派。系统会提示用户提供其 Live ID/Microsoft 帐户凭据，然后他们需要同意相关条款。

![][19]

请参见以下资源来实施此方案：

-   [OAuth Web 客户端示例](http://go.microsoft.com/fwlink/?LinkId=219162)
-   [OAuth 丰富客户端示例](http://go.microsoft.com/fwlink/?LinkId=219163)

###应用程序访问 Marketplace API

在此方案中，您构建了一个可访问
Marketplace API 的应用程序。Marketplace API 需要进行身份验证才能成功实现对该应用程序的调用。将该应用程序部署到
Azure Marketplace。

![][20]

有关身份验证实现的详细信息，请参见 Marketplace 发布工具包。

请参见以下资源来实施此方案：

-   [下载应用发布工具包](http://go.microsoft.com/fwlink/?LinkId=221323)
-   [面向应用程序的 Azure Marketplace 简介](https://datamarket.azure.com)

##安全设置

本节概述了 Windows Identity Foundation 和
Azure AD 访问控制的安全设置。在设计并部署您的应用程序时，可使用它作为这些技术的基本安全清单。

###Windows Identity Foundation

下面是 WIF 的关键安全设置。以下信息摘自 [WIF 设计注意事项](http://msdn.microsoft.com/zh-cn/library/ee517298.aspx) and [Windows Identity Foundation
和 ASP.NET Web 应用的 (WIF) 安全性 - 威胁和对策](http://blogs.msdn.com/b/alikl/archive/2010/12/02/windows-identity-foundation-wif-security-for-asp-net-web-applications-threats-amp-countermeasures.aspx)
。

-   **IssuerNameRegistry**。指定受信任的安全令牌服务 (STS)。确保仅列出了受信任的 STS。
-   **cookieHandler requireSsl="true"**。指定是否通过 SSL 协议传输会话 Cookie。
-   **wsFederation's requireHttps="true"**。指定是否通过 SSL 协议执行与标识提供程序的联合身份验证协议通信。
-   **tokenReplayDetection enabled="true"**。指定是否启用令牌重放检测功能。请注意，此功能在管理已用令牌的本地副本时会创建服务器关联。
-   **audienceUris**。指定令牌的目标受众。如果您的应用程序收到的令牌不适用于您的应用，则 WIF 会拒绝该令牌。
-   **requestValidation** 和 **httpRuntime requestValidationType**。启用/禁用 ASP.NET 验证功能。请参阅 [Windows Identity Foundation (WIF)：从客户端检测到潜在危险的 Request.Form 值](http://social.technet.microsoft.com/wiki/contents/articles/1725.windows-identity-foundation-wif-a-potentially-dangerous-request-form-value-was-detected-from-the-client-wresult-t-requestsecurityto.aspx)中所述的指南

###Azure AD 访问控制

请考虑 Azure AD 访问控制部署中的以下安全设置。以下信息摘自 [ACS 安全
指南](http://msdn.microsoft.com/zh-cn/library/gg185962.aspx) and [Certificates and Keys Management Guidelines](http://msdn.microsoft.com/zh-cn/library/hh204521.aspx).

-   **STS 令牌有效期**。使用 Azure AD 访问控制管理门户来设置活动令牌有效期。
-   **使用"错误 URL"功能时的数据验证**。Azure AD 访问控制"错误 URL"功能需要匿名访问它向其发送错误消息的应用的页面。假定发送到此页面的所有数据都是来自不受信任源的危险数据。
-   **为高度敏感方案加密令牌**。若要缓解令牌中存在的信息泄露威胁，请考虑加密令牌。
-   **在部署到 Azure 时，可使用 RSA 加密 Cookie**。默认情况下，WIF 使用 DPAPI 加密 Cookie。在部署到 Web 场和 Azure 环境时，它会创建服务器关联并可能导致异常。在 Web 场和 Azure 方案中可改用 RSA。
-   **令牌签名证书**。定期续订令牌签名证书以避免拒绝服务。Azure AD 访问控制对其颁发的所有安全令牌进行签名。在您生成使用 ACS 颁发的 SAML 令牌的应用程序时，需使用 X.509 证书进行签名。在签名证书过期后，如果您尝试请求令牌，将收到错误。
-   **令牌签名密钥**。定期续订令牌签名密钥以避免拒绝服务。Azure AD 访问控制对其颁发的所有安全令牌进行签名。在您生成使用 ACS 颁发的 SWT 令牌的应用程序时，需使用 256 位对称签名密钥。如果您在签名密钥过期后尝试请求令牌，将收到错误。
-   **令牌加密证书**。定期续订令牌加密证书以避免拒绝服务。如果信赖方应用程序是通过 WS-Trust 协议使用所有权确认令牌的 Web 服务，则令牌加密是必需的。在其他情况下，令牌加密是可选的。在加密证书过期后，如果您尝试请求令牌，将收到错误。
-   **令牌解密证书**。定期续订令牌解密证书以避免拒绝服务。Azure AD 访问控制可以接受来自 WS 联合身份验证标识提供程序（例如，AD FS 2.0）的加密令牌。将使用 Azure AD 访问控制中托管的 X.509 证书进行解密。在解密证书过期后，如果您尝试请求令牌，将收到错误。
-   **服务标识凭据**。定期续订服务标识凭据以避免拒绝服务。服务标识使用为您的 Azure AD 访问控制命名空间全局配置的凭据，这些凭据允许应用程序或客户端使用 Azure AD 访问控制直接进行身份验证并接收令牌。Azure AD 访问控制服务标识可以与三种凭据类型相关联：对称密钥、密码和 X.509 证书。在凭据过期后，您将开始收到异常。
-   **Azure AD 访问控制管理服务帐户凭据**。定期续订管理服务凭据以避免拒绝服务。Azure AD 访问控制管理服务是允许您以编程方式管理和配置您的 Azure AD 访问控制命名空间设置的关键组件。管理服务帐户可以与三种凭据类型相关联：对称密钥、密码和 X.509 证书。在凭据过期后，您将开始收到异常。
-   **WS 联合身份验证标识提供程序签名和加密证书**。查询 WS 联合身份验证标识提供程序的证书有效性以避免拒绝服务。WS 联合身份验证标识提供程序证书可通过其元数据获得。在配置 WS 联合身份验证标识提供程序（例如 AD FS）时，将使用通过 URL 或以文件形式提供的 WS 联合身份验证元数据来配置 WS 联合身份验证签名证书。配置完 WS 联合身份验证标识提供程序后，可使用 Azure AD 访问控制管理服务来查询其证书有效性。在证书过期后，您将开始收到异常。

##使用 Azure Web 应用的共享宿主

当应用程序托管在 Azure Web 应用上时，本主题中概述的所有方案和解决方案都有效。

##Azure 虚拟机

当应用程序托管在 Azure 虚拟机上时，本主题中概述的所有方案和解决方案都有效。

##资源

-   [标识开发人员培训工具包](http://go.microsoft.com/fwlink/?LinkId=214555)
-   [MSDN 托管的标识开发人员培训课程](http://go.microsoft.com/fwlink/?LinkId=214561)
-   [基于声明的身份识别和访问控制指南](http://go.microsoft.com/fwlink/?LinkId=214562)
-   [访问控制服务](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg429786.aspx)
-   [ACS 操作指南](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg185939.aspx)
-   [使用访问控制服务 v2.0 保护 Azure Web 角色 ASP.NET Web 应用](http://social.technet.microsoft.com/wiki/contents/articles/2590.aspx)
-   [Azure AD 访问控制服务 (ACS) 学会视频](http://social.technet.microsoft.com/wiki/contents/articles/2777.aspx)
-   [Microsoft 安全开发生命周期](http://www.microsoft.com/security/sdl/default.aspx)
-   [SDL Threat Modeling Tool 3.1.8](http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=2955)
-   [安全和隐私博客](http://www.microsoft.com/about/twc/en/us/blogs.aspx)
-   [安全响应中心](http://www.microsoft.com/security/msrc/default.aspx)
-   [安全智能报告](http://www.microsoft.com/security/sir)
-   [安全开发生命周期](http://www.microsoft.com/security/sdl/default.aspx)
-   [安全开发人员中心 (MSDN)](http://msdn.microsoft.com/security)


[01]:./media/SecurityRX/01_SecuringTheApplication.gif
[02]:./media/SecurityRX/02_ThreatsVulnerabilitiesandAttacks.gif
[03]:./media/SecurityRX/03_WindowsAzureADAccesscontrol.gif
[04]:./media/SecurityRX/04_WCF(SOAP)Service.gif
[05]:./media/SecurityRX/05_AzureADAccessControl.gif
[06]:./media/SecurityRX/06_RESTService.gif
[07]:./media/SecurityRX/07_WIFisOptional.gif
[08]:./media/SecurityRX/08_ASPNETWebApptoREST.gif
[09]:./media/SecurityRX/09_RBAC.gif
[10]:./media/SecurityRX/10_WIFClaimsAuthenticationManager.gif
[11]:./media/SecurityRX/11_SecurityTokenRequriementmapping.gif
[12]:./media/SecurityRX/12_CustomRoleManager.gif
[13]:./media/SecurityRX/13_ClaimsAuthorizationManager.gif
[14]:./media/SecurityRX/14_WindowsAzurestorage.gif
[15]:./media/SecurityRX/15_SQLAzureIdentityandAccessScenarios.gif
[16]:./media/SecurityRX/16_WindowsAzureServiceBusIdentity.gif
[17]:./media/SecurityRX/17_WindowsAzureCacheIdentity.gif
[18]:./media/SecurityRX/18_IAccessMyDataset.gif
[19]:./media/SecurityRX/19_UsersAccessMyDatasets.gif
[20]:./media/SecurityRX/20_ApplicationAccessMarketplaceAPI.gif

[Web SSO 设计]: http://technet.microsoft.com/zh-cn/library/dd807033(WS.10).aspx
[联合 Web SSO 设计]: http://technet.microsoft.com/zh-cn/library/dd807050(WS.10).aspx
<!--HONumber=41-->
