<properties 
	pageTitle="移动服务 .NET 后端故障排除 | Azure" 
	description="了解如何诊断和修复使用 .NET 后端的移动服务遇到的问题" 
	services="mobile-services" 
	documentationCenter="" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="mollybos"/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/07/2016" 
	wacn.date="03/28/2016"/>

# 移动服务 .NET 后端故障排除

使用移动服务进行开发通常是简单轻松的工作，但即便如此，有时还是可能会出错。本教程提供了一些技巧，以帮助你排查移动服务 .NET 后端可能出现的常见问题。

1. [HTTP 调试](#HttpDebugging)
2. [运行时调试](#RuntimeDebugging)
3. [分析诊断日志](#Logs)
4. [调试云程序集解析](#AssemblyResolution)
5. [实体框架迁移疑难解答](#EFMigrations)

<a name="HttpDebugging"></a>
## HTTP 调试

使用移动服务开发应用程序时，通常会将移动服务客户端 SDK 用于你所使用的平台（Windows 应用商店、iOS、Android 等）。但有时候，深入了解 HTTP 级别并观察网络上发生的原始调用可能会有所帮助。此方法在调试连接和序列化问题时特别有用。通过移动服务 .NET 后端，你可以将此方法与 Visual Studio 本地和远程调试搭配使用（下一部分将详细说明），以了解 HTTP 调用在调用服务代码之前所创建的路径。

你可以使用任何 HTTP 调试器来发送和检查 HTTP 流量。[Fiddler](http://www.telerik.com/fiddler) 是开发人员针对此目的而经常使用的工具。为了让开发人员的工作更轻松，移动服务捆绑了基于 Web 的 HTTP 调试器（也称为测试客户端），以减少使用外部工具的需求。当你在本地托管移动服务时，可从类似于 http://localhost:59233 的 URI 获取该移动服务；而托管在云中时，此 URI 的格式为 [http://todo-list.azure-mobile.net](http://todo-list.azure-mobile.net)。无论在何处托管服务，以下步骤都会以相同方式运行：

1. 首先，在 **Visual Studio 2013 Update 2** 或更高版本中打开移动服务服务器项目。如果你尚无此项目，可以通过依次选择“文件”、“新建”、“项目”、“云”节点和“Microsoft Azure 移动服务”模板来创建一个项目。
2. 按 **F5**，以生成并运行该项目。在起始页上，选择“试用”。 

    >[AZURE.NOTE]
    >如果服务在本地托管，则单击链接会定向至下一页。但是，如果在云中托管，则系统会提示你提供一组凭据。这是为了确保未经授权的用户无法访问你的 API 和负载的相关信息。若要查看该页，你需要使用**空白用户名**以及**应用程序密钥**（充当密码）来登录。若要获取应用程序密钥，你可以在 Azure 经典门户中导航至移动服务的“仪表板”选项卡，并选择“管理密钥”。
    >
    > ![访问帮助页时的身份验证提示][HelpPageAuth]

3. 你所查看的页面（称为“帮助页”）将显示移动服务启用的 HTTP API 完整列表。选择其中一个 API（如果你是在 Visual Studio 中开始使用移动服务项目的，则应会看见 **GET tables/TodoItem**）。

    ![帮助页][HelpPage]

4. 生成的页面将显示 API 预期的请求和响应的任何文档与 JSON 示例。选择“试用”按钮。

    ![API 的测试页][HelpPageApi]

5. 这是“测试客户端”，可让你发送 HTTP 请求以试用你的 API。选择“发送”。

    ![测试客户端][TestClient]

6. 你将在浏览器窗口中看到移动服务传回的 HTTP 响应。

    ![包含 HTTP 响应的测试客户端][TestClientResponse]

现在，你可以在 Web 浏览器中浏览移动服务公开的不同 HTTP API。

<a name="RuntimeDebugging"></a>
## 运行时调试

.NET 后端的主要功能之一是在本地调试服务代码，以及调试在云环境中实时运行的代码。执行以下步骤:

1. 在 **Visual Studio 2013 Update 2** 或更高版本中打开你要调试的移动服务项目。
2. 配置符号加载。导航至“调试”菜单，然后选择“选项和设置”。确保未选中“启用‘仅我的代码’”，并选中“启用源服务器支持”。

    ![配置符号加载][SymbolLoading]

3. 选择左边的“符号”节点，并使用 URI [http://srv.symbolsource.org/pdb/Public](http://srv.symbolsource.org/pdb/Public) 添加对 [SymbolSource] 服务器的引用。移动服务 .NET 后端的符号将以最新版本启用。

    ![配置符号服务器][SymbolServer]

4. 在代码段中要进行调试的位置设置一个断点。你可以在 Visual Studio 中移动服务项目模板随附的 **TodoItemController** 的 **GetAllTodoItems()** 方法中设置一个断点。
5. 按 **F5** 在本地调试服务。第一次加载可能会比较慢，因为 Visual Studio 必须为移动服务 .NET 后端下载符号。
6. 如前面有关 HTTP 调试的部分中所述，你可以使用测试客户端将 HTTP 请求发送到设置断点的方法中。例如，可以选择帮助页上的 **GET tables/TodoItem**，然后选择“试用”，然后选择“发送”，以将请求发送到 **GetAllTodoItems()** 方法。
7. Visual Studio 应会在你设置的断点处中断，Visual Studio 中的“调用堆栈”窗口中应会提供附有源代码的完整堆栈跟踪。

    ![命中断点][Breakpoint]

8. 现在，你可以将服务发布到 Azure，我们能够以上述方式使用调试。在“解决方案资源管理器”中，右键单击该项目并选择“发布”以发布该项目。
9. 在发布向导的“设置”选项卡上，选择“调试”配置。这可以确保随你的代码一起发布相关的调试符号。

    ![发布调试][PublishDebug]

10. 成功发布服务后，打开“服务器资源管理器”并展开“Microsoft Azure”和“移动服务”节点。视需要登录。
11. 右键单击刚刚发布到的移动服务，然后选择“附加调试器”。

    ![附加调试程序][AttachDebugger]

12. 就像在步骤 6 中一样，发送 HTTP 请求以调用你已在其中设置断点的方法。这次请使用 Azure 托管的移动服务的帮助页和测试客户端。
13. Visual Studio 将在断点处中断。

现在，当你在 Azure 中进行本地开发以及针对实时移动服务进行开发时，可以访问 Visual Studio 调试器的完整功能。

<a name="Logs"></a>
## 分析诊断日志

当移动服务处理来自客户的请求时，会生成各种有用的诊断信息，并捕获遇到的任何异常。除此之外，你还可以利用每个 [**TableController**](http://msdn.microsoft.com/library/microsoft.windowsazure.mobile.service.tables.tablecontroller.aspx) 的 [**Services**](http://msdn.microsoft.com/library/microsoft.windowsazure.mobile.service.tables.tablecontroller.services.aspx) 属性上提供的 [**Log**](http://msdn.microsoft.com/library/microsoft.windowsazure.mobile.service.apiservices.log.aspx) 属性，参考其他日志来检测控制器代码。

在本地调试时，日志显示在 Visual Studio 的“输出”窗口中。

![Visual Studio“输出”窗口中的日志][LogsOutputWindow]

将服务发布到 Azure 后，你可以在 Visual Studio 的“服务器资源管理器”中，右键单击移动服务，然后选择“查看日志”，以获取云中运行的服务实例的日志。

![Visual Studio 服务器资源管理器中的日志][LogsServerExplorer]

也可以从 Azure 经典门户中移动服务的“日志”选项卡获取相同的日志。

![登录 Azure 经典门户][LogsPortal]

<a name="AssemblyResolution"></a>
## 调试云程序集解析

当你将移动服务发布到 Azure 时，此服务将由移动服务托管环境加载，以确保能够无缝地升级和修补托管控制器代码的 HTTP 管道。这包括 [.NET 后端 NuGet 包](http://www.nuget.org/packages?q=%22mobile+services+.net+backend%22)引用的所有程序集：团队需持续更新服务才能使用这些程序集的最新版本。

有时，版本会因为引用了所需程序集的不同主要版本而发生冲突（允许不同的次要版本）。当 NuGet 提示你升级到移动服务 .NET 后端所用某个包的最新版本时，经常发生此情况。

>[AZURE.NOTE]移动服务目前只与 ASP.NET 5.1 兼容；当前不支持 ASP.NET 5.2。部署后，将 ASP.NET NuGet 包升级到 5.2.* 可能会导致错误。

如果升级了任何此类的包，当你将更新的服务发布到 Azure 时，将会看到指出冲突的警告页：

![指示程序集加载冲突的帮助页][HelpConflict]

同时，将在服务日志中记录如下所示的异常消息：

    Found conflicts between different versions of the same dependent assembly 'Microsoft.ServiceBus': 2.2.0.0, 2.3.0.0. Please change your project to use version '2.2.0.0' which is the one currently supported by the hosting environment.

此问题很容易解决：只需恢复到所需程序集的支持版本，然后重新发布服务。

<a name="EFMigrations"></a>
## 实体框架迁移故障排除

使用包含 SQL 数据库的移动服务 .NET 后端时，将以实体框架作为数据访问技术，让你查询数据库及保存对象。EF 代表开发人员处理的重要方面之一是数据库列（也称为架构）如何随着代码中指定的模型类的更改而更改。此过程称为[代码优先迁移](http://msdn.microsoft.com/zh-cn/data/jj591621)。

迁移可能很复杂，只有数据库状态与 EF 模型保持同步才能成功。有关如何处理移动服务的迁移和可能发生的错误的说明，请参阅[如何对 .NET 后端移动服务进行数据模型更改](/documentation/articles/mobile-services-dotnet-backend-how-to-use-code-first-migrations/)。

<!-- IMAGES -->

[HelpPageAuth]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/6.png
[HelpPage]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/7.png
[HelpPageApi]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/8.png
[TestClient]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/9.png
[TestClientResponse]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/10.png
[SymbolLoading]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/1.png
[SymbolServer]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/2.png
[Breakpoint]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/3.png
[PublishDebug]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/4.png
[AttachDebugger]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/5.png
[LogsOutputWindow]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/11.png
[LogsServerExplorer]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/12.png
[LogsPortal]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/13.png
[HelpConflict]: ./media/mobile-services-dotnet-backend-how-to-troubleshoot/14.png


<!-- Links -->
[SymbolSource]: http://symbolsource.org

<!---HONumber=Mooncake_0118_2016-->