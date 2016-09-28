<!-- deleted in Global -->

本主题说明如何创建一个同时包含移动客户端和 Web 客户端的应用。你将创建一个移动应用和一个网站，并为两者使用相同的基础数据库。

首先，你将创建一个新的移动应用后端以及一个简单的*待办事项列表*应用，此应用将应用数据存储在新的移动应用后端。移动应用后端对服务器端业务逻辑使用支持的 .NET 语言。客户端应用可以使用移动应用支持的任何客户端平台，包括 iOS、Windows、Xamarin iOS 和 Xamarin Android。

然后，你将使用与移动应用相同的数据库创建一个网站。在本教程结束时，你将拥有一个使用相同数据的 Web 客户端和一个移动客户端。

>[AZURE.NOTE] 若要在注册 Azure 帐户前开始使用 Azure 网站，请转至[试用 Azure 网站](https://tryappservice.azure.com/)，在此处，可立即在 Azure 网站中创建临时初学者网站。你不需要使用信用卡，也不需要做出承诺。

## 创建新的移动应用后端和客户端

* 遵循[创建移动应用]教程中的步骤创建一个移动应用后端和一个客户端。你可以使用移动应用支持的任何客户端平台，包括 iOS、Windows、Xamarin iOS 和 Xamarin Android。

* 确保已将移动应用后端部署到 Azure，并将移动客户端应用程序连接到托管的后端。移动应用代码项目使用 Entity Framework Code First，并在移动客户端应用的第一个 REST 请求之后初始化数据库。

## 从 Visual Studio 发布 TodoList Web API

在本部分中，你将使用示例网站方案创建一个新的网站。你将修改示例以使用与移动应用相同的数据库架构名称和连接字符串。

>[AZURE.NOTE] 完成这些步骤之前，请确保已将客户端连接到移动应用后端数据库以初始化该数据库。否则，网站将无法连接到同一数据库。

1. 在 [Azure 门户预览](https://portal.azure.cn/)中，使用与移动应用相同的资源组和托管计划创建新的网站。

2. 下载示例解决方案 [MultiChannelToDo] 并在 Visual Studio 中打开。该解决方案包含一个 Web API 项目和一个适用于 Web 客户端 UI 的网站项目。

3. 在 Web API 项目中，编辑 MultiChannelToDoContext.cs。在 `OnModelCreating` 中，将架构名称更新为与移动应用名称相同：

        modelBuilder.HasDefaultSchema("your_mobile_app"); // your service name, replacing dashes with underscore

4. 接下来，我们将从 Azure 门户获取移动应用连接字符串：

    - 在门户中选择你的移动应用，然后单击标有“用户代码”的部分。

    - 在打开的边栏选项卡中，选择“配置”，然后选择“应用程序设置”。

    - 在“连接字符串”下，单击“显示连接字符串”。复制 **MS\_TableConnectionString** 设置的值。这是移动应用用来连接 SQL 数据库的连接字符串。

5. 在 Visual Studio 中，右键单击 Web API 项目并选择“发布”。选择“Azure 网站”作为发布目标，然后选择前面创建的网站。一直单击“下一步”，直到显示“发布 Web”向导的“设置”部分为止。

6. 在“数据库”部分中，粘贴移动应用连接字符串作为 **MultiChannelToDoContext** 的值。只选中“在运行时使用此连接字符串”复选框。

7. Web API 已成功发布到 Azure 之后，你将看到确认页。复制已发布的服务的 URL。

## 从 Visual Studio 发布 TodoList Web 客户端 UI

在本部分中，你将使用以 AngularJS 实现的示例 Web 客户端应用程序。然后，你要使用 Visual Studio 将该项目发布到 Azure 中托管的新 Azure 网站。

1. 在 Visual Studio 中，打开 **MultiChannelToDo.Web** 项目。编辑文件 `js/service/ToDoService.js`，将 URL 添加到刚刚发布的 Web API 中：

        var apiPath = "https://your-web-api-site-name.chinacloudsites.cn";

2. 右键单击“MultiChannelToDo.Web”项目，然后选择“发布”。

3. 在“发布 Web”向导中，选择“Azure 网站”作为发布目标，并创建一个不包含数据库的新网站。

4. 成功发布项目之后，你将在浏览器中看到 Web UI。

## 测试移动应用和网站

1. 在 Web UI 中，添加或编辑一些待办事项。

    ![浏览器中的网站视图](./media/app-service-mobile-dotnet-backend-web-and-mobile/web-app-in-browser.png)

2. 运行你在[创建移动应用]教程中创建的移动应用。你将看到一些待办事项，它们与网站中显示的事项相同。

    ![Xamarin 移动应用的视图](./media/app-service-mobile-dotnet-backend-web-and-mobile/xamarin-ios-quickstart-device.png)

## 发生的更改
* 有关从网站更改为 Azure 网站的指南，请参阅：[Azure 网站及其对现有 Azure 服务的影响](/documentation/services/web-sites/)
* 有关从门户更改为新门户的指南，请参阅：[有关在 Azure 门户中导航的参考](https://portal.azure.cn/)

<!-- Links -->

[MultiChannelToDo]: https://github.com/Azure/mobile-services-samples/tree/web-mobile/MultiChannelToDo
[创建移动应用]: /documentation/articles/app-service-mobile-xamarin-ios-get-started/

<!---HONumber=Mooncake_0118_2016-->