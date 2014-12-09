<properties linkid="manage-services-integration-hybrid-connection" urlDisplayName="Integration Hybrid Connection" pageTitle="集成和网站中的混合连接 | Azure" metaKeywords="BizTalk Services, BizTalk, web sites, hybrid connection, Azure" description="了解如何创建混合连接、管理连接以及安装混合连接管理器。" metaCanonical="" services="integration-services" documentationCenter="" title="混合连接" authors="mandia" solutions="" manager="paulettm" editor="cgronlun" />

# 混合连接

本主题列出了创建和管理混合连接以及安装混合连接管理器的步骤。具体内容包括：

-   [什么是混合连接][什么是混合连接]
-   [创建混合连接][创建混合连接]
-   [本地安装混合连接管理器][本地安装混合连接管理器]
-   [管理混合连接][管理混合连接]

## <a name="HCOverview"></a>什么是混合连接

使用混合连接可以轻松方便地将 Azure 应用程序（例如网站和移动服务）连接到本地资源。混合连接的优势包括：

-   可以连接到任何使用静态 TCP 端口的本地资源，例如 SQL Server、MySQL、HTTP Web API 和大多数自定义 Web 服务。
-   可供多个 Azure 网站和移动服务使用。
-   不会对 Azure 网站或网站页更改任何代码。

    <div class="dev-callout">

    **说明**
    不支持使用动态端口（FTP 被动模式或扩展被动模式）的基于 TCP 的服务。

    </div>

在以下示例中，您可以使用混合连接来创建和管理连接性：

-   有一个要连接到本地 SQL Server 的使用 ADO.NET 的 Azure 网站。
-   有一个可连接到本地 HTTP Web 服务的 Azure 网站，例如 SharePoint。

## <a name="CreateHybridConnection"></a>创建混合连接

可在 Azure 管理门户中使用网站**或**使用 BizTalk 服务创建混合连接。

**若要使用 Azure 网站创建混合连接**，请参阅[将 Azure 网站连接到本地资源][将 Azure 网站连接到本地资源]。

**若要开始使用混合连接和 Azure 移动服务**，请参阅 [Azure 移动服务和混合连接][Azure 移动服务和混合连接]。

**若要在 BizTalk 服务中创建混合连接**：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中，选择“BizTalk 服务”，然后选择您的 BizTalk 服务。
    如果您没有现有的 BizTalk 服务，则可以[创建 BizTalk 服务][创建 BizTalk 服务]。
3.  选择“混合连接”选项卡：
    ![“混合连接”选项卡][“混合连接”选项卡]

4.  选择“创建混合连接”或选择任务栏中的“添加”按钮。输入以下内容：

    |------------|------------------------------------------------------------------------------------------------------------------------|
    | **名称**   | 您可以输入任何名称，但是尽量使其目的明确。示例包括：                                                                   
                   Payroll*SQLServerName*                                                                                                 
                   SupplyList*SharepointServerName*                                                                                       
                   Customers*OracleServerName*                                                                                            |
    | **主机名** | 输入本地资源的完全限定的域名、计算机名或 IPv4 地址。示例包括：                                                         
                   *SQLServerName*                                                                                                        
                   *SQLServerName*.*Domain*.corp.*yourCompany*.com                                                                        
                   *myHTTPSharePointServer*                                                                                               
                   *myHTTPSharePointServer*.*yourCompany*.com                                                                             
                   *myFTPServer*                                                                                                          
                   10.100.10.10                                                                                                           |
    | **Port**   | 输入本地资源的端口号。例如，如果您使用的是网站，请输入端口 80 或端口 443。如果您使用的是 SQL Server，请输入端口 1433。 |

5.  选中复选标记。

可创建多个混合连接。有关允许的连接数，请参阅[BizTalk 服务：版本图表][BizTalk 服务：版本图表]。

## <a name="InstallHCM"></a>本地安装混合连接管理器

创建混合连接后，下载混合连接管理器并将其安装在本地资源上：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中，选择“BizTalk 服务”，然后选择您的 BizTalk 服务。
3.  选择“混合连接”选项卡：
    ![“混合连接”选项卡][“混合连接”选项卡]
4.  在任务栏中，选择“本地安装”：
    ![本地安装][本地安装]
5.  选择“安装和配置”以在本地系统上运行或下载混合连接管理器。
6.  选中复选标记。

混合连接支持安装在以下操作系统上的本地资源：

-   Windows Server 2008 R2
-   Windows Server 2012
-   Windows Server 2012 R2

安装混合连接管理器后，将发生以下情况：

-   在 Azure 上托管的混合连接将自动配置为使用主应用程序连接字符串。
-   本地资源将自动配置为使用主本地连接字符串。

本主题中的[管理混合连接][管理混合连接]提供有关连接字符串的详细信息。

## <a name="ManageHybridConnection"></a>管理混合连接

混合连接在 Azure BizTalk 服务中管理。[REST API][REST API] 还可用于管理混合连接。

**若要复制/重新生成混合连接字符串**：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  在左侧导航窗格中，选择“BizTalk 服务”，然后选择您的 BizTalk 服务。
3.  选择“混合连接”选项卡：
    ![“混合连接”选项卡][“混合连接”选项卡]
4.  选择混合连接。在任务栏中，选择“管理连接”：
    ![管理选项][管理选项]
    “管理连接”列出了应用程序和本地连接字符串。您可以复制连接字符串或重新生成在连接字符串中使用的访问密钥。
    **如果您选择重新生成**，将更改在连接字符串内使用的共享访问密钥。请执行以下操作：

-   在 Azure 管理门户中，选择 Azure 应用程序中的“同步密钥”。
-   重新运行“本地安装”。当您重新运行本地安装时，本地资源将自动配置为使用更新的主连接字符串。

## 下一步

-   [将 Azure 网站连接到本地资源][将 Azure 网站连接到本地资源]
-   [分步混合连接：从 Azure 网站连接到本地 SQL Server][分步混合连接：从 Azure 网站连接到本地 SQL Server]
-   [Azure 移动服务和混合连接][Azure 移动服务和混合连接]

## 另请参阅

-   [用于在 Windows Azure 上管理 BizTalk 服务的 REST API][REST API]
-   [BizTalk 服务：版本图表][BizTalk 服务：版本图表]
-   [使用 Azure 管理门户创建 BizTalk 服务][使用 Azure 管理门户创建 BizTalk 服务]
-   [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡][BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]

  [什么是混合连接]: HCOverview
  [创建混合连接]: #CreateHybridConnection
  [本地安装混合连接管理器]: #InstallHCM
  [管理混合连接]: #ManageHybridConnection
  [将 Azure 网站连接到本地资源]: http://go.microsoft.com/fwlink/p/?LinkId=397538
  [Azure 移动服务和混合连接]: http://azure.microsoft.com/zh-cn/documentation/articles/mobile-services-dotnet-backend-hybrid-connections-get-started
  [Azure 管理门户]: http://go.microsoft.com/fwlink/p/?LinkID=213885
  [创建 BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=329870
  [“混合连接”选项卡]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionTab.png
  [BizTalk 服务：版本图表]: http://go.microsoft.com/fwlink/p/?LinkID=302279
  [本地安装]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionOnPremSetup.png
  [REST API]: http://msdn.microsoft.com/library/azure/dn232347.aspx
  [管理选项]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionManageConn.png
  [分步混合连接：从 Azure 网站连接到本地 SQL Server]: http://go.microsoft.com/fwlink/?LinkID=397979
  [使用 Azure 管理门户创建 BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=302280
  [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]: http://go.microsoft.com/fwlink/p/?LinkID=302281
