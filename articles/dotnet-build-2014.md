<properties pageTitle="Azure Spring 2014 版本亮点 - .NET 开发人员中心" metaKeywords="azure .net sdk 2.3" description="了解提供给 Azure .NET 开发人员的新工具和功能。" documentationCenter=".NET" title="Azure Spring 2014 版本亮点" authors="mollybos" solutions="" manager="carolz" editor="mollybos" />

# Azure Spring 2014 版本亮点

本文概述了在“版本 2014”大会上讨论的以及在 Azure SDK for .NET 2.3、Visual Studio 2013 Update 2 和其他 Spring 版本中提供的适合 Azure .NET 开发人员使用的新工具、功能和主题。

**获取以下工具：**

-   [Visual Studio 2013 Update 2 RC][Visual Studio 2013 Update 2 RC]
-   [Azure SDK 2.3][Azure SDK 2.3]
-   [Azure PowerShell][Azure PowerShell]
-   [Azure 跨平台命令行界面][Azure 跨平台命令行界面]

有关 Spring 2014 工具发布版本的完整详细信息，请参阅 [Azure SDK for.NET 2.3 发行说明][Azure SDK for.NET 2.3 发行说明]和 [Visual Studio 2013 产品更新页面][Visual Studio 2013 产品更新页面]。

[观看 Build 视频][观看 Build 视频]按需流式处理。

## 目录

-   [Web 开发和发布][Web 开发和发布]
-   [诊断和调试][诊断和调试]
-   [管理 Visual Studio 中的 Azure 服务][管理 Visual Studio 中的 Azure 服务]
-   [使用 PowerShell 自动执行][使用 PowerShell 自动执行]
-   [使用.NET 的移动开发][使用.NET 的移动开发]
-   [存储客户端库 3.0 和新的存储模拟器][存储客户端库 3.0 和新的存储模拟器]
-   [资源管理器][资源管理器]

## <span id="webdeploy"></span></a>Web 开发和发布

Azure SDK 2.3 和 Visual Studio 2013 Update 2 RC 包括几个通过 Azure 来简化 Web 开发和发布的更新。现在，当你创建新的 Web 应用程序时，还可以创建 Azure 网站或虚拟机。当你准备好发布网站时，你可以使用更新的 Web 发布对话框或已添加到你的解决方案中的 PowerShell 脚本将你的网站直接部署到 Azure 网站或虚拟机。请查看下列资源，了解更多详细信息以及介绍如何利用新功能的教程：

-   [Azure 和 ASP.NET 入门][Azure 和 ASP.NET 入门]
-   [Azure Tools for Visual Studio 入门][Azure Tools for Visual Studio 入门]
-   [在 Visual Studio 2013 中创建 ASP.NET Web 项目][在 Visual Studio 2013 中创建 ASP.NET Web 项目]
-   [版本 2014：Visual Studio 2013 Update 2 及更高版本中针对 ASP.NET 和 Web 的新功能（视频）][版本 2014：Visual Studio 2013 Update 2 及更高版本中针对 ASP.NET 和 Web 的新功能（视频）]

## <span id="diagnostics"></span></a>诊断和调试

使用新的虚拟机远程调试功能以及新的本机代码调试功能远程诊断应用程序问题：

-   [在 Visual Studio 中调试云服务或虚拟机][在 Visual Studio 中调试云服务或虚拟机]

Emulator Express 是新的针对云服务的轻量型本地模拟器。了解如何使用它来测试本地计算机上的云服务：

-   [使用 Emulator Express 来运行和调试云服务][使用 Emulator Express 来运行和调试云服务]

安装 Azure SDK 2.3 以后，你现在就可以直接从服务器资源管理器远程查看和编辑网站文件，以及查看站点日志文件。当你保存已编辑的文件时，它会保存回你的站点，无需发布。有关详细信息，请参阅：

-   [在 Visual Studio 中对 Azure 网站进行故障排除][在 Visual Studio 中对 Azure 网站进行故障排除]

## <span id="service-management"></span></a>管理 Visual Studio 中的 Azure 服务

利用 Visual Studio 提供的改进型虚拟机管理功能，包括在 IDE 中创建 VM 的功能：

-   [从服务器资源管理器创建虚拟机][从服务器资源管理器创建虚拟机]
-   [从服务器资源管理器访问 Azure 虚拟机][从服务器资源管理器访问 Azure 虚拟机]

我们还提供了几项改进，帮助你通过服务器资源管理器更有效地管理其他 Azure 服务。有关详细信息，请参阅：

-   [使用 Visual Studio 服务器资源管理器浏览 Service Bus 资源][使用 Visual Studio 服务器资源管理器浏览 Service Bus 资源]
-   [使用服务器资源管理器浏览存储资源][使用服务器资源管理器浏览存储资源]

## <span id="automation"></span></a>使用 PowerShell 和 API 执行自动操作

安装 Azure Powershell，以便对网站、Web 作业等使用 cmdlet。编写更多的使用 Service Management API for .NET 的自定义自动化功能。有关详细信息，请参阅：

-   [如何安装和配置 Azure PowerShell][如何安装和配置 Azure PowerShell]
-   [Azure PowerShell 文档][Azure PowerShell 文档]
-   [版本 2014：在 Azure 中通过新的 SDK、工具和服务实现无处不在的自动化（视频）][版本 2014：在 Azure 中通过新的 SDK、工具和服务实现无处不在的自动化（视频）]

直接在 Visual Studio 中创建 PowerShell 脚本，并使用它们自动执行你的环境创建操作：

-   [使用 Windows PowerShell 脚本发布到开发和测试环境][使用 Windows PowerShell 脚本发布到开发和测试环境]

## <span id="mobile"></span></a>使用.NET 的移动开发

Azure 移动服务现在提供一个选项，允许你将基于 .NET 的后端用于面向 Windows 应用商店、Windows Phone、iOS 和 Android 等移动平台的移动应用程序。若要了解详细信息，请查看：

-   [云量：Azure 移动服务 .NET 后端（视频）][云量：Azure 移动服务 .NET 后端（视频）]
-   [Azure 移动开发人员中心][Azure 移动开发人员中心]

Visual Studio 2013 Update 2 还提供针对移动开发的新支持，包括服务器资源管理器中的远程调试支持和通知中心集成。有关详细信息，请参阅：

-   [快速入门：添加移动服务][快速入门：添加移动服务]
-   [如何使用 Visual Studio 向运行的应用程序发送推送通知][如何使用 Visual Studio 向运行的应用程序发送推送通知]
-   [如何在移动服务中创建自定义 API 和计划的作业][如何在移动服务中创建自定义 API 和计划的作业]

## <span id="storage"></span></a>存储客户端库 3.0

Azure SDK 2.3 包括更新后的存储模拟器，而存储客户端库 3.0 则集成到了 SDK 中包含的项目模板中。

有关详细信息，请参阅：

-   [Azure 存储客户端库 3.0][Azure 存储客户端库 3.0]
-   [Azure 存储空间简介][Azure 存储空间简介]
-   [版本 2014：Microsoft Azure 存储空间 – 新功能、最佳做法和模式（视频）][版本 2014：Microsoft Azure 存储空间 – 新功能、最佳做法和模式（视频）]
-   [Microsoft Azure 存储空间 @ 版本 2014][Microsoft Azure 存储空间 @ 版本 2014]

## <span id="arm"></span></a>资源管理器

资源管理器是一个新框架，用于部署和管理跨资源的应用程序。尝试使用包含新 JSON 编辑器、PowerShell cmdlet 和 CLI 支持的资源管理器。有关详细信息，请查看：

-   [将 Azure PowerShell 用于资源管理器][将 Azure PowerShell 用于资源管理器]
-   [将 Azure 跨平台命令行界面用于资源管理器][将 Azure 跨平台命令行界面用于资源管理器]
-   [版本 2014：Azure 资源组模型：适合现代云的现代管理（视频）][版本 2014：Azure 资源组模型：适合现代云的现代管理（视频）]

  [Visual Studio 2013 Update 2 RC]: http://aka.ms/vs2013update2rc
  [Azure SDK 2.3]: http://www.windowsazure.com/zh-cn/downloads/
  [Azure PowerShell]: http://go.microsoft.com/?linkid=9811175
  [Azure 跨平台命令行界面]: http://go.microsoft.com/?linkid=9828653
  [Azure SDK for.NET 2.3 发行说明]: http://go.microsoft.com/fwlink/p/?LinkId=393548
  [Visual Studio 2013 产品更新页面]: http://go.microsoft.com/fwlink/?LinkId=272487
  [观看 Build 视频]: http://go.microsoft.com/fwlink/?LinkId=394377&clcid=0x409
  [Web 开发和发布]: #webdeploy
  [诊断和调试]: #diagnostics
  [管理 Visual Studio 中的 Azure 服务]: #service-management
  [使用 PowerShell 自动执行]: #automation
  [使用.NET 的移动开发]: #mobile
  [存储客户端库 3.0 和新的存储模拟器]: #storage
  [资源管理器]: #arm
  [Azure 和 ASP.NET 入门]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-dotnet-get-started/
  [Azure Tools for Visual Studio 入门]: http://msdn.microsoft.com/zh-cn/library/azure/ff687127.aspx
  [在 Visual Studio 2013 中创建 ASP.NET Web 项目]: http://asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio
  [版本 2014：Visual Studio 2013 Update 2 及更高版本中针对 ASP.NET 和 Web 的新功能（视频）]: http://channel9.msdn.com/Events/Build/2014/3-602
  [在 Visual Studio 中调试云服务或虚拟机]: http://msdn.microsoft.com/zh-cn/library/azure/ff683670.aspx
  [使用 Emulator Express 来运行和调试云服务]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn339018.aspx
  [在 Visual Studio 中对 Azure 网站进行故障排除]: http://www.windowsazure.com/zh-cn/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio
  [从服务器资源管理器创建虚拟机]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn569263.aspx
  [从服务器资源管理器访问 Azure 虚拟机]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj131259.aspx
  [使用 Visual Studio 服务器资源管理器浏览 Service Bus 资源]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj149828.aspx
  [使用服务器资源管理器浏览存储资源]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff683677.aspx
  [如何安装和配置 Azure PowerShell]: http://www.windowsazure.com/zh-cn/documentation/articles/install-configure-powershell/
  [Azure PowerShell 文档]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj156055.aspx
  [版本 2014：在 Azure 中通过新的 SDK、工具和服务实现无处不在的自动化（视频）]: http://channel9.msdn.com/Events/Build/2014/3-621
  [使用 Windows PowerShell 脚本发布到开发和测试环境]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dn642480.aspx
  [云量：Azure 移动服务 .NET 后端（视频）]: http://channel9.msdn.com/Shows/Cloud+Cover/Episode-137-The-Azure-Mobile-Services-NET-Backend-with-Yavor-Georgiev
  [Azure 移动开发人员中心]: /zh-cn/develop/mobile/
  [快速入门：添加移动服务]: http://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/dn629482.aspx
  [如何使用 Visual Studio 向运行的应用程序发送推送通知]: http://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/dn614131.aspx
  [如何在移动服务中创建自定义 API 和计划的作业]: http://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/dn614130.aspx
  [Azure 存储客户端库 3.0]: http://go.microsoft.com/fwlink/?LinkId=394927
  [Azure 存储空间简介]: /zh-cn/documentation/articles/storage-introduction/
  [版本 2014：Microsoft Azure 存储空间 – 新功能、最佳做法和模式（视频）]: http://channel9.msdn.com/Events/Build/2014/3-628
  [Microsoft Azure 存储空间 @ 版本 2014]: http://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/08/microsoft-azure-storage-build-2014.aspx
  [将 Azure PowerShell 用于资源管理器]: http://go.microsoft.com/fwlink/?LinkID=394767
  [将 Azure 跨平台命令行界面用于资源管理器]: /zh-cn/documentation/articles/xplat-cli-azure-resource-manager/
  [版本 2014：Azure 资源组模型：适合现代云的现代管理（视频）]: http://channel9.msdn.com/Events/Build/2014/2-607
