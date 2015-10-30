<properties
	pageTitle="Azure  批处理( Batch )开发库和工具 | Windows Azure"
	description="获取开发 Azure  批处理( Batch )应用程序时所需的库和工具"
	services="batch"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="07/24/2015"
	wacn.date="10/30/2015"/>


# Azure  批处理( Batch )开发库和工具
<p>获取这些库和工具，以开发 Azure  批处理( Batch )应用程序。

##  批处理( Batch )

+ [适用于 .NET 的 批处理( Batch )客户端库](http://www.nuget.org/packages/Azure.Batch/) (NuGet) – 开发客户端应用程序以使用 批处理( Batch )服务运行作业
+ [ 批处理( Batch )管理库](http://www.nuget.org/packages/Microsoft.Azure.Management.Batch/) (NuGet) – 管理 批处理( Batch )帐户
+ [ 批处理( Batch )资源管理器](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer) (GitHub) – 用于浏览、访问和更新 批处理( Batch )帐户中的主要元素（包括作业和任务、计算节点和池以及计算节点上的文件）的 GUI 应用程序与示例。请参阅[博客文章](http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx)。


## 批处理（ Batch ） Apps

+ [Batch Apps 云 SDK](http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps.Cloud/1.1.1-preview) (NuGet) – 使应用程序可将作业卸载到 批处理( Batch )服务
+ [Batch Apps Visual Studio 扩展](https://visualstudiogallery.msdn.microsoft.com/8b294850-a0a5-43b0-acde-57a07f17826a)（visual Studio 库）– 使用 Batch Apps 云 SDK 使 Visual Studio 中的应用程序支持云
+ [Batch Apps 客户端 SDK](http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps/2.3.0-preview) (NuGet) – 与应用程序交互，以便将作业卸载到 批处理( Batch )服务
+ [Batch Apps Python 客户端](https://github.com/Azure/azure-batch-apps-python) (GitHub) - Python 客户端模块，用来与 Batch Apps 服务中设置的应用程序交互

>[AZURE.IMPORTANT]Azure 只会以预览版形式发布 批处理（ Batch ） Apps API。你只应该针对测试或概念证明项目开发它。在将来的服务版本中，关键的批处理（ Batch ） Apps 功能将集成到 批处理( Batch ) API。

## 其他资源

+ [代码示例](https://github.com/Azure/azure-batch-samples) (GitHub)
+ [Azure  批处理( Batch )论坛](https://social.msdn.microsoft.com/forums/azure/home?forum=azurebatch)

<!--Anchors-->
[Batch]: #batch
[Batch Apps]: #batch-apps
[Additional resources]: #additional-resources

<!---HONumber=66-->