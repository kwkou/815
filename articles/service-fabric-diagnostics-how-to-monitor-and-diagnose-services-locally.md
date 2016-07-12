<properties
   pageTitle="在本地监视和诊断使用 Azure Service Fabric 编写的服务 | Azure"
   description="了解如何监视和诊断本地开发计算机上使用 Microsoft Azure Service Fabric 编写的服务。"
   services="service-fabric"
   documentationCenter=".net"
   authors="kunaldsingh"
   manager="samgeo"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/30/2016"
   wacn.date="07/04/2016"/>


# 在本地计算机开发安装过程中监视和诊断服务
监视、检测、诊断和故障排除允许服务继续运行，同时对用户体验造成最小中断。虽然监视和诊断在实际部署的生产环境中相当重要，但是效力将取决于在服务开发过程中采用类似的模型，以便在你移到实际安装时能确保其正常工作。Service Fabric 使服务开发人员能够轻松实现可以跨单个计算机本地开发安装和实际生产群集安装无缝工作的诊断。

## Windows 事件跟踪的优势
[Windows 事件跟踪](https://msdn.microsoft.com/zh-cn/library/windows/desktop/bb968803.aspx) (ETW) 是推荐的用于跟踪 Service Fabric 中的消息的技术。这是因为：

* **ETW 速度很快。** 它是作为一种对代码执行时间影响最小的跟踪技术而构建的。
* **ETW 跟踪可以跨本地开发环境和实际群集安装无缝工作。** 这意味着当你准备好将你的代码部署到实际群集时，无需重新编写跟踪代码。
* **Service Fabric 系统代码也使用 ETW 进行内部跟踪。** 这可让你查看与 Service Fabric 系统跟踪交错的应用程序跟踪。同时帮助你更轻松地了解基础系统中应用程序代码与事件之间的序列和相互关系。
* **Service Fabric Visual Studio 工具中有内置支持，可以查看 ETW 事件。**


## 在 Visual Studio 中查看 Service Fabric 系统事件

Service Fabric 发出 ETW 事件以帮助应用程序开发人员了解平台中发生的情况。如果你还没有这么做，请继续遵循[在 Visual Studio 中创建第一个应用程序](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)中的步骤。此信息将帮助你启动应用程序，并运行可显示跟踪消息的诊断事件查看器。

1. 如果诊断事件窗口未自动显示，请在 Visual Studio 中转到“视图”选项卡上，选择“其他窗口”，然后选择“诊断事件查看器”。

2. 每个事件都有标准元数据信息，它将告诉你事件来自的节点、应用程序和服务。你也可以使用事件窗口顶端的“筛选事件”框来筛选事件列表。例如，你可以根据“节点名称”或“服务名称”进行筛选。 而当你查看事件详细信息时，你也可以使用事件窗口顶端的“暂停”按钮来暂停，稍后再继续，而不会丢失任何事件。

  ![Visual Studio 诊断事件查看器](./media/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/DiagEventsExamples2.png)

## 将你自己的自定义跟踪添加到应用程序代码
Service Fabric Visual Studio 项目模板包含示例代码。该代码演示如何添加自定义应用程序代码 ETW 跟踪（与来自 Service Fabric 的系统跟踪一起显示在 Visual Studio ETW 查看器中）。此方法的优点是元数据将自动添加到跟踪中，并已配置了 Visual Studio 诊断事件查看器来显示它们。

对于从**服务模板**（无状态或有状态）创建的项目，只要搜索 `RunAsync` 实现即可：

1. 对 `RunAsync` 方法中 `ServiceEventSource.Current.ServiceMessage` 的调用显示了应用程序代码中的自定义 ETW 跟踪的一个示例。
2. 在 **ServiceEventSource.cs** 文件中，你会找到 `ServiceEventSource.ServiceMessage` 方法的重载，由于性能方面的原因，应该将其用于高频率事件。

对于从**Actor 模板**（无状态或有状态）创建的项目：

1. 打开 **"ProjectName".cs** 文件，其中，ProjectName 是为你的 Visual Studio 项目选择的名称。  
2. 在 DoWorkAsync 方法中查找代码 `ActorEventSource.Current.ActorMessage(this, "Doing Work");`。这是根据应用程序代码编写的自定义 ETW 跟踪的一个示例。  
3. 在 **ActorEventSource.cs** 文件中，你会找到 `ActorEventSource.ActorMessage` 方法的重载，由于性能方面的原因，应该将其用于高频率事件。

将自定义 ETW 跟踪添加到你的服务代码之后，可以再次生成、部署和运行应用程序以在诊断事件查看器中查看你的事件。如果使用 **F5** 调试应用程序，则将自动打开诊断事件查看器。

## 后续步骤
你添加到上述应用程序用于进行本地诊断的相同跟踪代码将与工具一起工作，在 Azure 群集上运行应用程序时，你可以使用这些工具查看这些事件。请查看以下文章，其中介绍了不同的工具选项，以及如何设置这些选项。

* [如何使用 Azure 诊断收集日志](/documentation/articles/service-fabric-diagnostics-how-to-setup-wad/)  
* [Using ElasticSearch as a Service Fabric application trace store（将 ElasticSearch 用作 Service Fabric 应用程序跟踪存储）](/documentation/articles/service-fabric-diagnostic-how-to-use-elasticsearch/)

<!---HONumber=Mooncake_0425_2016-->