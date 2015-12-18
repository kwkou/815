<properties
	pageTitle="Azure 网站部署文档"
	description="查找那些说明如何将应用部署到 Azure 网站的文档。"
	services="app-service"
	documentationCenter=""
	authors="tdykstra"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="web-sites"
	ms.date="11/06/2015"
	wacn.date="12/17/2015"/>

# Azure 网站部署文档

## 概述

本文列出的方法适用于将你自己的内容部署到 [Azure 网站](/documentation/services/web-sites/)，其中包括内含操作方法信息的文章和博客的链接。随着更多的文章的发布，这些文章也会添加到该列表中。

部署网站的最佳方法是设置与[源代码管理系统](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control)集成的[持续交付工作流](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/continuous-integration-and-continuous-delivery)。自动化不仅使开发过程更高效，还可以使你的备份和还原流程更可控、更可靠。

##### 从云托管的源代码管理系统进行部署

* [使用 Git 的存储库网站](#git)
* [使用 Mercurial 的存储库网站](#mercurial)

##### 从本地源代码管理系统进行部署

* [使用 Team Foundation Service (TFS) 进行持续交付](#tfs)
* [本地 Git 或 Mercurial 存储库](#onpremises)

##### 使用命令行工具自动进行部署

* [使用 MSBuild 自动进行部署](#msbuild)
* [使用 FTP 工具和脚本复制文件](#ftp)
* [使用 Windows PowerShell 自动进行部署](#powershell)
* [使用 .NET 管理 API 自动进行部署](#api)
* [从 Azure 命令行界面 (Azure CLI) 进行部署](#cli)
* [从 Web 部署命令行进行部署](#webdeploy)
 
##### 从集成开发环境 (IDE) 进行部署

* [从 Visual Studio 直接进行部署](#vs)
* [从 WebMatrix 直接进行部署](#webmatrix)

另一个部署选项是使用基于云的服务，例如 [Octopus 部署](http://en.wikipedia.org/wiki/Octopus_Deploy)。有关更多信息，请参阅[将 ASP.NET网站部署到 Azure 网站](https://octopusdeploy.com/blog/deploy-aspnet-applications-to-azure-websites)。


##<a name="git"></a>使用 Git 的存储库网站

[Git](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control#gittfs) 是一种流行的分布式源代码管理系统。Azure 具有的内置功能可轻松实现从流行的基于 Web 的存储库网站（存储 Git 存储库，包括 [GitHub](http://www.github.com)、[CodePlex](http://www.codeplex.com/) 和 [BitBucket](https://bitbucket.org/)）自动部署到网站。使用 Git 进行部署的一个优点是比较容易回滚到以前的部署（一旦有必要）。

有关详细信息，请参阅以下资源：

* [从源代码管理发布到使用 Git 的网站](/documentation/articles/web-sites-publish-source-control)。如何使用 Git 直接从本地计算机发布到网站（在 Azure 中，此发布方法称为“本地 Git”）。还将演示如何启用从 GitHub、CodePlex 或 BitBucket 进行 Git 存储库的连续部署。
* [网站的“部署到 Azure”按钮](http://azure.microsoft.com/blog/2014/11/13/deploy-to-azure-button-for-azure-websites-2/)。有关用于触发从 Git 存储库的部署方法的博客。
* [Git 和 Mercurial 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchshome?forum=azuregit)。

##<a name="mercurial"></a>使用 Mercurial 的存储库网站

如果将 [Mercurial](http://mercurial.selenic.com/) 用作源代码管理系统，并将存储库存储在 [CodePlex](http://www.codeplex.com/) 或 [BitBucket](https://bitbucket.org/) 中，则可以使用 Azure 网站中的内置功能来自动部署内容。

有关如何使用 Mercurial 进行部署的信息，请参阅以下资源：

* [从源代码管理发布到使用 Git 的网站](/documentation/articles/web-sites-publish-source-control)。尽管本教程演示的是如何发布 Git 存储库，但 CodePlex 或 BitBucket 中托管的 Mercurial 存储库的发布过程与此类似。
* [Git 和 Mercurial 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchshome?forum=azuregit)。

##<a name="vs"></a>从 Visual Studio 直接进行部署

有关如何从 Visual Studio 部署到网站的信息，请参阅以下资源：

* [Azure 和 ASP.NET 入门](/documentation/articles/web-sites-dotnet-get-started)。如何使用 Visual Studio 和 Web 部署来创建和部署一个简单的 ASP.NET MVC Web 项目。
* [如何使用 Visual Studio 部署 Azure Web 作业](/documentation/articles/websites-dotnet-deploy-webjobs)。如何配置控制台应用程序项目，以便它们部署为 Web 作业。  
* [将包含成员资格、OAuth 和 SQL 数据库的安全 ASP.NET MVC 5 应用程序部署到网站](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)。如何使用 Visual Studio、Web 部署和 Entity Framework Code First 迁移通过 SQL 数据库来创建和部署 ASP.NET MVC Web 项目。
* [Visual Studio 和 ASP.NET 的 Web 部署概述](http://msdn.microsoft.com/zh-cn/library/dd394698.aspx)。使用 Visual Studio 的 Web 部署的基本简介。虽然已过时，但包括仍然相关的信息，其中包括用于部署数据库和网站的选项的概述，以及其他部署任务的列表（这些任务可能需要你来执行，或需要你手动配置 Visual Studio 以代替你执行）。本主题提供有关常规部署的信息，不只是有关如何部署到网站的信息。
* [使用 Visual Studio 的 ASP.NET Web 部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/introduction)。共 12 篇的系列教程涵盖了比此列表中其他部署任务更完整的部署任务。自编写本教程以来添加了一些 Azure 部署功能，但注释是后来添加的，说明缺少哪些内容。
* [在 Visual Studio 2012 中直接从 Git 存储库将 ASP.NET 网站部署到 Azure](http://www.dotnetcurry.com/ShowArticle.aspx?ID=881)。说明如何在 Visual Studio 中部署 ASP.NET Web 项目（使用 Git 插件将代码提交到 Git 并将 Azure 连接到 Git 存储库）。从 Visual Studio 2013 开始，Git 支持是内置的，不需要安装插件。

有关详细信息，请参阅以下资源：

* [创建 PHP-MySQL网站并使用 FTP 进行部署](/documentation/articles/web-sites-php-mysql-deploy-use-ftp)。

##<a name="tfs"></a>使用 Team Foundation Service (TFS) 进行持续交付

Team Foundation Server 是 Microsoft 针对源代码管理和团队协作的本地解决方案。你可以设置 TFS 以进行到网站的连续交付。

有关详细信息，请参阅以下资源：

* [在 Azure 中持续交付云服务](/documentation/articles/cloud-services-dotnet-continuous-delivery)。本文档适用于 Azure 云服务，但其部分内容与网站有关。

##<a name="gitmercurial"></a>本地 Git 或 Mercurial 存储库

在 Azure 中，你可以输入使用 Git 或 Mercurial 的任何存储库的 URL，以从该位置进行部署。你还可以直接从本地 Git 存储库推送到网站。

有关详细信息，请参阅以下资源：

* [从源代码管理发布到使用 Git 的网站](/documentation/articles/web-sites-publish-source-control)。如何使用 Git 直接从本地计算机发布到网站（在 Azure 中，此发布方法称为“本地 Git”）。还将演示如何启用从 GitHub、CodePlex 或 BitBucket 进行 Git 存储库的连续部署。
* [从任何 git/hg 存储库发布到网站](http://blog.davidebbo.com/2013/04/publishing-to-azure-web-sites-from-any.html)。该博客介绍了网站中的“外部存储库”功能。
* [Git 和 Mercurial 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchshome?forum=azuregit)。
* [将两个网站从一个 Git 存储库部署到 Azure](http://www.hanselman.com/blog/DeployingTWOWebsitesToWindowsAzureFromOneGitRepository.aspx)。Scott Hanselman 的博客文章。

##<a name="msbuild"></a>使用 MSBuild 自动进行部署

如果要将 [Visual Studio IDE](#vs) 用于部署，则可以使用 [MSBuild](http://msbuildbook.com/) 来自动执行可在 IDE 中完成的任何任务。可以对 MSBuild 进行配置，以使用 [Web 部署](#webdeploy)或 [FTP/FTPS](#ftp) 来复制文件。Web 部署还可以自动执行许多其他与部署相关的任务，例如部署数据库。

有关使用 MSBuild 的命令行部署的详细信息，请参阅以下资源：

* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。有关使用 Visual Studio 部署到 Azure 的一系列教程中的第十篇。演示在 Visual Studio 中设置发布配置文件之后如何使用命令行来进行部署。
* [深入了解 Microsoft 生成引擎：使用 MSBuild 和 Team Foundation Build](http://msbuildbook.com/)。纸印书籍，其中包含了有关如何使用 MSBuild 进行部署的章节。

##<a name="ftp"></a>使用 FTP 工具和脚本复制文件

你可以使用 [FTP](http://zh.wikipedia.org/wiki/File_Transfer_Protocol) 来复制文件，从而将内容部署到应用。可以轻松地创建针对网站的 FTP 凭据，并可以将其用于任何可使用 FTP 的脚本或应用程序，包括浏览器（例如 Internet Explorer）和功能全面的免费实用程序（例如 [FileZilla](https://filezilla-project.org/)）。网站还支持更安全的 FTPS 协议。

尽管使用 FTP 实用程序将网站的文件复制到 Azure 很容易，但它们并不会自动负责或协调相关的部署任务，例如部署数据库或更改连接字符串。此外，许多 FTP 工具不会对源文件和目标文件进行比较以跳过对未更改的文件的复制。对于大型应用，始终复制所有文件可能会导致部署时间较长，即使对于少量更新也是如此，因为将始终复制所有文件。

有关详细信息，请参阅以下资源：

* [使用 FTP 批处理脚本](http://support.microsoft.com/kb/96269)。

##<a name="powershell"></a>使用 Windows PowerShell 自动进行部署

你可以通过 [Windows PowerShell](http://msdn.microsoft.com/zh-cn/library/dd835506.aspx) 执行 MSBuild 或 FTP 部署功能。如果你这样做，则还可以使用 Windows PowerShell cmdlet 集合（可使 Azure REST 管理 API 易于调用）。

有关详细信息，请参阅以下资源：* [使用 Azure 构建真实世界云应用 – 使一切自动化](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything)。电子书章节，其中介绍电子书中所示的示例应用程序如何使用 Windows PowerShell 脚本来创建 Azure 测试环境并部署到该环境中。请参阅[资源](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything#resources)部分以获取指向其他 Azure PowerShell 文档的链接。* [使用 Windows PowerShell 脚本发布到开发和测试环境](http://msdn.microsoft.com/zh-cn/library/dn642480.aspx)。如何使用 Visual Studio 生成的 Windows PowerShell 部署脚本。

##<a name="api"></a>使用 .NET 管理 API 自动进行部署

可以编写 C# 代码以执行 MSBuild 或 FTP 功能进行部署。如果你这样做，则可以访问 Azure 管理 REST API 来执行站点管理功能。

有关详细信息，请参阅以下资源：

* [使用 Azure 管理库和 .NET 使一切自动化](http://www.hanselman.com/blog/PennyPinchingInTheCloudAutomatingEverythingWithTheWindowsAzureManagementLibrariesAndNET.aspx)。.NET 管理 API 的简介和指向更多文档的链接。

##<a name="cli"></a>从 Azure 命令行界面 (Azure CLI) 进行部署

通过使用 FTP，可以在 Mac 或 Linux 计算机中通过 Windows 中的命令行来进行部署。如果你这样做，则还可以访问使用 Azure CLI 的 Azure REST 管理 API。

有关详细信息，请参阅以下资源：

* [Azure 命令行工具](/downloads/#cmd-line-tools)。Azure.com 中有关命令行工具信息的门户页面。

##<a name="webdeploy"></a>从 Web 部署命令行进行部署

[Web 部署](http://www.iis.net/downloads/microsoft/web-deploy)是用于部署到 IIS 的 Microsoft 软件，它不仅提供智能文件同步功能，还可以执行或协调许多其他与部署相关的、使用 FTP 时无法自动执行的任务。例如，Web 部署可以部署新的数据库或数据库更新以及你的网站。Web 部署还可以尽量减少更新现有站点所需的时间，因为它可以智能地仅复制更改过的文件。Microsoft WebMatrix、Visual Studio 和 Team Foundation Server 都支持内置 Web 部署，但你也可以直接从命令行使用 Web 部署以使部署自动化。Web 部署命令非常强大，但在学习过程中可能会遇到困难。

有关详细信息，请参阅以下资源：

* [简单的网站：部署](http://azure.microsoft.com/blog/2014/07/28/simple-azure-websites-deployment/)。David Ebbo 的博客，介绍如何更轻松地使用 Web 部署。
* [Web 部署工具](http://technet.microsoft.com/zh-cn/library/dd568996)。Microsoft TechNet 网站上的正式文档。虽然已过时，但仍是一个不错的起点。
* [使用 Web 部署](http://www.iis.net/learn/publish/using-web-deploy)。Microsoft IIS.NET 网站上的正式文档。虽然也已过时，但仍是一个不错的起点。
* [StackOverflow](http://www.stackoverflow.com)。获得有关如何从命令行中使用 Web 部署的最新信息的最佳位置。
* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。MSBuild 是由 Visual Studio 使用的生成引擎，还可以从命令行使用它以将网站部署到网站。本教程是主要介绍 Visual Studio 部署的一系列教程的一部分。

##<a name="nextsteps"></a>后续步骤

在某些情况下，你可能想要能够轻松地在网站的过渡版本和生产版本之间来回切换。有关详细信息，请参阅 [网站上的过渡部署](/documentation/articles/web-sites-staged-publishing)。

准备好备份和还原计划是任何部署工作流的一个重要部分。有关网站的备份和还原功能的信息，请参阅 [网站备份](/documentation/articles/web-sites-backup)。

有关如何使用 Azure 的基于角色的访问控制来管理网站部署访问权限的信息，请参阅 [RBAC 和网站发布](http://azure.microsoft.com/blog/2015/01/05/rbac-and-azure-websites-publishing)。

有关其他部署主题的信息，请参阅 [网站文档](/documentation/services/web-sites/)中的“部署”部分。
 

<!---HONumber=Mooncake_1207_2015-->