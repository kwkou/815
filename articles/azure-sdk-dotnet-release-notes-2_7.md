
<properties 
   pageTitle="Azure SDK for .NET 2.7 发行说明" 
   description="Azure SDK for .NET 2.7 发行说明" 
   services="app-service/web" 
   documentationCenter=".net" 
   authors="Juliako" 
   manager="dwrede" 
   editor=""/>

<tags
   ms.service="app-service"
   ms.date="07/20/2015"
   wacn.date=""/>


# Azure SDK for .NET 2.7 发行说明

本文档包含 Azure SDK for .NET 2.7 发行版的发行说明。

只有 Visual Studio 2015 和 Visual Studio 2013 才支持 Azure SDK 2.7。[Azure SDK 2.6](http://azure.microsoft.com/downloads/) 是 Visual Studio 2012 支持的最后一个 SDK。

有关此版本的详细信息，请参阅 [Azure SDK 2.7 通告文章](https://azure.microsoft.com/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)。

##Visual Studio 2015 的登录改进

适用于 Visual Studio 2015 的 Azure SDK 2.7 支持 Visual Studio 2015 中的新标识管理功能。这包括支持通过基于角色的访问控制、云解决方案提供者、DreamSpark 以及其他帐户和订阅类型访问 Azure 的帐户。

只能在 Visual Studio 2015 中使用这些登录改进。将来的更新将包含对 Visual Studio 2013 的支持。


##移动 SDK

更新了 **Mobile Apps** 模板，以反映最新的 [NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/)和设置过程。

##服务总线 

一般性的 Bug 修复与改进。有关更新和功能的详细信息，请参阅最新[服务总线 NuGet](http://www.nuget.org/packages/WindowsAzure.ServiceBus/) 的发行说明。

##HDInsight 工具 

此版本做了以下更新。这些更新目前以预览版提供。有关详细信息，请参阅[此博客](http://go.microsoft.com/fwlink/?LinkId=619108)。

- 为 Tez 上的 Hive 作业绘制 Hive 图形
- 完全支持 Hive DML IntelliSense
- Pig 模板支持
- Azure 服务的 Storm 模板

###重大变化

- 使用此版本的工具时，必须先升级旧的 **Storm** 项目。有关详细信息，请参阅[此博客](http://go.microsoft.com/fwlink/?LinkId=619108)。
- 不再支持 Visual Studio Web Express。有关详细信息，请参阅[此博客](http://go.microsoft.com/fwlink/?LinkId=619108)。

##Azure App Service 工具

此版本对 Web 工具扩展做了以下更新。有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)。

- 添加了对 DreamSpark 帐户的支持
- 对 Azure 工具做了全面的更改，以支持新的 Azure 资源管理 API
- 在[云资源管理器](azure-sdk-dot-net-release-notes-2_7.md#cloud_explorer)中添加了对 Azure App Service 的支持

###已知问题

服务器资源管理器的“槽”节点下面不会出现 Web 应用部署槽节点，而云资源管理器下面不会加载 Web 应用部署槽的子节点。此问题已解决，下一个 SDK 版本将不再发生。


##<a id="cloud_explorer"></a>适用于 Visual Studio 2015 的云资源管理器

Azure SDK 2.7 包含适用于 Visual Studio 2015 的云资源管理器，可让你从 Visual Studio 中查看 Azure 资源、检查其属性，以及执行重要的开发人员操作。

云资源管理器支持以下功能：

- Azure 资源的“资源组”和“资源类型”视图 
- 按名称搜索资源（可在“资源类型”视图中使用）
- 支持应用了基于角色的访问控制 (RBAC) 的订阅和资源 
- 集成式“操作”面板，针对特定选择的资源显示以开发人员为焦点的操作。例如：附加使用资源管理器堆栈创建的虚拟机的远程调试器、查看虚拟机的诊断数据，等等。
- 集成式“属性”面板，显示以开发人员为焦点，在开发/测试期间经常需要的属性 
- 快速切换枚举资源时要使用的帐户（使用工具栏上的“设置”命令） 
- 筛选枚举资源时要使用的订阅（使用工具栏上的“设置”命令） 
- Azure Preview 门户中用于管理资源与资源组的深层链接 
 
 
##Azure 资源管理器工具 

Azure 资源管理器工具已更新为使用基于角色的访问控制 (RBAC) 和新的订阅类型。除了经典存储之外，这些更改还附带了使用新存储帐户在部署期间存储项目的功能。

如果在 SDK 2.7 中使用旧版 SDK 中的 Azure 资源组，则需要使用新的存储帐户而不是经典存储帐户来部署新的部署脚本。在你更改项目以添加新的脚本之前，系统会发出提示。旧脚本将重命名，你需要手动修改新的脚本。
 
 
##存储资源管理器工具 

- 支持查看附加 Blob。有关详细信息，请参阅[此博客文章](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/04/13/introducing-azure-storage-append-blob.aspx)。 
- 支持通过服务器资源管理器查看高级存储帐户。服务器资源管理器只显示高级存储帐户的页 Blob，因为它们是高级存储帐户唯一支持的类型。

##适用于 Visual Studio 的 Azure 数据工厂工具 

即将引入适用于 Visual Studio 的 **Azure 数据工厂工具**。以下是启用的功能。有关详细信息，请参阅[此博客](http://go.microsoft.com/fwlink/?LinkId=617530)。

- **基于模板的创作**：选择基于用例的模板、数据移动模板或数据处理模板，以部署端到端数据集成解决方案，以及快速开始使用数据工厂。 
- **与解决方案资源管理器集成，方便创作和部署数据工厂实体**：创建和部署管道与相关实体作为 Visual Studio 项目。 
- **与图表视图集成，以便在创作时进行可视互动**：借助图表视图，以可视方式创作管道和数据集。 
- **与服务器资源管理器集成，方便浏览已部署的实体并与之互动**：利用服务器资源管理器浏览已部署的数据工厂和相应的实体。将已部署的数据工厂或任何实体（管道、链接的服务、数据集）导入你的项目。 
- **使用架构验证与丰富智能感知进行 JSON 编辑**：使用丰富智能感知与架构验证，有效地配置和编辑数据工厂实体的 JSON 文档 
- **多环境发布**：通过为每个环境创建不同的配置文件，将创作的管道发布到开发、测试或生产环境。
- **支持基于 Pig、Hive 和 .Net 的数据处理**：支持在数据工厂项目中使用 Pig 和 Hive 脚本。支持引用 C# 项目来管理 .Net 活动。

##另请参阅

[Azure SDK 2.7 通告文章](https://azure.microsoft.com/blog/2015/07/20/announcing-the-azure-sdk-2-7-for-net/)

<!---HONumber=66-->