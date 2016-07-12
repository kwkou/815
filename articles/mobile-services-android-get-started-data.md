<properties
	pageTitle="开始使用 Android 上的数据（JavaScript 后端）| Azure"
	description="了解如何开始使用移动服务来利用 Android 应用中的数据（JavaScript 后端）。"
	services="mobile-services"
	documentationCenter="android"
	authors="RickSaling"
	manager="erikre"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="04/07/2016"
	wacn.date="05/23/2016"/>

# 将移动服务添加到现有 Android 应用（JavaScript 后端）


## 摘要

本主题说明如何使用 Azure 移动服务将持久性数据添加到 Android 应用程序。在本教程中，你将要下载一个可在内存中存储数据的应用，创建一个新的移动服务，将该应用与该移动服务相集成以便在 Azure 移动服务而不是在本地存储和更新数据，然后使用 Azure 管理门户查看运行该应用时对数据所做的更改。


本教程将帮助你详细了解 Azure 移动服务如何从 Android 应用程序存储和检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门教程中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门](/documentation/articles/mobile-services-android-get-started/)教程。

## 先决条件

若要完成本教程，您需要以下各项：

- 一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/" target="_blank">Azure 试用</a>。


- [Azure 移动服务 Android SDK]；
- [Android Studio 集成开发环境](https://developer.android.com/sdk/index.html)，其中包含 Android SDK；Android 4.2 或更高版本。下载的 GetStartedWithData 项目需要 Android 4.2 或更高版本。但是，移动服务 SDK 只需要 Android 2.2 或更高版本。

## 代码示例

若要查看完整的源代码，请转到<a href="https://github.com/Azure/mobile-services-samples/tree/master/GettingStartedWithData/AndroidStudio">此处</a>。

## 下载 GetStartedWithData 项目

###获取示例代码

[AZURE.INCLUDE [download-android-sample-code](../includes/download-android-sample-code.md)]

###检查并运行示例代码

[AZURE.INCLUDE [mobile-services-android-run-sample-code](../includes/mobile-services-android-run-sample-code.md)]

## 在 Azure 管理门户中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

## 将新表添加到移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data-2](../includes/mobile-services-create-new-service-data-2.md)]

## 更新应用程序以使用移动服务进行数据访问

[AZURE.INCLUDE [mobile-services-android-getting-started-with-data](../includes/mobile-services-android-getting-started-with-data.md)]


## 针对新移动服务测试应用程序

更新应用程序以使用移动服务作为后端存储后，便可以使用 Android 模拟器或 Android 手机针对移动服务测试该应用程序。

1. 在“运行”菜单中，单击“运行应用程序”以启动项目。

	这将执行使用 Android SDK 构建的应用，该应用使用客户端库发送一个查询，该查询从你的移动服务返回项目。

2. 和前面一样，输入有意义的文本，然后单击“添加”。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在 [Azure 管理门户](https://manage.windowsazure.cn)中，单击“移动服务”，然后单击你的移动服务。

4. 单击“数据”选项卡，然后单击“浏览”。

   	![][9]
  
   	请注意，**TodoItem** 表现在包含了数据以及由移动服务生成的某些值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

针对 Android 的**数据处理入门**教程到此结束。

## 故障排除

###验证 Android SDK 版本

[AZURE.INCLUDE [验证 SDK](../includes/mobile-services-verify-android-sdk-version.md)]

## 后续步骤

本教程演示了有关如何使 Android 应用程序处理移动服务中的数据的基础知识。请试着学习下列其他 Android 教程：

* [身份验证入门](/documentation/articles/mobile-services-android-get-started-users/)
	<br/>了解如何对应用程序用户进行身份验证。


<!-- Anchors. -->
[Download the Android app project]: #download-app
[Create the mobile service]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use Mobile Services]: #update-app
[Test the app against Mobile Services]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

[8]: ./media/mobile-services-android-get-started-data/mobile-dashboard-tab.png
[9]: ./media/mobile-services-android-get-started-data/mobile-todoitem-data-browse.png
[12]: ./media/mobile-services-android-get-started-data/mobile-eclipse-project.png
[13]: ./media/mobile-services-android-get-started-data/mobile-quickstart-startup-android.png
[14]: ./media/mobile-services-android-get-started-data/mobile-services-import-android-workspace.png
[15]: ./media/mobile-services-android-get-started-data/mobile-services-import-android-project.png


<!-- URLs. -->

[Azure 管理门户]: https://manage.windowsazure.cn/
[Azure 移动服务 Android SDK]: http://aka.ms/Iajk6q
[GitHub]: http://go.microsoft.com/fwlink/p/?LinkID=282122
[Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=280125

<!---HONumber=Mooncake_0118_2016-->