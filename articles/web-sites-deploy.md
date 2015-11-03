<properties
	pageTitle="在 Azure 网站中部署 Web 应用"
	description="了解将内容部署到 Web Apps 的可用方法。"
	services="app-service\web"
	documentationCenter=""
	authors="tdykstra"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service-web"
	ms.date="08/14/2015"
	wacn.date="10/03/2015"/>

#如何部署 Azure 网站

你具有许多可用于将自己的内容部署到 Azure 网站的选项。本主题将提供每个选项的简要概述和指向详细信息的链接。

###<a name="cloud"></a>从云托管的源代码管理系统进行部署
部署 Web 应用的最佳方法是设置与[源代码管理系统](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control)集成的[持续交付工作流](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/continuous-integration-and-continuous-delivery)。自动化不仅使开发过程更高效，还可以使你的备份和还原流程更可控、更可靠。

如果你尚未设置源代码管理，最简单的方法是从使用云托管的源代码管理系统开始。

* [Visual Studio Online](#vso)
* [使用 Git 的存储库网站](#git)
* [使用 Mercurial 的存储库网站](#mercurial)
* [Dropbox](#dropbox)

###<a name="ide"></a>从 IDE 部署

[Visual Studio](http://www.visualstudio.com/) 和 [WebMatrix](http://www.microsoft.com/web/webmatrix/) 是可用于 Web 开发的 Microsoft IDE（集成开发环境）。两者都提供内置功能，使你可以轻松部署到 Web 应用。两者都可以使用 [Web 部署](http://www.iis.net/downloads/microsoft/web-deploy)来自动执行其他与部署相关的任务，例如数据库部署和连接字符串的更改。两者也都可以通过使用 [FTP 或 FTPS](http://zh.wikipedia.org/wiki/File_Transfer_Protocol) 进行部署。

WebMatrix 安装快速且易于学习，但 Visual Studio 提供了更多用于处理 Web Apps 的功能。你可以从 Visual Studio IDE 中创建、停止、启动和删除 Web 应用、在实时创建日志时查看日志、进行远程调试等。Visual Studio 还可以与源代码管理系统（如 [Visual Studio Online](#vso)、[Team Foundation Server](#tfs) 和 [Git 存储库](#git)）集成。

* [Visual Studio](#vs)
* [WebMatrix](#webmatrix)

###<a name="ftp"></a>使用 FTP 实用工具部署

无论你使用哪种 IDE，你都可以通过使用 [FTP](http://zh.wikipedia.org/wiki/File_Transfer_Protocol) 复制文件来将内容部署到你的应用。可以轻松地创建针对 Web 应用的 FTP 凭据，并可以将其用于任何可使用 FTP 的应用程序，包括浏览器（例如 Internet Explorer）和功能全面的免费实用程序（例如 [FileZilla](https://filezilla-project.org/)）。Web Apps 还支持更安全的 FTPS 协议。

尽管使用 FTP 实用程序将 Web 应用的文件复制到 Azure 很容易，但它们并不会自动负责或协调相关的部署任务，例如部署数据库或更改连接字符串。此外，许多 FTP 工具不会对源文件和目标文件进行比较以跳过对未更改的文件的复制。对于大型应用，始终复制所有文件可能会导致部署时间较长，即使对于少量更新也是如此，因为将始终复制所有文件。

###<a name="onpremises"></a>从本地源代码管理系统进行部署

如果要在本地（非云托管的）存储库中使用 TFS、Git 或 Mercurial，则可以直接从存储库部署到 Web 应用。

* [Team Foundation Server (TFS)](#tfs)
* [本地 Git 或 Mercurial 存储库](#onpremises)

###<a name="commandline"></a>使用命令行工具和 Azure REST 管理 API

最好自动执行开发工作流，但如果不能直接在源代码管理系统中做到这一点，则可以使用命令行工具手动进行设置。这通常涉及到多个工具或框架的使用，因为部署过程通常涉及到执行网站管理功能以及复制内容。

Azure 通过提供 REST 管理 API 和几个可使该 API 更易使用的框架，来简化你可能需要为部署执行的站点管理任务。

* [FTP](#ftp)
* [MSBuild](#msbuild)
* [FTP 脚本](#ftp2)
* [Windows PowerShell](#powershell)
* [.NET 管理 API](#api)
* [Azure 命令行界面 (Azure CLI)](#cli)
* [Web 部署命令行](#webdeploy)
 
###<a name="octopus"></a>Octopus 部署

[Octopus 部署](http://en.wikipedia.org/wiki/Octopus_Deploy)可以与 Web Apps 一同使用。有关更多信息，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站](https://octopusdeploy.com/blog/deploy-aspnet-applications-to-azure-websites)。


##<a name="vso"></a>Visual Studio Online

[Visual Studio Online](http://www.visualstudio.com/)（以前称为 Team Foundation Service）是 Microsoft 针对源代码管理和团队协作的基于云的解决方案。该服务对最多 5 个开发人员的团队免费。你可以向 Azure 网站持续交付，并且你的存储库可以使用 [Git 或 TFVC](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control#gittfs)。

有关详细信息，请参阅以下资源：

* [使用 Visual Studio Online 和 TFVC 向 Azure 持续传送项目](/documentation/articles/cloud-services-continuous-delivery-use-vso)。分步教程演示如何使用 TFVC 设置从 Visual Studio Online 的持续交付。TFVC 是集中式源代码管理选项，与分布式源代码管理选项 Git 截然相反。
* [使用 Visual Studio Online 和 Git 向 Azure 持续传送项目](/documentation/articles/cloud-services-continuous-delivery-use-vso-git)。与上一个教程类似，但使用 Git 而不是 TFVC。

##<a name="git"></a>使用 Git 的存储库网站

[Git](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control#gittfs) 是一种流行的分布式源代码管理系统。Azure 具有的内置功能可轻松实现从流行的基于 Web 的存储库网站（存储 Git 存储库，包括 [GitHub](http://www.github.com)、[CodePlex](http://www.codeplex.com/) 和 [BitBucket](https://bitbucket.org/)）自动部署到 Web 应用。使用 Git 进行部署的一个优点是比较容易回滚到以前的部署（一旦有必要）。

有关详细信息，请参阅以下资源：

* [从源代码管理发布到具有 Git 的 Azure 网站](/documentation/articles/web-sites-publish-source-control)。如何使用 Git 直接从本地计算机发布到 Azure 网站（在 Azure 中，此发布方法称为“本地 Git”）。还将演示如何启用从 GitHub、CodePlex 或 BitBucket 进行 Git 存储库的连续部署。
* [部署到 Azure 网站的 Azure 按钮](http://azure.microsoft.com/blog/2014/11/13/deploy-to-azure-button-for-azure-websites-2/)。有关用于触发从 Git 存储库的部署方法的博客。
* [Git、Mercurial 和 DropBox 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchszh-cn/home?forum=azuregit)。

##<a name="mercurial"></a>使用 Mercurial 的存储库网站

如果将 [Mercurial](http://mercurial.selenic.com/) 用作源代码管理系统，并将存储库存储在 [CodePlex](http://www.codeplex.com/) 或 [BitBucket](https://bitbucket.org/) 中，则可以使用 Azure 网站中的内置功能来自动部署内容。

有关如何使用 Mercurial 进行部署的信息，请参阅以下资源：

* [从源控件发布到 Azure 网站](/documentation/articles/web-sites-publish-source-control/)。尽管本教程演示的是如何发布 Git 存储库，但 CodePlex 或 BitBucket 中托管的 Mercurial 存储库的发布过程与此类似。
* [Git、Mercurial 和 DropBox 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchszh-cn/home?forum=azuregit)。

<!--
###<a name="dropbox"></a>DropBox

[DropBox](https://www.dropbox.com/) is not a source control system, but if you store your source code in DropBox you can automate deployment from your DropBox account.

* [Deploy To Windows Azure Using Dropbox](http://blogs.msdn.com/b/windowsazure/archive/2013/03/19/new-deploy-to-windows-azure-web-sites-from-dropbox.aspx). How to use the Azure Management Portal to set up DropBox deployment.
* [DropBox and Azure  Websites](http://channel9.msdn.com/Series/Windows-Azure-Web-Sites-Tutorials/Dropbox-Deployment-to-Windows-Azure-Web-Sites). This video walks through the process of connecting a DropBox folder to an Azure  Website, and shows how quickly you can get a  Website up and running or maintain it using simple drag-and-drop deployment.
* [Azure Forum for Git, Mercurial, and DropBox](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchszh-cn/home?forum=azuregit).
-->

##<a name="vs"></a>Visual Studio

有关如何从 Visual Studio 部署到 Azure 网站的信息，请参阅以下资源：

* [Azure 和 ASP.NET 入门](/documentation/articles/web-sites-dotnet-get-started)。如何使用 Visual Studio 和 Web 部署来创建和部署一个简单的 ASP.NET MVC Web 项目。
* [如何将 Azure Web 作业部署到 Azure 网站](/documentation/articles/websites-dotnet-deploy-webjobs)。如何配置控制台应用程序项目，以便它们部署为 Web 作业。  
* [将包含成员资格、OAuth 和 SQL 数据库的安全 ASP.NET MVC 5 应用部署到 Azure 网站](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)。如何使用 Visual Studio、Web 部署和 Entity Framework Code First 迁移通过 SQL 数据库来创建和部署 ASP.NET MVC Web 项目。
* [Visual Studio 和 ASP.NET 的 Web 部署概述](http://msdn.microsoft.com/zh-cn/library/dd394698.aspx)。使用 Visual Studio 的 Web 部署的基本简介。虽然已过时，但包括仍然相关的信息，其中包括用于部署数据库和 Web 应用程序的选项的概述，以及其他部署任务的列表（这些任务可能需要你来执行，或需要你手动配置 Visual Studio 以代替你执行）。本主题提供有关常规部署的信息，不只是关于部署到 Azure 网站。
* [使用 Visual Studio 的 ASP.NET Web 部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/introduction)。共 12 篇的系列教程涵盖了比此列表中其他部署任务更完整的部署任务。自编写本教程以来添加了一些 Azure 部署功能，但注释是后来添加的，说明缺少哪些内容。 
* [在 Visual Studio 2012 中直接从 Git 存储库将 ASP.NET 网站部署到 Azure](http://www.dotnetcurry.com/ShowArticle.aspx?ID=881)。说明如何在 Visual Studio 中部署 ASP.NET Web 项目（使用 Git 插件将代码提交到 Git 并将 Azure 连接到 Git 存储库）。从 Visual Studio 2013 开始，Git 支持是内置的，不需要安装插件。


有关详细信息，请参阅以下资源：

* [创建 PHP-MySQL Azure 网站并使用 FTP 进行部署](/documentation/articles/web-sites-php-mysql-deploy-use-ftp/)。 

##<a name="tfs"></a>Team Foundation Server (TFS)

Team Foundation Server 是 Microsoft 针对源代码管理和团队协作的本地解决方案。你可以设置 TFS 以进行到 Web 应用的连续交付。

有关详细信息，请参阅以下资源：

* [在 Azure 中持续交付云服务](/documentation/articles/cloud-services-dotnet-continuous-delivery)。本文档适用于 Azure 云服务，但其部分内容与网站有关。

###<a name="gitmercurial"></a>本地 Git 或 Mercurial 存储库

在 Azure 中，你可以输入使用 Git 或 Mercurial 的任何存储库的 URL，以从该位置进行部署。你可以直接从本地 Git 存储库推送到 Azure 网站。

有关详细信息，请参阅以下资源：

* [从源代码管理发布到具有 Git 的 Azure 网站](/documentation/articles/web-sites-publish-source-control)。如何使用 Git 直接从本地计算机发布到 Azure 网站（在 Azure 中，此发布方法称为“本地 Git”）。还将演示如何启用从 GitHub、CodePlex 或 BitBucket 进行 Git 存储库的连续部署。
* [从任何 git/hg 存储库发布到 Azure 网站](http://blog.davidebbo.com/2013/04/publishing-to-azure-web-sites-from-any.html)。该博客介绍了 Azure 网站中的“外部存储库”功能。
* [Git、Mercurial 和 DropBox 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchszh-cn/home?forum=azuregit)。
* [将两个网站从一个 Git 存储库部署到 Azure](http://www.hanselman.com/blog/DeployingTWOWebsitesToWindowsAzureFromOneGitRepository.aspx)。Scott Hanselman 的博客文章。


##<a name="msbuild"></a>MSBuild

如果要将 [Visual Studio IDE](#vs) 用于部署，则可以使用 [MSBuild](http://msbuildbook.com/) 来自动执行可在 IDE 中完成的任何任务。可以对 [Web Deploy](#webdeploy) 进行配置，以使用 Web 部署或 [FTP/FTPS](#ftp) 来复制文件。Web 部署还可以自动执行许多其他与部署相关的任务，例如部署数据库。

有关使用 MSBuild 的命令行部署的详细信息，请参阅以下资源：

* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。有关使用 Visual Studio 部署到 Azure 的一系列教程中的第十篇。演示在 Visual Studio 中设置发布配置文件之后如何使用命令行来进行部署。
* [深入了解 Microsoft 生成引擎：使用 MSBuild 和 Team Foundation Build](http://msbuildbook.com/)。纸印书籍，其中包含了有关如何使用 MSBuild 进行部署的章节。

##<a name="ftp2"></a>FTP 脚本

可以轻松地创建针对 Web 应用的 [FTP/FTPS](http://zh.wikipedia.org/wiki/File_Transfer_Protocol) 凭据，然后可以将这些凭据用于 FTP 批处理脚本。

有关详细信息，请参阅以下资源：

* [使用 FTP 批处理脚本](http://support.microsoft.com/kb/96269)。

##<a name="powershell"></a>Windows PowerShell

可以从 [Windows PowerShell](http://msdn.microsoft.com/zh-cn/library/dd835506.aspx) 执行 MSBuild 或 FTP 部署功能。如果你这样做，则还可以使用 Windows PowerShell cmdlet 集合（可使 Azure REST 管理 API 易于调用）。

有关详细信息，请参阅以下资源：

* [使用 Azure 构建真实世界云应用 – 使一切自动化](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything)。电子书章节，其中介绍电子书中所示的示例应用程序如何使用 Windows PowerShell 脚本来创建 Azure 测试环境并部署到该环境中。请参阅[资源](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything#resources)部分以获取指向其他 Azure PowerShell 文档的链接。
* [使用 Windows PowerShell 脚本发布到开发和测试环境](http://msdn.microsoft.com/zh-cn/library/dn642480.aspx)。如何使用 Visual Studio 生成的 Windows PowerShell 部署脚本。

###<a name="api"></a>.NET 管理 API

可以编写 C# 代码以执行 MSBuild 或 FTP 功能进行部署。如果你这样做，则可以访问 Azure 管理 REST API 来执行站点管理功能。

有关详细信息，请参阅以下资源：

* [使用 Azure 管理库和 .NET 使一切自动化](http://www.hanselman.com/blog/PennyPinchingInTheCloudAutomatingEverythingWithTheWindowsAzureManagementLibrariesAndNET.aspx)。.NET 管理 API 的简介和指向更多文档的链接。

##<a name="cli"></a>Azure 命令行界面 (Azure CLI)

通过使用 FTP，可以在 Mac 或 Linux 计算机中通过 Windows 中的命令行来进行部署。如果你这样做，则还可以访问使用 Azure CLI 的 Azure REST 管理 API。

有关详细信息，请参阅以下资源：

* [命令行工具](/zh-cn/downloads/#cmd-line-tools)。windowsazure.cn 中有关命令行工具信息的门户页面。

##<a name="webdeploy"></a>Web 部署命令行

[Web 部署](http://www.iis.net/downloads/microsoft/web-deploy)是用于部署到 IIS 的 Microsoft 软件，它不仅提供智能文件同步功能，还可以执行或协调许多其他与部署相关的、使用 FTP 时无法自动执行的任务。例如，Web 部署可以部署新的数据库或数据库更新以及你的 Web 应用。Web 部署还可以尽量减少更新现有站点所需的时间，因为它可以智能地仅复制更改过的文件。Microsoft WebMatrix、Visual Studio、Visual Studio Online 和 Team Foundation Server 都支持内置 Web 部署，但你也可以直接从命令行使用 Web 部署以使部署自动化。Web 部署命令非常强大，但在学习过程中可能会遇到困难。

有关详细信息，请参阅以下资源：

* [简单的 Azure 网站：部署](http://azure.microsoft.com/blog/2014/07/28/simple-azure-websites-deployment/)。David Ebbo 的博客，介绍如何更轻松地使用 Web 部署。
* [Web 部署工具](http://technet.microsoft.com/zh-cn/library/dd568996)。Microsoft TechNet 网站上的正式文档。虽然已过时，但仍是一个不错的起点。
* [使用 Web 部署](http://www.iis.net/learn/publish/using-web-deploy)。Microsoft IIS.NET 网站上的正式文档。虽然也已过时，但仍是一个不错的起点。
* [堆栈溢出](http://www.stackoverflow.com)获得有关如何从命令行中使用 Web 部署的最新信息的最佳位置。
* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。MSBuild 是由 Visual Studio 使用的生成引擎，还可以从命令行使用它以将 Web 应用程序部署到 Azure 网站。本教程是主要介绍 Visual Studio 部署的一系列教程的一部分。

##<a name="nextsteps"></a>后续步骤

在某些情况下，你可能想要能够轻松地在 Web 应用的过渡版本和生产版本之间来回切换。有关详细信息，请参阅 [Web Apps 网站上的过渡部署](/documentation/articles/web-sites-staged-publishing)。

准备好备份和还原计划是任何部署工作流的一个重要部分。有关 Web Apps 的备份和还原功能的信息，请参阅 [Web Apps 备份](/documentation/articles/web-sites-backup)。

有关如何使用 Azure 的基于角色的访问控制来管理 Web Apps 部署访问权限的信息，请参阅 [RBAC 和 Web App 发布](http://azure.microsoft.com/blog/2015/01/05/rbac-and-azure-websites-publishing)。

有关其他部署主题的信息，请参阅 [Azure 网站文档](/zh-cn/documentation/services/web-sites/)中的“部署”部分。

<!---HONumber=71-->