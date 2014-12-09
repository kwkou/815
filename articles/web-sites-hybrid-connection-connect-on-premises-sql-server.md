<properties linkid="web-sites-hybrid-connection-getting-started" title="混合连接分步指南：从 Azure 网站连接至本地 SQL Server" pageTitle="混合连接分步指南：从 Azure 网站连接至本地 SQL Server" description="在 Microsoft Azure 上创建一个网站并将其连接至本地 SQL Server 数据库" metaKeywords="" services="web-sites" solutions="web" documentationCenter="" authors="timamm" manager="paulettm" editor="mollybos" videoId="" scriptId="" />

# 使用混合连接从 Azure 网站连接至本地 SQL Server

## 介绍

混合连接可以将 Microsoft Azure 网站连接至使用静态 TCP 端口的本地资源。受支持的资源包括 Microsoft SQL Server、HTTP Web API 以及大多数自定义 Web 服务。

在本教程中，您将了解如何在 Azure 预览版门户中创建网站、使用新的混合连接功能将该网站连接至您的本地 SQL Server 数据库以及创建将使用混合连接的简单 ASP.NET Web 应用程序并将该应用程序部署到 Azure 网站。在 Azure 上完成的网站将用户凭据存储在本地成员资格数据库中。本教程假定读者之前未使用过 Azure 或 ASP.NET。

> [WACOM.NOTE] 混合连接功能的网站部分仅在 [Azure 预览版门户][Azure 预览版门户]中有提供。若要在 BizTalk 服务中创建连接，请参阅[混合连接][混合连接]。

## 先决条件

若要完成本教程，你需要安装以下产品。所有这些产品都提供了免费版，方便你开始针对 Azure 进行开发且无需支付任何费用。

-   **Azure 订阅** — 有关免费订阅，请参见 [Azure 免费试用][Azure 免费试用]。

-   **Visual Studio 2013** - 若要下载 Visual Studio 2013 免费试用版，请参阅 [Visual Studio 下载][Visual Studio 下载]。请先安装此产品，然后继续。

-   **Microsoft .NET Framework 3.5 Service Pack 1** — 如果您的操作系统是 Windows 8.1、Windows Server 2012 R2、Windows 8、Windows Server 2012、Windows 7 或 Windows Server 2008 R2，可通过“控制面板”\>“程序和功能”\> 打开或关闭 Windows 功能来启用它。另外，您可以从 [Microsoft 下载中心][Microsoft 下载中心]进行下载。

-   **SQL Server 2014 Express with Tools** - 从 [Microsoft Web 平台数据库页][Microsoft Web 平台数据库页]免费下载 Microsoft SQL Server Express。选择 **Express**（而不是 LocalDB）版本。**Express with Tools** 版本包含本教程中将要使用的 SQL Server Management Studio。

-   **SQL Server Management Studio Express** — 其中包括上述 SQL Server 2014 Express with Tools 下载，但若需要单独安装，可从 [SQL Server Express 下载页面][Microsoft Web 平台数据库页]进行下载和安装。

本教程假定您订阅了 Azure、已安装 Visual Studio 2013 并安装或启用了 .NET Framework 3.5。本教程将向您介绍如何在与 Azure 混合连接功能协作良好的配置中安装 SQL Server 2014 Express（默认实例带有静态 TCP 端口）。开始本教程之前，如果您还未安装 SQL Server，请先从上述位置下载 SQL Server 2014 Express with Tools。

### 说明

若要通过混合连接使用本地 SQL Server 或 SQL Server Express 数据库，需要在静态端口上启用 TCP/IP。SQL Server 上的默认实例使用静态端口 1433，而命名的实例则不使用该端口。

要在其上安装本地混合连接管理器代理的计算机：

-   必须能够通过端口 5671 连接至 Azure
-   必须能够访问您的本地资源的 *hostname*:*portnumber*。

本文中的分步指南假定您从将托管本地混合连接代理的计算机使用浏览器。

如果已在符合上述条件的配置和环境中安装了 SQL Server，则可以跳到后面的部分，开始[在本地创建 SQL Server 数据库][在本地创建 SQL Server 数据库]。

## 本文内容

[A. 安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库][A. 安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库]

[B. 在 Azure 预览版门户中创建网站][B. 在 Azure 预览版门户中创建网站]

[C. 创建混合连接和 BizTalk 服务][C. 创建混合连接和 BizTalk 服务]

[D. 安装本地混合连接管理器以完成连接][D. 安装本地混合连接管理器以完成连接]

[E. 创建基本的 ASP.NET Web 项目、编辑数据库连接字符串并本地运行项目][E. 创建基本的 ASP.NET Web 项目、编辑数据库连接字符串并本地运行项目]

[F. 将 Web 应用程序发布到 Azure 并对其进行测试][F. 将 Web 应用程序发布到 Azure 并对其进行测试]

<a name="InstallSQL"></a>

## A. 安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库

本节将向您介绍如何安装 SQL Server Express、启用 TCP/IP 并创建一个数据库，以便您的 Web 应用程序可与 Azure 预览版环境结合使用。

### 安装 SQL Server Express

1.  若要安装 SQL Server Express，请运行下载的 **SQLEXPRWT\_x64\_ENU.exe** 或 **SQLEXPR\_x86\_ENU.exe** 文件。SQL Server 安装中心向导显示。

    ![SQL Server 安装][SQL Server 安装]

2.  选择“全新 SQL Server 独立安装或向现有安装添加功能”。按照说明操作，接受默认选项和设置，直至“实例配置”页面显示。

3.  在“实例配置”页面上，选择“默认实例”。

    ![选择默认实例][选择默认实例]

    默认情况下，SQL Server 的默认实例在静态端口 1433 上侦听来自 SQL Server 客户端的请求，而这正是混合连接功能所需的。命名实例使用不受混合连接支持的动态端口和 UDP。

4.  接受“服务器配置”页面上的默认值。

5.  在“数据库引擎配置”页面上，在“身份验证模式”下选择“混合模式（SQL Server 身份验证和 Windows 身份验证）”并输入密码。

    ![选择混合模式][选择混合模式]

    在本教程中，您将使用 SQL Server 身份验证。请确保记住您输入的密码，稍后将用到此密码。

6.  按向导提示逐步执行完其余操作以完成安装。

### 启用 TCP/IP

若要启用 TCP/IP，需要用到随 SQL Server Express 一起安装的 SQL Server 配置管理器。继续之前，请按照[针对 SQL Server 启用 TCP/IP 网络协议][针对 SQL Server 启用 TCP/IP 网络协议]的步骤进行。

<a name="CreateSQLDB"></a>

### 在本地创建 SQL Server 数据库

您的 Visual Studio Web 应用程序需要一个可由 Azure 访问的成员资格数据库。这要求是 SQL Server 或 SQL Server Express 数据库（不是 MVC 模板默认使用的 LocalDB 数据库），因此，接下来需要创建成员资格数据库。

1.  在 SQL Server Management Studio 中，连接至刚才安装的 SQL Server。（如果“连接到服务器”对话框没有自动显示，请导航至左侧窗格中的“对象资源管理器”，单击“连接”，然后单击“数据库引擎”。）
    ![连接到服务器][连接到服务器]

    对于“服务器类型”，请选择“数据库引擎”。对于“服务器名称”，可使用 **localhost** 或正在使用的计算机的名称。选择“SQL Server 身份验证”，然后以 sa 用户名和之前创建的密码进行登录。

2.  若要通过使用 SQL Server Management Studio 新建数据库，请右键单击“对象资源管理器”中的“数据库”，然后单击“新建数据库”。

    ![创建新的数据库][创建新的数据库]

3.  在“新建数据库”对话框中，输入 MembershipDB 作为数据库名称，然后单击“确定”。

    ![提供数据库名称][提供数据库名称]

    请注意，此时未对数据库进行任何更改。成员资格信息稍后将在您运行 Web 应用程序时由该程序自动进行添加。

4.  在“对象资源管理”中，如果展开“数据库”，您会看到成员资格数据库已经创建。

    ![MembershipDB 已创建][MembershipDB 已创建]

<a name="CreateSite"></a>

## B. 在 Azure 预览版门户中创建网站

> [WACOM.NOTE] 如果已在 Azure 预览版门户中创建了希望用于此教程的网站，可直接跳至[创建混合连接和 BizTalk 服务][C. 创建混合连接和 BizTalk 服务]。

1.  在 [Azure 预览版门户][Azure 预览版门户]的左下角处，单击“新建”，然后选择“网站”。

    ![新建按钮][新建按钮]

    ![新建网站][新建网站]

2.  在“网站”分页上为您的网站提供名称，然后单击“创建”。

    ![网站名称][网站名称]

3.  几分钟之后，网站创建成功并显示其网站分页。该分页是一个可垂直滚动的仪表板，便于您管理站点。

    ![运行中的网站][运行中的网站]

4.  若要验证该站点是否已激活，可单击“浏览”图标显示默认页面。

    ![单击浏览查看您的网站][单击浏览查看您的网站]

    ![默认的网站页面][默认的网站页面]

接下来，您将为该网站创建混合连接和 BizTalk 服务。

<a name="CreateHC"></a>

## C. 创建混合连接和 BizTalk 服务

1.  返回预览版门户，向下滚动网站的分页并选择“混合连接”。

    ![混合连接][1]

2.  在“混合连接”分页上，单击“添加”。

    ![添加混合连接][添加混合连接]

3.  “添加混合连接”分页打开。因为这是您的第一个混合连接，所以预先选择了“新建混合连接”选项，“创建混合连接”分页将打开。

    ![创建混合连接][创建混合连接]

    在“创建混合连接”分页上：

    -   在“名称”处为连接提供一个名称。
    -   在“主机名”处输入您的 SQL Server 主机的计算机名。
    -   在“端口”处输入 1433（SQL Server 的默认端口）。
    -   单击“Biz Talk 服务”

4.  “Biz Talk 服务”分页打开。为 BizTalk 服务输入一个名称，然后单击“确定”。

    ![创建 BizTalk 服务][创建 BizTalk 服务]

    “创建 Biz Talk 服务”分页关闭，您将返回“创建混合连接”分页。

5.  在“创建混合连接”分页上，单击“确定”。

    ![单击确定][单击确定]

6.  完成这一流程后，门户中的“通知”区域将告知您连接已创建成功。

    ![成功通知][成功通知]

7.  在网站分页上，“混合连接”图标现在显示已创建了一个混合连接。

    ![已创建一个混合连接][已创建一个混合连接]

此时，您已完成了云混合连接基础架构中最重要的一部分。接下来，您将创建相应的本地部分。

<a name="InstallHCM"></a>

## D. 安装本地混合连接管理器以完成连接

1.  在网站分页上，单击“混合连接”图标

    ![混合连接图标][混合连接图标]

2.  在“混合连接”分页上，最近添加的终结点的“状态”栏显示的是“未连接”。单击该连接对其进行配置。

    ![未连接][未连接]

    “混合连接”分页打开。

    ![未连接分页][未连接分页]

3.  在该分页上，单击“侦听器安装”。

    ![单击侦听器安装][单击侦听器安装]

4.  “混合连接属性”分页打开。在“本地混合连接管理器”下，选择“单击此处进行安装”。

    ![单击此处进行安装][单击此处进行安装]

5.  在“应用程序运行安全性警告”对话框中，选择“运行”以继续。

    ![选择运行以继续][选择运行以继续]

6.  在“用户帐户控制”对话框中，选择“是”。

    ![选择是][选择是]

7.  “混合连接管理器”开始下载并安装。

    ![安装][安装]

8.  安装完成后，单击“关闭”。

    ![单击关闭][单击关闭]

    在“混合连接”分页上，“状态”栏现在显示“已连接”。

    ![已连接状态][已连接状态]

现在，混合连接基础架构已完成，接下来创建一个 Web 应用程序开始使用此架构。

<a name="CreateASPNET"></a>

## E. 创建基本的 ASP.NET Web 项目、编辑数据库连接字符串并本地运行项目

### 创建基本的 ASP.NET 项目

1.  在 Visual Studio 中的“文件”菜单上，创建一个新项目：

    ![新建 Visual Studio 项目][新建 Visual Studio 项目]

2.  在“新建项目”对话框的“模板”部分中，依次选择“Web”和“ASP.NET Web 应用程序”，然后单击“确定”。

    ![选择 ASP.NET Web 应用程序][选择 ASP.NET Web 应用程序]

3.  在“新建 ASP.NET 项目”对话框中，选择“MVC”，然后单击“确定”。

    ![选择 MVC][选择 MVC]

4.  创建完项目后，应用程序自述页面显示。请先不要运行该 Web 项目。

    ![自述页面][自述页面]

### 为应用程序编辑数据库连接字符串

在此步骤中，将编辑要告知应用程序本地 SQL Server Express 数据库位置的连接字符串。连接字符串位于应用程序的 Web.config 文件中，其中包含该应用程序的配置信息。

> [WACOM.NOTE] 若要确保应用程序使用在 SQL Server Express 中创建的数据库而不是 Visual Studio 默认 LocalDB 中的数据库，在运行项目之前完成此步骤至关重要。

1.  在“解决方案资源管理器”中，双击 Web.config 文件。

    ![Web.config][Web.config]

2.  编辑 **connectionStrings** 部分以指向本地计算机上的 SQL Server 数据库，请遵循以下示例中的语法：

    ![连接字符串][连接字符串]

    撰写连接字符串时，请记住：

    -   如果连接至命名实例而不是默认实例（例如 YourServer\\SQLEXPRESS），必须对 SQL Server 进行配置以使用静态端口。有关配置静态端口的信息，请参见[如何配置 SQL Server 以在特定端口上进行侦听][如何配置 SQL Server 以在特定端口上进行侦听]。默认情况下，命名实例使用 UDP 和动态端口，而混合连接对此不支持。

    -   建议指定端口（默认为 1433，如示例中所示）和连接字符串，以便可确保本地 SQL Server 启用了 TCP 并使用的是正确的端口。

    -   请记住使用 SQL Server 身份验证进行连接，以在连接字符串中指定用户 ID 和密码。

3.  在 Visual Studio 中单击“保存”以保存 Web.config 文件。

### 本地运行项目并注册新用户

1.  现在，通过单击“调试”下的浏览按钮本地运行新的 Web 项目。本示例中使用的是 Internet Explorer。

    ![运行项目][运行项目]

2.  在默认网页的右上方，选择“注册”注册新帐户：

    ![注册新帐户][注册新帐户]

3.  输入用户名和密码：

    ![输入用户名和密码][输入用户名和密码]

    这将自动在为应用程序保存成员资格信息的本机 SQL Server 上创建数据库。表之一 (**dbo.AspNetUsers**) 保存类似刚才输入的网站用户凭据。您将在本教程稍后部分看到此表。

4.  关闭默认网页的浏览器窗口。这将停止 Visual Studio 中的应用程序。

现在可以进行下一步了 — 将应用程序发布到 Azure 并对其进行测试。

<a name="PubNTest"></a>

## F. 将 Web 应用程序发布到 Azure 并对其进行测试

现在，将您的应用程序发布到 Azure 上的网站，然后对其进行测试以查看如何使用之前配置的混合连接将您的网站应用程序连接至本地计算机上的数据库。

### 发布 Web 应用程序

1.  可在 Azure 门户中下载网站的发布配置文件。在网站的分页上，选择“下载发布配置文件”，然后将文件保存到您的计算机。

    ![下载发布配置文件][下载发布配置文件]

    ![下载文件夹中的发布配置文件][下载文件夹中的发布配置文件]

    接下来，将该文件导入您的 Visual Studio Web 应用程序。

2.  在 Visual Studio 中，右键单击“解决方案资源管理器”中的项目名称并选择“发布”。

    ![选择发布][选择发布]

3.  在“发布 Web”对话框中的“配置文件”选项卡上，选择“导入”。

    ![导入][导入]

4.  浏览至下载的发布配置文件，将其选定然后单击“确定”。

    ![浏览至配置文件][浏览至配置文件]

5.  您的发布信息获得导入并显示在对话框的“连接”选项卡上。

    ![单击发布][单击发布]

    单击“发布”。

    发布完成后，浏览器启动并显示的网站对于您而言似曾相识 — 只不过该网站运行于 Azure 云中！

接下来，将使用实时 Web 应用程序查看其正在使用的混合连接。

### 在 Azure 上测试已完成的 Web 应用程序

1.  在 Azure 上的网页的右上角选择“登录”。

    ![测试登录][测试登录]

2.  您的 Azure 网站现已连接至本地计算机上 Web 应用程序的成员资格数据库。若要对此进行验证，请输入与之前在本地数据库中所输相同的凭据登录。

    ![问候语][问候语]

3.  若要进一步测试新的混合连接，请注销你的 Azure Web 应用程序，并注册另一个用户。提供新的用户名和密码，然后单击“注册”。

    ![测试注册其他用户][测试注册其他用户]

4.  若要验证新用户的凭据是否已通过混合连接存储在本地数据库中，请在本地计算机上打开 SQL Management Studio。在“对象资源管理器”中，依次展开“MembershipDB”数据库和“表”。右键单击 **dbo.AspNetUsers** 成员资格表，选择“选择前 1000 行”以查看结果。

    ![查看结果][查看结果]

5.  本地成员资格表现在显示了两个帐户 — 一个是本地创建的，另一个是在 Azure 云中创建的。在云中创建的帐户已通过 Azure 的混合连接功能保存至本地数据库。

    ![本地数据库中注册的用户][本地数据库中注册的用户]

您现在已创建并部署了一个 ASP.NET Web 应用程序，此程序在本地 SQL Server 数据库与 Azure 云中的网站之间使用混合连接。祝贺你！

## 另请参阅

[混合连接网站][混合连接网站]

[混合连接概述][混合连接]

[BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡][BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡]

[使用混合连接从 Azure 移动服务连接到本地 SQL Server][使用混合连接从 Azure 移动服务连接到本地 SQL Server]

[ASP.NET 标识概述][ASP.NET 标识概述]

<!-- IMAGES -->

  [Azure 预览版门户]: https://portal.azure.com
  [混合连接]: http://go.microsoft.com/fwlink/p/?LinkID=397274
  [Azure 免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [Visual Studio 下载]: http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs
  [Microsoft 下载中心]: http://www.microsoft.com/zh-cn/download/details.aspx?displaylang=en&id=22
  [Microsoft Web 平台数据库页]: http://www.microsoft.com/web/platform/database.aspx
  [在本地创建 SQL Server 数据库]: #CreateSQLDB
  [A. 安装 SQL Server Express，启用 TCP/IP 并在本地创建一个 SQL Server 数据库]: #InstallSQL
  [B. 在 Azure 预览版门户中创建网站]: #CreateSite
  [C. 创建混合连接和 BizTalk 服务]: #CreateHC
  [D. 安装本地混合连接管理器以完成连接]: #InstallHCM
  [E. 创建基本的 ASP.NET Web 项目、编辑数据库连接字符串并本地运行项目]: #CreateASPNET
  [F. 将 Web 应用程序发布到 Azure 并对其进行测试]: #PubNTest
  [SQL Server 安装]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A01SQLServerInstall.png
  [选择默认实例]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A02ChooseDefaultInstance.png
  [选择混合模式]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A03ChooseMixedMode.png
  [针对 SQL Server 启用 TCP/IP 网络协议]: http://technet.microsoft.com/zh-cn/library/hh231672.aspx
  [连接到服务器]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A04SSMSConnectToServer.png
  [创建新的数据库]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A05SSMScreateNewDBlh.png
  [提供数据库名称]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A06SSMSprovideDBname.png
  [MembershipDB 已创建]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A07SSMSMembershipDBCreated.png
  [新建按钮]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B01New.png
  [新建网站]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B02NewWebsite.png
  [网站名称]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B03WebsiteCreationBlade.png
  [运行中的网站]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B04WebSiteRunningBlade.png
  [单击浏览查看您的网站]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B05Browse.png
  [默认的网站页面]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B06DefaultWebSitePage.png
  [1]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C01CreateHCHCIcon.png
  [添加混合连接]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C02CreateHCAddHC.png
  [创建混合连接]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C03TwinCreateHCBlades.png
  [创建 BizTalk 服务]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C04CreateHCCreateBTS.png
  [单击确定]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C05CreateBTScomplete.png
  [成功通知]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C06CreateHCSuccessNotification.png
  [已创建一个混合连接]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C07CreateHCOneConnectionCreated.png
  [混合连接图标]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D01HCIcon.png
  [未连接]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D02NotConnected.png
  [未连接分页]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D03NotConnectedBlade.png
  [单击侦听器安装]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D04ClickListenerSetup.png
  [单击此处进行安装]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D05ClickToInstallHCM.png
  [选择运行以继续]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D06ApplicationRunWarning.png
  [选择是]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D07UAC.png
  [安装]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D08HCMInstalling.png
  [单击关闭]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D09HCMInstallComplete.png
  [已连接状态]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D10HCStatusConnected.png
  [新建 Visual Studio 项目]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E01HCVSNewProject.png
  [选择 ASP.NET Web 应用程序]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E02HCVSChooseASPNET.png
  [选择 MVC]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E03HCVSChooseMVC.png
  [自述页面]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E04HCVSReadmePage.png
  [Web.config]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E05HCVSChooseWebConfig.png
  [连接字符串]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E06HCVSConnectionString.png
  [如何配置 SQL Server 以在特定端口上进行侦听]: http://support.microsoft.com/kb/823938
  [运行项目]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E06HCVSRunProject.png
  [注册新帐户]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E07HCVSRegisterLocally.png
  [输入用户名和密码]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E08HCVSCreateNewAccount.png
  [下载发布配置文件]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F01PortalDownloadPublishProfile.png
  [下载文件夹中的发布配置文件]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F02HCVSPublishProfileInDownloadsFolder.png
  [选择发布]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F03HCVSRightClickProjectSelectPublish.png
  [导入]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F04HCVSPublishWebDialogImport.png
  [浏览至配置文件]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F05HCVSBrowseToImportPubProfile.png
  [单击发布]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F06HCVSClickPublish.png
  [测试登录]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F07HCTestLogIn.png
  [问候语]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F08HCTestHelloContoso.png
  [测试注册其他用户]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F09HCTestRegisterRelecloud.png
  [查看结果]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F10HCTestSSMSTree.png
  [本地数据库中注册的用户]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F11HCTestShowMemberDb.png
  [混合连接网站]: http://azure.microsoft.com/zh-cn/services/biztalk-services/
  [BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡]: http://azure.microsoft.com/zh-cn/documentation/articles/biztalk-dashboard-monitor-scale-tabs/
  [使用混合连接从 Azure 移动服务连接到本地 SQL Server]: http://azure.microsoft.com/zh-cn/documentation/articles/mobile-services-dotnet-backend-hybrid-connections-get-started/
  [ASP.NET 标识概述]: http://www.asp.net/identity
