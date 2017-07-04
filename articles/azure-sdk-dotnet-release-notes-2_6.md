<properties 
   pageTitle="Azure SDK for .NET 2.6 发行说明" 
   description="Azure SDK for .NET 2.6 发行说明" 
   services="app-service/web" 
   documentationCenter=".net" 
   authors="Juliako" 
   manager="dwrede" 
   editor=""/>

<tags
	ms.service="app-service"
	ms.date="01/19/2016"
	wacn.date="03/28/2016"/>


# Azure SDK for .NET 2.6 发行说明

本文档包含 Azure SDK for .NET 2.6 发行版的发行说明。

使用 Azure SDK 2.6，你可以开发针对 .NET 4.5.2 或 .NET 4.6 的云服务应用程序 (PaaS)，前提是你在云服务角色上手动安装目标 .NET Framework。请参阅[在云服务角色上安装 .NET](/documentation/articles/cloud-services-dotnet-install-dotnet/)。


##Service Bus 更新

- 事件中心： 

	- 现在允许在发送事件时，通过公开事件中心的其他发布服务器终结点进行针对性的访问控制。
	- 通过提高稳定性和进行改进来增强事件中心功能。
	- 增加了对基于 WebSocket 且适用于消息传送和事件中心的 Amqp 协议的支持。

##HDInsight Tools for Visual Studio 更新

- **IntelliSense 增强功能**：远程元数据建议

	现在，HDInsight Tools for Visual Studio 支持在你编辑 Hive 脚本时获取远程元数据。例如，键入 **SELECT * FROM** 即会显示所有表名称。另外，在指定表后，还会显示列名称。

- **HDInsight Emulator 支持**

	HDInsight Tools for Visual Studio 现在支持连接到 HDInsight Emulator，因此，你可以在本地开发 Hive 脚本而不会引入任何成本，然后再针对 HDInsight 群集执行这些脚本即可。

	有关详细信息，请参阅[此手册](/documentation/articles/hdinsight-hadoop-emulator-get-started/)。

- **针对泛型 Hadoop 群集的 HDInsight Tools for Visual Studio 支持**（预览版）

	HDInsight Tools for Visual Studio 现在支持泛型 Hadoop 群集，因此，你可以使用 HDInsight Tools for Visual Studio 执行以下操作：

	- 连接到你的群集； 
	- 通过增强型 IntelliSense/自动完成支持编写 Hive 查询； 
	- 在你的群集中通过直观的 UI 查看所有作业。 

	有关详细信息，请参阅[此手册](/documentation/articles/hdinsight-hadoop-emulator-get-started/)。

##角色中缓存更新

- **角色中缓存**已更新，可以使用 **Azure 存储空间 SDK** 版本 4.3。到目前为止，**角色中缓存**一直在使用 Azure 存储空间 SDK 版本 1.7。

	使用 Azure SDK 2.5 或以下版本的客户应更新到 Azure SDK 2.6，并选择使用新版 Azure 存储空间 SDK。

	目前已计划在 2016 年 8 月 1 日删除 Azure 存储空间版本 2011-08-18。从 Azure SDK 2.5 或更低版本到 2.6 的任何角色中缓存迁移必须在该日期之前完成。有关停用 Azure 存储空间版本 2011-08-18 的详细信息，请参阅[有关删除 Azure 存储服务版本的最新信息：延期到 2016 年](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/10/19/microsoft-azure-storage-service-version-removal-update-extension-to-2016.aspx)。

>[AZURE.IMPORTANT] 我们特此宣布将在 2016 年 11 月 30 日停用 Azure 托管缓存服务和 Azure 角色中缓存。我们建议你迁移到 Azure Redis 缓存，以便为这次停用做好准备。有关日期和迁移指导的详细信息，请参阅[哪种 Azure 缓存产品适合我？](/documentation/articles/cache-faq/#which-azure-cache-offering-is-right-for-me)

##Azure Web 应用工具

以下各项已在 Azure SDK 2.6 版本中更新。

- Azure 发布功能已增强，其中包括了作为部署目标的 Azure API Apps。
- API Apps 预配功能，允许用户创建 API App 并行使预配功能。
- 服务器资源管理器已更改，可以反映新的 Azure Web 应用节点，并按资源组对 Web、移动和 API 应用进行分组。
- 将 Azure API 应用客户端手势添加到了大多数 C# 项目中，这样就可以自动生成可以在用户的 Azure 订阅中运行且支持 Swagger 的 API Apps。
- 服务器资源管理器中的 API Apps 工具和 Azure Web 应用节点仅在 Visual Studio 2013 中可用。 

##Azure 资源管理器工具更新

Azure 资源管理器工具已更新，现在包括适用于虚拟机、网络和存储的模板。JSON 编辑体验已更新，现在包括新的模板概况视图，并且可以使用 JSON 代码段来编辑模板。从 Visual Studio 部署的模板使用随项目提供的 PowerShell 脚本，因此对脚本所做的更改会被 Visual Studio 使用。

##针对云服务的诊断改进

Azure SDK 2.6 重新支持在 Azure 计算模拟器中收集诊断日志，并可将其传输到开发存储空间中。当应用程序在模拟器中运行时生成的任何诊断日志（包括应用程序跟踪日志、Windows 事件跟踪 (ETW) 日志、性能计数器、基础结构日志和 Windows 事件日志）都可以传输到开发存储空间中，以便验证你的诊断日志功能在本地计算机上是否正常工作。

现在，诊断存储帐户可以在服务配置 (.cscfg) 文件中指定，因此可以更轻松地针对不同环境使用不同的诊断存储帐户。链接字符串在 Azure SDK 2.4 和 Azure SDK 2.6 中的工作方式有明显的区别。

##重大变化

###Azure 资源管理器工具 

- 在 Azure SDK 2.5 中提供的“云部署项目”项目类型已重命名为“Azure 资源组”。
- 在 Azure SDK 2.5 中创建的项目的“云部署项目”类型可以在 2.6 中使用，但通过 Visual Studio 部署模板会失败。不过，仍可使用 PowerShell 脚本进行部署，就像以前一样。
 
##已知问题

- 在模拟器中收集诊断日志需要 64 位操作系统。在 32 位操作系统上运行时，不会收集诊断日志。这不会影响任何其他模拟器功能。 

- 在 2015 年 4 月 29 日发布的 Azure SDK 2.6 有两大问题：

	- 在计算机上安装了 Azure SDK 2.6 时，无法在 Visual Studio 2015 中加载通用应用程序。
	- 在 Visual Studio 2013 和 Visual Studio 2015 中调试云服务项目会失败，其中 Visual Studio 停止响应或崩溃，同时出现包含消息“为模拟器配置诊断”的对话框。
	
	Azure SDK 2.6 的更新已于 2015 年5 月 18 日发布。更新后的版本为 2.6.30508.1601；它包含上述两大问题的修补程序。若要确定 SDK 的版本，请转到“控制面板 -> 程序和功能 -> Azure Tools for Microsoft Visual Studio 2013 - v 2.6”。“版本”栏会显示版本号。

	如果仍出现上述问题，请安装最新版的 Azure 2.6 SDK，该版本适用于 [VS 2012](http://go.microsoft.com/fwlink/p/?linkid=323511&clcid=0x409)、[VS 2013](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids) 或 [VS 2015](http://go.microsoft.com/fwlink/?linkid=518003&clcid=0x409)。
 
##另请参阅

[Azure SDK for .NET 和 API 的支持和停用信息](https://msdn.microsoft.com/zh-cn/library/azure/dn479282.aspx/)

<!---HONumber=Mooncake_0118_2016-->