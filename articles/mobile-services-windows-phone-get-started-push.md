<properties pageTitle="推送通知 （Windows Phone） 入门 |移动开发人员中心" metaKeywords="" description="了解如何使用 Azure 移动服务向 Windows Phone 应用程序发送推送通知。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="" manager="" editor="" />



# 移动服务中的推送通知入门（旧推送）

<div class="dev-center-tutorial-selector sublanding">
    <a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push" title="Windows Store C#" >Windows Store C#</a>
    <a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
    <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push" title="Windows Phone" class="current">Windows Phone</a>
    <a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS">iOS</a>
    <a href="/zh-cn/documentation/articles/mobile-services-android-get-started-push" title="Android">Android</a>
<!--    <a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-push" title="Xamarin.iOS">Xamarin.iOS</a>
    <a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-push" title="Xamarin.Android">Xamarin.Android</a>    -->
	<a href="/zh-cn/documentation/articles/partner-appcelerator-mobile-services-javascript-backend-appcelerator-get-started-push" title="Appcelerator">Appcelerator</a>
</div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started-push/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push/"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>

本主题说明如何使用 Azure 移动服务向 Windows Phone 8 应用程序发送推送通知。 
在本教程中，你将要使用 Microsoft 推送通知服务 (MPNS) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

>[WACOM.NOTE]本主题支持 <em>现有的</em> 尚未升级的 <em>移动服务</em> 使用通知中心集成。在创建 <em>新的</em> 移动服务时，会自动启用此集成功能。有关新建移动服务，请参阅[推送通知入门](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/)。
>
>移动服务将与 Azure 通知中心集成，以支持附加的推送通知功能，如模板、多个平台和改进的规模。 <em>您应升级现有的移动服务以便在可能的情况下使用通知中心</em>。升级之后，请参阅此版本的[推送通知入门](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/)。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [创建 Registrations 表]
2. [向应用程序添加推送通知]
3. [更新脚本以发送推送通知]
4. [插入数据以接收通知]

本教程需要安装 [Visual Studio 2012 Express for Windows Phone] 或更高版本。

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门]。 

   >[WACOM.NOTE]如果你每天向每个用户发送超过 500 条的消息，则必须改用通知中心。有关详细信息，请参阅 <a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started">通知中心入门</a>。

## <a name="create-table"></a>创建新表

[WACOM.INCLUDE [mobile-services-create-new-push-table](../includes/mobile-services-create-new-push-table.md)]

<h2><a name="add-push"></a>向应用程序添加推送通知</h2>
		
1. 在 Visual Studio 中，打开项目文件 MainPage.xaml.cs 并添加以下代码，以创建新的 **Registrations** 类：

	    public class Registrations
	    {
	        public string Id { get; set; }
	
	        [JsonProperty(PropertyName = "handle")]
	        public string Handle { get; set; }
	    }
	
	Handle 属性用于存储通道 URI。

2. 打开文件 App.xaml.cs 并添加以下 using 语句：

        using Microsoft.Phone.Notification;

3. 将以下代码添加到 App.xaml.cs：
	
        public static HttpNotificationChannel CurrentChannel { get; private set; }

		private async void AcquirePushChannel()
        {
            CurrentChannel = HttpNotificationChannel.Find("MyPushChannel");

            if (CurrentChannel == null)
            {
                CurrentChannel = new HttpNotificationChannel("MyPushChannel");
                CurrentChannel.Open();
                CurrentChannel.BindToShellTile();
            }
                  
	       IMobileServiceTable<Registrations> registrationsTable = App.MobileService.GetTable<Registrations>();
	       var registration = new Registrations { Handle = CurrentChannel.ChannelUri.AbsoluteUri };
	       await registrationsTable.InsertAsync(registration);
        }

   	此代码获取并存储推送通知订阅的通道，并将其绑定到应用程序的默认磁贴上。

	<div class="dev-callout"><b>注意</b>
		<p>在本教程中，移动服务向设备发送翻转磁贴通知。当你发送 toast 通知时，必须改为对通道调用 <strong>BindToShellToast</strong> 。若要同时支持 toast 和磁贴通知，请同时调用 <strong>BindToShellTile</strong> 和  <strong>BindToShellToast</strong> </p>
	</div>
    
4. 在 App.xaml.cs 中 **Application_Launching** 事件处理程序的顶部，添加对新的 **AcquirePushChannel** 方法的以下调用：

        AcquirePushChannel();

   	这确保每次启动应用程序时都初始化 **CurrentChannel** 属性。


5.	在解决方案资源管理器中，展开**属性**，打开 WMAppManifest.xml 文件，单击**功能**选项卡，并确保选中 **ID___CAP___PUSH_NOTIFICATION** 功能。

   	![][1]

   	这样可确保你的应用程序可收到推送通知。

<h2><a name="update-scripts"></a>在管理门户中更新已注册的插入脚本</h2>

[WACOM.INCLUDE [mobile-services-update-registrations-script](../includes/mobile-services-update-registrations-script.md)]

4. 单击"TodoItem"，再单击"脚本"并选择"插入"。 

   	![][10]

3. 将 insert 函数替换为以下代码，然后单击"保存"：

	    function insert(item, user, request) {
    	    request.execute({
        	    success: function() {
            	    request.respond();
            	    sendNotifications();
        	    }
    	    });

	        function sendNotifications() {
        	    var registrationsTable = tables.getTable('Registrations');
        	    registrationsTable.read({
            	    success: function(registrations) {
                	    registrations.forEach(function(registration) {
                    	    push.mpns.sendFlipTile(registration.handle, {
                        	    title: item.text
                    	    }, {
                        	    success: function(pushResponse) {
                            	    console.log("Sent push:", pushResponse);
                        	    }
                    	    });
                	    });
            	    }
        	    });
    	    }
	    }

    此插入脚本会向 **Registrations** 表中存储的所有通道发送推送通知（包括已插入项的文本）。

<h2><a name="test"></a>在应用程序中测试推送通知</h2>

1. 在 Visual Studio 中，选择**生成**菜单上的**部署解决方案**。

2. 在模拟器中，向左轻扫以显示已安装应用程序的列表，并找到新的 **TodoList** 应用程序。

3. 点击并按住该应用程序图标，然后从上下文菜单中选择**固定到'开始'屏幕**。

  	![][2]

  	这会将名为 **TodoList** 的磁贴固定到到"开始"菜单。

4. 点击名为 **TodoList** 的磁贴以启动应用程序。 

  	![][3]

5. 在应用程序中的文本框内输入文本"hello push"，然后单击**保存**。

   	![][4]

  	此时会将一个插入请求发送到移动服务，以存储添加的项。

6. 按**按钮**按钮以返回到"开始"菜单。 

  	![][5]

  	请注意，应用程序已收到该推送通知，并且磁贴的标题现在是 **hello push**。

## <a name="next-steps"> </a>后续步骤

本教程演示了移动服务提供的基本推送通知功能。如果你的应用程序需要更高级功能，例如发送跨平台通知、基于订阅的路由或极大量的通知，请考虑为移动服务使用 Azure 通知中心。有关详细信息，请参阅下列通知中心主题之一：

+ [通知中心入门]
  <br/>了解如何在 Windows 应用商店应用程序中利用通知中心。

+ [什么是通知中心？]
	<br/>了解如何创建通知并将通知发送到多个平台上的用户。

+ [向订阅者发送通知]
	<br/>了解用户如何注册并接收其感兴趣的类别的推送通知。

<!--+ [向用户发送通知]
	<br/>了解如何通过移动服务向任何设备上的特定用户发送推送通知。

+ [向用户发送跨平台通知]
	<br/>了解如何使用模板通过移动服务发送推送通知，而无需在后端处理特定于平台的负载。
-->

建议进一步了解以下移动服务主题：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何使用 Windows 帐户对应用程序的用户进行身份验证。

* [移动服务服务器脚本参考]
  <br/>了解有关注册和使用服务器脚本的详细信息。

* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。 

<!-- Anchors. -->
[创建 Registrations 表]: #create-table
[更新脚本以发送推送通知]: #update-scripts
[向应用程序添加推送通知]: #add-push
[插入数据以接收通知]: #test
[后续步骤]:#next-steps

<!-- Images. -->
[1]: ./media/mobile-services-windows-phone-get-started-push/mobile-app-enable-push-wp8.png
[2]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push1-wp8.png
[3]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push2-wp8.png
[4]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push3-wp8.png
[5]: ./media/mobile-services-windows-phone-get-started-push/mobile-quickstart-push4-wp8.png
[10]: ./media/mobile-services-windows-phone-get-started-push/mobile-insert-script-push2.png



<!-- URLs. -->
[移动服务 SDK]: https://go.microsoft.com/fwLink/p/?LinkID=268375
[Visual Studio 2012 Express for Windows Phone]: https://go.microsoft.com/fwLink/p/?LinkID=268374
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-wp8
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-wp8
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-wp8
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-wp8
[向应用程序用户推送通知]: /zh-cn/documentation/articles/mobile-services-windows-phone-push-notifications-app-users
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-windows-phone-authorize-users-in-scripts

[Azure 管理门户]: https://manage.windowsazure.cn/
[mpns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=271130
[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/
[通知中心入门]: /zh-cn/documentation/articles/notification-hubs-windows-phone-get-started/
[什么是通知中心？]: /zh-cn/documentation/articles/notification-hubs-overview/
[向订阅者发送通知]: /zh-cn/documentation/articles/notification-hubs-windows-phone-send-breaking-news/
[向用户发送通知]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/
[向用户发送跨平台通知]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users-xplat-mobile-services/
