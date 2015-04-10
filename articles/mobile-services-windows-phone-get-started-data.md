<properties linkid="develop-mobile-tutorials-get-started-with-data-wp8" urlDisplayName="数据处理入门" pageTitle="数据处理 (WP8) 入门 - Azure 移动服务" metaKeywords="" description="了解如何使用来自 Azure 移动服务 Windows Phone 8 应用程序的数据。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-phone" ms.devlang="dotnet" ms.topic="article" ms.date="09/19/2014" ms.author="glenga" />


# 将移动服务添加到现有应用程序

[WACOM.INCLUDE [mobile-services-selector-get-started-data-legacy](../includes/mobile-services-selector-get-started-data-legacy.md)]

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">

<p>此主题说明如何通过 Azure 移动服务来利用 Windows Phone 8 应用程序中的数据。在本教程中，您将要下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。</p>
</div>
<div class="dev-onpage-video-wrapper"><a href="http://go.microsoft.com/fwlink/?LinkID=298628" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/mobile-wp8-get-started-data-180x120.png') !important;" href="http://go.microsoft.com/fwlink/?LinkID=298628" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">12:54</span></div>
</div>

本教程将指导您完成以下基本步骤：

1. [下载 Windows Phone 8 应用程序项目] 
2. [创建移动服务]
3. [添加用于存储的数据表]
4. [更新应用程序以使用移动服务]
5. [针对移动服务测试应用程序]

本教程需要针对 Windows Phone 8 的 Visual Studio 2012 Express 和运行于 Windows 8 之上的 [Windows Phone 8 SDK]。若要在完成本教程后创建一个 Windows Phone 8.1 应用程序，您必须使用 Visual Studio 2013 Update 2 或更高版本。

>[WACOM.NOTE]若要完成本教程，您需要一个 Azure 帐户。如果没有帐户，则可创建一个免费的试用帐户，只需几分钟即可完成。有关详细信息，请参阅： <a href="/zh-cn/pricing/1rmb-trial/?WT.mc_id=A756A2826&amp;returnurl=http%3A%2F%2Fwww.windowsazure.cn%2Fzh-cn%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-with-data-wp8%2F" target="_blank">Azure 免费试用版</a>。

## <a name="download-app"></a>下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序][开发人员代码示例站点] 的基础上制作的，后者是一个 Windows Phone Silverlight 8 应用程序项目。  

1. 从 [开发人员代码示例站点] 下载 GetStartedWithData 示例应用程序项目。 

	>[WACOM.NOTE]若要创建 Windows Phone Silverlght 8.1 应用程序，只需在下载的 Windows Phone Silverlight 8 应用程序项目中将目标操作系统更改为 Windows Phone 8.1。若要创建 Windows Phone 应用商店应用程序，请下载 GetStartedWithData 示例应用程序项目的 [Windows Phone 应用商店应用程序版本](http://go.microsoft.com/fwlink/p/?LinkId=397372)。 

2. 在 Visual Studio 中打开下载的项目，然后检查 MainPage.xaml.cs 文件。

   	请注意，添加的 **TodoItem** 对象存储在内存中的 **ObservableCollection&lt;TodoItem&gt;** 内。

3. 按 **F5** 键重新生成项目并启动应用程序。

4. 在应用程序中的文本框内键入一些文本，然后单击 **保存** 按钮。

   	![][0]  

   	可以看到，保存的文本已显示在下面的列表中。

<h2><a name="create-service"></a>在管理门户中创建新移动服务</h2>

[WACOM.INCLUDE [mobile-services-create-new-service-data](../includes/mobile-services-create-new-service-data.md)]

<h2><a name="add-table"></a>将新表添加到移动服务</h2>

[WACOM.INCLUDE [mobile-services-create-new-service-data-2](../includes/mobile-services-create-new-service-data-2.md)]

<h2><a name="update-app"></a>更新应用程序以使用移动服务进行数据访问</h2>

将移动服务准备就绪后，您可以更新应用，以便在移动服务而不是本地集合中存储项。 

1. 在 Visual Studio 的"解决方案资源管理器"中，右键单击项目名称，然后选择"管理 NuGet 包"。

2. 在左窗格中，选择 **联机** 类别，搜索 `WindowsAzure.MobileServices`，在 **Azure 移动服务** 包上单击 **安装**，然后接受许可协议。

  	![][7]

  	这会将移动服务客户端库添加到项目。

3. 在管理门户中单击"移动服务"，然后单击您刚刚创建的移动服务。

4. 单击"仪表板"选项卡并记下"站点 URL"中的值，然后单击"管理密钥"并记下"应用程序密钥"中的值。

   	![][8]

  	从应用代码访问移动服务时，您需要使用这些值。

5. 在 Visual Studio 中，打开文件 App.xaml.cs 并添加或取消注释以下  `using` 语句：

       	using Microsoft.WindowsAzure.MobileServices;

6. 在这同一文件中，取消注释定义 **MobileService** 变量的代码，并在 **MobileServiceClient** 构造函数中依次提供移动服务的 URL 和应用程序密钥。

		//public static MobileServiceClient MobileService = new MobileServiceClient( 
        //    "AppUrl", 
        //    "AppKey" 
        //); 

  	这将创建用于访问移动服务的 **MobileServiceClient** 的新实例。

6. 在 MainPage.xaml.cs 文件中，添加或取消注释以下  `using` 语句：

       	using Microsoft.WindowsAzure.MobileServices;
		using Newtonsoft.Json;

7. 在这同一文件中，将 **TodoItem** 类定义替换为下面的代码：

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

   	此代码将创建一个移动服务感知型绑定集合 (**items**) 和 SQL Database 表 **TodoItem** (**todoTable**) 的代理类。 

7. 在 **InsertTodoItem** 方法中，删除设置 **TodoItem**.**Id** 属性的代码行，为该方法添加 **async** 修饰符，并取消注释下面的代码行：

        await todoTable.InsertAsync(todoItem);

  	此代码将在表中插入一个新项。

8. 在 **RefreshTodoItems** 方法中，为该方法添加 **async** 修饰符，然后取消注释下面的代码行：

        items = await todoTable.ToCollectionAsync();

   	这将设置 todoTable 中的项目集合的绑定，其中包含从移动服务返回的所有 TodoItem 对象。 

9. 在 **UpdateCheckedTodoItem** 方法中，为该方法添加 **async** 修饰符，然后取消注释下面的代码行：

         await todoTable.UpdateAsync(item);

   	这会将项目更新发送给移动服务。

更新应用程序以使用移动服务作为后端存储后，便可以针对移动服务测试该应用程序。

<h2><a name="test-app"></a>针对移动服务测试应用程序</h2>

1. 在 Visual Studio 中，按 F5 键运行应用程序。

2. 如前所述，在文本框中键入文本，然后单击 **保存**。

   	此时会将一个新项作为 insert 发送到移动服务。

3. 在[管理门户]中单击"移动服务"，然后单击您的移动服务。

4. 单击"数据"选项卡，然后单击"浏览"。

   	![][9]
  
   	可以看到，**TodoItem** 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

针对 Windows Phone 8 的 **数据处理入门教程** 到此结束。

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使 Windows Phone 8 应用程序处理移动服务中的数据的基础知识。接下来，建议你完成下列教程，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

* [使用脚本验证和修改数据]
  <br/>了解有关在移动服务中使用服务器脚本验证和更改从应用程序发送的数据的详细信息。

* [使用分页优化查询]
  <br/>了解如何在查询中使用分页来控制单个请求中处理的数据量。

完成了数据系列后，你还可以尝试以下 Windows Phone 8 教程之一：

* [身份验证入门] 
  <br/>了解如何对应用程序的用户进行身份验证。

* [推送通知入门] 
  <br/>了解如何通过移动服务将非常基本的推送通知发送到应用程序。
 
<!-- Anchors. -->
[下载 Windows Phone 8 应用程序项目]: #download-app
[创建移动服务]: #create-service
[添加用于存储的数据表]: #add-table
[更新应用程序以使用移动服务]: #update-app
[针对移动服务测试应用程序]: #test-app
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-windows-phone-get-started-data/mobile-quickstart-startup-wp8.png






[7]: ./media/mobile-services-windows-phone-get-started-data/mobile-add-nuget-package-wp.png
[8]: ./media/mobile-services-windows-phone-get-started-data/mobile-dashboard-tab.png
[9]: ./media/mobile-services-windows-phone-get-started-data/mobile-todoitem-data-browse.png



<!-- URLs. -->
[使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-phone-validate-modify-data-server-scripts
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-windows-phone-add-paging-data
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-wp8
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-wp8
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-wp8
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-wp8

[Azure 管理门户]: https://manage.windowsazure.cn/
[管理门户]: https://manage.windowsazure.cn/
[Windows Phone 8 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=268374
[移动服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=268375
[开发人员代码示例站点]:  http://go.microsoft.com/fwlink/p/?LinkId=271146
