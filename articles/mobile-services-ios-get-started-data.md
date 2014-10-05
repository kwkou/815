<properties linkid="develop-mobile-tutorials-get-started-with-data-ios" urlDisplayName="Get Started with Data" pageTitle="Get started with data (iOS) | Mobile Dev Center" metaKeywords="Azure iOS data, Azure mobile services data, " description="Learn how to get started using Mobile Services to leverage data in your iOS app." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with data in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 移动服务中的数据处理入门

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-ios" title="iOS" class="current">iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>  
</div>	

本主题说明如何通过 Azure 移动服务来利用 iOS 应用程序中的数据。在本教程中，你将要下载一个可在内存中存储数据的应用程序，创建一个新的移动服务，将该移动服务与该应用程序相集成，然后登录到 Azure 管理门户以查看运行该应用程序时对数据所做的更改。

<div class="dev-callout"><b>说明</b>

<p>本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 来存储数据以及从 iOS 应用程序检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成<a href="/zh-cn/develop/mobile/tutorials/get-started-ios"移动服务入门</a>教程。</p>
</div>

本教程将指导你完成以下基本步骤：

1.  [下载 iOS 应用程序项目][]
2.  [创建移动服务][]
3.  [添加用于存储的数据表][]
4.  [更新应用程序以使用移动服务][]
5.  [针对移动服务测试应用程序][]

本教程需要安装[移动服务 iOS SDK][]、[XCode 4.5][] 和 iOS 5.0 或更高版本。

<div class="dev-callout"><b>说明</b>

<p>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A756A2826&amp;returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-with-data-ios%2F" target="_blank">Azure 免费试用</a>。</p>
</div>

<a name="download-app"></a>
## 下载项目下载 GetStartedWithData 项目

本教程是在 [GetStartedWithData 应用程序][]（一个 iOS 应用程序）的基础上制作的。此应用程序的 UI 与移动服务 iOS 快速入门中生成的应用程序相同，不过，前者的一些新增项本地存储在内存中。

1.  下载 GetStartedWithData [示例应用程序][GetStartedWithData 应用程序]。

2.  在 Xcode 中打开下载的项目，然后检查 QSTodoService.m 文件。

    注意，有八条 "// TODO" 注释指定了将此应用程序用于你的移动服务时必须执行的步骤。

3.  按“运行”按钮（或按 Command+R 键）重新生成项目并启动应用程序 。

4.  在应用程序中的文本框内键入一些文本，然后单击“+”按钮 。

    ![][]

    可以看到，保存的文本已显示在下面的列表中。

<a name="create-service"></a>
## 创建移动服务在管理门户中创建新的移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-data][]]

<a name="add-table"></a>
## 添加新表将新表添加到移动服务

[WACOM.INCLUDE [mobile-services-create-new-service-data-2][]]

<a name="update-app"></a>
## 更新应用程序更新应用程序以使用移动服务进行数据访问

将移动服务准备就绪后，你可以更新应用程序，以便在移动服务而不是本地集合中存储项。

1.  如果你尚未安装[移动服务 iOS SDK][]，现在请安装它。

2.  在 Xcode 中的项目导航器中，打开 Quickstart 文件夹中的 TodoService.m 和 TodoService.h 文件，然后添加以下 import 语句：

        #import <WindowsAzureMobileServices/WindowsAzureMobileServices.h>  

3.  在 ToDoService.h 文件中，找到以下注释代码行：

        // Create an MSClient property comment in the #interface declaration for the TodoService. 

    在此注释后面添加以下代码行：

        @property (nonatomic, strong)   MSClient *client;

    这会创建一个属性，该属性表示与服务连接的 MSClient

4.  在 TodoService.m 文件中，找到以下注释代码行：

        // Create an MSTable property for your items. 

    在此注释的后面，在 @interface 声明中添加以下代码行：

        @property (nonatomic, strong)   MSTable *table;

    这将会创建移动服务表的属性表示形式。

5.  在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

6.  单击“仪表板”选项卡并记下“站点 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值 。

    ![][1]

    从应用程序代码访问移动服务时，你需要使用这些值。

7.  返回 Xcode，打开 TodoService.m 并找到以下注释代码行：

        // Initialize the Mobile Service client with your URL and key.

    在此注释后面添加以下代码行：

        self.client = [MSClient clientWithApplicationURLString:@"APPURL" applicationKey:@"APPKEY"];

    这将会创建移动服务客户端的实例。

8.  将此代码中的 "APPURL" 和 "APPKEY" 值替换为你在执行步骤 6 时从移动服务获取的 URL 和应用程序密钥。

9.  找到以下注释代码行：

        // Create an MSTable instance to allow us to work with the TodoItem table.

    在此注释后面添加以下代码行：

        self.table = [self.client tableWithName:@"TodoItem"];

    这将会创建 TodoItem 表实例。

10. 找到以下注释代码行：

        // Create a predicate that finds items where complete is false comment in the refreshDataOnSuccess method. 

    在此注释后面添加以下代码行：

        NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];

    这将会创建一个查询，用于返回尚未完成的所有任务。

11. 找到以下注释代码行：

        // Query the TodoItem table and update the items property with the results from the service.

    将该注释和后面的 "completion" 块调用替换为以下代码：

        // Query the TodoItem table and update the items property with the results from the service
        [self.table readWithPredicate:predicate completion:^(NSArray *results, NSInteger totalCount, NSError *error) 
        {
        self.items = [results mutableCopy];
        completion();
        }]; 

12. 找到 "addItem" 方法，并将方法的正文替换为以下代码：

        // Insert the item into the TodoItem table and add to the items array on completion
        [self.table insert:item completion:^(NSDictionary *result, NSError *error) {
        NSUInteger index = [items count];
        [(NSMutableArray *)items insertObject:item atIndex:index];

        // Let the caller know that we finished
        completion(index);
        }];

    此代码会将一个插入请求发送到移动服务。

13. 找到 "completeItem" 方法，并将方法的正文替换为以下代码：

        // Update the item in the TodoItem table and remove from the items array on completion
        [self.table update:mutable completion:^(NSDictionary *item, NSError *error) {

        // TODO
        // Get a fresh index in case the list has changed
        NSUInteger index = [items indexOfObjectIdenticalTo:mutable];

        [mutableItems removeObjectAtIndex:index];

        // Let the caller know that we have finished
        completion(index);
        }]; 

    此代码将删除标记为已完成的 TodoItem。

更新应用程序以使用移动服务作为后端存储后，便可以针对移动服务测试该应用程序。

<a name="test-app"></a>
## 测试应用程序针对新的移动服务测试应用程序

1.  在 Xcode 中，选择要部署到的模拟器（iPhone 或 iPad），按“运行”按钮（或 Command+R 键）以重新生成项目并启动应用程序 。

    这样可以执行你的 Azure 移动服务客户端，它使用 iOS SDK 构建，可查询来自你的移动服务的项。

2.  如前所述，在文本框中键入文本，然后单击“+”按钮 。

    此时会将一个新项作为 insert 发送到移动服务。

3.  在[管理门户][]中单击“移动服务”，然后单击你的移动服务 。

4.  单击“数据”选项卡，然后单击“浏览” 。

    ![][2]

    可以看到，"TodoItem" 表现在包含了数据以及移动服务生成的 ID 值，并且已在该表中自动添加了列，以匹配应用程序中的 TodoItem 类。

针对 iOS 的"数据处理入门"教程到此结束。

<a name="next-steps"> </a>
## 后续步骤

本教程演示了有关如何使 iOS 应用程序处理移动服务中的数据的基础知识。

接下来，建议你完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

-   [使用脚本验证和修改数据][]
    了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

-   [使用分页优化查询][]
    了解如何使用查询中的分页控制单个请求中处理的数据量。

完成了数据系列教程后，请试着学习以下其他 iOS 教程：

-   [身份验证入门][]
    了解如何对应用程序用户进行身份验证。

-   [推送通知入门][]
    了解如何使用移动服务将非常基本的推送通知发送到应用程序。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/get-started-with-data-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/get-started-with-data-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-data-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/get-started-with-data-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/get-started-with-data-html "HTML"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-android "Xamarin.Android"
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-ios
  [下载 iOS 应用程序项目]: #download-app
  [创建移动服务]: #create-service
  [添加用于存储的数据表]: #add-table
  [更新应用程序以使用移动服务]: #update-app
  [针对移动服务测试应用程序]: #test-app
  [移动服务 iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
  [XCode 4.5]: https://go.microsoft.com/fwLink/p/?LinkID=266532
  [Azure 免费试用]: http://www.windowsazure.com/zh-cn/pricing/free-trial/?WT.mc_id=A756A2826&returnurl=http%3A%2F%2Fwww.windowsazure.com%2Fen-us%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-with-data-ios%2F
  [GetStartedWithData 应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=268622
  []: ./media/mobile-services-ios-get-started-data/mobile-quickstart-startup-ios.png
  [mobile-services-create-new-service-data]: ../includes/mobile-services-create-new-service-data.md
  [mobile-services-create-new-service-data-2]: ../includes/mobile-services-create-new-service-data-2.md
  [1]: ./media/mobile-services-ios-get-started-data/mobile-dashboard-tab.png
  [管理门户]: https://manage.windowsazure.cn/
  [2]: ./media/mobile-services-ios-get-started-data/mobile-todoitem-data-browse.png
  [使用脚本验证和修改数据]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-ios
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-ios
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-ios
