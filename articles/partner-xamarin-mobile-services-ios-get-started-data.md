<properties
	pageTitle="将移动服务添加到现有应用 (Xamarin.iOS) | Windows Azure"
	description="了解如何存储数据，以及如何从 Azure 移动服务 Xamarin.iOS 应用程序访问数据。"
	documentationCenter="xamarin"
	authors="ggailey777"
	manager="dwrede"
	services="mobile-services"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="08/18/2015"
	wacn.date="10/22/2015"/>

#  将移动服务添加到现有应用程序

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

此主题说明如何通过 Azure 移动服务来利用 Xamarin.iOS 应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

> [AZURE.NOTE]本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 来存储数据以及从 Xamarin.iOS 应用程序检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门](/documentation/articles/partner-xamarin-mobile-services-ios-get-started)教程。

本教程将指导你完成以下基本步骤：

1. [下载 Xamarin.iOS 应用程序项目][GitHub] 
2. [创建移动服务]
3. [添加用于存储的数据表]
4. [更新应用程序以使用移动服务]
5. [针对移动服务测试应用程序]

本教程需要安装 [Azure 移动服务组件]、[XCode 6.0][Install Xcode]、[Xamarin.iOS] 和 iOS 7.0 或更高版本。

> [AZURE.IMPORTANT]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial target="\_blank)。

##  <a name="download-app"></a>下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序][GitHub]（一个 Xamarin.iOS 应用程序）的基础上制作的。此应用程序的 UI 与移动服务 Xamarin.iOS 快速入门中生成的应用程序相同，不过，前者的一些新增项本地存储在内存中。

1. 下载 [GetStartedWithData 应用程序][GitHub]。 

2. 在 Xamarin Studio 中，打开下载的项目并检查 **TodoService** 类。

   	可以看到，有几条 **// TODO** 注释指定了将此应用程序用于你的移动服务时必须执行的步骤。

3. 转到“运行”菜单，选择“开始执行(不调试)”以启动应用程序。

4. 在应用程序中的文本框内键入一些文本，然后单击“+”按钮。

   	![][0]

   	可以看到，保存的文本已显示在下面的列表中。

##  <a name="create-service"></a>在管理门户中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

##  <a name="add-table"></a>将新表添加到移动服务

为了能够在新移动服务中存储应用数据，必须先在关联的 SQL 数据库实例中创建一个新表。

1. 在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 单击“数据”选项卡，然后单击“+创建”。

   	![][5]

   	此时将显示“创建新表”对话框。

3. 在“表名”中键入 _TodoItem_，然后单击勾选按钮。

  	![][6]

  	这将创建一个新的设置了默认权限的存储表 **TodoItem**，这意味着任何应用程序用户均可访问和更改该表中的数据。

   > [AZURE.NOTE]移动服务快速入门中使用了相同的表名。但是，每个表是在特定于给定移动服务的架构中创建的。这是为了防止当多个移动服务使用同一数据库时发生数据冲突。

4. 单击新的 **TodoItem** 表，然后验证是否不存在任何数据行。

5. 单击“列”选项卡，并验证是否只有一个“ID”列，该列是自动为你创建的。

	  这是对移动服务中的表的最低要求。

    > [AZURE.NOTE]如果在移动服务中启用了动态架构，则通过插入或更新操作向移动服务发送 JSON 对象时，将自动创建新列。

现在，您可以将新移动服务用作应用的数据存储。

##  <a name="update-app"></a>更新应用程序以使用移动服务进行数据访问

将移动服务准备就绪后，您可以更新应用，以便在移动服务而不是本地集合中存储项。

1. 如果“Components”文件夹中尚未列出“Azure 移动服务”，则可以通过右键单击“组件”，选择“获取更多组件”，然后搜索“Azure 移动服务”来获取该服务。

2. 在 Xamarin Studio 的“解决方案”视图中，打开 **TodoService** 类并取消注释以下 `using` 语句：

        using Microsoft.WindowsAzure.MobileServices;
               
3. 在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务。

4. 单击“仪表板”选项卡并记下“站点 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值。

   	![][8]

5. 在 **Constants** 类中，取消注释以下常量：

        public const string ApplicationURL = @"AppUrl";
        public const string ApplicationKey = @"AppKey";

6. 将上述常量中的 **AppUrl** 和 **AppKey** 替换为上面在管理门户中找到的值。

7. 在 **TodoService** 类中，取消注释以下代码行：

        private MobileServiceClient client;

   	这会创建一个属性，该属性表示与服务连接的 MobileServiceClient

8. 在 **TodoService** 类中，取消注释以下代码行：

        private IMobileServiceTable<TodoItem> todoTable;  

   	这将会创建移动服务表的属性表示形式。

9. 取消注释 **TodoService** 构造函数中的以下行：

		// TODO:: Uncomment these lines to use Mobile Services
		client = new MobileServiceClient(Constants.ApplicationURL, Constants.ApplicationKey).WithFilter(this); 
		todoTable = client.GetTable<TodoItem>(); 

    这将创建移动服务客户端的一个实例，并创建 TodoItem 表实例。

10. 取消注释 **TodoService** 的 **RefreshDataAsync** 方法中的以下行

			// TODO:: Uncomment these lines to use Mobile Services
			try
	    {
				// This code refreshes the entries in the list view by querying the TodoItems table.
				// The query excludes completed TodoItems
				Items = await todoTable
					.Where (todoItem => todoItem.Complete == false).ToListAsync();
			}
	    catch (MobileServiceInvalidOperationException e)
	    {
				Console.Error.WriteLine (@"ERROR {0}", e.Message);
				return null;
			}

    这将会创建一个查询，用于返回尚未完成的所有任务。

11. 找到 **InsertTodoItemAsync** 方法，并取消注释以下行：

		await todoTable.InsertAsync(todoItem);
		
    此代码会将一个插入请求发送到移动服务。

12. 找到 **CompleteItemAsync** 方法，并取消注释以下行：

		await todoTable.UpdateAsync(item);
		
   	此代码将删除标记为已完成的 TodoItem。

既然此应用已更新从而将移动服务用于后端存储，就可以针对移动服务测试该应用。

##  <a name="test-app"></a>针对新移动服务测试应用程序

1. 在 Xamarin Studio 中，从某个主组合框选择要部署到的模拟器或设备，然后转到“运行”菜单并选择“开始执行(不调试)”以启动应用程序。

   	这样可以执行你的 Azure 移动服务客户端，它使用 Xamarin.iOS 构建，可查询来自你的移动服务的项。

2. 如前所述，在文本框中键入文本，然后单击“+”按钮。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击“移动服务”，然后单击你的移动服务。

4. 单击“数据”选项卡，然后单击“浏览”。

   	![][9]
  
   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

针对 Xamarin.iOS 的**数据处理入门**教程到此结束。

##  获取已完成的示例
下载[已完成的示例项目]。请务必使用你自己的 Azure 设置更新 **applicationURL** 和 **applicationKey** 变量。

##  <a name="next-steps"></a>后续步骤

本教程演示了有关如何使 iOS 应用程序处理移动服务中的数据的基础知识。

接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  <br/>了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

* [使用分页优化查询]
  <br/>了解如何使用查询中的分页控制单个请求中处理的数据量。

完成了数据系列教程后，请试着学习以下其他 iOS 教程：

* [身份验证入门]
  <br/>了解如何对应用程序用户进行身份验证。

* [推送通知入门]
  <br/>了解如何使用移动服务将非常基本的推送通知发送到应用程序。

<!-- Anchors. -->

[Get the Windows Store app]: #download-app
[创建移动服务]: #create-service
[添加用于存储的数据表]: #add-table
[更新应用程序以使用移动服务]: #update-app
[针对移动服务测试应用程序]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

[0]: ./media/partner-xamarin-mobile-services-ios-get-started-data/mobile-quickstart-startup-ios.png




[5]: ./media/partner-xamarin-mobile-services-ios-get-started-data/mobile-data-tab-empty.png
[6]: ./media/partner-xamarin-mobile-services-ios-get-started-data/mobile-create-todoitem-table.png
[8]: ./media/partner-xamarin-mobile-services-ios-get-started-data/mobile-dashboard-tab.png
[9]: ./media/partner-xamarin-mobile-services-ios-get-started-data/mobile-todoitem-data-browse.png


[使用脚本验证和修改数据]: /develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-ios
[使用分页优化查询]: /develop/mobile/tutorials/add-paging-to-data-xamarin-ios
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-xamarin-ios
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-xamarin-ios
[身份验证入门]: /develop/mobile/tutorials/get-started-with-users-xamarin-ios
[推送通知入门]: /develop/mobile/tutorials/get-started-with-push-xamarin-ios

[Azure Management Portal]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[GitHub]: http://go.microsoft.com/fwlink/p/?LinkId=331302
[GitHub repo]: http://go.microsoft.com/fwlink/p/?LinkId=268784
[Azure 移动服务组件]: http://components.xamarin.com/view/azure-mobile-services/

[已完成的示例项目]: http://go.microsoft.com/fwlink/p/?LinkId=331302
[Xamarin.iOS]: http://xamarin.com/download
 

<!---HONumber=74-->