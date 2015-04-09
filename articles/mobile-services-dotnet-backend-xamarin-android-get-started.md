<properties urlDisplayName="Get Started with Mobile Services for Xamarin Android apps" pageTitle="用于 Xamarin Android 应用程序的移动服务入门 - Azure 移动服务" metaKeywords="" description="按照本教程进行操作，开始使用 Azure 移动服务进行 Xamarin Android 开发" metaCanonical="" services="" documentationCenter="Mobile" title="Get Started with Mobile Services for Xamarin Android apps" authors="donnam" solutions="" manager="dwrede" editor="mollybos" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-xamarin-android" ms.devlang="dotnet" ms.topic="article" ms.date="11/11/2014" ms.author="donnam" />

# <a name="getting-started"> </a>移动服务入门

[WACOM.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]

本教程说明如何使用 Azure 移动服务向 Xamarin Android 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单待办事项列表应用程序。要创建的移动服务将使用支持的 .NET 语言，你可以使用 Visual Studio 来提供服务器端业务逻辑和管理移动服务。若要创建允许用 JavaScript 编写服务器端业务逻辑的移动服务，请参阅本主题中的 [JavaScript 后端版本]。

>[AZURE.NOTE]本主题演示如何使用 Azure 管理门户创建新的移动服务项目。通过使用 Visual Studio 2013 Update 2，还可以向现有的 Visual Studio 解决方案添加新的移动服务项目。有关详细信息，请参阅[快速入门：添加移动服务（.NET 后端）](http://msdn.microsoft.com/library/windows/apps/dn629482.aspx)

以下是完成的应用程序的屏幕截图：

![][0]

只有在完成本教程后，才可以学习有关 Xamarin Android 应用程序的所有其他移动服务教程。 

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并获取最多 10 个免费移动服务，这些移动服务即使在你的试用期结束后也可以继续使用。有关详细信息，请参阅 <a href="/zh-cn/pricing/1rmb-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdocumentation%2Farticles%2Fmobile-services-dotnet-backend-xamarin-android-get-started" target="_blank">Azure 免费试用</a>。<br />本教程需要安装 <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Professional 2013</a>。可以使用免费试用版。

## 创建新的移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-create-new-service](../includes/mobile-services-dotnet-backend-create-new-service.md)]

## 创建新的 Xamarin Android 应用程序

创建移动服务后，你可以在管理门户中遵照一个简易的快速入门项目来创建新应用程序或修改现有应用程序，以连接到你的移动服务。 

在本部分中，你将为移动服务下载新的 Xamarin.android 应用程序和服务项目。

1. 在管理门户中单击"移动服务"，然后单击您刚刚创建的移动服务。
   
2. 在快速入门选项卡中，单击**"选择平台"**下的**"Xamarin"**，然后展开**"创建新的 Xamarin 应用程序"**。

   	![][6]

   	此时将显示三个简单步骤，描述如何创建与移动服务连接的 Xamarin Android 应用程序。

  	![][7]

3. 在本地计算机或虚拟机上下载并安装 <a href="https://go.microsoft.com/fwLink/p/?LinkID=257546" target="_blank">Visual Studio Professional 2013</a>（如果尚未这么做）。  

4. 下载并安装 [Xamarin Studio] 或 Xamarin for Visual Studio（如果尚未这样做）。

5. 在**"下载你的服务并将其发布到云"**下，选择**"Android"**并单击**"下载"**。 

  	随即将会下载一个解决方案，其中包含移动服务的项目，以及已连接到移动服务的示例待办事项列表应用程序的项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

6. 下载发布配置文件，将下载的文件保存到本地计算机，然后记下保存位置。

## 测试移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-test-local-service](../includes/mobile-services-dotnet-backend-test-local-service.md)]

## 发布移动服务

[WACOM.INCLUDE [mobile-services-dotnet-backend-publish-service](../includes/mobile-services-dotnet-backend-publish-service.md)]

## 运行 Xamarin Android 应用程序

本教程的最后一个阶段是生成和运行您的新应用。

1. 在 Visual Studio 或 Xamarin Studio 中，导航到移动服务解决方案中的客户端项目。

	![][8]

	![][9]

2. 按**"运行"**按钮生成项目并启动应用程序。系统将要求你选择模拟器或已连接的 USB 设备。 

	> [AZURE.NOTE] 若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。

3. 在应用中键入有意义的文本（例如 _Complete the tutorial_），然后单击加号 (**+**) 图标。

	![][10]

	这样可向在 Azure 中托管的新移动服务发送 POST 请求。请求中的数据将插入到 TodoItem 表中。移动服务将返回存储在表中的项，数据显示在列表中。

	> [AZURE.NOTE] 
   	> 你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 ToDoActivity.cs C# 文件中。
    
## 后续步骤
完成快速入门后，请了解如何在移动服务中执行其他重要任务： 

* [脱机数据同步入门]
  <br/>了解快速入门如何使用脱机数据同步，让应用程序响应迅速且可靠。

* [身份验证入门]
  <br/>了解如何使用标识提供程序对应用程序的用户进行身份验证。

* [推送通知入门]
  <br/>了解如何将非常简单的推送通知发送到你的应用程序。

* [排查移动服务 .NET 后端问题]
  <br/> 了解如何诊断和修复使用移动服务 .NET 后端可能会出现的问题。 

<!-- Anchors. -->
[移动服务入门]:#getting-started
[创建新的移动服务]:#create-new-service
[后续步骤]:#next-steps



<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-quickstart-completed-android.png
[6]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-portal-quickstart-xamarin.png
[7]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-quickstart-steps-xamarin-android.png
[8]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-xamarin-project-android-vs.png
[9]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-xamarin-project-android-xs.png
[10]: ./media/mobile-services-dotnet-backend-xamarin-android-get-started/mobile-quickstart-startup-android.png

<!-- URLs. -->
[脱机数据同步入门]: /zh-cn/documentation/articles/mobile-services-xamarin-android-get-started-offline-data
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started-users
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-xamarin-android-get-started-push
[Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
[移动服务 SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[管理门户]: https://manage.windowsazure.cn/
[JavaScript 后端版本]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started
[使用 Visual Studio 2012 的移动服务中的数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-data-vs2012
[排查移动服务 .NET 后端问题]: /zh-cndocumentation/articles/mobile-services-dotnet-backend-how-to-troubleshoot/


[Xamarin Studio]: http://xamarin.com/download
[Xcode]: https://go.microsoft.com/fwLink/?LinkID=266532&clcid=0x409
[用于 Windows 的 Xamarin]: https://go.microsoft.com/fwLink/?LinkID=330242&clcid=0x409
