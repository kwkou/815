<properties pageTitle="什么是 Azure .NET SDK" metaKeywords="azure .net sdk" description="了解 Azure .NET SDK 中包含的内容。" documentationCenter=".NET" title="What is the Azure SDK for .NET" authors="tdykstra" solutions="" manager="wpickett" editor="mollybos" />
<tags 
	ms.service="multiple" 
	ms.date="08/14/2015" 
	wacn.date="10/3/2015"/>
# 什么是 Azure SDK for .NET？

Azure SDK for .NET 是一套应用程序，其中包括 Visual Studio 工具、命令行工具、运行时二进制文件和客户端库，可帮助你开发、测试和部署在 Azure 中运行的应用程序。本文详细介绍了安装 SDK 时获得的内容。你可以从[“Azure 下载”](/downloads/)页下载 SDK。

Azure SDK for .NET 还包含使用 Azure 服务所需的客户端库。这些库使用 NuGet 单独进行安装。

## 目录

- [在安装 Azure SDK for .NET 时获得的内容](#included)
- [在安装 Azure SDK for .NET 时未获得的内容](#notincluded)
- [常见问题](#faq)
- [版本](#versions)
- [资源](#resources)

##Azure SDK for .NET 中包含的内容

Azure SDK for .NET 将安装以下产品：

- [Visual Studio Express for Web](#vwd)
- [Microsoft ASP.NET 和 Web Tools for Visual Studio](#wte)
- [Microsoft Azure Tools for Microsoft Visual Studio](#tools)
- [Microsoft Azure 创作工具](#auth)
- [Microsoft Azure 模拟器](#emulator)
- [Microsoft Azure 存储模拟器](#stgemulator)
- [Microsoft Azure 存储工具](#stgtools)
- [用于 .NET 的 Microsoft Azure 库](#libraries)
- [用于 Visual Studio 的 LightSwitch Azure Publishing 外接程序](#ls)

###Visual Studio Express for Web

如果你的计算机上没有 Visual Studio，SDK 将安装 Visual Studio Express for Web。
 
###Microsoft ASP.NET 和 Web Tools for Visual Studio

这使你可以使用 Azure 网站：

* 将 Web 项目发布到 Azure 网站。
* 将控制台应用程序项目发布到 Azure WebJobs。
* 在创建新的 Web 项目或发布 Web 项目时创建 Azure 网站和 SQL 数据库资源。
* 在创建新网站时创建 PowerShell 部署脚本。
* 在服务器资源管理器中管理和故障诊断 Azure 网站。
* 在调试模式下针对网站和 WebJobs 远程运行。 

>[WACOM.NOTE]无需安装 Azure SDK for .NET 即可使用这些功能；它们还包括在 Visual Studio 更新中。

###Microsoft Azure Tools for Microsoft Visual Studio

这使你可以使用 Azure 云服务和虚拟机：

* 创建、打开和发布云服务项目。
* 创建云服务项目的部署包。
* 在创建新的 Web 项目时创建 Azure 虚拟机。
* 在创建新的虚拟机时创建 PowerShell 脚本。
* 查看和管理 Visual Studio 项目属性窗口中的云服务项目设置。
* 在服务器资源管理器中查看和管理 云服务、虚拟机 和 Service Bus。 
* 针对云服务和虚拟机在调试模式下远程运行。

###Microsoft Azure 创作工具

其中包括：

* CSPack 命令行工具，用于创建部署程序包。
* CSEncrypt 命令行工具，用于加密密码，以便使用密码通过远程桌面连接访问云服务角色实例。
* 运行时二进制文件，云服务项目需要使用该文件与运行时环境通信以及进行诊断。这些二进制文件在 NuGet 包中不提供。

###Microsoft Azure 模拟器

Azure 模拟器模拟云服务环境，这样你就可以先在本地计算机上测试云服务项目，然后再将其部署到 Azure。

###Microsoft Azure 存储模拟器

Azure 存储模拟器使用 SQL Server 实例和本地文件系统来模拟 Azure 存储空间（队列、表、Blob），以便在本地进行测试。

###Microsoft Azure 存储工具

这将安装命令行工具 AzCopy，以便将数据传入和传出 Azure 存储帐户。

###用于 .NET 的 Microsoft Azure 库

其中包括：

* 用于 Azure 存储空间、Service Bus 和 Caching 的 NuGet 包，存储在你的计算机上，方便 Visual Studio 脱机创建新的云服务项目。
* Visual Studio 插件，允许角色中缓存项目在 Visual Studio 中本地运行。 

###用于 Visual Studio 的 LightSwitch Azure Publishing 外接程序

这使你可以将 LightSwitch 项目发布到 Azure 网站。LightSwitch 外接程序包括在 Visual Studio 更新和 Azure SDK for .NET 中。安装 SDK 可确保你拥有最新版本的外接程序。

##在安装 Azure SDK for .NET 时未获得的内容

有几项功能需要用于 Azure 开发，但却未包括在 SDK 安装内容中。这其中，最重要的是下列功能：

* 客户端库。 

	Azure SDK 包括使用 Azure 服务时所需的客户端库，但你在安装 SDK 时并未安装所有这些库。如果你的应用程序需要的客户端库 SDK 并没有安装，你可以从 NuGet.org 获取。如果你的应用程序使用的客户端库是 SDK 安装的，则最好是使用 NuGet.org 上的最新版本对其进行更新。

  	**客户端库的本地副本。** Azure SDK for.NET 将某些 Azure 客户端库的 NuGet 包（如存储空间、服务总线和 Caching）复制到你的计算机上。这些客户端库将自动包括在新的云服务项目中，因此本地的 NuGet 程序包会启用 Visual Studio 来创建项目，即使你未连接到 Internet 也是如此。通常，客户端库的更新频率比 SDK 新版本的发布频率更为频繁，因此 NuGet.org 上的客户端库通常比你所获得的 SDK 更新。

	**包括客户端库的项目模板。** 只有 [Azure 云服务](/zh-cn/documentation/articles/cloud-services-dotnet-get-started/)和 [Azure 移动服务](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/)项目模板会自动包含一些客户端库。对于其他库或其他模板，安装你所需的[客户端库 NuGet 包](http://go.microsoft.com/fwlink/?LinkId=510472)。

* [Azure PowerShell](/zh-cn/documentation/articles/install-configure-powershell/)

	使用 Azure PowerShell，你可以自动化 Azure 环境的创建和部署。

* Azure 移动服务项目模板。

	移动服务模板仅在 Visual Studio 2013 Update 2 及更高版本中提供。这些模板在 Visual Studio 2012 或更早的版本中不提供，在 Visual Studio 2013 Update 1 或更早的版本中也不提供，即使你安装了 Azure SDK for .NET。

##常见问题

- 许多 Azure 功能已在 Visual Studio 中。我需要安装 Azure SDK for .NET 吗？
- 我想要一个客户端库。我必须安装 Azure SDK for .NET 才能获取它吗？
- [哪里可以找到较旧版本的 Azure SDK for .NET？](#olderversions)
- [Azure SDK for .NET 版本的生命周期策略是什么？](#lifecycle)
- [哪些来宾操作系统版本是 Azure SDK for .NET 兼容的？](#guestos)

###许多 Azure 功能已在 Visual Studio 中。我需要安装 Azure SDK for .NET 吗？

如果你想要使用最新工具针对 Azure 进行开发，则最好是安装该 SDK。如果你不愿意安装该 SDK，则在符合以下条件的情况下，你可以这样做：

* 你已安装了最新版的 Visual Studio 更新。
* 你的开发仅针对 Azure 网站或移动服务，不针对云服务或虚拟机。
* 你的应用程序不使用存储，或者它使用存储，但你不需要存储模拟器或 AzCopy 工具。

###我想要一个客户端库。我必须安装 Azure SDK for .NET 才能获取它吗？

该 SDK 仅安装客户端库，因此在没有连接到 Internet 的情况下，你也可以创建云服务项目。最新客户端库在 NuGet.org 上的 NuGet 包中提供。有关详细信息，请参阅本文档前面部分的[在安装 Azure SDK for .NET 时未获得的内容](#notincluded)。

###哪里可以找到较旧版本的 Azure SDK for .NET？

如需较旧版本，请参阅 Azure SDK for .NET 下载页。

###Azure SDK for .NET 版本的生命周期策略是什么？

请参阅 Microsoft Azure 云服务支持生命周期策略。

###哪些来宾操作系统版本是 Azure SDK for .NET 兼容的？

请参阅 Azure 来宾操作系统版本和 SDK 兼容性矩阵。

##版本

若要查看哪一个版本是最新版本或者需要下载较旧版本，请参阅 Azure SDK for .NET 版本历史记录 页。

##<a id="resources"></a>资源

若要下载最新的 Azure SDK for .NET 或客户端库，请参阅 “Azure 下载”页。

如需 Azure SDK for .NET 源代码，包括客户端库，请参阅 GitHub.com/Azure。
 
<!--HONumber=43-->
