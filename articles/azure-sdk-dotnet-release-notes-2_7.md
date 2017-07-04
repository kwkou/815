
<properties 
   pageTitle="Azure SDK for .NET 2.7 和 .NET 2.7.1 发行说明" 
   description="Azure SDK for .NET 2.7 和 .NET 2.7.1 发行说明" 
   services="app-service\web" 
   documentationCenter=".net" 
   authors="Juliako" 
   manager="dwrede" 
   editor=""/>

<tags
	ms.service="app-service"
	ms.date="01/19/2016"
	wacn.date="03/28/2016"/>


# Azure SDK for .NET 2.7 和 .NET 2.7.1 发行说明

##概述

本文档包含 Azure SDK for .NET 2.7 发行版的发行说明。

本文档还包含 Azure SDK for .NET 2.7.1 发行版的发行说明。

只有 Visual Studio 2015 和 Visual Studio 2013 才支持 Azure SDK 2.7。[Azure SDK 2.6](/downloads/) 是 Visual Studio 2012 支持的最后一个 SDK。

有关此版本的详细信息，请参阅 [Azure SDK 2.7 通告文章](https://azure.microsoft.com/zh-cn/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)和 [Azure SDK 2.7.1 通告文章](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-7-1-for-net/)。

##Azure SDK for .NET 2.7

###Visual Studio 2015 的登录改进

适用于 Visual Studio 2015 的 Azure SDK 2.7 支持 Visual Studio 2015 中的新标识管理功能。这包括支持通过基于角色的访问控制、云解决方案提供者、DreamSpark 以及其他帐户和订阅类型访问 Azure 的帐户。

只能在 Visual Studio 2015 中使用 Azure SDK 2.7 附带的登录改进。Azure SDK 2.7.1 中附带了对 Visual Studio 2013 的支持。


###移动 SDK

更新了 **Mobile Apps** 模板，以反映最新的 [NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/)和设置过程。

###服务总线 

一般性的 Bug 修复与改进。有关更新和功能的详细信息，请参阅最新[服务总线 NuGet](http://www.nuget.org/packages/WindowsAzure.ServiceBus/) 的发行说明。

###HDInsight 工具 

此版本做了以下更新。这些更新目前以预览版提供。有关详细信息，请参阅[此博客](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)。

- 为 Tez 上的 Hive 作业绘制 Hive 图形
- 完全支持 Hive DML IntelliSense
- Pig 模板支持
- Azure 服务的 Storm 模板

####重大变化

- 使用此版本的工具时，必须先升级旧的 **Storm** 项目。有关详细信息，请参阅[此博客](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)。
- 不再支持 Visual Studio Web Express。有关详细信息，请参阅[此博客](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)。

###Azure Web 应用工具

此版本对 Web 工具扩展做了以下更新。有关详细信息，请参阅[此](https://azure.microsoft.com/zh-cn/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)博客。

- 添加了对 DreamSpark 帐户的支持
- 对 Azure 工具做了全面的更改，以支持新的 Azure 资源管理 API
- 在[云资源管理器](#cloud_explorer)中添加了对 Azure Web 应用的支持

####已知问题

服务器资源管理器的“槽”节点下面不会出现 Web 应用部署槽节点，而云资源管理器下面不会加载 Web 应用部署槽的子节点。此问题已解决，下一个 SDK 版本将不再发生。


###<a id="cloud_explorer"></a>适用于 Visual Studio 2015 的云资源管理器

Azure SDK 2.7 包含适用于 Visual Studio 2015 的云资源管理器，可让你从 Visual Studio 中查看 Azure 资源、检查其属性，以及执行重要的开发人员操作。

云资源管理器支持以下功能：

- Azure 资源的“资源组”和“资源类型”视图 
- 按名称搜索资源（可在“资源类型”视图中使用）
- 支持应用了基于角色的访问控制 (RBAC) 的订阅和资源 
- 集成式“操作”面板，针对特定选择的资源显示以开发人员为焦点的操作。例如：附加使用资源管理器堆栈创建的虚拟机的远程调试器、查看虚拟机的诊断数据，等等。
- 集成式“属性”面板，显示以开发人员为焦点，在开发/测试期间经常需要的属性 
- 快速切换枚举资源时要使用的帐户（使用工具栏上的“设置”命令） 
- 筛选枚举资源时要使用的订阅（使用工具栏上的“设置”命令） 
 
 
###Azure 资源管理器工具 

Azure 资源管理器工具已更新为使用基于角色的访问控制 (RBAC) 和新的订阅类型。除了经典存储之外，这些更改还附带了使用新存储帐户在部署期间存储项目的功能。

如果在 SDK 2.7 中使用旧版 SDK 中的 Azure 资源组，则需要使用新的存储帐户而不是经典存储帐户来部署新的部署脚本。在你更改项目以添加新的脚本之前，系统会发出提示。旧脚本将重命名，你需要手动修改新的脚本。
 
 
###存储资源管理器工具 

- 支持查看附加 Blob。有关详细信息，请参阅[此博客文章](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/04/13/introducing-azure-storage-append-blob.aspx)。
- 支持在服务器浏览器查看高级存储账户。服务器浏览器中只显示高级存储账户的页 blob，因为高级存储账户只支持这个类型。 

##Azure SDK for .NET 2.7.1

以下部分包含的更新是 Azure SDK for .NET 2.7.1 发行版中引入的。
###HDInsight 工具 

有关 HDInsight 工具更新的更多详细说明，请参阅[此博客](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-7-1-for-net/)。

- Hive 作业运算符视图（新功能）

	为了帮助你更好地了解 Hive 查询，我们添加了 Hive 运算符视图功能。若要查看某个顶点中的所有运算符，可双击作业图的相应顶点。若要查看特定运算符的更多详细信息，可将鼠标悬停在该运算符上方。
- Hive 错误标记（新功能）

	为了让你能够即时查看语法错误，我们添加了 Hive 错误标记功能。此外，我们还增强了错误消息功能，你现在可以即时查看详细的语法错误（在此版本发布之前，你需要将 Hive 脚本提交给群集，然后再等待一段时间才能获得详细的错误消息）。  
- Storm 拓扑图（新功能）

	当你想要了解你的拓扑是否正常工作时，可视化十分重要。在此版本中，我们添加了 Storm 图的可视化功能。你可以将拓扑的重要度量值可视化（例如，可以用颜色来表示某个 Bolt 是否“忙碌”）。也可双击“Bolt/Spout”来查看更多详细信息。

- 支持 Azure 经典管理门户中创建的 HDInsight 群集（Bug 修复）

	现在，你可以使用 Visual Studio 来查看作业并将其提交给所有 HDInsight 群集，不管群集是在哪里创建的。

- 更多 IntelliSense 支持，更快的 Hive 元数据加载速度（改进）

	我们改进了 IntelliSense，添加了更多用户友好建议。例如，现在还可以在 IntelliSense 中提供表别名建议，方便你撰写查询。此外，我们还改进了 Hive 元数据加载，只需数秒即可列出 Hive 源存储中的所有数据库、表和列。

有关 HDInsight 工具更新的更多详细说明，请参阅[此博客](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-7-1-for-net/)。

###Visual Studio 2013 中的改进

- 有了 Azure SDK 2.7.1，Visual Studio 2013 就可以通过基于角色的访问控制、云解决方案提供程序和 Dreamspark 访问 Azure 帐户和订阅。
- 现在，由于有了 Azure SDK 2.7.1，在 Visual Studio 2013 中还可以使用新的云资源管理器工具窗口。

###已知问题

在非英语 OS 上安装适用于 Visual Studio Community 2013 的 Azure SDK 2.6 或 2.7.1 会显示一个警告，指出 Visual Studio 的英语资源和非英语资源可能不匹配。此警告可以忽略，不会出什么问题。仅当计算机此前没有安装 Visual Studio Community 2013，而你是在非英语 OS 上安装 SDK 时，才会出现这种情况。只有在语言包向 Visual Studio 应用 RTM 资源后，才会显示此警告，此时尚未应用 Update 4。忽略此警告后，语言包就可以继续应用语言包内容的 Update 4 版本。

LightSwitch 项目与此版本不兼容。下一个 SDK 版本将解决此问题。

##另请参阅
[Azure SDK 2.7.1 通告文章](https://azure.microsoft.com/zh-cn/blog/announcing-the-azure-sdk-2-7-1-for-net/)

[Azure SDK 2.7 通告文章](https://azure.microsoft.com/zh-cn/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)

[Azure SDK for .NET 和 API 的支持和停用信息](https://msdn.microsoft.com/zh-cn/library/azure/dn479282.aspx/)

<!---HONumber=Mooncake_0118_2016-->