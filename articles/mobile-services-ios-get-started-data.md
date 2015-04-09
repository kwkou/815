<properties linkid="develop-mobile-tutorials-get-started-with-data-ios" urlDisplayName="数据处理入门" pageTitle="数据处理入门 (iOS) | 移动开发人员中心" metaKeywords="Azure iOS data, Azure mobile services data, " description="了解如何开始使用移动服务来利用 iOS 应用程序中的数据。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="glenga" solutions="" manager="" editor="" />



# 将移动服务添加到现有应用程序

[WACOM.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

本主题说明如何通过 Azure 移动服务来利用 iOS 应用程序中的数据。在本教程中，你将下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

> [AZURE.NOTE] 本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 来存储数据以及从 iOS 应用程序检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门](/zh-cn/documentation/articles/mobile-services-ios-get-started/)教程。

本教程将指导你完成以下基本步骤：

1. [下载 iOS 应用程序项目]
2. [创建移动服务]
3. [添加用于存储的数据表]
4. [更新应用程序以使用移动服务]
5. [针对移动服务测试应用程序]

本教程需要以下项：

+ [移动服务 iOS SDK] 和 [XCode 4.5][安装 Xcode] 以及 iOS 5.0 或更高版本。
+ Microsoft Azure 帐户。如果你没有帐户，则可以创建一个免费的试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="/zh-cn/pricing/1rmb-trial/?WT.mc_id=A756A2826&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-with-data-ios%2F" target="_blank">Azure 免费试用</a>。</p></div>

## <a name="download-app"></a>下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序][GitHub]（一个 iOS 应用）的基础上制作的。此应用程序的 UI 与移动服务 iOS 快速入门中生成的应用程序相同，不过，前者的一些新增项本地存储在内存中。

1. 下载 GetStartedWithData [示例应用程序][GitHub]。 

2. 在 Xcode 中打开下载的项目，然后检查 TodoService.m 文件。

   	注意，有八条 **// TODO** 注释指定了将此应用程序用于你的移动服务时必须执行的步骤。

3. 按"运行"按钮（或按 Command+R 键）重新生成项目并启动应用程序。

4. 在应用程序中的文本框内键入一些文本，然后单击**+**按钮。

   	![][0]  

   	可以看到，保存的文本已显示在下面的列表中。

## <a name="create-service"></a>在管理门户中创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

## <a name="add-table"></a>将新表添加到移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-data-2](../includes/mobile-services-create-new-service-data-2.md)]

## <a name="update-app"></a>更新应用程序以使用移动服务进行数据访问

[WACOM.INCLUDE [mobile-services-ios-enable-mobile-service-access](../includes/mobile-services-ios-enable-mobile-service-access.md)]

## <a name="test-app"></a>针对新移动服务测试应用程序

1. 在 Xcode 中，选择要部署到的模拟器（iPhone 或 iPad），按"运行"按钮（或 Command+R 键）以重新生成项目并启动应用程序。

   	这样可以执行你的 Azure 移动服务客户端，它使用 iOS SDK 构建，可查询来自你的移动服务的项。

2. 如前所述，在文本框中键入文本，然后单击**+**按钮。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击"移动服务"，然后单击你的移动服务。

4. 单击"数据"选项卡，然后单击"浏览"。

   	![][9]
  
   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

针对 iOS 的"数据处理入门"教程到此结束。

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使 iOS 应用程序处理移动服务中的数据的基础知识。 

接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  <br/>了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

* [使用分页优化查询]
  <br/>了解如何使用查询中的分页控制单个请求中处理的数据量。

完成了数据系列教程后，请试着学习以下其他 iOS 教程：

* [身份验证入门] 
	<br/>了解如何对应用程序的用户进行身份验证。

* [推送通知入门] 
  <br/>了解如何使用移动服务将非常基本的推送通知发送到应用程序。

<!-- Anchors. -->
[下载 iOS 应用程序项目]: #download-app
[创建移动服务]: #create-service
[添加用于存储的数据表]: #add-table
[更新应用程序以使用移动服务]: #update-app
[针对移动服务测试应用程序]: #test-app
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-ios-get-started-data/mobile-quickstart-startup-ios.png







[8]: ./media/mobile-services-ios-get-started-data/mobile-dashboard-tab.png
[9]: ./media/mobile-services-ios-get-started-data/mobile-todoitem-data-browse.png



<!-- URLs. -->
[使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-ios-add-paging-data
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-ios
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-ios
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-ios
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios

[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[安装 Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[移动服务 iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[GitHub]:  http://go.microsoft.com/fwlink/p/?LinkId=268622
[GitHub 存储库]: http://go.microsoft.com/fwlink/p/?LinkId=268784
