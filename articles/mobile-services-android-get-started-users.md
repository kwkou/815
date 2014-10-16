<properties linkid="develop-mobile-tutorials-get-started-with-users-android" urlDisplayName="Get Started with Authentication" pageTitle="Get started with authentication (Android) | Mobile Dev Center" metaKeywords="" description="Learn how to use Mobile Services to authenticate users of your Android app through a variety of identity providers, including Google, Facebook, Twitter, and Microsoft." metaCanonical="" services="" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="ricksal" solutions="" manager="" editor="" />

# 移动服务中的身份验证入门

<div class="dev-center-tutorial-selector sublanding">   
<a href="/en-us/develop/mobile/tutorials/get-started-with-users-dotnet" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/en-us/develop/mobile/tutorials/get-started-with-users-js" title="Windows 应用商店 JavaScript">Windows 应用商店 JavaScript</a><a href="/en-us/develop/mobile/tutorials/get-started-with-users-wp8" title="Windows Phone">Windows Phone</a><a href="/en-us/develop/mobile/tutorials/get-started-with-users-ios" title="iOS">iOS</a><a href="/en-us/develop/mobile/tutorials/get-started-with-users-android" title="Android" class="current">Android</a><a href="/en-us/develop/mobile/tutorials/get-started-with-users-html" title="HTML" class="current">HTML</a><a href="/en-us/develop/mobile/tutorials/get-started-with-users-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a><a href="/en-us/develop/mobile/tutorials/get-started-with-users-xamarin-android" title="Xamarin.Android" class="current">Xamarin.Android</a></div>

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">

<p>本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。</p>
</div>

<div class="dev-onpage-video-wrapper"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Android-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/mobile-android-get-started-authentication-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Android-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a><span class="time">10:42</span></div>

</div>

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1.  [注册应用程序以进行身份验证并配置移动服务][注册应用程序以进行身份验证并配置移动服务]
2.  [将表权限限制给已经过身份验证的用户][将表权限限制给已经过身份验证的用户]
3.  [向应用程序添加身份验证][向应用程序添加身份验证]

本教程基于移动服务快速入门。因此，你还必须先完成[移动服务入门][移动服务入门]教程。

完成本教程需要 Eclipse 和 Android 4.2 或更高版本。

## <a name="register"></a><span class="short-header">注册应用程序</span>注册应用程序以进行身份验证并配置移动服务

为了能够对用户进行身份验证，你必须通过标识提供程序注册你的应用程序。然后，你需要向移动服务注册标识提供程序生成的客户端密钥。

1.  登录到 [Azure 管理门户][Azure 管理门户]，单击“移动服务”，然后单击你的移动服务。

    ![][]

2.  单击“仪表板”选项卡，记下**站点 URL** 值。

    ![][1]

    注册你的应用程序时，可能需要向标识提供程序提供此值。

3.  从以下列表中选择支持的标识提供程序，并按步骤向该标识提供程序注册你的应用程序：

-   [Microsoft 帐户][Microsoft 帐户]
-   [Facebook 登录][Facebook 登录]
-   [Twitter 登录][Twitter 登录]
-   [Google 登录][Google 登录]
-   [Azure Active Directory][Azure Active Directory]

    请记住，要记下标识提供程序生成的客户端标识和密钥值。

    <div class="dev-callout"><b>安全说明</b>
<p>标识提供程序生成的密钥是一个重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。</p>
</div>

1.  回到管理门户中，单击“标识”选项卡，输入从标识提供程序获取的应用程序标识符和共享密钥值，然后单击“保存”。

    ![][2]

你的移动服务和应用程序现已配置为使用你选择的身份验证提供程序。

## <a name="permissions"></a><span class="short-header">限制权限</span>将权限限制给已经过身份验证的用户

1.  在管理门户中，单击“数据”选项卡，然后单击“TodoItem”表。

    ![][3]

2.  单击“权限”选项卡，将所有权限设置为“仅经过身份验证的用户”，然后单击“保存”。这样可以确保对 **TodoItem** 表的所有操作都要求用户经过身份验证。这样还可简化下一个教程中的脚本，因为它们无需再允许匿名用户。

    ![][4]

3.  在 Eclipse 中，打开你在完成[移动服务入门][移动服务入门]教程后创建的项目。

4.  然后，从“运行”菜单中单击“运行”以启动应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。

    发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 *TodoItem* 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a><span class="short-header">添加身份验证</span>向应用程序添加身份验证

1.  在 Eclipse 的包资源管理器中打开 ToDoActivity.java 文件，然后添加以下 import 语句。

        import com.microsoft.windowsazure.mobileservices.MobileServiceUser;
        import com.microsoft.windowsazure.mobileservices.MobileServiceAuthenticationProvider;
        import com.microsoft.windowsazure.mobileservices.UserAuthenticationCallback;

2.  将以下方法添加到 **ToDoActivity** 类：

        private void authenticate() {

            // Login using the Google provider.
            mClient.login(MobileServiceAuthenticationProvider.Google,
                    new UserAuthenticationCallback() {

                        @Override
                        public void onCompleted(MobileServiceUser user,
                                Exception exception, ServiceFilterResponse response) {

                            if (exception == null) {
                                createAndShowDialog(String.format(
                                                "You are now logged in - %1$2s",
                                                user.getUserId()), "Success");
                                createTable();
                            } else {
                                createAndShowDialog("You must log in. Login Required", "Error");
                            }
                        }
                    });
        }

    这将会创建一个用于处理身份验证过程的新方法。将使用 Google 登录对用户进行身份验证。此时将出现一个对话框，其中显示了已经过身份验证的用户的 ID。如果未正常完成身份验证，你将无法继续操作。

    <div class="dev-callout"><b>说明</b>
<p>如果使用的标识提供程序不是 Google，请将传递给上述 <strong>login</strong> 方法的值更改为下列其中一项：<em>MicrosoftAccount</em>、<em>Facebook</em>、<em>Twitter</em> 或 <em>windowsazureactivedirectory</em>。</p>
</div>

3.  在 **onCreate** 方法中，在实例化`MobileServiceClient` 对象的代码后面添加以下代码行。

        authenticate();

    此调用启动身份验证过程。

4.  将 `authenticate();` （在 **onCreate** 方法中）后面的剩余代码移到新的 **createTable** 方法，如下所示：

        private void createTable() {

            // Get the Mobile Service Table instance to use
            mToDoTable = mClient.getTable(ToDoItem.class);

            mTextNewToDo = (EditText) findViewById(R.id.textNewToDo);

            // Create an adapter to bind the items with the view
            mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);
            ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
            listViewToDo.setAdapter(mAdapter);

            // Load the items from the Mobile Service
            refreshItemsFromTable();
        }

5.  然后，从“运行”菜单中单击“运行”以启动应用程序，并使用所选的标识提供程序登录。

    当你成功登录时，应用程序应该运行而不出现错误，你应该能够查询移动服务，并对数据进行更新。

## <a name="next-steps"></a>后续步骤

在下一教程[使用脚本为用户授权][使用脚本为用户授权]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

<!-- Anchors. -->  

<!-- URLs. -->

  [Windows 应用商店 C\#]: /en-us/develop/mobile/tutorials/get-started-with-users-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /en-us/develop/mobile/tutorials/get-started-with-users-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /en-us/develop/mobile/tutorials/get-started-with-users-wp8 "Windows Phone"
  [iOS]: /en-us/develop/mobile/tutorials/get-started-with-users-ios "iOS"
  [Android]: /en-us/develop/mobile/tutorials/get-started-with-users-android "Android"
  [HTML]: /en-us/develop/mobile/tutorials/get-started-with-users-html "HTML"
  [Xamarin.iOS]: /en-us/develop/mobile/tutorials/get-started-with-users-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /en-us/develop/mobile/tutorials/get-started-with-users-xamarin-android "Xamarin.Android"
  [观看教程]: http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Android-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services
  [注册应用程序以进行身份验证并配置移动服务]: #register
  [将表权限限制给已经过身份验证的用户]: #permissions
  [向应用程序添加身份验证]: #add-authentication
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started-android
  [Azure 管理门户]: https://manage.windowsazure.cn/

  [Microsoft 帐户]: /zh-cn/develop/mobile/how-to-guides/register-for-microsoft-authentication/
  [Facebook 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-facebook-authentication/
  [Twitter 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-twitter-authentication/
  [Google 登录]: /zh-cn/develop/mobile/how-to-guides/register-for-google-authentication/
  [Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/

<!-- Images. -->
  []: ./media/mobile-services-android-get-started-users/mobile-services-selection.png
  [1]: ./media/mobile-services-android-get-started-users/mobile-service-uri.png

  [2]: ./media/mobile-services-android-get-started-users/mobile-identity-tab.png
  [3]: ./media/mobile-services-android-get-started-users/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-android-get-started-users/mobile-portal-change-table-perms.png
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-android
