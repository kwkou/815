<properties linkid="develop-mobile-tutorials-get-started-with-data-dotnet" urlDisplayName="Get Started with Data" pageTitle="Get started with data - Azure Mobile Services" metaKeywords="" description="Learn how to get started using data with Azure Mobile Services." metaCanonical="" disqusComments="1" umbracoNaviHide="1" title="Get started with data in Mobile Services using Visual Studio 2012" documentationCenter="Mobile" authors="" />

# 使用 Visual Studio 2012 的移动服务中的数据处理入门

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet-vs2012" title="Windows Store C#" class="current">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-js-vs2012" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android" title="Xamarin.Android">Xamarin.Android</a></div>

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>此主题说明如何通过 Azure 移动服务来利用 Windows 应用商店应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的 Visual Studio 2012 应用程序项目，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。单击右侧的剪辑可观看本教程的视频版本。</p>
</div>

<div class="dev-onpage-video-wrapper"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Windows-Store-app-Getting-Started-with-Data-Connecting-your-app-to-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/get-started-with-data-windows-store-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Windows-Store-app-Getting-Started-with-Data-Connecting-your-app-to-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">15:33</span></div>
</div>


<div class="dev-callout"><b>说明</b>

<p>本教程将向在 Visual Studio 2012 中创建的 Windows 应用商店应用程序添加移动服务功能。使用 Visual Studio 2013 包含的新功能，可以轻松将你的 Windows 应用商店应用程序连接到移动服务。有关详细信息，请参阅<a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet/">移动服务中的数据处理入门</a>。</p>
</div>

本教程将指导你完成以下基本步骤：

1.  [下载 Windows 应用商店应用程序项目][]
2.  [创建移动服务][]
3.  [添加用于存储的数据表][]
4.  [更新应用程序以使用移动服务][]
5.  [针对移动服务测试应用程序][]

<div class="dev-callout"><b>说明</b>

<p>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28" target="_blank">Azure 免费试用</a>。</p>
</div>

<a name="download-app"></a>
## 下载项目下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序][]（一个 Windows 应用商店应用程序）的基础上制作的。此应用程序的 UI 与移动服务快速入门中生成的应用程序相同，不过，前者的一些新增项本地存储在内存中。

1.  从[开发人员代码示例站点][GetStartedWithData 应用程序]下载 GetStartedWithData 示例应用程序的 C\# 版本。

    ![][]

2.  在 Visual Studio 2012 Express for Windows 8 中打开下载的项目，然后检查 MainPage.xaml.cs 文件。

    请注意添加的 "TodoItem" 对象存储在内存中的 "ObservableCollection\<TodoItem\>" 内。

3.  按 "F5" 键重新生成项目并启动应用程序。

4.  在应用程序的“插入 TodoItem”中键入一些文本 ，然后单击“保存” 。

    ![][1]

    可以看到，保存的文本已显示在“查询和更新数据”下的第二列中 。

<a name="create-service"></a>
## 创建移动服务在管理门户中创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-data][]]

<a name="add-table"></a>
## 添加新表将新表添加到移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-data-2][]]

<a name="update-app">
## 更新应用程序更新应用程序以使用移动服务进行数据访问

将移动服务准备就绪后，你可以更新应用程序，以便在移动服务而不是本地集合中存储项。

1.  在 Visual Studio 的“解决方案资源管理器” 中，右键单击项目名称，然后选择“管理 NuGet 包” 。

2.  在左窗格中，选择“联机” 类别，选择“包括预发行版” ，搜索 `WindowsAzure.MobileServices`，在“Azure 移动服务” 包上单击“安装” ，然后接受许可协议。

    ![][2]

    这会将移动服务客户端库添加到项目。

3.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

4.  单击“仪表板”选项卡并记下“站点 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值 。

    ![][3]

    从应用程序代码访问移动服务时，你需要使用这些值。

5.  在 Visual Studio 中，打开文件 App.xaml.cs 并添加或取消注释以下 `using` 语句：

        using Microsoft.WindowsAzure.MobileServices;

6.  在这同一文件中，取消注释定义 "MobileService" 变量的代码，并在 "MobileServiceClient" 构造函数中依次提供移动服务的 URL 和应用程序密钥。

        //public static MobileServiceClient MobileService = new MobileServiceClient( 
        //    "AppUrl", 
        //    "AppKey" 
        //); 

    这将创建用于访问移动服务的 "MobileServiceClient" 的新实例。

7.  在 MainPage.xaml.cs 文件中，添加或取消注释以下 `using` 语句：

        using Microsoft.WindowsAzure.MobileServices;
        using Newtonsoft.Json;

8.  在这同一文件中，将 "TodoItem" 类定义替换为下面的代码：

        public class TodoItem
        {
        public string Id { get; set; }

        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }

        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }
        }

9.  注释定义现有 "items" 集合的行，然后取消注释以下行：

        private MobileServiceCollection<TodoItem, TodoItem> items;
        private IMobileServiceTable<TodoItem> todoTable = 
        App.MobileService.GetTable<TodoItem>();

    此代码将创建一个移动服务感知型绑定集合 ("items") 和 SQL Database 表 "TodoItem" ("todoTable") 的代理类。

10. 在 "InsertTodoItem" 方法中，删除设置 "TodoItem"."Id" 属性的代码行，为该方法添加 "async" 修饰符，并取消注释下面的代码行：

        await todoTable.InsertAsync(todoItem);

    此代码将在表中插入一个新项。

11. 在 "RefreshTodoItems" 方法中，为该方法添加 "async" 修饰符，然后取消注释下面的代码行：

        items = await todoTable.ToCollectionAsync();

    这将设置 todoTable 中的项目集合的绑定，其中包含从移动服务返回的所有 TodoItem 对象。

12. 在 "UpdateCheckedTodoItem" 方法中，为该方法添加 "async" 修饰符，然后取消注释下面的代码行：

         await todoTable.UpdateAsync(item);

    这会将项目更新发送给移动服务。

更新应用程序以使用移动服务作为后端存储后，便可以针对移动服务测试该应用程序。

<a name="test-app"></a>
## 测试应用程序针对新的移动服务测试应用程序

1.  在 Visual Studio 中，按 F5 键运行应用程序。

2.  如前所述，在“插入 TodoItem” 中键入文本，然后单击“保存” 。

    此时会将一个新项作为 insert 发送到移动服务。

3.  在[管理门户][]中单击“移动服务”，然后单击你的移动服务 。

4.  单击“数据”选项卡，然后单击“浏览” 。

    ![][4]

    可以看到，"TodoItem" 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

5.  在应用程序中，选中列表中的某个项，然后返回到门户中的“浏览”选项卡并单击“刷新” 。

    可以看到，整个值已从 "false" 更改为 "true"。

6.  在 MainPage.xaml.cs 项目文件中，将现有 "RefreshTodoItems" 方法替换为用于筛选出已完成项目的以下代码：

        private async void RefreshTodoItems()
        {                       
        // This query filters out completed TodoItems. 
        items = await todoTable
        .Where(todoItem => todoItem.Complete == false)
        .ToCollectionAsync();

        ListItems.ItemsSource = items;            
        }

7.  在应用程序中，选中列表中的另一个项目，然后单击“刷新” 按钮。

    可以看到，选中的项现已从列表中消失。每次刷新都会导致往返访问移动服务，随后返回筛选的数据。

"数据处理入门"教程到此结束。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序处理移动服务中的数据的基础知识。接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

-   [使用脚本验证和修改数据][]
    了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

-   [使用分页优化查询][]
    了解如何使用查询中的分页控制单个请求中处理的数据量。

完成数据系列教程后，请试着学习下列教程之一：

-   [身份验证入门][]
    了解如何对应用程序用户进行身份验证。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet-vs2012 "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-data-js-vs2012 "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-data-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-data-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/get-started-with-data-html "HTML"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android "Xamarin.Android"
  [观看教程]: http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Windows-Store-app-Getting-Started-with-Data-Connecting-your-app-to-Windows-Azure-Mobile-Services
  [移动服务中的数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet/
  [下载 Windows 应用商店应用程序项目]: #download-app
  [创建移动服务]: #create-service
  [添加用于存储的数据表]: #add-table
  [更新应用程序以使用移动服务]: #update-app
  [针对移动服务测试应用程序]: #test-app
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=AE564AB28
  [GetStartedWithData 应用程序]: http://go.microsoft.com/fwlink/?LinkId=262308
  []: ./media/mobile-services-windows-store-dotnet-get-started-data-vs2012/mobile-data-sample-download-dotnet.png
  [1]: ./media/mobile-services-windows-store-dotnet-get-started-data-vs2012/mobile-quickstart-startup.png
  [mobile-services-create-new-service-data]: ../includes/mobile-services-create-new-service-data.md
  [mobile-services-create-new-service-data-2]: ../includes/mobile-services-create-new-service-data-2.md
  [2]: ./media/mobile-services-windows-store-dotnet-get-started-data-vs2012/mobile-add-nuget-package-dotnet.png
  [3]: ./media/mobile-services-windows-store-dotnet-get-started-data-vs2012/mobile-dashboard-tab.png
  [管理门户]: https://manage.windowsazure.cn/
  [4]: ./media/mobile-services-windows-store-dotnet-get-started-data-vs2012/mobile-todoitem-data-browse.png
  [使用脚本验证和修改数据]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-dotnet
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet-vs2012/
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library
