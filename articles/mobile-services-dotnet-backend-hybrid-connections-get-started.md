<properties linkid="mobile-services-dotnet-backend-hybrid-connections-get-started" urlDisplayName="Connect to an on-premises SQL Server from an Azure mobile service using Hybrid Connections" pageTitle="Connect to an on-premises SQL Server from an Azure mobile service using Hybrid Connections - Azure Mobile Services" metaKeywords="" description="Learn how to connect to an on-premises SQL Server from an Azure mobile service using Hybrid Connections" metaCanonical="" services="" documentationCenter="Mobile" title="Connect to an on-premises SQL Server from an Azure mobile service using Hybrid Connections" authors="yavorg" solutions="" manager="" editor="mollybos" />

# 使用混合连接从 Azure 移动服务连接到本地 SQL Server

当企业过渡到云时，通常会出于技术、法规或安全原因而需要将某些资产保留在本地。借助移动服务，你可以轻松地在这些资产的顶部创建云托管的移动层，同时还可以使用混合连接在你的场所安全地与它们重新建立连接。支持的资产包括静态 TCP 端口上运行的任何资源，例如 Microsoft SQL Server、MySQL、HTTP Web API 和大多数自定义 Web 服务。

在本教程中，你将学习如何修改 .NET 后端移动服务，以使用本地 SQL Server 数据库，而不是随服务一起提供的默认 Azure SQL Database。尽管 JavaScript 后端移动服务支持混合连接，但本主题只涉及 .NET 后端移动服务。

本主题将指导你完成以下基本步骤：

1.  [先决条件][]
2.  [安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库][]
3.  [创建混合连接][]
4.  [安装本地混合连接管理器以完成连接][]
5.  [修改移动服务以使用连接][]

<a name="Prerequisites"></a>
## 先决条件

若要完成本教程，你需要安装以下产品。所有这些产品都提供了免费版，方便你开始针对 Azure 进行开发且无需支付任何费用。

-   "Visual Studio 2013" - 若要下载 Visual Studio 2013 免费试用版，请参阅 [Visual Studio 下载][]。请先安装此产品，然后继续。

-   "SQL Server 2014 Express with Tools" - 从 [Microsoft Web 平台数据库页][]免费下载 Microsoft SQL Server Express。选择 "Express"（而不是 LocalDB）版本。"Express with Tools" 版本包含本教程中将要使用的 SQL Server Management Studio。

你还需要准备一台使用混合连接与 Azure 建立连接的本地计算机。该计算机必须符合以下条件：

-   能够通过端口 5671 连接到 Azure
-   能够访问你想要连接的本地资源的 *hostname*:*portnumber*。该资源不一定托管在同一台计算机上。

<a name="InstallSQL"></a>
## 安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库

若要通过混合连接使用本地 SQL Server 或 SQL Server Express 数据库，需要在静态端口上启用 TCP/IP。SQL Server 上的默认实例使用静态端口 1433，而命名的实例则不使用该端口。

有关如何配置 SQL Server 以使其符合上述条件的详细说明，请参阅[安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库][1]。如果已在符合上述条件的配置和环境中安装了 SQL Server，则可以跳到后面的部分，开始[在本地创建 SQL Server 数据库][]。

在本教程中，我们假设数据库名称为 "OnPremisesDB"，该数据库在端口 "1433" 上运行，计算机的主机名为 "onPremisesServer"。

<a name="CreateHC"></a>
## 创建混合连接

1.  在本地计算机上，登录到 [Azure 管理门户][]。

2.  在导航窗格的底部，选择“+新建”，然后依次选择“应用服务”、“BizTalk 服务”和“自定义创建” 。

    ![创建 BizTalk 服务][]

3.  指定“BizTalk 服务名称”并选择“版本” 。

    ![配置新的 BizTalk 服务][]

    本教程使用 "mobile1"。你需要为新的 BizTalk 服务提供唯一名称。

4.  创建 BizTalk 服务后，请选择“混合连接”选项卡，然后单击“添加” 。

    ![添加混合连接][]

    这将会创建一个新的混合连接。

5.  指定混合连接的“名称”和“主机名”，然后将“端口”设置为 `1433` 。

    ![配置混合连接][]

    主机名就是本地服务器的名称。这会将混合连接配置为访问端口 1433 上运行的 SQL Server。

6.  创建新连接后，该连接将出现在下表中。

    ![已成功创建混合连接][]

    新连接的状态显示为“本地安装未完成” 。

现在，我们需要在本地计算机上安装混合连接管理器。

<a name="InstallHCM"></a>
## 安装本地混合连接管理器以完成连接

混合连接管理器可让你的本地计算机连接到 Azure 以及中继 TCP 流量。你必须在尝试通过 Azure 访问的本地计算机上安装此软件。

1.  选择刚刚创建的连接，然后在底部栏中单击“本地安装” 。

    ![本地安装][]

2.  单击“安装并配置” 。

    ![安装混合连接管理器][]

    此时将会安装连接管理器的自定义实例，该实例已预先经过配置，可以处理你刚刚创建的混合连接。

3.  完成连接管理器的余下安装步骤。

    ![混合连接管理器安装][]

    完成安装后，混合连接状态将更改为“已连接 1 个实例” 。你可能需要刷新浏览器并等待几分钟。本地安装现已完成。

    ![已连接混合连接][]

<a name="CreateService"></a>
## 修改移动服务以使用连接

### 将混合连接与服务相关联

1.  在门户的“移动服务”选项卡中，选择现有的移动服务或创建一个新的移动服务 。

    > [WACOM.NOTE]请务必选择使用 .NET 后端创建的服务，或者创建一个新的 .NET 后端移动服务。若要了解如何创建新的 .NET 后端移动服务，请参阅[移动服务入门][]

2.  在移动服务的“配置”选项卡上，找到“混合连接”部分并选择“添加混合连接” 。

    ![关联混合连接][]

3.  在“BizTalk 服务”选项卡上选择刚刚创建的混合连接，然后按“确定” 。

    ![选取已关联的混合连接][]

移动服务现在已与新的混合连接相关联。

### 更新服务以使用本地连接字符串

最后，我们需要创建一个应用程序设置，以便将连接字符串的值存储到本地 SQL Server。然后我们需要修改移动服务，以使用新的连接字符串。

1.  在“配置”选项卡上的“应用程序设置”中，添加名为 `onPremisesDatabase`、值类似于 `Server=onPremisesServer,1433;Database=OnPremisesDB;User ID=sa;Password={password}` 的新应用程序设置 。

    ![连接字符串的应用程序设置][]

    将 `{password}` 替换为本地数据库的安全密码。

2.  按“保存”以保存混合连接以及刚刚创建的应用程序设置 。

3.  在 Visual Studio 2013 中，打开定义了基于 .NET 的移动服务的项目。

    若要了解如何下载 .NET 后端项目，请参阅[移动服务入门][]。

4.  在解决方案资源管理器中，展开“Models” 文件夹并打开文件名以 *Context.cs* 结尾的数据模型文件。

5.  修改 "DbContext" 实例构造函数，使其类似于以下代码段：

        public class hybridService1Context :DbContext
        {
        public hybridService1Context()
        : base(ConfigurationManager.AppSettings["onPremisesDatabase"])
            {
            }

        // snipped
        }

    现在，服务将会使用 Azure 上应用程序设置中定义的新混合连接字符串。

6.  发布你的更改，并使用已连接到移动服务的客户端应用程序来调用某些操作，以生成数据库更改。

7.  在运行 SQL Server 的本地计算机上打开 SQL Management Studio，然后在对象资源管理器中依次展开“OnPremisesDB” 数据库和“表” 。

8.  右键单击“hybridService1.TodoItems” 表，然后选择“选择前 1000 行”以查看结果 。

    ![SQL Management Studio][]

移动服务已将应用程序中生成的更改推送到本地数据库。

## 另请参阅

-   [混合连接网站][]
-   [混合连接概述][]
-   [BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡][]

  [先决条件]: #Prerequisites
  [安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库]: #InstallSQL
  [创建混合连接]: #CreateHC
  [安装本地混合连接管理器以完成连接]: #InstallHCM
  [修改移动服务以使用连接]: #CreateService
  [Visual Studio 下载]: http://www.visualstudio.com/downloads/download-visual-studio-vs
  [Microsoft Web 平台数据库页]: http://www.microsoft.com/web/platform/database.aspx
  [1]: /zh-cn/documentation/articles/web-sites-hybrid-connection-connect-on-premises-sql-server#InstallSQL
  [在本地创建 SQL Server 数据库]: /zh-cn/documentation/articles/web-sites-hybrid-connection-connect-on-premises-sql-server#CreateSQLDB
  [Azure 管理门户]: http://go.microsoft.com/fwlink/p/?linkid=213885&clcid=0x409
  [创建 BizTalk 服务]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/1.png
  [配置新的 BizTalk 服务]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/2.png
  [添加混合连接]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/3.png
  [配置混合连接]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/4.png
  [已成功创建混合连接]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/5.png
  [本地安装]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/5-1.png
  [安装混合连接管理器]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/6.png
  [混合连接管理器安装]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/7.png
  [已连接混合连接]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/8.png
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
  [关联混合连接]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/9.png
  [选取已关联的混合连接]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/10.png
  [连接字符串的应用程序设置]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/11.png
  [SQL Management Studio]: ./media/mobile-services-dotnet-backend-hybrid-connections-get-started/12.png
  [混合连接网站]: http://azure.microsoft.com/zh-cn/services/biztalk-services/
  [混合连接概述]: http://go.microsoft.com/fwlink/p/?LinkID=397274
  [BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡]: http://azure.microsoft.com/zh-cn/documentation/articles/biztalk-dashboard-monitor-scale-tabs/
