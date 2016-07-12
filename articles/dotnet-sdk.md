<properties 
	pageTitle="什么是 Azure .NET SDK" 
	description="了解 Azure .NET SDK 中包含的内容。" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="mollybos" 
	services=""/>

<tags 
	ms.service="multiple" 
	ms.date="01/08/2016" 
	wacn.date="05/30/2016"/>

# 什么是 Azure SDK for .NET？

## 概述

Azure SDK for .NET 是一套应用程序，其中包括 Visual Studio 工具、命令行工具、运行时二进制文件和客户端库，可帮助你开发、测试和部署在 Azure 中运行的应用程序。本文详细介绍了安装 SDK 时获得的内容。您可以从[“Azure 下载”页](/downloads/)下载 SDK。

Azure SDK for .NET 还包含[使用 Azure 服务所需的客户端库](http://go.microsoft.com/fwlink/?LinkId=510472)。这些库使用 NuGet 单独进行安装。

## <a id="included"></a>Azure SDK for .NET 中包含的内容

Azure SDK for .NET 将安装以下产品：

- [Visual Studio Express for Web](#vwd)
- [Microsoft ASP.NET 和 Web Tools for Visual Studio](#wte)
- [Azure Tools for Microsoft Visual Studio](#tools)
- [Azure 创作工具](#auth)
- [Azure 模拟器](#emulator)
- [Azure 存储模拟器](#stgemulator)
- [Azure 存储空间工具](#stgtools)
- [Azure Libraries for .NET](#libraries)
- [HDInsight Tools for Visual Studio](#hdinsight) 和 [Microsoft Hive ODBC 驱动程序](#hdinsight)
- [Azure Mobile App SDK V1.0](#mobile)
- [Azure PowerShell](#ps)

### <a id="vwd"></a>Visual Studio Express for Web

如果你的计算机上没有 Visual Studio，SDK 将安装 [Visual Studio Express for Web](http://www.visualstudio.com/products/visual-studio-express-vs.aspx)。
 
### <a id="wte"></a>Microsoft ASP.NET 和 Web Tools for Visual Studio

这使你可以使用 Azure 网站：

* [将 Web 项目发布到 Azure 网站](/documentation/articles/web-sites-dotnet-get-started/)。
* [将控制台应用程序项目发布到 Azure WebJobs](/documentation/articles/websites-dotnet-deploy-webjobs/)。
* [在创建新的 Web 项目或发布 Web 项目时创建 Azure 网站和 SQL 数据库资源](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/)。
* [在创建新网站时创建 PowerShell 部署脚本](http://msdn.microsoft.com/zh-cn/library/dn642480.aspx)。

>[AZURE.NOTE] 无需安装 Azure SDK for .NET 即可使用这些功能；它们还包括在 Visual Studio 更新中。

### <a id="tools"></a>Azure Tools for Microsoft Visual Studio

这使您可以使用 Azure 资源，主要是云服务和虚拟机：

* [创建、打开和发布云服务项目](/documentation/articles/cloud-services-dotnet-get-started/)。
* [创建云服务项目的部署包](http://msdn.microsoft.com/zh-cn/library/ff683672.aspx)。
* [在创建新的 Web 项目时创建 Azure 虚拟机](/documentation/articles/virtual-machines-windows-classic-web-app-visual-studio/)。
* [在创建新的虚拟机时创建 PowerShell 脚本](http://msdn.microsoft.com/zh-cn/library/dn642480.aspx)。
* [查看和管理 Visual Studio 项目属性窗口中的云服务项目设置](http://msdn.microsoft.com/zh-cn/library/ee405486.aspx)。
* 在服务器资源管理器中查看和管理[云服务](http://msdn.microsoft.com/zh-cn/library/ff683675.aspx)、[虚拟机](http://msdn.microsoft.com/zh-cn/library/jj131259.aspx)和[服务总线](http://msdn.microsoft.com/zh-cn/library/jj149828.aspx)。 
* [针对云服务和虚拟机在调试模式下远程运行](http://msdn.microsoft.com/zh-cn/library/ff683670.aspx)。
* [使用 Azure 资源组部署项自动执行资源预配](https://msdn.microsoft.com/zh-cn/library/dn872471.aspx)

### <a id="auth"></a>Azure 创作工具

其中包括：

* [CSPack 命令行工具](http://msdn.microsoft.com/zh-cn/library/gg432988.aspx)，用于创建部署程序包。
* [CSEncrypt 命令行工具](http://msdn.microsoft.com/zh-cn/library/hh404001.aspx)，用于加密密码，以便使用密码通过远程桌面连接访问云服务角色实例。
* 运行时二进制文件，云服务项目需要使用该文件与运行时环境通信以及进行诊断。这些二进制文件在 NuGet 包中不提供。

### <a id="emulator"></a>Azure 模拟器

[Azure 模拟器](http://msdn.microsoft.com/zh-cn/library/dn339018.aspx)模拟云服务环境，这样您就可以先在本地计算机上测试云服务项目，然后再将其部署到 Azure。

### <a id="stgemulator"></a>Azure 存储模拟器

[Azure 存储模拟器](http://msdn.microsoft.com/zh-cn/library/hh403989.aspx)使用 SQL Server 实例和本地文件系统来模拟 Azure 存储空间（队列、表、Blob），以便在本地进行测试。

### <a id="stgtools"></a>Azure 存储空间工具

这将安装命令行工具 [AzCopy](http://aka.ms/AzCopy)，可以使用它将数据传入和传出 Azure 存储帐户。

### <a id="libraries"></a>Azure Libraries for .NET

其中包括：

* 用于 Azure 存储空间、Service Bus 和 Caching 的 NuGet 包，存储在你的计算机上，方便 Visual Studio 脱机创建新的云服务项目。
* Visual Studio 插件，允许[角色中缓存](http://msdn.microsoft.com/zh-cn/library/dn386103.aspx)项目在 Visual Studio 中本地运行。 

### <a id="hdinsight"></a>HDInsight Tools for Visual Studio，和 Microsoft Hive ODBC 驱动程序

在服务器资源管理器中的 HDInsight 工具，可以导航 Hive 数据库和 HDInsight 群集的链接存储帐户、创建表，并创建和提交 Hive 查询。有关详细信息，请参阅 [HDInsight Hadoop Tools for Visual Studio 入门](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)。

### <a id="mobile">Azure Mobile App SDK

用于 [Azure App Service Mobile Apps](/documentation/articles/app-service-mobile-value-prop-preview/) 的工具。

### <a id="ps"></a>Azure PowerShell

使用 Azure PowerShell，您可以 [自动化 Azure 环境的创建和部署](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything)。

## <a id="notincluded"></a>在安装 Azure SDK for .NET 时未获得的内容

有几项功能需要用于 Azure 开发，但却未包括在 SDK 安装内容中。这其中，最重要的是下列功能：

* [客户端库](http://go.microsoft.com/fwlink/?LinkId=510472)。 

	Azure SDK 包括使用 Azure 服务时所需的客户端库，但你在安装 SDK 时并未安装所有这些库。如果您的应用程序需要的客户端库 SDK 并没有安装，您可以从 [NuGet.org](http://go.microsoft.com/fwlink/?LinkId=510472) 获取。如果你的应用程序使用的客户端库是 SDK 安装的，则最好是使用 NuGet.org 上的最新版本对其进行更新。

  	**客户端库的本地副本。** Azure SDK for.NET 将某些 Azure 客户端库的 NuGet 包（如存储空间、服务总线和 Caching）复制到您的计算机上。这些客户端库将自动包括在新的云服务项目中，因此本地的 NuGet 程序包会启用 Visual Studio 来创建项目，即使您未连接到 Internet 也是如此。通常，客户端库的更新频率比 SDK 新版本的发布频率更为频繁，因此 NuGet.org 上的客户端库通常比您所获得的 SDK 更新。

	**包括客户端库的项目模板。** 只有 [Azure 云服务](/documentation/articles/cloud-services-dotnet-get-started/)和 [Azure 移动服务](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/)项目模板会自动包含一些客户端库。对于其他库或其他模板，安装您所需的[客户端库 NuGet 包](http://go.microsoft.com/fwlink/?LinkId=510472)。

* [Azure 移动服务项目模板](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/)。

	移动服务模板仅在 Visual Studio 2013 Update 2 及更高版本中提供。这些模板在 Visual Studio 2012 或更早的版本中不提供，在 Visual Studio 2013 Update 1 或更早的版本中也不提供，即使你安装了 Azure SDK for .NET。

## <a id="faq"></a>常见问题

- [许多 Azure 功能已在 Visual Studio 中。我需要安装 Azure SDK for .NET 吗？](#azinvs)
- [我想要一个客户端库。我必须安装 Azure SDK for .NET 才能获取它吗？](#clientlib)
- [哪里可以找到较旧版本的 Azure SDK for .NET？](#olderversions)
- [Azure SDK for .NET 版本的生命周期策略是什么？](#lifecycle)
- [哪些来宾操作系统版本是 Azure SDK for .NET 兼容的？](#guestos)
- [我如何卸载 Azure SDK for .NET？](#uninstall)

### <a id="azinvs"></a>许多 Azure 功能已在 Visual Studio 中。我需要安装 Azure SDK for .NET 吗？

如果你想要使用最新工具针对 Azure 进行开发，则最好是安装该 SDK。如果你不愿意安装该 SDK，则在符合以下条件的情况下，你可以这样做：

* 你已安装了最新的 [Visual Studio 更新](http://www.visualstudio.com/downloads/download-visual-studio-vs#DownloadFamilies_5)。
* 你的开发仅针对 Azure 网站或移动服务，不针对云服务或虚拟机。
* 你的应用程序不使用存储，或者它使用存储，但你不需要存储模拟器或 AzCopy 工具。

### <a id="clientlib"></a>我想要一个客户端库。我必须安装 Azure SDK for .NET 才能获取它吗？

该 SDK 仅安装客户端库，因此在没有连接到 Internet 的情况下，你也可以创建云服务项目。最新的客户端库在 [NuGet.org](http://go.microsoft.com/fwlink/?LinkId=510472) 上的 NuGet 包中提供。有关详细信息，请参阅本文档前面部分的[在安装 Azure SDK for .NET 时未获得的内容](#notincluded)。

### <a id="olderversions"></a>哪里可以找到较旧版本的 Azure SDK for .NET？

如需较旧版本，请参阅 [Azure SDK for .NET](/downloads/archive-net-downloads/) 下载页。

### <a id="lifecycle"></a>Azure SDK for .NET 版本的生命周期策略是什么？

请参阅 [Azure 云服务支持生命周期策略](http://support.microsoft.com/gp/azure-cloud-lifecycle-faq)。

### 哪些来宾操作系统版本是 Azure SDK for .NET 兼容的？

请参阅 [Azure 来宾 OS 版本和 SDK 兼容性矩阵](http://msdn.microsoft.com/zh-cn/library/ee924680.aspx)。

### <a id="uninstall"></a>我如何卸载 Azure SDK for .NET？

在 [Azure SDK for .NET 中包含的内容](#included) 一文中所列出的每一个项目在 Windows 控制面板的**程序和功能**中是作为单独的程序列出的。因此，无法将所有项目作为组一起卸载；必须单独卸载每个程序。

如果您已经安装了 Azure SDK for.NET，在安装新版本时，通常无需卸载旧版本。在大多数情况下，SDK 安装会更新现有程序，而不是添加一个新程序并保留旧程序。

但是，如果您想要删除不再需要的早期版本的残留文件，只有在显示存在同一程序的更新版本时才能进行卸载，并且只能卸载指明了较旧版本号的程序。例如，在从 2.5 版本更新到 2.6 后，你可能会同时看到“Micrsoft Azure Tools for Microsoft Visual Studio 2013”存在 2.5 和 2.6 两个版本。在此情况下，你可以卸载 2.5 版本。但是，你也可能只看到“Azure 创作工具”的 2.5 版本。在此情况下，卸载程序则是不安全的。

请注意，在**程序和功能**中所显示的程序版本号可能会产生误导。例如，SDK 2.6 版包括“Micrsoft Azure Mobile App SDK V1.0”。如果你安装了适用于 Visual Studio 2013 的 SDK 和适用于 Visual Studio 2015 的”Micrsoft Azure Mobile App SDK V2.0”，在此情况下，显示的版本号并非 SDK 版本号，而是该程序适用的 Visual Studio 版本号。

本文不会列出 Azure SDK 每个旧版本所包括的所有程序；因为在 SDK 的更高版本中有时会删除旧版本中的一些程序，所以您可以从较早的版本卸载这些程序。例如，版本 2.5 安装有“适用于 Visual Studio 的 Azure 资源管理器工具”，但是因为在**程序和功能**中不再将其作为单独的程序列出，所以本文也未将它列出。本文仅列出包含在 Azure SDK for.NET 版本 2.6 中的那些程序。

> **注意：**在其他环境中可能会单独安装 SDK 的某些程序，因此，即使您不需要完整的 SDK，但这些程序也可能是需要的。同样地，即使在新版本的 SDK 中删除了旧版本中的某些程序，但仍然可能需要这些程序。因此，在卸载程序时，请务必小心谨慎，以免删除了计算机上仍然需要的内容。



## <a id="versions"></a>版本

若要查看哪一个版本是最新版本或者需要下载较旧版本，请参阅 [Azure SDK for .NET 版本历史记录](/downloads/archive-net-downloads/)页。

## <a id="resources"></a>资源

若要下载最新的 Azure SDK for .NET 或客户端库，请参阅[“Azure 下载”页](/downloads/)。

如需 Azure SDK for .NET 源代码，包括客户端库，请参阅 [GitHub.com/Azure](https://github.com/azure/)。

有关 Azure 客户端库的参考文档，请参阅 [Azure.NET 参考](/documentation/api/)。

<!---HONumber=Mooncake_0215_2016-->