<properties 
	pageTitle="使用 Azure 移动服务开发 Android 应用程序入门" 
	description="遵照本教程开始使用 Azure 移动服务进行 Android 开发。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/18/2015" 
	wacn.date="10/03/2015"/>

# <a name="getting-started"></a>移动服务入门

[AZURE.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started-EC.md)]

本教程说明如何使用 Azure 移动服务向 Android 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单_待办事项列表_应用程序。要创建的移动服务将使用支持的 .NET 语言，你可以使用 Visual Studio 来提供服务器端业务逻辑和管理移动服务。若要创建允许以 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题的 [JavaScript 后端版本](/documentation/articles/mobile-services-android-get-started-EC)。

以下是完成的应用程序的屏幕快照：

![][0]

完成本教程需要你安装 [Android 开发人员工具][Android SDK]，其中包含 Eclipse 集成开发环境 (IDE)、Android 开发人员工具 (ADT) 插件和最新的 Android 平台。需要使用 Android 4.2 或更高版本。

下载的快速入门项目包含适用于 Android 的移动服务 SDK。尽管此项目需要 Android 4.2 或更高版本，但移动服务 SDK 只需要 Android 2.2 或更高版本。

> [AZURE.IMPORTANT]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并取得多达 10 个免费的移动服务，即使在试用期结束之后仍可继续使用这些服务。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。

## <a name="create-new-service"></a>创建新的移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## 将移动服务下载到本地计算机

在创建移动服务后，请下载可在本地计算机或虚拟机上运行的个性化移动服务项目。

1. 单击刚刚创建的移动服务，在快速入门选项卡中单击“选择平台”下的“Android”，然后展开“创建新的 Android 应用程序”。

	![][1]

2. 如果你尚未安装 Visual Studio，请下载和安装 [Visual Studio Professional 2013](https://go.microsoft.com/fwLink/p/?LinkID=391934) 或更高版本。

3. 在“下载你的服务并将其发布到云”下面单击“下载”。

	这样可以下载实现你的移动服务的 Visual Studio 项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

4. 另外，请下载发布配置文件，将下载的文件保存到本地计算机，然后记下保存位置。

## 测试移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

## 发布移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

## 创建新的 Android 应用程序

在本部分中，你将要创建一个连接到移动服务的新的 Android 应用程序。

1. 在[管理门户]中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 在快速入门选项卡中，单击“选择平台”下的“Android”，然后展开“创建新的 Android 应用程序”。
 
	![][2]

3. 在本地计算机或虚拟机上下载并安装 [Android 开发人员工具][Android SDK]（如果尚未这么做）。

4. 在“下载并运行应用程序”下面单击“下载”。

  	随即将会下载已连接到移动服务的示例_待办事项列表_应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

## 运行 Android 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1. 浏览到压缩的项目文件所保存到的位置，然后在计算机上展开这些文件。

2. 在 Eclipse 中，依次单击“文件”、“导入”，展开“Android”，单击“工作区中的现有 Android 代码”，然后单击“下一步”。

 	![][14]

3. 单击“浏览”，浏览到已展开项目文件所在的位置，单击“确定”，并确认 TodoActivity 项目已选中，然后单击“完成”。

 	![][15]

    这样可将项目文件导入当前工作区。
    
    ![][8]

4. 在“运行”菜单中，单击“运行”，以便在 Android 模拟器中启动项目。

	> [AZURE.IMPORTANT]若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。

5. 在应用程序中键入有意义的文本（例如 _Complete the tutorial_），然后单击“添加”。

   	![][10]

   	这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据被插入到 TodoItem 表。移动服务返回存储在表中的项，数据显示在列表中。

	> [AZURE.NOTE]你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 ToDoActivity.java 文件中。

<!--This shows how to run your new client app against the mobile service running in Azure. Before you can test the Android app with the mobile service running on a local computer, you must configure the Web server and firewall to allow access from your Android development computer. For more information, see [Configure the local web server to allow connections to a local mobile service](/documentation/articles/mobile-services-dotnet-backend-how-to-configure-iis-express).-->

## <a name="next-steps"></a>后续步骤
完成快速入门后，请了解如何在移动服务中执行其他重要任务：

* [身份验证入门]<br/>了解如何使用标识提供程序对应用程序的用户进行身份验证。

* [推送通知入门]<br/>了解如何向应用程序发送一条很基本的推送通知。

* [移动服务 .NET 后端故障排除]<br/>了解如何诊断和修复移动服务 .NET 后端可能会出现的问题。

<!-- Anchors. -->

[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->

[0]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-completed-android.png
[1]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-vs-EC.png
[2]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-android-EC.png


[6]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-portal-quickstart-android.png
[7]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-android.png
[8]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-eclipse-quickstart.png

[10]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-startup-android.png
[11]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-data-tab.png
[12]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-data-browse.png

[14]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-services-import-android-workspace.png
[15]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-services-import-android-project.png

<!-- URLs. -->
[Get started with data]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-data
[身份验证入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-users
[推送通知入门]: /documentation/articles/mobile-services-dotnet-backend-android-get-started-push
[Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=280125
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[移动服务 .NET 后端故障排除]: /documentation/articles/mobile-services-dotnet-backend-how-to-troubleshoot
[管理门户]: https://manage.windowsazure.cn/

<!---HONumber=71-->