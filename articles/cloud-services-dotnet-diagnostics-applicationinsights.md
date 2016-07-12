<properties
   pageTitle="使用 Application Insights 排查云服务问题 | Azure"
   description="了解如何通过使用 Application Insights 来处理 Azure 诊断发送的数据，以便排查云服务问题。"
   services="cloud-services"
   documentationCenter=".net"
   authors="sbtron"
   manager=""
   editor="tysonn" />
<tags
   ms.service="cloud-services"
   ms.date="12/15/2015"
   wacn.date="01/15/2016" />


# 使用 Application Insights 排查云服务问题

现在，你可以使用 [Azure SDK 2.8](/downloads) 和 Azure 诊断扩展模块 1.5 将云服务的 Azure 诊断数据直接发送到 Application Insights。可以将 Azure 诊断收集的各种日志（包括应用程序日志、Windows 事件日志、ETW 日志和性能计数器）发送到 Application Insights，并在 Application Insights 门户 UI 中将其可视化。与 Application Insights SDK 配合使用时，你可以深入分析来自应用程序和系统的指标与日志，以及来自 Azure 诊断的基础结构级别数据。
  
## 配置 Azure 诊断以将数据发送到 Application Insights

请遵循以下步骤设置云服务项目，以将 Azure 诊断数据发送到 Application Insights。

1) 在 Visual Studio 解决方案资源管理器中，右键单击某个角色，然后选择“属性”打开角色设计器
	
![解决方案资源管理器角色属性][1]

2) 在角色设计器的诊断部分下，选中“将诊断数据发送到 Application Insights”复选框

![角色设计器将诊断数据发送到 Application Insights][2]

3) 在弹出的对话框中，选择要向其发送 Azure 诊断数据的 Application Insights 资源。该对话框可让你从订阅中选择现有的 Application Insights 资源，或者为 Application Insights 资源手动指定检测键。如果你没有 Application Insights 资源，可以通过单击“创建新资源”链接创建一个，这会在浏览器窗口中打开 Azure 经典门户，你可以在其中创建 Application Insights 资源。有关创建 Application Insights 资源的详细信息，请参阅[创建新的 Application Insights 资源](/documentation/articles/app-insights-create-new-resource/)

![选择 Application Insights 资源][3]

4) 添加 Application Insights 资源后，该资源的检测键将存储为名为 **APPINSIGHTS\_INSTRUMENTATIONKEY** 的服务配置设置。你可以通过从服务配置下拉列表中选择不同的配置，并为该配置指定新的检测键，来更改每个服务配置或环境的这项配置设置。

![选择服务配置][4]
	
在发布期间，Visual Studio 将使用 **APPINSIGHTS\_INSTRUMENTATIONKEY** 配置设置来配置诊断扩展模块的相应 Application Insights 资源信息。配置设置是为不同服务配置定义不同检测键的便利方式。发布时，Visual Studio 将转换该设置，并将它插入诊断扩展模块公共配置。为了简化使用 PowerShell 配置诊断扩展的过程，Visual Studio 的程序包输出还包含公共配置 XML 以及相应的 Application Insights 检测键。公共配置文件在 Extensions 文件夹中创建，并遵循模式 PaaSDiagnostics.<RoleName>.PubConfig.xml。任何基于 PowerShell 的部署都可以使用此模式将每个配置映射到角色。

5) 启用“将诊断数据发送到 Application Insights”会自动将 Azure 诊断配置为向 Application Insights 发送 Azure 诊断代理程序收集的所有性能计数器和错误级别日志。如果你想进一步配置要将哪些数据发送到 Application Insights，必须手动编辑每个角色的 *diagnostics.wadcfgx* 文件。若要了解有关手动更新配置的详细信息，请参阅[配置 Azure 诊断以将数据发送到 Application Insights](/documentation/articles/azure-diagnostics-configure-applicationinsights/)。

将云服务配置为向 Application Insights 发送 Azure 诊断数据后，可以像平时一样将云服务部署到 Azure，并确保 Azure 诊断扩展已启用。请参阅[使用 Visual Studio 发布云服务](/documentation/articles/vs-azure-tools-publishing-a-cloud-service/)。

## 在 Application Insights 中查看 Azure 诊断数据
Azure 诊断遥测数据会显示在为云服务配置的 Application Insights 资源中。

下面说明了各种 Azure 诊断日志类型与 Application Insights 概念之间的对应：

-  性能计数器在 Application Insights 中显示为自定义指标
-  Windows 事件日志在 Application Insights 中显示为跟踪和自定义事件
-  应用程序日志、ETW 日志和任何诊断基础结构日志在 Application Insights 中显示为跟踪。

若要在 Application Insights 中查看 Azure 诊断数据，请执行以下操作：

- 使用“[指标资源管理器](/documentation/articles/app-insights-metrics-explorer/)”可视化任何自定义性能计数器，或不同类型的 Windows 事件日志事件的计数。

![指标资源管理器中的自定义指标][5]

- 使用“[搜索](/documentation/articles/app-insights-diagnostic-search/)”在 Azure 诊断发送的各种跟踪日志中搜索。例如，如果角色中有未处理的异常造成该角色崩溃和回收，该信息会显示在 *Windows 事件日志* 的 *应用程序* 通道中。你可以使用搜索功能查看 Windows 事件日志错误并获取异常的完整堆栈跟踪，以便找出问题的根本原因。 

![搜索跟踪][6]

## 后续步骤

- [将 Application Insights SDK 添加到云服务](/documentation/articles/app-insights-cloudservices/)，以便从应用程序发送有关请求、异常、依赖性和任何自定义遥测的数据。与 Azure 诊断数据相结合可以在相同的 Application Insight 资源中获取应用程序和系统的完整视图。  


<!--Image references-->
[1]: ./media/cloud-services-dotnet-diagnostics-applicationinsights/solution-explorer-properties.png
[2]: ./media/cloud-services-dotnet-diagnostics-applicationinsights/role-designer-sendtoappinsights.png
[3]: ./media/cloud-services-dotnet-diagnostics-applicationinsights/select-appinsights-resource.png
[4]: ./media/cloud-services-dotnet-diagnostics-applicationinsights/role-designer-appinsights-serviceconfig.png
[5]: ./media/cloud-services-dotnet-diagnostics-applicationinsights/metrics-explorer-custom-metrics.png
[6]: ./media/cloud-services-dotnet-diagnostics-applicationinsights/search-windowseventlog-error.png

<!---HONumber=Mooncake_0104_2016-->
