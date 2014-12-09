<properties linkid="manage-services-how-to-deploy-websites" pageTitle="如何部署 Azure 网站" metaKeywords="Azure deploy publish web site" description="了解可使用哪些方法将内容部署到 Azure 网站。" metaCanonical="" services="web-sites" documentationCenter="" title="如何部署 Azure 网站" authors="tdykstra" solutions="" manager="wpickett" editor="mollybos" />

# 如何部署 Azure 网站

您具有许多可用于将自己的内容部署到 Azure 网站的选项。本主题将提供每个选项的简要概述和指向详细信息的链接。

-   [从云托管的源代码管理系统进行部署][从云托管的源代码管理系统进行部署]

    -   Visual Studio Online (VSO)

<!--     * Repository web sites using Git     * Repository web sites using Mercurial     * Dropbox  -->

-   [从 IDE 进行部署][从 IDE 进行部署]

    -   Visual Studio
    -   WebMatrix
-   [通过使用 FTP 实用程序进行部署][通过使用 FTP 实用程序进行部署]
-   [从本地源代码管理系统进行部署][从本地源代码管理系统进行部署]

    -   Team Foundation Server (TFS)

<!--     * On-premises Git or Mercurial repositories  -->

-   [使用命令行工具和 Azure REST 管理 API][使用命令行工具和 Azure REST 管理 API]

    -   MSBuild
    -   FTP 脚本
    -   Windows PowerShell
    -   .NET 管理 API
    -   跨平台命令行 (xpat-cli)
    -   Web 部署命令行

## <a name="cloud"></a>从云托管的源代码管理系统进行部署

部署网站的最佳方法是设置与[源代码管理系统][源代码管理系统]集成的[持续交付工作流][持续交付工作流]。自动化不仅使开发过程更高效，还可以使您的备份和还原流程更可控、更可靠。

如果您尚未设置源代码管理，最简单的方法是从使用云托管的源代码管理系统开始。

### <a name="vso"></a>Visual Studio Online (VSO)

[Visual Studio Online][Visual Studio Online]（以前称为 Team Foundation Service）是 Microsoft 针对源代码管理和团队协作的基于云的解决方案。该服务对最多 5 个开发人员的团队免费。您可以设置 VSO 以向 Azure 网站持续交付，并且您的存储库可以使用 [Git 或 TFVC][Visual Studio Online]。

有关详细信息，请参阅以下资源：

-   [使用 VSO 和 TFVC 向 Azure 持续交付][使用 VSO 和 TFVC 向 Azure 持续交付]。分步教程演示如何使用 TFVC 设置从 VSO 到 Azure 网站的持续交付。TFVC 是 VSO 中的集中式源代码管理选项，与分布式源代码管理选项 Git 截然相反。
-   [使用 VSO 与 TFVC 向 Azure 持续交付][使用 VSO 与 TFVC 向 Azure 持续交付]。所介绍的方案类似于前面的教程，但本教程还遍历了注册 VSO 帐户和将项目签入到源代码管理的步骤。本教程主要针对云服务所编写，因此它的部分内容不适用于网站。

### <a name="git"></a>使用 Git 的存储库网站

[Git][Visual Studio Online] 是一种流行的分布式源代码管理系统。Azure 具有的内置功能可轻松实现从流行的基于 Web 的存储库网站（存储 Git 存储库，包括 [GitHub][GitHub]）自动部署到 Azure 网站。使用 Git 进行部署的一个优点是比较容易回滚到以前的部署（一旦有必要）。

有关详细信息，请参阅以下资源：

-   [从源代码管理发布到 Azure 网站][从源代码管理发布到 Azure 网站]。如何使用 Git 直接从本地计算机发布到 Azure 网站（在 Azure 中，此发布方法称为“本地 Git”）。还将演示如何启用从 GitHub 进行 Git 存储库的连续部署。
-   [Git、Mercurial 的 Azure 论坛][Git、Mercurial 的 Azure 论坛]。

### <a name="mercurial"></a>使用 Mercurial 的存储库网站

如果将 [Mercurial][Mercurial] 用作源代码管理系统，并将存储库存储在 [CodePlex][CodePlex] 或 [BitBucket][BitBucket] 中，则可以使用 Azure 网站中的内置功能来自动部署内容。

有关如何使用 Mercurial 进行部署的信息，请参阅以下资源：

-   [从源代码管理发布到 Azure 网站][从源代码管理发布到 Azure 网站]。尽管本教程演示的是如何发布 Git 存储库，但 CodePlex 或 BitBucket 中托管的 Mercurial 存储库的发布过程与此类似。
-   [Git、Mercurial 的 Azure 论坛][Git、Mercurial 的 Azure 论坛]。

<!-- ###<a name="dropbox"></a>DropBox  [DropBox](https://www.dropbox.com/) is not a source control system, but if you store your source code in DropBox you can automate deployment from your DropBox account.  * [Deploy To Windows Azure Using Dropbox](http://blogs.msdn.com/b/windowsazure/archive/2013/03/19/new-deploy-to-windows-azure-web-sites-from-dropbox.aspx). How to use the Azure Management Portal to set up DropBox deployment. * [DropBox and Azure Web Sites](http://channel9.msdn.com/Series/Windows-Azure-Web-Sites-Tutorials/Dropbox-Deployment-to-Windows-Azure-Web-Sites). This video walks through the process of connecting a DropBox folder to an Azure Web Site, and shows how quickly you can get a web site up and running or maintain it using simple drag-and-drop deployment. * [Azure Forum for Git, Mercurial, and DropBox](http://social.msdn.microsoft.com/Forums/windowsazure/en-US/home?forum=azuregit). -->

## <a name="ide"></a>从 IDE 进行部署

[Visual Studio][Visual Studio] 和 [WebMatrix][WebMatrix] 是可用于 Web 开发的 Microsoft IDE（集成开发环境）。两者都提供内置功能，使您可以轻松部署到 Azure 网站。两者都可以使用 [Web 部署][Web 部署]来自动执行其他与部署相关的任务，例如数据库部署和连接字符串的更改。两者也都可以通过使用 [FTP 或 FTPS][FTP 或 FTPS] 进行部署。

WebMatrix 可快速安装且易于学习，但 Visual Studio 提供了更多用于处理 Azure 网站的功能。您可以从 Visual Studio IDE 中创建、停止、启动和删除 Azure 网站、在实时创建日志时查看日志、进行远程调试等等。Visual Studio 还可以与源代码管理系统（如 [Visual Studio Online][1]、[Team Foundation Server][Team Foundation Server] 和 [Git 存储库][Git 存储库]）集成。

### <a name="vs"></a>Visual Studio

有关如何从 Visual Studio 部署到 Azure 网站的信息，请参阅以下资源：

-   [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]。如何使用 Visual Studio 和 Web 部署来创建和部署一个简单的 ASP.NET MVC Web 项目。
-   [使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 5 应用部署到 Azure 网站][使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 5 应用部署到 Azure 网站]。如何使用 Visual Studio、Web 部署和 Entity Framework Code First 迁移通过 SQL 数据库来创建和部署 ASP.NET MVC Web 项目。
-   [Visual Studio 和 ASP.NET 的 Web 部署概述][Visual Studio 和 ASP.NET 的 Web 部署概述]。使用 Visual Studio 的 Web 部署的基本简介。虽然已过时，但包括仍然相关的信息，其中包括用于部署数据库和 Web 应用程序的选项的概述，以及其他部署任务的列表（这些任务可能需要您来执行，或需要您手动配置 Visual Studio 以代替您执行）。本主题提供有关常规部署的信息，不只是关于部署到 Azure 网站。
-   [使用 Visual Studio 的 ASP.NET Web 部署][使用 Visual Studio 的 ASP.NET Web 部署]。共 12 篇的系列教程涵盖了比此列表中其他部署任务更完整的部署任务。
-   [在 Visual Studio 2012 中直接从 Git 存储库将 ASP.NET 网站部署到 Azure][在 Visual Studio 2012 中直接从 Git 存储库将 ASP.NET 网站部署到 Azure]。说明如何在 Visual Studio 中部署 ASP.NET Web 项目（使用 Git 插件将代码提交到 Git 并将 Azure 连接到 Git 存储库）。

### <a name="webmatrix"></a>WebMatrix

有关如何从 WebMatrix 部署到 Azure 网站的信息，请参阅以下资源：

-   [使用 Microsoft WebMatrix 开发和部署网站][使用 Microsoft WebMatrix 开发和部署网站]。如何使用 WebMatrix 模板创建简单的 ASP.NET 网站，并使用 WebMatrix 和 Web 部署将其部署到 Azure 网站。
-   [使用 WebMatrix 构建 Node.js 网站并将其部署到 Azure][使用 WebMatrix 构建 Node.js 网站并将其部署到 Azure]。

<!-- * [Create and deploy a PHP-MySQL Azure Web Site using WebMatrix](http://www.windowsazure.com/zh-cn/documentation/articles/web-sites-php-mysql-use-webmatrix/).  -->

-   [WebMatrix 3:集成 Git 与到 Azure 的部署][WebMatrix 3:集成 Git 与到 Azure 的部署]。如何使用 WebMatrix 从 Git 源代码管理存储库进行部署。

## <a name="ftp"></a>通过使用 FTP 实用程序进行部署

无论您使用哪种 IDE，您都还可以通过使用 [FTP][FTP 或 FTPS] 复制文件来将内容部署到您的网站。可以轻松地创建针对 Azure 网站的 FTP 凭据，并可以将其用于任何可使用 FTP 的应用程序，包括浏览器（例如 Internet Explorer）和功能全面的免费实用程序（例如 [FileZilla][FileZilla]）。Azure 网站还支持更安全的 FTPS 协议。

尽管使用 FTP 实用程序将网站的文件复制到 Azure 很容易，但它们并不会自动负责或协调相关的部署任务，例如部署数据库或更改连接字符串。此外，许多 FTP 工具不会对源文件和目标文件进行比较以跳过对未更改的文件的复制。对于大型站点，始终复制所有文件可能会导致部署时间较长，即使对于少量更新也是如此，因为将始终复制所有文件。

有关详细信息，请参阅以下资源：

<!-- * [Create a PHP-MySQL Azure Web Site and Deploy Using FTP](/zh-cn/documentation/articles/web-sites-php-mysql-deploy-use-ftp/).   -->

-   [如何管理网站][如何管理网站]。包括 PHP 教程中未包括的有关如何设置 FTP 凭据的其他信息。有关文档本身缺少的有关如何获取凭据的信息，请参阅页面底部的注释。

## <a name="onpremises"></a>从本地源代码管理系统进行部署

如果要在本地（非云托管的）存储库中使用 TFS、Git 或 Mercurial，则可以直接从存储库部署到 Azure 网站。

### <a name="tfs"></a>Team Foundation Server (TFS)

Team Foundation Server 是 Microsoft 针对源代码管理和团队协作的本地解决方案。您可以设置 TFS 以进行到 Azure 网站的连续交付。

有关详细信息，请参阅以下资源：

-   [在 Azure 中持续交付云服务][在 Azure 中持续交付云服务]。本文档适用于 Azure 云服务，但其部分内容与网站有关。

### <a name="gitmercurial"></a>本地 Git 或 Mercurial 存储库

在 Azure 中，您可以输入使用 Git 或 Mercurial 的任何存储库的 URL，以从该位置进行部署。您可以直接从本地 Git 存储库推送到 Azure 网站。

有关详细信息，请参阅以下资源：

-   [从源代码管理发布到 Azure 网站][从源代码管理发布到 Azure 网站]。如何使用 Git 直接从本地计算机发布到 Azure 网站（在 Azure 中，此发布方法称为“本地 Git”）。还将演示如何启用从 GitHub 进行 Git 存储库的连续部署。
-   [从任何 git/hg 存储库发布到 Azure 网站][从任何 git/hg 存储库发布到 Azure 网站]。David Ebbo 的博客，其中介绍了 Azure 网站中的“外部存储库”功能。
-   [Git、Mercurial 和 DropBox 的 Azure 论坛][Git、Mercurial 的 Azure 论坛]。
-   [将两个网站从一个 Git 存储库部署到 Azure][将两个网站从一个 Git 存储库部署到 Azure]。Scott Hanselman 的博客文章。

## <a name="commandline"></a>使用命令行工具和 Azure REST 管理 API

最好自动执行开发工作流，但如果不能直接在源代码管理系统中做到这一点，则可以使用命令行工具手动进行设置。这通常涉及到多个工具或框架的使用，因为部署过程通常涉及到执行网站管理功能以及复制内容。

Azure 通过提供 REST 管理 API 和几个可使该 API 更易使用的框架，来简化您可能需要为部署执行的站点管理任务。

### <a name="msbuild"></a>MSBuild

如果要将 [Visual Studio IDE][Visual Studio IDE] 用于部署，则可以使用 [MSBuild][MSBuild] 来自动执行可在 IDE 中完成的任何任务。可以对 MSBuild 进行配置，以使用 [Web 部署][2]或 [FTP/FTPS][通过使用 FTP 实用程序进行部署] 来复制文件。Web 部署还可以自动执行许多其他与部署相关的任务，例如部署数据库。

有关使用 MSBuild 的命令行部署的详细信息，请参阅以下资源：

-   [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署][使用 Visual Studio 的 ASP.NET Web 部署：命令行部署]。有关使用 Visual Studio 部署到 Azure 的一系列教程中的第十篇。演示在 Visual Studio 中设置发布配置文件之后如何使用命令行来进行部署。
-   [深入了解 Microsoft 生成引擎：使用 MSBuild 和 Team Foundation Build][MSBuild]。纸印书籍，其中包含了有关如何使用 MSBuild 进行部署的章节。

### <a name="ftp2"></a>FTP 脚本

可以轻松地创建针对 Azure 网站的 [FTP/FTPS][FTP 或 FTPS] 凭据，然后可以将这些凭据用于 FTP 批处理脚本。

有关详细信息，请参阅以下资源：

-   [使用 FTP 批处理脚本][使用 FTP 批处理脚本]。

### <a name="powershell"></a>Windows PowerShell

可以从 [Windows PowerShell][Windows PowerShell] 执行 MSBuild 或 FTP 功能。如果您这样做，则还可以使用 Windows PowerShell cmdlet 集合（可使 Azure REST 管理 API 易于调用）。

有关详细信息，请参阅以下资源：

-   [使用 Azure 构建真实世界云应用 – 使一切自动化][使用 Azure 构建真实世界云应用 – 使一切自动化]。电子书章节，其中介绍电子书中所示的示例应用程序如何使用 Windows PowerShell 脚本来创建 Azure 测试环境并部署到该环境中。请参阅[资源][资源]部分以获取指向其他 Azure PowerShell 文档的链接。

### <a name="api"></a>.NET 管理 API

可以编写 C# 代码以执行 MSBuild 或 FTP 功能进行部署。如果您这样做，则可以访问 Azure 管理 REST API 来执行站点管理功能。

有关详细信息，请参阅以下资源：

-   [使用 Azure 管理库和 .NET 使一切自动化][使用 Azure 管理库和 .NET 使一切自动化]。.NET 管理 API 的简介和指向更多文档的链接。

### <a name="cli"></a>跨平台命令行 (xpat-cli)

通过使用 FTP，可以在 Mac 或 Linux 计算机中通过命令行来进行部署。如果您这样做，则还可以访问使用 Azure 跨平台命令行接口 (xpat-cli) 的 Azure REST 管理 API。还可以在 Windows 计算机上使用 xpat-cli。

有关详细信息，请参阅以下资源：

-   [命令行工具][命令行工具]。WindowsAzure.com 中有关命令行工具信息的门户页面。

### <a name="webdeploy"></a>Web 部署命令行

[Web 部署][Web 部署]是用于部署到 IIS 的 Microsoft 软件，它不仅提供智能文件同步功能，还可以执行或协调许多其他与部署相关的、使用 FTP 时无法自动执行的任务。例如，Web 部署可以部署新的数据库或数据库更新以及您的网站。Web 部署还可以尽量减少更新现有站点所需的时间，因为它可以智能地仅复制更改过的文件。Microsoft WebMatrix、Visual Studio、Visual Studio Online 和 Team Foundation Server 都支持内置 Web 部署，但您也可以直接从命令行使用 Web 部署以使部署自动化。Web 部署命令非常强大，但在学习过程中可能会遇到困难。

有关详细信息，请参阅以下资源：

-   [Web 部署工具][Web 部署工具]。Microsoft TechNet 网站上的正式文档。虽然已过时，但仍是一个不错的起点。
-   [使用 Web 部署][使用 Web 部署]。Microsoft IIS.NET 网站上的正式文档。虽然也已过时，但仍是一个不错的起点。
-   [堆栈溢出][堆栈溢出]。获得有关如何从命令行中使用 Web 部署的最新信息的最佳位置。
-   [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署][使用 Visual Studio 的 ASP.NET Web 部署：命令行部署]。MSBuild 是由 Visual Studio 使用的生成引擎，还可以从命令行使用它以将 Web 应用程序部署到 Azure 网站。本教程是主要介绍 Visual Studio 部署的一系列教程的一部分。

## <a name="nextsteps"></a> 后续步骤

在某些情况下，您可能想要能够轻松地在网站的过渡版本和生产版本之间来回切换。可以使用 Azure 网站过渡部署功能来实现该操作。有关详细信息，请参阅 [Microsoft Azure 网站上的过渡部署][Microsoft Azure 网站上的过渡部署]。

准备好备份和还原计划是任何部署工作流的一个重要部分。有关 Azure 网站的备份和还原功能的信息，请参阅 [Azure 网站备份][Azure 网站备份]。

有关其他部署主题的信息，请参阅 [Azure 网站文档][Azure 网站文档]中的“部署”部分。

  [从云托管的源代码管理系统进行部署]: #cloud
  [从 IDE 进行部署]: #ide
  [通过使用 FTP 实用程序进行部署]: #ftp
  [从本地源代码管理系统进行部署]: #onpremises
  [使用命令行工具和 Azure REST 管理 API]: #commandline
  [源代码管理系统]: http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control
  [持续交付工作流]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/continuous-integration-and-continuous-delivery
  [Visual Studio Online]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control#gittfs
  [使用 VSO 和 TFVC 向 Azure 持续交付]: http://www.visualstudio.com/zh-cn/learn/continuous-delivery-in-vs
  [使用 VSO 与 TFVC 向 Azure 持续交付]: /zh-cn/documentation/articles/cloud-services-continuous-delivery-use-vso/
  [GitHub]: http://www.github.com
  [从源代码管理发布到 Azure 网站]: /zh-cn/documentation/articles/web-sites-publish-source-control/
  [Git、Mercurial 的 Azure 论坛]: http://social.msdn.microsoft.com/Forums/windowsazure/en-US/home?forum=azuregit
  [Mercurial]: http://mercurial.selenic.com/
  [CodePlex]: http://www.codeplex.com/
  [BitBucket]: https://bitbucket.org/
  [Visual Studio]: http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs.aspx
  [WebMatrix]: http://www.microsoft.com/web/webmatrix/
  [Web 部署]: http://www.iis.net/downloads/microsoft/web-deploy
  [FTP 或 FTPS]: http://en.wikipedia.org/wiki/File_Transfer_Protocol
  [1]: #vso
  [Team Foundation Server]: #tfs
  [Git 存储库]: #git
  [Azure 和 ASP.NET 入门]: /zh-cn/develop/net/tutorials/get-started/
  [使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 5 应用部署到 Azure 网站]: /zh-cn/develop/net/tutorials/web-site-with-sql-database/
  [Visual Studio 和 ASP.NET 的 Web 部署概述]: http://msdn.microsoft.com/zh-cn/library/dd394698.aspx
  [使用 Visual Studio 的 ASP.NET Web 部署]: http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/introduction
  [在 Visual Studio 2012 中直接从 Git 存储库将 ASP.NET 网站部署到 Azure]: http://www.dotnetcurry.com/ShowArticle.aspx?ID=881
  [使用 Microsoft WebMatrix 开发和部署网站]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-using-webmatrix/
  [使用 WebMatrix 构建 Node.js 网站并将其部署到 Azure]: http://www.windowsazure.com/zh-cn/documentation/articles/web-sites-nodejs-use-webmatrix/
  [WebMatrix 3:集成 Git 与到 Azure 的部署]: http://www.codeproject.com/Articles/577581/Webmatrixplus3-3aplusIntegratedplusGitplusandplusD
  [FileZilla]: https://filezilla-project.org/
  [如何管理网站]: http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-manage-websites/
  [在 Azure 中持续交付云服务]: /zh-cn/develop/net/common-tasks/continuous-delivery/
  [从任何 git/hg 存储库发布到 Azure 网站]: http://blog.davidebbo.com/2013/04/publishing-to-azure-web-sites-from-any.html
  [将两个网站从一个 Git 存储库部署到 Azure]: http://www.hanselman.com/blog/DeployingTWOWebsitesToWindowsAzureFromOneGitRepository.aspx
  [Visual Studio IDE]: #vs
  [MSBuild]: http://msbuildbook.com/
  [2]: #webdeploy
  [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署]: http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment
  [使用 FTP 批处理脚本]: http://support.microsoft.com/kb/96269
  [Windows PowerShell]: http://msdn.microsoft.com/zh-cn/library/dd835506.aspx
  [使用 Azure 构建真实世界云应用 – 使一切自动化]: http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything
  [资源]: http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything#resources
  [使用 Azure 管理库和 .NET 使一切自动化]: http://www.hanselman.com/blog/PennyPinchingInTheCloudAutomatingEverythingWithTheWindowsAzureManagementLibrariesAndNET.aspx
  [命令行工具]: /zh-cn/downloads/#cmd-line-tools
  [Web 部署工具]: http://technet.microsoft.com/zh-cn/library/dd568996
  [使用 Web 部署]: http://www.iis.net/learn/publish/using-web-deploy
  [堆栈溢出]: http://www.stackoverflow.com
  [Microsoft Azure 网站上的过渡部署]: /zh-cn/documentation/articles/web-sites-staged-publishing/
  [Azure 网站备份]: /zh-cn/documentation/articles/web-sites-backup/
  [Azure 网站文档]: /zh-cn/documentation/services/web-sites/
