<properties
    pageTitle="使用命令行工具自动执行 Azure 应用部署 | Azure"
    description="发现如何从命令行部署 Azure 应用的信息"
    services="app-service"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="8b65980c-eb75-44a2-8e0f-f9eb9e617d16"
    ms.service="app-service"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="02/10/2017"
    ms.author="cephalin" />  


# 使用命令行工具自动执行 Azure 应用部署
可以使用命令行工具自动执行 Azure 应用部署。本文列出了可用工具以及说明如何在部署工作流中使用它们的有用链接。

## <a name="msbuild"></a>使用 MSBuild 自动部署
如果使用 Visual Studio IDE 进行部署，可以使用 [MSBuild](http://msbuildbook.com/) 自动执行可在 IDE 中完成的任何任务。可以将 MSBuild 配置为使用 [Web 部署](#webdeploy)或 FTP/FTPS 复制文件。Web 部署还可以自动执行许多其他与部署相关的任务，例如部署数据库。

有关使用 MSBuild 的命令行部署的详细信息，请参阅以下资源：

* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。有关使用 Visual Studio 部署到 Azure 的一系列教程中的第十篇。演示在 Visual Studio 中设置发布配置文件后如何使用命令行进行部署。
* [深入了解 Microsoft 生成引擎：使用 MSBuild 和 Team Foundation Build](http://msbuildbook.com/)。纸印书籍，其中包含了有关如何使用 MSBuild 进行部署的章节。

## <a name="powershell"></a>使用 Windows PowerShell 自动部署
你可以通过 [Windows PowerShell](http://msdn.microsoft.com/zh-cn/library/dd835506.aspx) 执行 MSBuild 或 FTP 部署功能。如果你这样做，则还可以使用 Windows PowerShell cmdlet 集合（可使 Azure REST 管理 API 易于调用）。

有关详细信息，请参阅以下资源：

* [部署链接到 GitHub 存储库的 Web 应用](/documentation/articles/app-service-web-arm-from-github-provision/)
* [设置使用 SQL 数据库的 Web 应用](/documentation/articles/app-service-web-arm-with-sql-database-provision/)
* [按可预见的方式在 Azure 中设置和部署微服务](/documentation/articles/app-service-deploy-complex-application-predictably/)
* [使用 Azure 构建真实世界云应用 – 使一切自动化](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything)。电子书章节，其中介绍电子书中所示的示例应用程序如何使用 Windows PowerShell 脚本创建 Azure 测试环境并部署到该环境中。请参阅[资源](http://asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything#resources)部分以获取指向其他 Azure PowerShell 文档的链接。

## <a name="api"></a>使用 .NET 管理 API 自动进行部署
可以编写 C# 代码以执行 MSBuild 或 FTP 功能进行部署。如果你这样做，则可以访问 Azure 管理 REST API 来执行站点管理功能。

有关详细信息，请参阅以下资源：

* [使用 Azure 管理库和 .NET 使一切自动化](http://www.hanselman.com/blog/PennyPinchingInTheCloudAutomatingEverythingWithTheWindowsAzureManagementLibrariesAndNET.aspx)。.NET 管理 API 的简介和指向更多文档的链接。

## <a name="cli"></a>从 Azure 命令行接口 (Azure CLI) 进行部署
通过使用 FTP，可以在 Mac 或 Linux 计算机中通过 Windows 中的命令行来进行部署。如果你这样做，则还可以访问使用 Azure CLI 的 Azure REST 管理 API。

有关详细信息，请参阅以下资源：

* [Azure 命令行工具](/downloads/)。Azure.com 中有关命令行工具信息的门户页面。

## <a name="webdeploy"></a>从 Web 部署命令行进行部署
[Web 部署](http://www.iis.net/downloads/microsoft/web-deploy)是用于部署到 IIS 的 Microsoft 软件，它不仅提供智能文件同步功能，还可以执行或协调许多其他与部署相关的，使用 FTP 时无法自动执行的任务。例如，Web 部署可以部署新的数据库或数据库更新以及你的 Web 应用。Web 部署还可以尽量减少更新现有站点所需的时间，因为它可以智能地仅复制更改过的文件。Microsoft Visual Studio 和 Team Foundation Server 支持内置 Web 部署，但你也可以直接从命令行使用 Web 部署自动进行部署。Web 部署命令非常强大，在学习过程中可能会遇到困难。

## 更多资源
使用命令行自动化的另一个部署选项是使用基于云的服务，例如 [Octopus 部署](http://en.wikipedia.org/wiki/Octopus_Deploy)。有关详细信息，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站](https://octopusdeploy.com/blog/deploy-aspnet-applications-to-azure-websites)。

有关命令行工具的详细信息，请参阅以下资源：

* [简单的 Web 应用：部署](https://azure.microsoft.com/blog/2014/07/28/simple-azure-websites-deployment/)。David Ebbo 的博客，介绍如何更轻松地使用 Web 部署。
* [Web 部署工具](http://technet.microsoft.com/zh-cn/library/dd568996)。Microsoft TechNet 站点上的官方文档。虽然已过时，但仍是一个不错的起点。
* [使用 Web 部署](http://www.iis.net/learn/publish/using-web-deploy)。Microsoft IIS.NET 站点上的官方文档。虽然也已过时，但仍是一个不错的起点。
* [使用 Visual Studio 的 ASP.NET Web 部署：命令行部署](http://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/command-line-deployment)。MSBuild 是由 Visual Studio 使用的生成引擎，还可以从命令行使用它以将 Web 应用程序部署到 Web 应用。本教程是主要介绍 Visual Studio 部署的一系列教程的一部分。

<!---HONumber=Mooncake_0206_2017-->