<properties linkid="develop-mobile-tutorials-create-pull-notifications-dotnet" urlDisplayName="Define a custom API that supports pull notifications" pageTitle="Define a custom API that supports pull notifications - Azure Mobile Services" metaKeywords="" description="Learn how to Define a custom API that supports periodic notifications in Windows Store apps that use Azure Mobile Services." metaCanonical="" services="" documentationCenter="" title="Define a custom API that supports periodic notifications" authors="glenga" solutions="" manager="" editor="" />

# 定义支持定期通知的自定义 API

<div class="dev-center-tutorial-selector"> 
	<a href="/zh-cn/develop/mobile/tutorials/create-pull-notifications-dotnet" title="Windows Store C#" class="current">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/create-pull-notifications-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
</div>

本主题说明如何使用自定义 API 在 Windows 应用商店应用程序中支持定期通知。启用定期通知后，Windows 将定期访问你的自定义 API 终结点，并使用返回的、采用磁贴特定格式的 XML 来更新开始菜单中的应用程序磁贴。有关详细信息，请参阅[定期通知][]。

需要将此功能添加到你在完成[移动服务入门][]或[数据处理入门][]教程后创建的应用程序。为此，你需要完成以下步骤：

1.  [定义自定义 API][]
2.  [更新应用程序以启用定期通知][]
3.  [测试应用程序][]

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]或[数据处理入门][]。

<a name="define-custom-api"></a>
## 定义自定义 API

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][]

2.  单击“API” 选项卡，然后单击“创建自定义 API” 。

    ![][1]

    这将显示“创建新的自定义 API” 对话框。

3.  将“获取权限” 更改为“任何人” ，在“API 名称” 中键入 *tiles*，然后单击勾选按钮。

    ![][2]

    这将创建具有公共 GET 访问权限的新 API。

4.  单击 API 表中新的磁贴条目。

    ![][3]

5.  单击“脚本” 选项卡，将现有代码替换为以下代码：

        exports.get = function(request, response) {
        var wns = require('wns');
        var todoItems = request.service.tables.getTable('TodoItem');
        todoItems.where({
        complete:false
        }).read({
        success:sendResponse
            });

        function sendResponse(results) {
        var tileText = {
        text1:"My todo list"
                };
        var i = 0;
        console.log(results)
        results.forEach(function(item) {
        tileText["text" + (i + 2)] = item.text;
        i++;
                });
        var xml = wns.createTileSquareText01(tileText);
        response.set('content-type', 'application/xml');
        response.send(200, xml);
            }
        };

    此代码将返回 TodoItem 表中前 3 个未完成的项目，然后将它们加载到传递给 "wns"."createTileSquareText01" 函数的 JSON 对象中。此函数返回以下磁贴模板 XML：

        <tile>
        <visual>
        <binding template="TileSquareText01">
        <text id="1">My todo list</text>
        <text id="2">Task 1</text>
        <text id="3">Task 2</text>
        <text id="4">Task 3</text>
        </binding>
        </visual>
        </tile>

    使用 "exports.get" 函数是因为客户端将发送 GET 请求以访问磁贴模板。

   	<div class="dev-callout"><b>说明</b>

    <p>此自定义 API 脚本使用 Node.js <a href="http://go.microsoft.com/fwlink/p/?LinkId=306750">wns 模块</a>，后者通过 <b>require</b> 函数进行引用。此模块不同于用于从服务器脚本发送推送通知的<a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554217.aspx">推送对象</a>返回的 <a href="http://go.microsoft.com/fwlink/p/?LinkId=260591">wns 对象</a>。</p>
	</div>

接下来，你将修改快速入门应用程序以启动定期通知，这些通知通过请求新的自定义 API 来更新动态磁贴。

<a name="update-app"></a>
## 更新应用程序更新应用程序以启用定期通知

1.  在 Visual Studio 中，按 F5 键运行前一教程中的快速入门应用程序。

2.  确保至少显示一个项目。如果没有任何项目，请在“插入 TodoItem” 中键入文本，然后单击“保存” 。

3.  在 Visual Studio 中，打开 App.xaml.cs 项目文件并添加以下 using 语句：

        using Windows.UI.Notifications;

4.  在 "OnLaunched" 事件处理程序中添加以下代码：

        TileUpdateManager.CreateTileUpdaterForApplication().StartPeriodicUpdate(
        new System.Uri(MobileService.ApplicationUri, "/api/tiles"),
        PeriodicUpdateRecurrence.Hour
        );

    此代码启用定期通知，以通过新的"磁贴"自定义 API 请求磁贴模板数据。选择与你的数据更新频率最匹配的 [PeriodicUpdateRecurrance] 值。

<a name="test-app"></a>
## 测试应用程序

1.  在 Visual Studio 中，按 F5 键再次运行应用程序。

    这将启用定期通知。

2.  导航到“开始”屏幕，找到应用程序的动态磁贴，并注意现在项目数据已显示在磁贴中。

    ![][4]

## 后续步骤

现在，你已创建定期通知，建议你了解有关以下移动服务主题的详细信息：

-   [推送通知入门][]
    定期通知由 Windows 管理，并且仅按预定义的时间表发生。推送通知可以由移动服务按需发送，并且可以是 toast 通知、磁贴通知和原始通知。

-   [移动服务服务器脚本参考][]
    了解有关创建自定义 API 的详细信息。

-   [移动服务 .NET 操作方法概念性参考][]
    了解有关如何将移动服务与 .NET 一起使用的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/create-pull-notifications-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/create-pull-notifications-js "Windows 应用商店 JavaScript"
  [定期通知]: http://msdn.microsoft.com/zh-cn/library/windows/apps/jj150587.aspx
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started/#create-new-service
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-dotnet
  [定义自定义 API]: #define-custom-api
  [更新应用程序以启用定期通知]: #update-app
  [测试应用程序]: #test-app
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-windows-store-dotnet-create-pull-notifications/mobile-services-selection.png
  [1]: ./media/mobile-services-windows-store-dotnet-create-pull-notifications/mobile-custom-api-create.png
  [2]: ./media/mobile-services-windows-store-dotnet-create-pull-notifications/mobile-custom-api-create-dialog.png
  [3]: ./media/mobile-services-windows-store-dotnet-create-pull-notifications/mobile-custom-api-select.png
  [wns 模块]: http://go.microsoft.com/fwlink/p/?LinkId=306750
  [推送对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554217.aspx
  [wns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=260591
  [4]: ./media/mobile-services-windows-store-dotnet-create-pull-notifications/mobile-custom-api-live-tile.png
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [移动服务 .NET 操作方法概念性参考]: /zh-cn/develop/mobile/how-to-guides/work-with-net-client-library
