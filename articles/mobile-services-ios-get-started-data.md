<properties
	pageTitle="将移动服务添加到现有应用程序 (iOS) | 移动开发人员中心"
	description="了解如何开始使用移动服务来利用 iOS 应用程序中的数据。"
	services="mobile-services"
	documentationCenter="ios"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="07/01/2015"
	wacn.date="10/03/2015"/>

#  将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

在本教程中，你将要下载一个可在内存中存储数据的现有应用程序，并将其更改为可与 Azure 移动服务配合工作。

在开始本教程之前，必须先完成 [快速入门] 教程。你将要重复使用“快速入门”教程中创建的移动服务。

## <a name="download-app"></a>下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData iOS 应用程序] 的基础上制作的。除了添加的项会存储在内存中以外，该应用程序在其他方面与 [快速入门] 相同。

下载 [GetStartedWithData iOS 应用程序]。在 Xcode 中打开该项目，然后检查 **TodoService.m**。有八条 **// TODO** 注释指定了使此应用程序发生作用而要执行的步骤。

## <a name="update-app"></a>更新应用程序以使用移动服务进行数据访问

[AZURE.INCLUDE [mobile-services-ios-enable-mobile-service-access](../includes/mobile-services-ios-enable-mobile-service-access.md)]

## <a name="test-app"></a>测试应用程序

1. 在 Xcode 中，单击“运行”启动应用程序。键入文本并单击 **+**，将项添加到待办事项列表。

2. 登录到 Azure 管理门户，单击“移动服务”，然后单击你的移动服务。单击“数据”选项卡，然后单击“浏览”。可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

<!-- Anchors. -->

[Download the iOS app project]: #download-app
[Create the mobile service]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use Mobile Services]: #update-app
[Test the app against Mobile Services]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

[0]: ./media/mobile-services-ios-get-started-data/mobile-quickstart-startup-ios.png
[8]: ./media/mobile-services-ios-get-started-data/mobile-dashboard-tab.png
[9]: ./media/mobile-services-ios-get-started-data/mobile-todoitem-data-browse.png



<!-- URLs. -->

[Validate and modify data with scripts]: /documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts
[Refine queries with paging]: /documentation/articles/mobile-services-ios-add-paging-data
[Get started with Mobile Services]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-ios
[Get started with data]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-ios
[Get started with push notifications]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios
[Azure Management Portal]: https://manage.windowsazure.cn/
[Management Portal]: https://manage.windowsazure.cn/
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[GitHub]: http://go.microsoft.com/fwlink/p/?LinkId=268622
[GitHub repo]: http://go.microsoft.com/fwlink/p/?LinkId=268784
[快速入门]: /documentation/articles/mobile-services-javascript-backend-ios-get-started
[GetStartedWithData iOS 应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=268622

<!---HONumber=71-->