<properties linkid="develop-mobile-tutorials-authorize-users-in-scripts-xamarin-ios" urlDisplayName="Authorize Users in Scripts (Xamarin.iOS)" pageTitle="Authorize users in scripts (Xamarin.iOS) - Azure Mobile Services" metaKeywords="Azure authorizing user, Xamarin.iOS scripts authorization, authorize mobile services" description="Learn how to authorize users with scripts in your Azure Mobile Services app for Xamarin.iOS." metaCanonical="" disqusComments="1" umbracoNaviHide="1" title="Use scripts to authorize users in Mobile Services" documentationCenter="Mobile" authors="" />

# 使用脚本为移动服务的用户授权

<div class="dev-center-tutorial-selector sublanding"> 
	<a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-wp8" title="Windows Phone">Windows Phone</a><a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-ios" title="iOS">iOS</a><a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-android" title="Android">Android</a><a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-html" title="HTML">HTML</a><a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-xamarin-ios" title="Xamarin.iOS" class="current">iOS C#</a><a href="/zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-xamarin-android" title="Xamarin.Android">Android C#</a>
</div>

本主题说明如何使用服务器脚本为已经过身份验证的用户授权，使其能够从 Xamarin.iOS 应用程序访问 Azure 移动服务中的数据。在本教程中，你将要在移动服务中注册脚本，以基于经过身份验证的用户的 userId 筛选查询，确保每个用户只会看到他们自己的数据。

本教程是在移动服务快速入门和前面的[身份验证入门][]教程的基础上制作的。在开始本教程之前，必须先完成[身份验证入门][]。

<a name="register-scripts"></a>
## 注册脚本

由于快速入门应用程序将会读取和插入数据，因此你需要针对 TodoItem 表注册这些操作的脚本。

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][]

2.  单击“数据” 选项卡，然后单击 TodoItem  表。

    ![][1]

3.  单击“脚本”，然后选择“插入”操作 。

    ![][2]

4.  将现有脚本替换为以下函数，然后单击“保存” 。

        function insert(item, user, request) {
        item.userId = user.userId;    
        request.execute();
        }

    在将项插入到 TodoItem 表之前，此脚本将向该项添加一个 userId 值（已经过身份验证的用户的 ID）。

    <div class="dev-callout"><b>说明</b>

    <p>首次运行此 insert 脚本时，必须启用动态架构。启用动态架构后，移动服务在首次执行时会自动将 <b>userId</b> 列添加到 <b>TodoItem</b> 表。默认情况下，将为新的移动服务启用动态架构，将应用程序发布到 Windows 应用商店之前，应该禁用动态架构。</p>
	</div>

5.  重复步骤 3 和 4，以将现有 "Read" 操作替换为以下函数：

        function read(query, user, request) {
        query.where({ userId:user.userId });    
        request.execute();
        }

    此脚本可筛选返回的 TodoItem 对象，使得每个用户只收到他们插入的项。

## 测试应用程序

1.  在 Xamarin Studio 或 Visual Studio 中，打开你在完成[身份验证入门][]教程后修改的项目。

2.  按“运行”按钮 以生成项目，在 iPhone 模拟器中启动应用程序，然后使用你选择的标识提供者登录。

    你会发现，尽管 TodoItem 表中已存在完成前面的教程后创建的项，但并没有返回任何项。发生此情况的原因是，以前插入项时并未使用 userId 列，而现在这些列的值为 null。

3.  在应用程序中的“插入 TodoItem”内输入文本，然后单击“保存” 。

    ![][3]

    这将会在移动服务的 TodoItem 表中插入文本和 userId。由于新项包含了正确的 userId 值，因此移动服务会返回该项，并在第二列显示它。

4.  返回到[管理门户][Azure 管理门户]中的 “todoitem”表，单击“浏览” ，并检查每个新增的项现在是否具有关联的 userId 值。

5.  （可选）如果你有其他登录帐户，可以通过关闭然后重新运行应用程序，来验证用户是否只能看到他们自己的数据。显示登录凭据对话框时，请输入一个不同的登录名，然后检查在前一帐户下输入的项是否未显示。

## 后续步骤

演示身份验证操作基础知识的教程到此结束。建议你了解有关以下移动服务主题的详细信息：

-   [数据处理入门][]
    了解有关使用移动服务存储和查询数据的详细信息。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-html "HTML"
  [iOS C\#]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-xamarin-ios "Xamarin.iOS"
  [Android C\#]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-xamarin-android "Xamarin.Android"
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-xamarin-ios
  [Azure 管理门户]: https://manage.windowsazure.com/
  [0]: ./media/partner-xamarin-mobile-services-ios-authorize-users-in-scripts/mobile-services-selection.png
  [1]: ./media/partner-xamarin-mobile-services-ios-authorize-users-in-scripts/mobile-portal-data-tables.png
  [2]: ./media/partner-xamarin-mobile-services-ios-authorize-users-in-scripts/mobile-insert-script-users.png
  [3]: ./media/partner-xamarin-mobile-services-ios-authorize-users-in-scripts/mobile-quickstart-startup-ios.png
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-xamarin-ios
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-xamarin-ios
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/p/?LinkId=262293
