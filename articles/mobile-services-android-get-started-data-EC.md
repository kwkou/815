<properties 
	pageTitle="数据处理入门 (Android) | 移动开发人员中心" 
	description="了解如何开始使用移动服务来利用 Android 应用程序中的数据。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/08/2015" 
	wacn.date="10/03/2015"/>

# 将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data-EC.md)]

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">

<p>本主题说明如何通过 Azure 移动服务来利用 Android 应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。</p>


> [AZURE.NOTE]本教程旨在帮助你更好地了解如何使用移动服务并通过 Azure 来存储数据以及从 Android 应用程序检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门](/documentation/articles/get-started-android-EC)教程。
> 
> 如果要查看已完成应用程序的源代码，请转到[此处](https://github.com/RickSaling/mobile-services-samples/tree/futures/GettingStartedWithData/Android/GetStartedWithData)。

若要完成本教程，您需要以下各项：

+ 一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。 

+ [移动服务 Android SDK]；<a href="http://developer.android.com/intl/zh-cn/sdk/index.html" target="_blank">Android SDK</a>，其中包含 Eclipse 集成开发环境 (IDE) 和 Android 开发人员工具 (ADT) 插件；以及 Android 4.2 或更高版本。

> [AZURE.NOTE]本教程提供了 Android SDK 和移动服务 Android SDK 的安装说明。下载的 GetStartedWithData 项目需要 Android 4.2 或更高版本。但是，移动服务 SDK 只需要 Android 2.2 或更高版本。

<!-- -->

> [AZURE.NOTE]本教程使用最新版本的移动服务 SDK。你可以在<a href="http://go.microsoft.com/fwlink/p/?LinkID=280126">此处</a>找到旨在向后兼容的早期版本，但这些教程中包含的代码并不适用于这些版本。

## <a name="download-app"></a>下载 GetStartedWithData 项目

### 获取示例代码

[AZURE.INCLUDE [download-android-sample-code](../includes/download-android-sample-code-EC.md)]

### 验证 Android SDK 版本

[AZURE.INCLUDE [验证 SDK](../includes/mobile-services-verify-android-sdk-version-EC.md)]


### 检查并运行示例代码

[AZURE.INCLUDE [mobile-services-android-run-sample-code](../includes/mobile-services-android-run-sample-code-EC.md)]

## <a name="create-service"></a>在管理门户中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

## <a name="add-table"></a>将新表添加到移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data-2](../includes/mobile-services-create-new-service-data-2.md)]

## <a name="update-app"></a>更新应用程序以使用移动服务进行数据访问

[AZURE.INCLUDE [mobile-services-android-getting-started-with-data](../includes/mobile-services-android-getting-started-with-data-EC.md)]

## <a name="test-app"></a>针对新移动服务测试应用程序

更新应用程序以使用移动服务作为后端存储后，便可以使用 Android 模拟器或 Android 手机针对移动服务测试该应用程序。

1. 在“运行”菜单中，单击“运行”以启动项目。

	这将执行使用 Android SDK 构建的应用，该应用使用客户端库发送一个查询，该查询从你的移动服务返回项目。

2. 和前面一样，输入有意义的文本，然后单击“添加”。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击“移动服务”，然后单击你的移动服务。

4. 单击“数据”选项卡，然后单击“浏览”。

   	![][9]
  
   	请注意，**TodoItem** 表现在包含了数据以及由移动服务生成的某些值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

针对 Android 的**数据处理入门**教程到此结束。

## <a name="next-steps"></a>后续步骤

本教程演示了有关如何使 Android 应用程序处理移动服务中的数据的基础知识。

接下来，请试着学习下列其他 Android 教程：

* [身份验证入门]<br/>了解如何对应用程序用户进行身份验证。

* [推送通知入门]<br/>了解如何使用移动服务将非常基本的推送通知发送到应用程序。

<!-- Anchors. -->
[Download the Android app project]: #download-app
[Create the mobile service]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use Mobile Services]: #update-app
[Test the app against Mobile Services]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

[8]: ./media/mobile-services-android-get-started-data/mobile-dashboard-tab.png
[9]: ./media/mobile-services-android-get-started-data-EC/mobile-todoitem-data-browse.png
[12]: ./media/mobile-services-android-get-started-data/mobile-eclipse-project.png
[13]: ./media/mobile-services-android-get-started-data/mobile-quickstart-startup-android.png
[14]: ./media/mobile-services-android-get-started-data/mobile-services-import-android-workspace.png
[15]: ./media/mobile-services-android-get-started-data/mobile-services-import-android-project.png


<!-- URLs. -->
[身份验证入门]: /documentation/articles/mobile-services-android-get-started-users
[推送通知入门]: /documentation/articles/mobile-services-javascript-backend-android-get-started-push-EC
[Azure Management Portal]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[移动服务 Android SDK]: http://aka.ms/Iajk6q
[GitHub]: http://go.microsoft.com/fwlink/p/?LinkID=282122
[Android SDK]: http://developer.android.com/intl/zh-cn/sdk/index.html

<!---HONumber=71-->