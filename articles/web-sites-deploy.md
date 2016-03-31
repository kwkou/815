<properties
	pageTitle="将你的应用部署到 Azure Web 应用"
	description="了解如何将你的应用部署到 Azure Web 应用。"
	services="app-service"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service"
	ms.date="01/07/2016"
	wacn.date="03/24/2016"/>
    
# 将应用部署到 Azure

本文可帮助你确定将 Web 应用、移动应用后端或 API 应用的文件部署到 [Azure Web 应用](/documentation/services/web-sites/)的最佳选项，然后将你引导到相应的文章和博客，其中包含特定于你的首选选项的操作说明。

## <a name="overview"></a>部署进程概述

你在 Azure Web 应用中创建或预配应用后，在将任何代码部署到该应用之前，Azure 会为你维护应用程序框架（ASP.NET、PHP、Node.js 等）。某些框架在默认情况下已启用，而其他框架（如 Java 和 Python）可能需要进行简单的复选标记配置才能启用它。此外，你还可以自定义应用程序框架，如运行时的 PHP 版本或位元。有关详细信息，请参阅[在 Azure Web 应用中配置你的应用](/documentation/articles/web-sites-configure)。

由于你无需担心 Web 服务器或应用程序框架，因此将你的应用部署到 Azure 只需将你的代码、二进制文件、内容文件及其各自的目录结构部署到 [**Azure 中的 /site/wwwroot** 目录](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure)（或将 Web 作业部署到 **/Data/Jobs** 目录）。Azure 支持以下三种部署进程。

- [FTP 或 FTPS](https://en.wikipedia.org/wiki/File_Transfer_Protocol)：使用你常用的支持 FTP 或 FTPS 的工具将文件移至 Azure，该工具可以是 [FileZilla](https://filezilla-project.org) 到功能齐全的 IDE（如 [NetBeans](https://netbeans.org)）。这完全是文件上载进程。Azure Web 应用不提供任何附加服务，例如版本控制、文件结构管理等。 
- [Web 部署](http://www.iis.net/learn/publish/using-web-deploy/introduction-to-web-deploy)：自动部署到 IIS 服务器的相同工具。将代码直接从你常用的 Microsoft 工具（如 Visual Studio、WebMatrix 和 Visual Studio Team Services）部署到 Azure。此工具支持仅差异部署、创建数据库、连接字符串转换等操作。与 FTP 类似，Azure Web 应用不提供任何附加服务。

常用的 Web 开发工具支持其中的一个或多个部署进程。虽然你选择的工具确定了你可以利用的部署进程，但是由你支配的实际 DevOps 功能取决于部署进程和你选择的特定工具的组合。例如，如果你从[包含 Azure SDK 的 Visual Studio](#vspros) 执行 Web 部署，你会在 Visual Studio 中自动执行程序包还原和 MSBuild。Azure SDK 还在 Visual Studio 界面中提供了易用的向导，以帮助你直接创建所需的 Azure 资源。

>[AZURE.NOTE] 这些部署进程并未实际预配你的应用可能需要的 Azure 资源，如 App Service 计划、Azure Web 应用和 SQL 数据库。但是，大多数链接的操作方法文章会向你展示如何预配应用并端到端地将代码部署到该应用。你还可以在[使用命令行工具自动部署](#automate)部分中找到用于预配 Azure 资源的其他选项。

## <a name="ftp"></a>通过将文件手动复制到 Azure 来进行部署
如果你习惯于手动将 Web 内容复制到 Web 宿主（对于 PHP 开发人员来说常见的工作流程），可以使用 [FTP](http://en.wikipedia.org/wiki/File_Transfer_Protocol) 实用工具来复制文件，如 Windows 资源管理器或 [FileZilla](https://filezilla-project.org/)。

手动复制文件将使用 FTP 协议进行部署（请参阅[部署进程概述](#overview)）。

手动复制文件的优点是：

- 很快熟悉 FTP 工具。 
- 确切地知道文件将复制到的位置。
- 使用 FTPS 可增加安全性。
- 如果你喜欢使用最小的工具进行 Web 开发（例如，使用记事本开发 Web 应用程序），它是一个很好的部署解决方案。

手动复制文件的缺点是：

- 你负责将文件部署到 Azure Web 应用中的正确目录。
- 发生故障时没有针对回退的版本控制。
- 许多 FTP 工具未提供仅差异复制，而只是复制所有文件。对于大型应用来说，这会导致部署时间较长，即使少量更新也是如此。

### <a name="howtoftp"></a>如何通过将文件手动复制到 Azure 来进行部署
将文件复制到 Azure 涉及几个简单步骤：

1. 在 [Azure 管理门户](https://manage.windowsazure.cn)中为你的应用创建部署凭据。为此，请在应用的边栏选项卡中，单击“设置”>“部署凭据”。
2. 配置部署凭据后，请转到“设置”>“属性”获取 FTP 连接信息，然后复制 **FTP/开发用户**、**FTP 主机名**和 **FTPS 主机名**的值。
3. 从 FTP 客户端，使用收集到的连接信息连接到你的应用。
4. 将你的文件及其各自的目录结构复制到 [**Azure 中的 /site/wwwroot** 目录](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure)（或者将 Web 作业复制到 **/Data/Jobs** 目录）。
5. 浏览到你的应用的 URL 以验证该应用是否正在正常运行。 

有关详细信息，请参阅以下资源：

* [创建 PHP-MySQL Web 应用并使用 FTP 进行部署](/documentation/articles/web-sites-php-mysql-deploy-use-ftp)。
* [使用 FTP 批处理脚本](http://support.microsoft.com/kb/96269)。

## 使用 IDE 进行部署
如果你已在使用包含 [Azure SDK](/downloads/) 的 [Visual Studio](https://www.visualstudio.com/products/visual-studio-community-vs.aspx) 或 WebMatrix 或其他 IDE 套件（如 [Xcode](https://developer.apple.com/xcode/) 和 [Eclipse](https://www.eclipse.org)），你可以直接从 IDE 内部署到 Azure。此选项非常适合于单个开发人员。

Visual Studio 支持所有这三种部署进程（FTP、Git 和 Web 部署），具体取决于你的首选项，而其他 IDE 在已集成 FTP 或 Git 时可部署到 Azure（请参阅[部署进程概述](#overview)）。

使用 IDE 进行部署的优点是：

- 可能最大程度地减少端到端应用程序生命周期中的工具使用。开发、调试、跟踪你的应用以及将其部署到 Azure，所有这些操作都无需移出你的 IDE。 

使用 IDE 进行部署的缺点是：

- 增加了工具使用方面的复杂性。
- 团队项目仍需要源代码管理系统。

<a name="vspros"></a> 使用包含 Azure SDK 的 Visual Studio 进行部署的其他优点是：

- Azure SDK 使 Azure 资源成为 Visual Studio 中的一等公民。可创建、删除、编辑、启动和停止应用、查询后端 SQL 数据库、 实时调试 Azure 应用，以及执行更多操作。 
- 实时编辑 Azure 上的代码文件。
- 实时调试 Azure 上的应用。
- 已集成 Azure 资源管理器。
- 仅差异部署。 

###<a name="vs"></a>如何从 Visual Studio 直接进行部署

* [Azure 和 ASP.NET 入门](/documentation/articles/web-sites-dotnet-get-started)。如何使用 Visual Studio 和 Web 部署来创建和部署一个简单的 ASP.NET MVC Web 项目。
* [如何使用 Visual Studio 部署 Azure Web 作业](/documentation/articles/websites-dotnet-deploy-webjobs)。如何配置控制台应用程序项目，以便它们部署为 Web 作业。  
* [将包含成员资格、OAuth 和 SQL 数据库的安全 ASP.NET MVC 5 应用程序部署到 Web Apps](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)。如何使用 Visual Studio、Web 部署和 Entity Framework Code First 迁移通过 SQL 数据库来创建和部署 ASP.NET MVC Web 项目。
* [Visual Studio 和 ASP.NET 的 Web 部署概述](http://msdn.microsoft.com/zh-cn/library/dd394698.aspx)。使用 Visual Studio 的 Web 部署的基本简介。虽然已过时，但包括仍然相关的信息，其中包括用于部署数据库和 Web 应用程序的选项的概述，以及其他部署任务的列表（这些任务可能需要你来执行，或需要你手动配置 Visual Studio 以代替你执行）。本主题提供有关常规部署的信息，不只是有关如何部署到 Web Apps 的信息。
* [使用 Visual Studio 的 ASP.NET Web 部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/introduction)。共 12 篇的系列教程涵盖了比此列表中其他部署任务更完整的部署任务。自编写本教程以来添加了一些 Azure 部署功能，但注释是后来添加的，说明缺少哪些内容。
* [在 Visual Studio 2012 中直接从 Git 存储库将 ASP.NET 网站部署到 Azure](http://www.dotnetcurry.com/ShowArticle.aspx?ID=881)。说明如何在 Visual Studio 中部署 ASP.NET Web 项目（使用 Git 插件将代码提交到 Git 并将 Azure 连接到 Git 存储库）。从 Visual Studio 2013 开始，Git 支持是内置的，不需要安装插件。

###<a name="webmatrix"></a>如何从 WebMatrix 直接进行部署

* [使用 WebMatrix 构建 Node.js 网站并将其部署到 Azure](/documentation/articles/web-sites-nodejs-use-webmatrix)。
* [WebMatrix 3：集成 Git 与到 Azure 的部署](http://www.codeproject.com/Articles/577581/Webmatrixplus3-3aplusIntegratedplusGitplusandplusD)。如何使用 WebMatrix 从 Git 源代码管理存储库进行部署。

## <a name="onprem"></a>从本地源代码管理系统进行部署

如果你在任意规模的开发团队中工作并使用本地源代码管理 (SCM) 系统（如 [Team Foundation Server](https://www.visualstudio.com/products/tfs-overview-vs.aspx) (TFS)、[Git](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control#gittfs) 或 [Mercurial](http://mercurial.selenic.com/)），你可以将 Azure 配置为与你的存储库集成，并在源代码管理工作流中直接部署到 Azure。

从本地源代码管理系统部署的优点是：

- 支持从任何语言框架或者 Git 或 Mercurial 客户端（包括 [Xcode](https://developer.apple.com/xcode/) 和 [Eclipse](https://www.eclipse.org)）进行部署。
- 分支特定的部署，可以将不同版本部署到单独的[槽](/documentation/articles/web-sites-staged-publishing)。
- 适用于任何规模的开发团队。

从本地源代码管理系统部署的缺点是：

- 需要了解所选的 SCM 系统。
- 可能会提供的功能和特性超过所需。
- 缺少通过 Git 客户端工具进行分支特定的部署的统包解决方案。 

使用 TFS 进行部署的其他优点是：

- 持续集成 (CI) 生成、测试和部署。
- 适用于现有 IDE 或编辑器的内置协作工具。
- 支持 Git 进行分布式版本控制或支持 Team Foundation 版本控制 (TFVC) 进行集中的版本控制。 
- 敏捷部署的丰富工具。
- 实现 [Jenkins](https://jenkins-ci.org)、[Slack](https://slack.com)、[ZenDesk](https://www.zendesk.com)、[Trello](https://trello.com)、[Azure 服务总线](/home/features/messaging/)等的现成集成。 
- [Team Foundation Server Express](https://www.microsoft.com/download/details.aspx?id=48259) 对于最多包含 5 个开发人员的团队是免费的。

###<a name="gitmercurial"></a>如何从本地 Git 或 Mercurial 存储库部署

* [从源代码管理发布到使用 Git 的 Web Apps](/documentation/articles/web-sites-publish-source-control)。如何使用 Git 直接从本地计算机发布到 Web 应用（在 Azure 中，此发布方法称为“本地 Git”）。
* [从任何 git/hg 存储库发布到 Web Apps](http://blog.davidebbo.com/2013/04/publishing-to-azure-web-sites-from-any.html)。该博客介绍了 Web Apps 中的“外部存储库”功能。
* [Git 和 Mercurial 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchshome?forum=azuregit)。
* [将两个网站从一个 Git 存储库部署到 Azure](http://www.hanselman.com/blog/DeployingTWOWebsitesToWindowsAzureFromOneGitRepository.aspx)。Scott Hanselman 的博客文章。

## 从基于云的源代码管理服务部署
如果你在任意规模的开发团队中工作，并使用基于云的源代码管理 (SCM) 服务，如 [GitHub](https://www.github.com)，你可以将 Azure 配置为与你的存储库集成。

从基于云的源代码管理服务部署的优点是：

- 支持任何语言框架。
- 分支特定的部署，可以将不同分支部署到不同的[槽](/documentation/articles/web-sites-staged-publishing)。
- 适用于任何规模的开发团队。

从基于云的源代码管理服务部署的缺点是：

- 需要了解所选的 SCM 服务。
- 可能会提供的功能和特性超过所需。

###<a name="cloudgitmercurial"></a>如何从基于云的 Git 或 Mercurial 存储库部署

- [从源代码管理发布到使用 Git 的 Web Apps](/documentation/articles/web-sites-publish-source-control)。尽管本教程演示的是如何发布 Git 存储库，但 CodePlex 或 BitBucket 中托管的 Mercurial 存储库的发布过程与此类似。
- [Web Apps 的“部署到 Azure”按钮](http://azure.microsoft.com/blog/2014/11/13/deploy-to-azure-button-for-azure-websites-2/)。有关用于触发从 Git 存储库的部署方法的博客。
- [Git 和 Mercurial 的 Azure 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/home?forum=windowsazurezhchshome?forum=azuregit)。

有关详细信息，请参阅以下资源：

- [从源代码管理发布到使用 Git 的 Web Apps](/documentation/articles/web-sites-publish-source-control)。如何使用 Git 直接从本地计算机发布到 Web Apps（在 Azure 中，此发布方法称为“本地 Git”）。 

## <a name="automate"></a>使用命令行工具自动进行部署

* [使用 MSBuild 自动进行部署](#msbuild)
* [使用 FTP 工具和脚本复制文件](#ftp)
* [使用 Windows PowerShell 自动进行部署](#powershell)
* [使用 .NET 管理 API 自动进行部署](#api)
* [从 Azure 命令行界面 (Azure CLI) 进行部署](#cli)
* [从 Web 部署命令行进行部署](#webdeploy)
* [使用 FTP 批处理脚本](http://support.microsoft.com/kb/96269)。
 
另一个部署选项是使用基于云的服务，例如 [Octopus 部署](http://en.wikipedia.org/wiki/Octopus_Deploy)。有关更多信息，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站](https://octopusdeploy.com/blog/deploy-aspnet-applications-to-azure-websites)。

###<a name="msbuild"></a>使用 MSBuild 自动进行部署

如果要将 [Visual Studio IDE](#vs) 用于部署，则可以使用 [MSBuild](http://msbuildbook.com/) 来自动执行可在 IDE 中完成的任何任务。可以对 MSBuild 进行配置，以使用 [Web 部署](#webdeploy)或 [FTP/FTPS](#ftp) 来复制文件。Web 部署还可以自动执行许多其他与部署相关的任务，例如部署数据库。

有关使用 MSBuild 的命令行部署的详细信息，请参阅以下资源：

* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。有关使用 Visual Studio 部署到 Azure 的一系列教程中的第十篇。演示在 Visual Studio 中设置发布配置文件之后如何使用命令行来进行部署。
* [深入了解 Microsoft 生成引擎：使用 MSBuild 和 Team Foundation Build](http://msbuildbook.com/)。纸印书籍，其中包含了有关如何使用 MSBuild 进行部署的章节。

###<a name="powershell"></a>使用 Windows PowerShell 自动进行部署

你可以通过 [Windows PowerShell](http://msdn.microsoft.com/zh-cn/library/dd835506.aspx) 执行 MSBuild 或 FTP 部署功能。如果你这样做，则还可以使用 Windows PowerShell cmdlet 集合（可使 Azure REST 管理 API 易于调用）。

有关详细信息，请参阅以下资源：
* [使用 Azure 构建真实世界云应用 – 使一切自动化](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything)。电子书章节，其中介绍电子书中所示的示例应用程序如何使用 Windows PowerShell 脚本来创建 Azure 测试环境并部署到该环境中。请参阅[资源](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything#resources)部分以获取指向其他 Azure PowerShell 文档的链接。

###<a name="api"></a>使用 .NET 管理 API 自动进行部署

可以编写 C# 代码以执行 MSBuild 或 FTP 功能进行部署。如果你这样做，则可以访问 Azure 管理 REST API 来执行站点管理功能。

有关详细信息，请参阅以下资源：

* [使用 Azure 管理库和 .NET 使一切自动化](http://www.hanselman.com/blog/PennyPinchingInTheCloudAutomatingEverythingWithTheWindowsAzureManagementLibrariesAndNET.aspx)。.NET 管理 API 的简介和指向更多文档的链接。

###<a name="cli"></a>从 Azure 命令行界面 (Azure CLI) 进行部署

通过使用 FTP，可以在 Mac 或 Linux 计算机中通过 Windows 中的命令行来进行部署。如果你这样做，则还可以访问使用 Azure CLI 的 Azure REST 管理 API。

有关详细信息，请参阅以下资源：

* [Azure 命令行工具](/downloads/#cmd-line-tools)。Azure.com 中有关命令行工具信息的门户页面。

###<a name="webdeploy"></a>从 Web 部署命令行进行部署

[Web 部署](http://www.iis.net/downloads/microsoft/web-deploy)是用于部署到 IIS 的 Microsoft 软件，它不仅提供智能文件同步功能，还可以执行或协调许多其他与部署相关的、使用 FTP 时无法自动执行的任务。例如，Web 部署可以部署新的数据库或数据库更新以及你的 Web 应用。Web 部署还可以尽量减少更新现有站点所需的时间，因为它可以智能地仅复制更改过的文件。Microsoft WebMatrix、Visual Studio 和 Team Foundation Server 都支持内置 Web 部署，但你也可以直接从命令行使用 Web 部署自动进行部署。Web 部署命令非常强大，但在学习过程中可能会遇到困难。

有关详细信息，请参阅以下资源：

* [简单的 Web Apps：部署](http://azure.microsoft.com/blog/2014/07/28/simple-azure-websites-deployment/)。David Ebbo 的博客，介绍如何更轻松地使用 Web 部署。
* [Web 部署工具](http://technet.microsoft.com/zh-cn/library/dd568996)。Microsoft TechNet 网站上的正式文档。虽然已过时，但仍是一个不错的起点。
* [使用 Web 部署](http://www.iis.net/learn/publish/using-web-deploy)。Microsoft IIS.NET 网站上的正式文档。虽然也已过时，但仍是一个不错的起点。
* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。MSBuild 是由 Visual Studio 使用的生成引擎，还可以从命令行使用它以将 Web 应用程序部署到 Web Apps。本教程是主要介绍 Visual Studio 部署的一系列教程的一部分。

##<a name="nextsteps"></a>后续步骤

在某些情况下，你可能想要能够轻松地在 Web 应用的过渡版本和生产版本之间来回切换。有关详细信息，请参阅 [Web Apps 上的过渡部署](/documentation/articles/web-sites-staged-publishing)。

准备好备份和还原计划是任何部署工作流的一个重要部分。有关 Web Apps 的备份和还原功能的信息，请参阅 [Web Apps 备份](/documentation/articles/web-sites-backup)。

有关其他部署主题的信息，请参阅 [Web Apps 文档](/documentation/services/web-sites/)中的“部署”部分。

 

<!---HONumber=Mooncake_0215_2016-->