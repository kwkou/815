<properties 
   pageTitle="Azure SDK for .NET 2.8 发行说明" 
   description="Azure SDK for .NET 2.8 发行说明" 
   services="app-service\web" 
   documentationCenter=".net" 
   authors="Juliako" 
   manager="dwrede" 
   editor=""/>

<tags
	ms.service="app-service"
	ms.date="01/31/2016"
	wacn.date="03/17/2016"/>

# Azure SDK for .NET 2.8、2.8.1 和 2.8.2

##概述
 
本文包含 Azure SDK for .NET 2.8、2.8.1 和 2.8.2 版本的发行说明（包括已知问题和重大更改）。

有关此版本中的新功能和更新的完整列表，请参阅 [Azure SDK 2.8 for Visual Studio 2013 和 Azure SDK 2.8 for Visual Studio 2015](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-8-for-net/) 通知。

##  Azure SDK for .NET 2.8

### 下载 Azure SDK for .NET 2.8

[Azure SDK for .NET 2.8 for Visual Studio 2015](http://go.microsoft.com/fwlink/?LinkId=699285)

[Azure SDK for .NET 2.8 for Visual Studio 2013](http://go.microsoft.com/fwlink/?LinkId=699287)
 
### .NET 4.5.2 支持 

####已知问题

Azure.NET SDK 2.8 可让你创建 .NET 4.5.2 云服务包。不过，.NET 4.5.2 Framework 要在 2016 年 1 月来宾 OS 版本发布后，才会安装在默认的来宾 OS 映像上。在此之前，可以通过独立的来宾 OS 版本 November 2015-02 获取 .NET 4.5.2 Framework。请参阅 [Azure 来宾 OS 版本和 SDK 兼容性对照表](/documentation/articles/cloud-services-guestos-update-matrix/)页跟踪该映像的发布时间。November 2015-02 映像发布后，你可以选择通过更新云服务配置文件 (.cscfg) 来使用该映像。在服务配置文件中，将 ServiceConfiguration 元素的 osVersion 属性设置为字符串“WA-GUEST-OS-4.26\_201511-02”。如果你选择使用此映像，将不再获得来宾 OS 的自动更新。若要获取自动更新，osVersion 必须设置为“\*”，并且只能在 2016 年 1 月通过自动更新获取 .NET 4.5.2。

###Azure 数据工厂

####已知问题 

如果计算机上安装的 Azure PowerShell 版本高于 0.9.8，则在创建与示例数据相关的 **数据工厂模板** 项目期间，Azure PowerShell 脚本可能会失败。

若要成功创建这种类型的项目，必须安装 [Azure PowerShell 0.9.8 版](https://github.com/Azure/azure-powershell/releases/download/v0.9.8-September2015/azure-powershell.0.9.8.msi)。


### Azure 资源管理器工具 

####重大变化

Azure 资源组项目提供的 PowerShell 脚本在此版本中已更新，可以配合新的 Azure PowerShell cmdlet 1.0 版使用。使用 2.8 以前的 SDK 版本时，将无法从 Visual Studio 使用此新脚本。

使用 2.8 SDK 时，无法从 Visual Studio 运行在旧版 SDK 上创建的项目中的脚本。如果使用适当的 Azure PowerShell cmdlet 版本，所有脚本可继续在 Visual Studio 外部运行。

2\.8 SDK 需要 1.0 版的 Azure PowerShell cmdlet。所有其他版本的 SDK 需要 0.9.8 版的 Azure PowerShell cmdlet。

###Web 工具扩展

####已知问题

以下版本将解决以下已知问题。

- 非生产环境（例如 Azure 中国区或 Azure Stack 客户）的 Azure Web 应用相关云与服务器资源管理器笔势无效。对于这些受影响区域中的客户，从 Azure 经典管理门户下载发布配置文件将启用发布功能。以后的版本将为 Azure 中国区和 Azure Stack 客户修复笔势，例如“附加调试器”和“查看流日志”。
- 当客户在创建 Azure Web 应用时，如果要部署的目标 App Insights 实例位于中国东部以外的区域，可能会看到错误。在这种情况下，在经典管理门户中创建 Azure 和下载发布配置文件将启用发布方案。

###Azure HDInsight 工具

####新的更新

- 可以通过 HiveServer2 在群集中运行 Hive 查询，而且几乎不需要额外的开销，即可实时查看作业日志。
- 使用新的 Hive 任务执行视图可以更深入地分析作业、查找更多详细信息和识别潜在问题。

有关信息，请参阅 [Azure SDK 2.8 for Visual Studio 2013 和 Azure SDK 2.8 for Visual Studio 2015](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-8-for-net/)。

## Azure SDK for .NET 2.8.1

### Visual Studio 2013 和 Visual Studio 2015 的已知问题
 
1. 以槽为目标的触发 Web 作业发布将显示错误，并且不会设置计划，但会将 Web 作业推送到 Azure。需要计划作业的客户可以使用 Azure 经典管理门户来设置 Web 作业的计划。 
2. Python 客户可能会遇到调试器问题。服务团队正在针对此问题推出修复程序，但如果客户受到影响，请通过论坛、通知博客或发行说明意见部分告知 Microsoft。 
3. 某些区域（如印度南部）的客户会遇到 Azure Web 应用预配错误。这与经典管理门户中的情况一致，遇到此问题的客户可以使用 Azure 经典管理门户来请求这些地理区域的发布访问权限。使用 Azure 经典管理门户请求这些区域的访问权限后，预配应能正常进行。

##Azure SDK for .NET 2.8.2

在安装 2.8.2 工具以后，客户可能会遇到以下问题。

- 如果你使用的是 Windows 10 且尚未安装 Internet Explorer，则可能会遇到“找不到 Internet Explorer”错误。若要解决此问题，请使用“添加/删除 Windows 组件”对话框来安装 Internet Explorer。

如果你发现此问题，请使用“发送微笑”功能进行报告。

有关详细信息，请参阅[此文章](https://azure.microsoft.com/zh-cn/blog/announcing-azure-sdk-2-8-2-for-net/)。

##其他更新

有关其他更新，请参阅 [Azure SDK 2.8 通知文章](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-8-for-net/)。

##另请参阅

[Azure SDK 2.8 通知文章](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-8-for-net/)

[Azure SDK for .NET 和 API 的支持和停用信息](https://msdn.microsoft.com/zh-cn/library/azure/dn479282.aspx)

<!---HONumber=Mooncake_0307_2016-->