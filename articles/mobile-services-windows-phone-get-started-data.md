<properties 
	pageTitle="将移动服务添加到现有应用 (WP8) | Azure" 
	description="了解如何使用来自 Azure 移动服务 Windows Phone 8 应用程序的数据。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="11/11/2015" 
	wacn.date="05/23/2016"/>


#  将移动服务添加到现有应用程序


[AZURE.INCLUDE [mobile-services-selector-get-started-data](../includes/mobile-services-selector-get-started-data.md)]

##概述

此主题说明如何通过 Azure 移动服务来利用 Windows Phone 8 应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 [Azure 管理门户]以查看运行该应用程序时对数据所做的更改。

## 先决条件 

+ Visual Studio 2012 Express for Windows Phone 8，以及 Windows 8 上运行的 [Windows Phone 8 SDK]。若要在完成本教程后创建一个 Windows Phone 8.1 应用程序，您必须使用 Visual Studio 2013 Update 2 或更高版本。 

+ 一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)</a>。

## <a name="download-app"></a>下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序][Developer Code Samples site]的基础上制作的，该应用程序是一个 Windows Phone Silverlight 8 应用程序项目。

1. 从[开发人员代码示例站点]下载 GetStartedWithData 示例应用程序项目。 

	>[AZURE.NOTE]若要创建 Windows Phone Silverlght 8.1 应用程序，只需在下载的 Windows Phone Silverlight 8 应用程序项目中将目标操作系统更改为 Windows Phone 8.1。若要创建 Windows Phone 应用商店应用程序，请下载 GetStartedWithData 示例应用程序项目的 [Windows Phone 应用商店应用程序版本](http://go.microsoft.com/fwlink/p/?LinkId=397372)。

2. 在 Visual Studio 中打开下载的项目，然后检查 MainPage.xaml.cs 文件。

   	请注意，添加的 **TodoItem** 对象存储在内存中的 **ObservableCollection&lt;TodoItem&gt;** 内。

3. 按 **F5** 键以重新构建项目并启动此应用。

4. 在应用程序中的文本框内键入一些文本，然后单击“保存”按钮。

   	![][0]

   	可以看到，保存的文本已显示在下面的列表中。

##<a name="create-service"></a>在 Azure 管理门户中创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

## <a name="add-table"></a>将新表添加到移动服务

[AZURE.INCLUDE [mobile-services-create-new-service-data-2](../includes/mobile-services-create-new-service-data-2.md)]

## <a name="update-app"></a>更新应用程序以使用移动服务进行数据访问

将移动服务准备就绪后，您可以更新应用，以便在移动服务而不是本地集合中存储项。

1. 在 Visual Studio 的“解决方案资源管理器”中，右键单击项目名称，然后选择“管理 NuGet 包”。

2. 在左窗格中，选择“联机”类别，搜索 `WindowsAzure.MobileServices`，单击“Azure 移动服务”包对应的“安装”，然后接受许可协议。

  	![][7]

  	这会将移动服务客户端库添加到项目。

3. 在 [Azure 管理门户]中单击“移动服务”，然后单击你刚刚创建的移动服务。

4. 单击“仪表板”选项卡并记下“站点 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值。

   	![][8]

  	从应用代码访问移动服务时，您需要使用这些值。

5. 在 Visual Studio 中，打开文件 App.xaml.cs 并添加或取消注释以下 `using` 语句：

       	using Microsoft.WindowsAzure.MobileServices;

6. 在同一个文件中，取消注释以下定义 **MobileService** 变量的代码，并在 **MobileServiceClient** 构造函数中依次提供移动服务的 URL 和应用程序密钥。

		//public static MobileServiceClient MobileService = new MobileServiceClient( 
        //    "AppUrl", 
        //    "AppKey" 
        //); 

  	这将创建用于访问移动服务的 **MobileServiceClient** 的新实例。

6. 在 MainPage.cs 文件中，添加或取消注释以下 `using` 语句：

       	using Microsoft.WindowsAzure.MobileServices;
		using Newtonsoft.Json;

7. 在此 DataModel 文件夹中，将 **TodoItem** 类定义替换为以下代码：

        public class TodoItem
        {
            public string Id { get; set; }

            [JsonProperty(PropertyName = "text")]
            public string Text { get; set; }

            [JsonProperty(PropertyName = "complete")]
            public bool Complete { get; set; }
        }

7. 注释定义现有 **items** 集合的行，然后取消注释以下行：

        private MobileServiceCollection<TodoItem, TodoItem> items;
        private IMobileServiceTable<TodoItem> todoTable = 
			App.MobileService.GetTable<TodoItem>();

   	此代码将创建一个移动服务感知型绑定集合 (**items**) 和 SQL 数据库表 **TodoItem** (**todoTable**) 的代理类。

7. 在 **InsertTodoItem** 方法中，删除设置 **TodoItem**.**Id** 属性的代码行，为该方法添加 **async** 修饰符，然后取消注释以下代码行：

        await todoTable.InsertAsync(todoItem);

  	此代码在表中插入一个新项。

8. 在 **RefreshTodoItems** 方法中，为该方法添加 **async** 修饰符，然后取消注释以下代码行：

        items = await todoTable.ToCollectionAsync();

   	这将设置 todoTable 中的项目集合的绑定，其中包含从移动服务返回的所有 TodoItem 对象。

9. 在 **UpdateCheckedTodoItem** 方法中，为该方法添加 **async** 修饰符，然后取消注释以下代码行：

         await todoTable.UpdateAsync(item);

   	这会将项更新发送给移动服务。

既然此应用已更新从而将移动服务用于后端存储，就可以针对移动服务测试该应用。

## <a name="test-app"></a>针对新移动服务测试应用程序

1. 在 Visual Studio 中，按 F5 键运行应用程序。

2. 如前所述，在文本框中键入文本，然后单击“保存”。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在 [Azure 管理门户]中，单击“移动服务”，然后单击你的移动服务。

4. 单击“数据”选项卡，然后单击“浏览”。

   	![][9]
  
   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

本教程到此结束。

##  <a name="next-steps"></a>后续步骤

本教程演示了有关如何使 Windows Phone 8 应用程序处理移动服务中的数据的基础知识。建议你接下来阅读下列其他主题之一：

* [向应用程序添加身份验证](/documentation/articles/mobile-services-windows-phone-get-started-users/)
  <br/>了解如何对应用程序用户进行身份验证。

* [向应用程序添加推送通知](/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/)
  <br/>了解如何使用移动服务将非常基本的推送通知发送到应用程序。
 
* [移动服务 C# 操作方法概念性参考 ](/documentation/articles/mobile-services-dotnet-how-to-use-client-library/)
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。
 
<!-- Anchors. -->
[Download the Windows Phone 8 app project]: #download-app
[Create the mobile service]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use Mobile Services]: #update-app
[Test the app against Mobile Services]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->
[0]: ./media/mobile-services-windows-phone-get-started-data/mobile-quickstart-startup-wp8.png
[7]: ./media/mobile-services-windows-phone-get-started-data/mobile-add-nuget-package-wp.png
[8]: ./media/mobile-services-windows-phone-get-started-data/mobile-dashboard-tab.png
[9]: ./media/mobile-services-windows-phone-get-started-data/mobile-todoitem-data-browse.png

<!-- URLs. -->

[Azure 经典门户]: https://manage.windowsazure.cn/
[Windows Phone 8 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=268374
[Mobile Services SDK]: http://go.microsoft.com/fwlink/p/?LinkID=268375
[Developer Code Samples site]: http://go.microsoft.com/fwlink/p/?LinkId=271146
[开发人员代码示例站点]: http://go.microsoft.com/fwlink/p/?LinkId=271146
 

<!---HONumber=Mooncake_0118_2016-->