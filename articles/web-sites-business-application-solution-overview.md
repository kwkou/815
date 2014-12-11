<properties linkid="websites-business-application" urlDisplayName="Create a Line-of-Business Application on Azure Web Sites" pageTitle="Create a Line-of-Business Application on Azure Web Sites" metaKeywords="Web Sites" description="This guide provides a technical overview of how to use Azure Web Sites to create intranet, line-of-business applications. This includes authentication strategies, service bus relay, and monitoring." umbracoNaviHide="0" disqusComments="1" editor="mollybos" manager="paulettm" title="Create a Line-of-Business Application on Azure Web Sites" authors="jroth" />

# 在 Azure 网站上创建业务线应用程序

本指南提供如何使用 Azure 网站创建业务线应用程序的技术概述。出于本文档的目的，假定这些应用程序是 Intranet 应用程序，并且应确保这些应用程序的安全以便用于内部业务。有两种不同的业务应用程序特性。这些应用程序要求身份验证，通常是针对公司目录。并且它们通常要求对内部数据和服务某种访问或集成。本指南主要说明如何在 [Azure 网站][Azure 网站]上构建业务应用程序。但是，有一些情形 [Azure 云服务][Azure 云服务]或 [Azure 虚拟机][Azure 虚拟机]可以更好地满足你的要求。了解这些选项之间的差别很重要，因此请参阅主题：[Azure 网站、云服务和虚拟机：何时使用何种产品？][Azure 网站、云服务和虚拟机：何时使用何种产品？]。

在本指南中将针对以下方面：

-   [考虑利益][考虑利益]
-   [选择身份验证策略][选择身份验证策略]
-   [创建支持身份验证的 Azure 网站][创建支持身份验证的 Azure 网站]
-   [使用 Service Bus 与本地资源进行集成][使用 Service Bus 与本地资源进行集成]
-   [监视应用程序][监视应用程序]

<div class="dev-callout">
<strong>说明</strong>
<p>本指南展示为面向公众的 .COM 站点开发而进行调整的一些最常见领域和任务。但是，还有可在特定实施中使用的其他一些 Azure 网站功能。若要查看这些功能，另请参阅<a href="http://www.windowsazure.com/en-us/manage/services/web-sites/global-web-presence-solution-overview/">全球网络影响力</a>和<a href="http://www.windowsazure.com/en-us/manage/services/web-sites/digital-marketing-campaign-solution-overview">数字市场营销活动</a>中的其他指南。</p>
</div>

## <a name="benefits"></a>考虑利益

因为业务线应用程序通常针对公司用户，所以，你应该考虑多种原因以便权衡是使用云还是本地公司资源和基础结构。首先，移到云有一些典型的好处，例如，能够伸缩以便适应动态工作负荷。我们以一个处理每年绩效审核的应用程序为例。对于一年中的大多数时间，此类型的应用程序将会处理非常少的流量。但在审核期间，对于大型公司而言流量将会突增到较高的水平。Azure 提供缩放选项，使公司能够在高流量审核期间进行扩展以便处理负荷，而对于一年中的其他时间能够收缩以便节约资金。云的另一个好处是能够更关注应用程序开发，而更少关注基础结构添置和管理。

除了这些标准优势之外，将业务应用程序放置于云中还为员工和合作伙伴从任何地点使用应用程序提供更好的支持。用户无需连接到公司网络以便使用应用程序，并且 IT 部门能够避免复杂的反向代理解决方案。有几种身份验证选项以便确保保护对公司应用程序的访问。在本指南的后面几节中将探讨这些选项。

## <a name="authentication"></a>选择身份验证策略

对于业务应用程序方案，你的身份验证策略是最重要的决策之一。有若干个选项：

-   使用 [Azure Active Directory 服务][Azure Active Directory 服务]。你可以将它用作独立目录，或者可以将它与本地 Active Directory 同步。应用程序然后与 Azure Active Directory 交互以便对用户进行身份验证。有关此方法的概述，请参阅[使用 Azure Active Directory][使用 Azure Active Directory]。
-   使用 Azure 虚拟机和虚拟网络可以安装 Active Directory。这使你可以选择将本地 Active Directory 安装扩展到云。还可以选择使用 Active Directory 联合身份验证服务 (ADFS) 将标识请求联合回 AD。然后，针对 Azure 应用程序的身份验证将对本地 Active Directory 执行 ADFS。有关此方法的概述，请参见[在虚拟机中运行 Windows Server Active Directory][在虚拟机中运行 Windows Server Active Directory] 和[在 Azure 虚拟机上部署 Windows Server Active Directory 的指南][在 Azure 虚拟机上部署 Windows Server Active Directory 的指南]。
-   使用 [Azure 访问控制服务][Azure 访问控制服务] (ACS) 以便通过多个标识服务对用户进行身份验证。这提供一个抽象以便通过 Active Directory 或者通过不同的提供程序进行身份验证。有关更多信息，请参阅[使用 Azure Active Directory 访问控制][使用 Azure Active Directory 访问控制]。

对于此业务应用程序方案，使用 Azure Active Directory 的第一个方案为你的应用程序实现身份验证策略提供最快的途径。本指南的剩余部分将着重介绍 Azure Active Directory。但是，根据你的业务要求，你可能会发现其他两个解决方案中的一个更适合。例如，如果不允许你将标识信息同步到云，则 ADFS 解决方案可能是更好的选项。或者，如果你必须支持 Facebook 之类的其他标识提供程序，则 ACS 解决方案将会更适合。

在开始设置新的 Azure Active Directory 之前，你应该注意到现有服务（例如 Office 365 或 Windows Intune）已使用 Azure Active Directory。在该情况下，你要将现有订阅与你的 Azure 订阅相关联。有关更多信息，请参见 [Azure AD 租户是什么？][Azure AD 租户是什么？]。

如果你当前没有已使用 Azure Active Directory 的服务，则可以在管理门户中创建一个新目录。使用管理门户的 **Active Directory** 部分可以创建新目录。

![BusinessApplicationsAzureAD][BusinessApplicationsAzureAD]

在创建后，你可以选择创建和管理用户、集成的应用程序和域。

![BusinessApplicationsADUsers][BusinessApplicationsADUsers]

有关这些步骤的完整演练，请参见[使用 Azure AD 将登录名添加到 Web 应用程序中][使用 Azure AD 将登录名添加到 Web 应用程序中]。如果你在将这个新目录用作独立资源，则下一步骤是开发与该目录集成的应用程序。但是，如果你具有本地 Active Directory 标识，则通常想要将它们与新的 Azure Active Directory 同步。有关执行此操作的详细信息，请参阅[目录集成][目录集成]。

在创建并填充了该目录后，你必须创建要求身份验证的 Web 应用程序，然后将它们与该目录相集成。下面两节中会介绍这些步骤。

## <a name="createintranetsite"></a>创建支持身份验证的 Azure 网站

在“全球网络影响力”方案中，我们探讨了用于创建和部署新网站的不同选项。如果你是 Azure 网站的新手，最好[查看该信息][查看该信息]。Visual Studio 中的 ASP.NET 应用程序是针对使用 Windows 身份验证的 Intranet Web 应用程序的常见选择。其中一个原因就是 ASP.NET 和 Visual Studio 为此方案提供的紧密的集成和支持。

例如，在 Visual Studio 中创建一个 ASP.NET MVC 4 项目时，你可以选择在项目创建对话框中创建“Intranet 应用程序”。

![BusinessApplicationsVSIntranetApp][BusinessApplicationsVSIntranetApp]

这导致对项目设置进行更改以便支持 Windows 身份验证。具体来说，将 web.config 文件的 **authentication** 元素的 **mode** 属性设置为 **Windows**。如果你创建不同的 ASP.NET 项目，例如 Web 窗体项目，或者如果你在使用某一现有项目，则必须手动进行此更改。

对于 MVC 项目，你还必须在项目属性窗口中更改两个值。将“Windows 身份验证”设置为“启用”，将“匿名身份验证”设置为“禁用”。

![BusinessApplicationsVSProperties][BusinessApplicationsVSProperties]

若要对 Azure Active Directory 进行身份验证，你必须向该目录注册此应用程序，然后必须修改该应用程序配置以便进行连接。这可以在 Visual Studio 中以两种方式实现：

-   [针对 Visual Studio 的身份认证和访问工具][针对 Visual Studio 的身份认证和访问工具]
-   [Microsoft ASP.NET Tools for Azure Active Directory][Microsoft ASP.NET Tools for Azure Active Directory]

### <a name="identityandaccessforvs"></a>针对 Visual Studio 的身份认证和访问工具：

一个方法是使用[身份认证和访问工具][身份认证和访问工具]（你可以下载并安装）。该工具在项目上下文菜单上与 Visual Studio 相集成。下面的说明和屏幕快照来自 Visual Studio 2012。右键单击该项目，然后选择“身份认证和访问”。有三项必须进行配置。在“提供程序”选项卡上，必须提供“指向 STS 元数据文档的路径”和“APP ID URI”（若要获取这些值，请参见[在 Azure Active Directory 中注册应用程序][在 Azure Active Directory 中注册应用程序]上的有关部分）。

![BusinessApplicationsVSIdentityAndAccess][BusinessApplicationsVSIdentityAndAccess]

必须进行的最后的配置更改是对“身份认证和访问”对话框的“配置”选项卡进行的更改。你必须选中“启用 Web 场 cookie”复选框。有关这些步骤的详细演练，请参见[使用 Azure AD 将登录名添加到 Web 应用程序中][使用 Azure AD 将登录名添加到 Web 应用程序中]。

#### <a name="registerwaadapp"></a>在 Azure Active Directory 中注册应用程序：

为了填充“提供程序”选项卡，你需要向 Azure Active Directory 注册你的应用程序。在 Azure 管理门户上的“Active Directory”部分中，选择你的目录，然后转到“应用程序”选项卡。这将提供一个选项以便按 URL 添加 Azure 网站。请注意，在你执行这些步骤时，开始时将该 URL 设置为针对在 Visual Studio 中进行本地调试而提供的本地主机地址。以后进行部署时，将该地址更改为你的网站的实际 URL。

![BusinessApplicationsADIntegratedApps][BusinessApplicationsADIntegratedApps]

在添加后，该门户提供 STS 元数据文档路径（“联合元数据文档 URL”）和“应用程序 ID URI”。在 Visual Studio 中“身份识别和访问”对话框的“提供程序”选项卡上使用这些值。

![BusinessApplicationsADAppAdded][BusinessApplicationsADAppAdded]

### <a name="aspnettoolsforwaad"></a>Microsoft ASP.NET Tools for Azure Active Directory：

实现前面介绍的步骤的另一个方法是使用 [Microsoft ASP.NET Tools for Azure Active Directory][1]。为此，请从 Visual Studio 中的“项目”菜单单击“启用 Azure 身份验证”项。这就会显示一个简单的对话框，该对话框请求你提供 Azure Active Directory 域的地址（而不是你的应用程序的 URL）。

![BusinessApplicationsVSEnableAuth][BusinessApplicationsVSEnableAuth]

如果你是该 Active Directory 域的管理员，则选中“在 Azure AD 中设置此应用程序”复选框。这将执行向 Active Directory 注册该应用程序的工作。如果你不是管理员，则取消选中该框，并且提供显示给管理员的信息。该管理员可以使用管理门户通过之前在身份认证和访问工具上执行的步骤创建集成的应用程序。有关如何使用 ASP.NET Tools for Azure Active Directory 的详细步骤，请参见 [Azure 身份验证][Azure 身份验证]。

在管理你的业务线应用程序时，你能够将任何支持的源代码管理系统用于部署。但是，由于在此方案中高度集成 Visual Studio，Team Foundation Service (TFS) 更可能是你选择的源代码管理系统。如果是这样，你应该注意到，Azure 网站提供与 TFS 的集成。在管理门户中，转到你的网站的“仪表板”选项卡。然后选择“从源代码管理设置部署”。遵循 TFS 使用说明。

![BusinessApplicationsDeploy][BusinessApplicationsDeploy]

## <a name="servicebusrelay"></a>使用 Service Bus 与本地资源进行集成

许多业务线应用程序都必须与本地数据和服务相集成。有多种原因会导致某些类型的数据不能移到云。这些原因可以是实践原因或法规原因。如果你处于计划阶段，需要确定要在 Azure 中托管哪些数据以及哪些数据应保留在本地，则一定要查看 [Azure 信任中心][Azure 信任中心]上的资源。混合 Web 应用程序在 Azure 中运行并且访问必须保留在本地的资源。

在使用虚拟机或云服务时，你可以使用虚拟网络将 Azure 中的应用程序与公司网络连接起来。但是，网站不支持虚拟网络，因此，执行此类型的与网站集成的最佳方法是通过使用 [Azure Service Bus 中继服务][Azure Service Bus 中继服务]。该 Service Bus 中继服务允许云中的应用程序安全地连接到在公司网络上运行的 WCF 服务。Service Bus 无需开放防火墙端口即允许此通信。

在下图中，云应用程序和本地 WCF 服务都通过以前创建的命名空间与 Service Bus 通信。该本地 WCF 服务能够访问不能移到云中的内部数据和服务。该 WCF 服务将在命名空间中注册一个终结点。在 Azure 中运行的网站也在 Service Bus 中连接到此终结点。它们只需能够发起传出公共 HTTP 请求即可完成该步骤。

![BusinessApplicationsServiceBusRelay][BusinessApplicationsServiceBusRelay]

Service Bus 然后将云应用程序连接到本地 WCF 服务。这为创建使用 Azure 以及本地上的服务和资源的混合应用程序提供基本的体系结构。有关更多信息，请参见[如何使用 Service Bus 中继服务][如何使用 Service Bus 中继服务]和教程 [Service Bus 中继消息传送教程][Service Bus 中继消息传送教程]。有关演示此技术的示例，请参见 [Enterprise Pizza - 使用 Service Bus 将网站连接到本地][Enterprise Pizza - 使用 Service Bus 将网站连接到本地]。

## <a name="monitor"></a>监视应用程序

业务应用程序可从标准网站功能（例如缩放和监视）中受益。对于在特定天数或小时数期间体验变化的负荷的业务应用程序而言，自动缩放（预览）功能有助于缩放站点以便高效地使用资源。监视选项包括终结点监视和配额监视。在[全球网络影响力][查看该信息]和[数字市场营销活动][2]方案中更详细地介绍了所有这些方案。

不同的业务线应用程序具有不同的业务重要性，因此其监视需要也各不相同。对于更关键的应用程序，请考虑对第三方监视解决方案（例如 [New Relic][New Relic]）进行投资。

业务线应用程序通常由 IT 员工进行管理。在遇到意外错误或行为时，IT 工作人员可启用更详细地日志记录，然后分析生成的数据以便确定问题。在管理门户中，转到“配置”选项卡，并且查看“应用程序诊断”和“网站诊断”部分中的选项。

![BusinessApplicationsDiagnostics][BusinessApplicationsDiagnostics]

使用不同的应用程序和网站日志解决与网站有关的问题。请注意，某些选项指定“文件系统”，这将日志文件放置于你的网站的文件系统上。可通过 FTP、Azure PowerShell 或 Azure 命令行工具访问它们。其他选项指定“存储”。这会将信息发送到你指定的 Azure 存储帐户。对于“Web 服务器日志记录”，你还具有为文件系统指定磁盘配额或者为存储选项指定保留策略的选项。这可以避免无限增加存储的日志记录数据量。

![BusinessApplicationsDiagRetention][BusinessApplicationsDiagRetention]

有关这些日志记录设置的更多信息，请参见[如何：为网站配置诊断和下载日志][如何：为网站配置诊断和下载日志]。

除了通过 FTP 或存储实用工具（例如 Azure 存储资源管理器）查看原始日志，你还可以在 Visual Studio 中查看日志信息。有关在故障排除情况中使用这些日志的详细信息，请参见[在 Visual Studio 中排除 Azure 网站的故障][在 Visual Studio 中排除 Azure 网站的故障]。

## <a name="summary"></a>摘要

Azure 使你能够在云中托管安全的 Intranet 应用程序。Azure Active Directory 提供对用户进行身份验证的功能，以便只有你的组织的成员可以访问应用程序。Service Bus 中继服务提供使 Web 应用程序与内部服务和数据进行通信的机制。通过此混合应用程序方法，能够更容易地将业务应用程序发布到云，而不必也迁移所有依赖数据和服务。在部署后，业务应用程序将受益于 Azure 网站提供的标准缩放和监视功能。有关详细信息，请参阅以下技术文章。

<table cellspacing="0" border="1">
<tr>
<th align="left" valign="top">区域</th>
<th align="left" valign="top">资源</th>
</tr>
<tr>
<td valign="middle"><strong>计划</strong></td>
<td valign="top">- <a href="http://www.windowsazure.com/en-us/manage/services/web-sites/choose-web-app-service">Azure 网站、云服务和虚拟机：何时使用何种产品？</a></td>
</tr>
<tr>
<td valign="middle"><strong>创建和部署</strong></td>
<td valign="top">- <a href ="http://www.windowsazure.com/en-us/develop/net/tutorials/get-started/">将 ASP.NET Web 应用程序部署到 Azure 网站</a><br/>- <a href="http://www.windowsazure.com/en-us/develop/net/tutorials/web-site-with-sql-database/">将安全的 ASP.NET MVC 应用程序部署到 Azure</a></td>
</tr>
<tr>
<td valign="middle"><strong>身份验证</strong></td>
<td valign="top">- <a href ="http://www.windowsazure.com/en-us/manage/windows/fundamentals/identity/">了解 Azure 标识选项</a><br/>- <a href="http://www.windowsazure.com/en-us/documentation/services/active-directory/">Azure Active Directory 服务</a><br/>- <a href="http://technet.microsoft.com/en-us/library/jj573650.aspx">什么是 Azure AD 租户？</a><br/>- <a href="http://msdn.microsoft.com/library/windowsazure/dn151790.aspx">使用 Azure AD 将登录名添加到 Web 应用程序中</a><br/>- <a href="http://www.asp.net/aspnet/overview/aspnet-and-visual-studio-2012/windows-azure-authentication">Azure 身份验证教程</a></td>
</tr>
<tr>
<td valign="middle"><strong>Service Bus 中继</strong></td>
<td valign="top">- <a href="http://www.windowsazure.com/en-us/develop/net/how-to-guides/service-bus-relay/">如何使用 Service Bus 中继服务</a><br/>- <a href="http://msdn.microsoft.com/en-us/library/windowsazure/ee706736.aspx">Service Bus 中继消息传送教程</a></td>
</tr>
<tr>
<td valign="middle"><strong>监视</strong></td>
<td valign="top">- <a href ="http://www.windowsazure.com/en-us/manage/services/web-sites/how-to-monitor-websites/">如何监视网站</a><br/>- <a href="http://msdn.microsoft.com/library/windowsazure/dn306638.aspx">如何：在 Azure 中接收警报通知和管理警报规则</a><br/>- <a href="http://www.windowsazure.com/en-us/manage/services/web-sites/how-to-monitor-websites/#howtoconfigdiagnostics">如何：为网站配置诊断和下载日志</a><br/>- <a href="http://www.windowsazure.com/en-us/develop/net/tutorials/troubleshoot-web-sites-in-visual-studio/">在 Visual Studio 中排除 Azure 网站的故障</a></td>
</tr>
</table>

  [Azure 网站]: /zh-cn/documentation/services/web-sites/
  [Azure 云服务]: /zh-cn/documentation/services/cloud-services/
  [Azure 虚拟机]: /zh-cn/documentation/services/virtual-machines/
  [Azure 网站、云服务和虚拟机：何时使用何种产品？]: /zh-cn/manage/services/web-sites/choose-web-app-service
  [考虑利益]: #benefits
  [选择身份验证策略]: #authentication
  [创建支持身份验证的 Azure 网站]: #createintranetsite
  [使用 Service Bus 与本地资源进行集成]: #servicebusrelay
  [监视应用程序]: #monitor
  [全球网络影响力]: http://www.windowsazure.com/en-us/manage/services/web-sites/global-web-presence-solution-overview/
  [数字市场营销活动]: http://www.windowsazure.com/en-us/manage/services/web-sites/digital-marketing-campaign-solution-overview
  [Azure Active Directory 服务]: /zh-cn/documentation/services/active-directory/
  [使用 Azure Active Directory]: /zh-cn/manage/windows/fundamentals/identity/#ad
  [在虚拟机中运行 Windows Server Active Directory]: /zh-cn/manage/windows/fundamentals/identity/#adinvm
  [在 Azure 虚拟机上部署 Windows Server Active Directory 的指南]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156090.aspx
  [Azure 访问控制服务]: http://msdn.microsoft.com/library/windowsazure/hh147631.aspx
  [使用 Azure Active Directory 访问控制]: /zh-cn/manage/windows/fundamentals/identity/#ac
  [Azure AD 租户是什么？]: http://technet.microsoft.com/zh-cn/library/jj573650.aspx
  [BusinessApplicationsAzureAD]: ./media/web-sites-business-application-solution-overview/BusinessApplications_AzureAD.png
  [BusinessApplicationsADUsers]: ./media/web-sites-business-application-solution-overview/BusinessApplications_AD_Users.png
  [使用 Azure AD 将登录名添加到 Web 应用程序中]: http://msdn.microsoft.com/library/windowsazure/dn151790.aspx
  [目录集成]: http://technet.microsoft.com/zh-cn/library/jj573653.aspx
  [查看该信息]: /zh-cn/manage/services/web-sites/global-web-presence-solution-overview/
  [BusinessApplicationsVSIntranetApp]: ./media/web-sites-business-application-solution-overview/BusinessApplications_VS_IntranetApp.png
  [BusinessApplicationsVSProperties]: ./media/web-sites-business-application-solution-overview/BusinessApplications_VS_Properties.png
  [针对 Visual Studio 的身份认证和访问工具]: #identityandaccessforvs
  [Microsoft ASP.NET Tools for Azure Active Directory]: #aspnettoolsforwaad
  [身份认证和访问工具]: http://visualstudiogallery.msdn.microsoft.com/e21bf653-dfe1-4d81-b3d3-795cb104066e
  [在 Azure Active Directory 中注册应用程序]: #registerwaadapp
  [BusinessApplicationsVSIdentityAndAccess]: ./media/web-sites-business-application-solution-overview/BusinessApplications_VS_IdentityAndAccess.png
  [BusinessApplicationsADIntegratedApps]: ./media/web-sites-business-application-solution-overview/BusinessApplications_AD_IntegratedApps.png
  [BusinessApplicationsADAppAdded]: ./media/web-sites-business-application-solution-overview/BusinessApplications_AD_AppAdded.png
  [1]: http://go.microsoft.com/fwlink/?LinkID=282306
  [BusinessApplicationsVSEnableAuth]: ./media/web-sites-business-application-solution-overview/BusinessApplications_VS_EnableAuth.png
  [Azure 身份验证]: http://www.asp.net/aspnet/overview/aspnet-and-visual-studio-2012/windows-azure-authentication
  [BusinessApplicationsDeploy]: ./media/web-sites-business-application-solution-overview/BusinessApplications_Deploy.png
  [Azure 信任中心]: /zh-cn/support/trust-center/
  [Azure Service Bus 中继服务]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj860549.aspx
  [BusinessApplicationsServiceBusRelay]: ./media/web-sites-business-application-solution-overview/BusinessApplications_ServiceBusRelay.png
  [如何使用 Service Bus 中继服务]: /zh-cn/develop/net/how-to-guides/service-bus-relay/
  [Service Bus 中继消息传送教程]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee706736.aspx
  [Enterprise Pizza - 使用 Service Bus 将网站连接到本地]: http://code.msdn.microsoft.com/windowsazure/Enterprise-Pizza-e2d8f2fa
  [2]: /zh-cn/manage/services/web-sites/digital-marketing-campaign-solution-overview
  [New Relic]: http://newrelic.com/azure
  [BusinessApplicationsDiagnostics]: ./media/web-sites-business-application-solution-overview/BusinessApplications_Diagnostics.png
  [BusinessApplicationsDiagRetention]: ./media/web-sites-business-application-solution-overview/BusinessApplications_Diag_Retention.png
  [如何：为网站配置诊断和下载日志]: /zh-cn/manage/services/web-sites/how-to-monitor-websites/#howtoconfigdiagnostics
  [在 Visual Studio 中排除 Azure 网站的故障]: /zh-cn/develop/net/tutorials/troubleshoot-web-sites-in-visual-studio/
  [3]: http://www.windowsazure.com/en-us/manage/services/web-sites/choose-web-app-service
  [将 ASP.NET Web 应用程序部署到 Azure 网站]: http://www.windowsazure.com/en-us/develop/net/tutorials/get-started/
  [将安全的 ASP.NET MVC 应用程序部署到 Azure]: http://www.windowsazure.com/en-us/develop/net/tutorials/web-site-with-sql-database/
  [了解 Azure 标识选项]: http://www.windowsazure.com/en-us/manage/windows/fundamentals/identity/
  [4]: http://www.windowsazure.com/en-us/documentation/services/active-directory/
  [什么是 Azure AD 租户？]: http://technet.microsoft.com/en-us/library/jj573650.aspx
  [5]: http://www.windowsazure.com/en-us/develop/net/how-to-guides/service-bus-relay/
  [6]: http://msdn.microsoft.com/en-us/library/windowsazure/ee706736.aspx
  [如何监视网站]: http://www.windowsazure.com/en-us/manage/services/web-sites/how-to-monitor-websites/
  [如何：在 Azure 中接收警报通知和管理警报规则]: http://msdn.microsoft.com/library/windowsazure/dn306638.aspx
  [7]: http://www.windowsazure.com/en-us/manage/services/web-sites/how-to-monitor-websites/#howtoconfigdiagnostics
  [8]: http://www.windowsazure.com/en-us/develop/net/tutorials/troubleshoot-web-sites-in-visual-studio/
