<properties
	pageTitle="将移动服务添加到现有应用 (Xamarin.Android) | Windows Azure"
	description="了解如何存储数据，以及如何从 Azure 移动服务 Xamarin.Android 应用程序访问数据。"
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

此主题说明如何通过 Azure 移动服务来利用 Xamarin.Android 应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

> [AZURE.NOTE]本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 来存储数据以及从 Xamarin.Android 应用程序检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门](/develop/mobile/tutorials/get-started-xamarin-android)教程。

本教程将指导你完成以下基本步骤：

1. [下载 Xamarin.Android 应用程序项目][GitHub] 
2. [创建移动服务]
3. [添加用于存储的数据表]
4. [更新应用程序以使用移动服务]
5. [针对移动服务测试应用程序]

> [AZURE.IMPORTANT]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。

本教程需要 [Azure 移动服务组件]、[Xamarin.Android] 和 Android SDK 4.2 或更高版本。

> [AZURE.NOTE]下载的 GetStartedWithData 项目需要针对 Android 4.2 或更高版本。但是，移动服务 SDK 只需要 Android 2.2 或更高版本。

##  <a name="download-app"></a>下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序][GitHub]（一个 Xamarin.Android 应用程序）的基础上制作的。此应用的 UI 与移动服务 Android 快速入门中生成的应用相同，不过，前者的一些新增项本地存储在内存中。

1. 下载 `GetStartedWithData` 示例应用程序，然后在你的计算机上提取文件。 

2. 在 Xamarin Studio 中，依次单击“文件”、“打开”，浏览到将 GetStartedWithData 示例项目解压缩到的位置，然后选择“XamarinTodoQuickStart.Android.sln”并将其打开。

3. 找到并打开 **TodoActivity** 类

   	可以看到，有几条 `// TODO::` 注释指定了将此应用程序用于你的移动服务时必须执行的步骤。

5. 从“运行”菜单中，单击“开始执行(不调试)”，然后系统会要求你选取模拟器或已连接的 USB Android 设备。

	> [AZURE.IMPORTANT]可以使用 Android 手机或 Android 模拟器运行此项目。使用 Android 手机运行会要求你下载手机特定的 USB 驱动程序。
	> 
	> 若要在 Android 模拟器中运行该项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。

6. 在应用程序中键入有意义的文本（例如 _Complete the tutorial_），然后单击“添加”。

   	![][13]

   	请注意，保存的文本将存储在内存中的集合中，并显示在下面的列表中。

##  <a name="create-service"></a>在管理门户中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

##  <a name="add-table"></a>将新表添加到移动服务

为了能够在新移动服务中存储应用程序数据，必须先创建一个新表。

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

  	这会将移动服务 SDK 组件添加到项目。

2. 打开 **AndroidManifest.xml** 文件，并确保存在以下权限行：

		<uses-permission android:name="android.permission.INTERNET" />

  	这将使应用程序能够在 Azure 中访问移动服务。

3. 从“解决方案”窗口打开 **TodoActivity** 类，并取消注释以下代码行：

		using Microsoft.WindowsAzure.MobileServices;
 
4. 我们将删除应用当前使用的内存中列表，因此可将其替换为移动服务。在 **ToDoActivity** 类中，取消注释以下定义现有 **toDoItemList** 列表的代码行。

		public List<TodoItem> todoItemList = new ArrayList<TodoItem>();

5. 上一步操作完成后，此项目将指示生成错误。搜索剩余三个使用 `todoItemList` 变量的位置，并取消注释所指示的部分。

6. 现在，我们将添加移动服务。取消注释以下代码行：

        private MobileServiceClient client; // Mobile Service Client references
        private IMobileServiceTable<TodoItem> todoTable; // Mobile Service Table used to access data   

7. 在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务。

8. 单击“仪表板”选项卡并记下“站点 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值。

   	![][8]

  	从应用代码访问移动服务时，您需要使用这些值。

9. 在 **Constants** 类中，取消注释以下成员变量：

        public const string ApplicationURL = @"AppUrl";
        public const string ApplicationKey = @"AppKey";
        
10. 将上述变量中的 **AppUrl** 和 **AppKey** 替换为上面从管理门户检索到的值。

11. 在 **onCreate** 方法中，取消注释以下定义 **MobileServiceClient** 变量的代码行：

		// Create the Mobile Service Client instance, using the provided
		// Mobile Service URL and key
		client = new MobileServiceClient(
			Constants.ApplicationURL,
			Constants.ApplicationKey).WithFilter(filter);

		// Get the Mobile Service Table instance to use
		todoTable = client.GetTable<TodoItem>();    

  	这将创建用于访问移动服务的 MobileServiceClient 的新实例。它还将创建用于代理移动服务中的数据存储的 MobileServiceTable 实例。

12. 找到文件底部的 ProgressFilter 类，并取消其注释。当 MobileServiceClient 运行网络操作时，此类显示“正在加载”指示器。

13. 取消注释 **checkItem** 方法的以下行：

		try {
			await todoTable.UpdateAsync(item);
			if (item.Complete)
				adapter.Remove(item);
		} catch (Exception e) {
			CreateAndShowDialog(e, "Error");
		}

   	这会将项更新发送到移动服务，并从适配器中删除已选中的项。
    
14. 取消注释 **AddItem** 方法的以下行：
	
		try 
		{
			// Insert the new item
			await todoTable.InsertAsync(item);

			if (!item.Complete) 
				adapter.Add(item);			
		} 
		catch (Exception e) 
		{
			CreateAndShowDialog(e, "Error");
		}   		

  	此代码将创建一个新项目并将其插入到远程移动服务的表中。

15. 取消注释 **RefreshItemsFromTableAsync** 方法的以下行：

		try {
			// Get the items that weren't marked as completed and add them in the adapter
			var list = await todoTable.Where(item => item.Complete == false).ToListAsync ();

			adapter.Clear();

			foreach (TodoItem current in list)
				adapter.Add(current);
		} 
        catch (Exception e) 
        {
			CreateAndShowDialog(e, "Error");
		}

	这将查询移动服务，并返回未标记为“完成”的所有项。这些项目将添加到用于绑定的适配器。
		

既然此应用已更新从而将移动服务用于后端存储，就可以针对移动服务测试该应用。

##  <a name="test-app"></a>针对新移动服务测试应用程序

1. 在“运行”菜单中，单击“开始执行(不调试)”以启动项目。系统将要求你选择现有模拟器映像或已连接的 USB Android 设备。

	这将执行使用 Xamarin.Android 构建的应用程序，该应用程序使用客户端库发送一个查询，该查询从你的移动服务返回项目。

5. 和前面一样，输入有意义的文本，然后单击“添加”。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击“移动服务”，然后单击你的移动服务。

4. 单击“数据”选项卡，然后单击“浏览”。

   	![][9]
  
   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

针对 Xamarin.Android 的**数据处理入门**教程到此结束。

##  获取已完成的示例
下载[已完成的示例项目]。请务必使用你自己的 Azure 设置更新 **applicationURL** 和 **applicationKey** 变量。

##  <a name="next-steps"></a>后续步骤

本教程演示了有关如何使 Xamarin.Android 应用程序处理移动服务中的数据的基础知识。

接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

* [使用分页优化查询]
  了解如何使用查询中的分页控制单个请求中处理的数据量。

完成了数据系列教程后，请试着学习以下其他 Xamarin.Android 教程：

* [身份验证入门] 
  了解如何对应用程序用户进行身份验证。

* [推送通知入门 ] 
  了解如何使用移动服务将非常基本的推送通知发送到应用程序。

<!-- Anchors. -->

[Get the Windows Store app]: #download-app
[创建移动服务]: #create-service
[添加用于存储的数据表]: #add-table
[更新应用程序以使用移动服务]: #update-app
[针对移动服务测试应用程序]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->





[5]: ./media/partner-xamarin-mobile-services-android-get-started-data/mobile-data-tab-empty.png
[6]: ./media/partner-xamarin-mobile-services-android-get-started-data/mobile-create-todoitem-table.png
[8]: ./media/partner-xamarin-mobile-services-android-get-started-data/mobile-dashboard-tab.png
[9]: ./media/partner-xamarin-mobile-services-android-get-started-data/mobile-todoitem-data-browse.png
[13]: ./media/partner-xamarin-mobile-services-android-get-started-data/mobile-quickstart-startup-android.png

<!-- URLs. TODO:: update 'Download the Android app project' download link, 'GitHub', completed project, etc. -->
[使用脚本验证和修改数据]: /develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-android
[使用分页优化查询]: /develop/mobile/tutorials/add-paging-to-data-xamarin-android
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-xamarin-android
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-xamarin-android
[身份验证入门]: /develop/mobile/tutorials/get-started-with-users-xamarin-android
[推送通知入门 ]: /develop/mobile/tutorials/get-started-with-push-xamarin-android

[Azure Management Portal]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[Azure 移动服务组件]: http://components.xamarin.com/view/azure-mobile-services/
[Download the Android app project]: http://www.google.com/
[GitHub]: http://go.microsoft.com/fwlink/p/?LinkId=331302
[Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=280125

[已完成的示例项目]: http://go.microsoft.com/fwlink/p/?LinkId=331302

<!---HONumber=74-->