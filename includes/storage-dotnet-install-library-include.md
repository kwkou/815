### 创建控制台应用程序，并获取程序集

在 Visual Studio 中创建新的控制台应用程序并安装包含 Azure 存储空间客户端库的 NuGet 包：

1. 在 Visual Studio 中，选择“文件”>“新建项目”，然后从 Visual C# 模板列表中选择“Windows”>“控制台应用程序”。
2. 提供控制台应用程序的名称，然后单击“确定”。
3. 创建项目后，在解决方案资源管理器中右键单击该项目并选择“管理 NuGet 包”。在线搜索“WindowsAzure.Storage”，然后单击“安装”以安装适用于 .NET 的 Azure 存储空间客户端库包和依赖项。

此文章中的代码示例还使用 [Microsoft Azure Configuration Manager 库](https://msdn.microsoft.com/zh-cn/library/azure/mt634646.aspx)从控制台应用程序中的 app.config 文件中检索存储连接字符串。使用 Azure Configuration Manager，你可以在运行时检索连接字符串，无论你的应用程序是在 Azure 中运行还是从桌面、移动、或 Web 应用程序中运行。

若要安装 Azure Configuration Manager 包，请在“解决方案资源管理器”中右键单击项目，然后选择“管理 NuGet 包”。在线搜索“ConfigurationManager”，然后单击“安装”以安装包。

不一定要使用 Azure Configuration Manager。还可以使用 API，如 .NET Framework 的 [ConfigurationManager 类](https://msdn.microsoft.com/zh-cn/library/system.configuration.configurationmanager.aspx)。



<!---HONumber=Mooncake_0405_2016-->