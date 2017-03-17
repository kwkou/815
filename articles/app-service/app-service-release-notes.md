<properties 
   pageTitle="用于 .NET 2.5.1 的 Azure SDK 发行说明" 
   description="用于 .NET 2.5.1 的 Azure SDK 发行说明" 
   services="app-service" 
   documentationCenter=".net,nodejs,java" 
   authors="Juliako" 
   manager="erikre" 
   editor=""/>

<tags
   ms.service="app-service"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration" 
   ms.date="10/10/2016" 
   wacn.date="03/17/2017"
   ms.author="juliako"/>


# 用于 .NET 2.5.1 的 Azure SDK 发行说明

本文档包含用于 .NET 2.5.1 的 Azure SDK 版本的发行说明。

##用于 .NET 2.5.1 的 Azure SDK 发行说明

下面是用于 .NET 2.5.1 的 Azure SDK 中的新功能和更新。

- 与 **Web 工具扩展**相关的新功能\\方案。

	- Azure 网站已重命名为 Azure 应用服务。有关详细信息，请参阅 [Azure App Service and existing Azure Services](/documentation/articles/app-service-changes-existing-services/)（Azure 应用服务和现有的 Azure 服务）。
	- 增加了 Azure API 应用（预览版）支持，因此客户可以将 ASP.NET 项目作为 API 应用发布，然后在 C# 项目中使用“添加”>“Azure API 应用客户端”手势根据已部署 API 应用的结构生成代码。
	- 已弃用服务器资源管理器中的“网站”节点，代之以“Azure 应用服务”节点，该节点支持对 Azure API 应用、移动应用和 Web 应用进行以资源组为基础的分组。
	- 增加了 Azure 移动应用（预览版）支持，因此客户可以创建新的移动应用项目、添加移动应用控制器、发布项目，以及对应用程序进行远程调试。
	- “添加”>“Azure API 应用客户端”手势现在支持本地 Swagger JSON 文件，因此 Web API 开发人员可以使用第三方 NuGet（例如 Swashbuckle）生成 Swagger，或者进行手动创作。客户端开发人员可以借助这种方式通过代码生成功能在 C# 项目中使用任何 Swagger 终结点。
	- 增强了 Web 应用和 API 应用发布对话框，使之支持 Azure 门户预览的资源分组概念，而选择/创建 Azure 资源组和应用服务计划的功能也在新的 Web 应用和 API 应用预配对话框中得到了体现。
	- Azure API 应用服务器资源管理器节点提供 Azure 门户预览中 API 应用深层链接的链接，以及其他功能（例如日志流式处理和远程调试）。

	如需了解用于 .NET 2.5.1 的 Azure SDK 中存在的已知问题以及目前存在的限制，请参阅下面的[此](/documentation/articles/app-service-release-notes/#known_issues_2_5_1)部分。


- Visual Studio 中与 **HDInsight 工具**相关的新功能\\方案已在此版本中启用。
	- 对 Hive 脚本进行本地验证。单击工具栏中的“验证脚本”按钮，看脚本中是否存在任何错误。
	- 改进了对 Hive 作业的调试。现在，访问 Visual Studio 中的 Yarn 日志即可调试 Hive 作业。如果应用程序存在性能问题，查看 YARN 日志可以获取有用信息。
	- （公开预览版）针对 Hive 的关键字自动完成功能和 IntelliSense 支持。用于 Visual Studio 的 HDInsight 工具增加了针对 Hive 的关键字自动完成功能和 IntelliSense 支持，方便用户创作 Hive 脚本。
	- Storm 支持。现在可通过用于 Visual Studio 的 HDInsight 工具在 C# 中开发 Storm 拓扑/Spout/Bolt。然后即可将已开发拓扑提交到 Storm 群集，并查看拓扑/bolt/spout 状态。可以通过系统日志和客户日志对 Storm 拓扑/Bolt/Spout 进行故障排除。也可使用 Storm on HDInsight 中的现有 JAVA 资产。
	
	有关详细信息，请参阅 [用于 Visual Studio 的 HDInsight Hadoop 工具入门](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)。

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-visual-studio-login-guide.md)]

##<a id="known_issues_2_5_1"></a> 用于 .NET 2.5.1 的 Azure SDK 的已知问题和限制

- Azure API 应用以移动应用部署目标的方式呈现。在后续版本发布之前，Web 应用应该是移动应用的唯一目标。
- Azure API 应用预配可能会成功，但偶尔会无法更新“Azure 应用服务活动”窗口中的进度。解决办法是在 Azure 门户预览中查看新的 Azure API 应用的状态。
- 单击“文件”>“新建项目”>“API 应用”>“F5”时出现 HTTP 错误，因为没有 default/index.html。解决办法是手动浏览到 /api/values URL。
- 服务器资源管理器图标偶尔显示为平展样式。重新启动 VS 可以解决此问题。
- 如果在 Web 应用或 API 应用预配期间引发异常（例如出现超出配额错误或者 Azure API 应用网关名称重复），错误会显示部分原始的 JSON 文本。
- 在项目创建时查看 Application Insights 偶尔会出现项目创建问题。
- 偶然情况下，生成的 Azure API 应用客户端代码会缺少命名空间，需手动添加（或通过 Visual Studio 提示自动导入）才能对代码进行编译。
- 移动应用项目会发布到 Web 应用，但需选取选择一个已在 Azure 门户预览中作为移动应用创建的站点，因为移动应用项目需要数据库。
- 移动应用的起始页使用术语“移动服务”而非“移动应用”
- 移动应用项目的创建可能需要长达一分钟的时间。
- 某些情况下，在 API 应用预配期间会从 Azure API 返回一个错误，指出权限无法正常设置，而 API 应用已完成正常预配并可供使用。可以通过 Azure 门户预览手动设置权限。
- API 应用模板和移动应用模板不支持 Application Insights。
- API 应用项目不能与云服务项目一起使用。
- 仅 C# 提供 API 应用项目模板。
- 仅 C# 支持通过“添加 Azure API 应用客户端”上下文菜单使用 API 应用。

 

<!---HONumber=Mooncake_0919_2016-->