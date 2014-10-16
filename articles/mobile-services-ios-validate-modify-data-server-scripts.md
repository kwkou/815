<properties linkid="develop-mobile-tutorials-validate-modify-and-augment-data-ios" urlDisplayName="Validate Data" pageTitle="Use server scripts to validate and modify data (iOS) | Mobile Dev Center" metaKeywords="" description="Learn how to validate and modify data sent using server scripts from your iOS app." metaCanonical="" services="" documentationCenter="Mobile" title="Validate and modify data in Mobile Services by using server scripts" authors="" solutions="" manager="" editor="" />

# 使用服务器脚本在移动服务中验证和修改数据

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts" title="Windows Store C#">Windows 应用商店 C\#</a><a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-validate-modify-data-server-scripts" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-windows-phone-validate-modify-data-server-scripts" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/mobile-services-ios-validate-modify-data-server-scripts" title="iOS" class="current">iOS</a><a href="/zh-cn/documentation/articles/mobile-services-android-validate-modify-data-server-scripts" title="Android">Android</a><a href="/zh-cn/documentation/articles/mobile-services-html-validate-modify-data-server-scripts" title="HTML">HTML</a><a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-validate-modify-data-server-scripts" title="Xamarin.iOS">Xamarin.iOS</a><a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-validate-modify-data-server-scripts" title="Xamarin.Android">Xamarin.Android</a></div>

本主题说明如何在 Azure 移动服务中利用服务器脚本。你可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。在本教程中，你将要定义并注册用于验证和修改数据的服务器脚本。由于服务器端脚本的行为往往会影响到客户端，因此你还要更新 iOS 应用程序以利用这些新行为。

本教程将指导你完成以下基本步骤：

1.  [添加字符串长度验证][]
2.  [更新客户端以支持验证][]

本教程以前一教程[数据处理入门][]中的步骤和示例应用程序为基础。在开始本教程之前，必须先完成[数据处理入门][]。

<a name="string-length-validation"></a>
## 添加验证

验证用户提交的数据的长度总不失为一种良好做法。首先，你要注册一个脚本，用于验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][]

2.  单击“数据” 选项卡，然后单击 TodoItem  表。

    ![][1]

3.  单击“脚本”，然后选择“插入”操作 。

    ![][2]

4.  将现有脚本替换为以下函数，然后单击“保存” 。

            function insert(item, user, request) {
        if (item.text.length > 10) {
        request.respond(statusCodes.BAD_REQUEST, 'Text length must be 10 characters or less.');
        } else {
        request.execute();
            }
        }

    此脚本将检查 "text" 属性的长度，如果该长度超过 10 个字符，则发送错误响应。如果未超过 10 个字符，将调用 "execute" 方法以完成插入。

    <div class="dev-callout"><b>说明</b>

    <p>在“脚本”选项卡中，依次单击“清除”和“保存”可以删除某个已注册的脚本 。</p>
	</div>

<a name="update-client-validation"></a>
## 更新客户端

移动服务会验证数据和发送错误响应，而你则需要更新你的应用程序，使之能够处理验证后生成的错误响应。

1.  在 Xcode 中，打开你在完成[数据处理入门][]教程后修改的项目。

2.  按“运行”按钮 (Command + R) 生成项目并启动应用程序，在文本框中键入 10 个字符以上的文本，然后单击加号 ("+") 图标 。

    可以看到，由于移动服务返回了 400 响应（“错误的请求”），应用程序引发了一个未处理的错误。

3.  在 QSTodoService.m 文件中，在 "addItem" 方法中找到以下代码行：

        [self logErrorIfNotNil:error]; 

    在此代码行的后面，将 completion 块的余下部分替换为以下代码：

        BOOL goodRequest = !((error) && (error.code == MSErrorMessageErrorCode));

        // detect text validation error from service.
        if (goodRequest) // The service responded appropriately
        {
        NSUInteger index = [items count];
        [(NSMutableArray *)items insertObject:result atIndex:index];

        // Let the caller know that we finished
        completion(index);
        }
        else{

        // if there's an error that came from the service
        // log it, and popup up the returned string.
        if (error && error.code == MSErrorMessageErrorCode) {
        NSLog(@"ERROR %@", error);
        UIAlertView *av =
        [[UIAlertView alloc]
        initWithTitle:@"Request Failed"
        message:error.localizedDescription
        delegate:nil
        cancelButtonTitle:@"OK"
        otherButtonTitles:nil
                 ];
        [av show];
            }
        }

    这会将错误记录到输出窗口并显示给用户。

4.  重新生成并启动应用程序。

    ![][3]

    可以看到，错误已被处理，并且已向用户显示了错误消息。

<a name="next-steps"> </a>
## 后续步骤

现在你已完成本教程，建议你继续学习数据系列中的最后一篇教程：[使用分页优化查询][]。

在为用户授权以及发送推送通知时，也可以使用服务器脚本。有关详细信息，请参阅以下教程：

-   [使用脚本为用户授权][]
    了解如何基于某个已经过身份验证的用户的 ID 筛选数据。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-validate-modify-data-server-scripts "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-validate-modify-data-server-scripts "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-validate-modify-data-server-scripts "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-android-validate-modify-data-server-scripts "Android"
  [HTML]: /zh-cn/documentation/articles/mobile-services-html-validate-modify-data-server-scripts "HTML"
  [Xamarin.iOS]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-validate-modify-data-server-scripts "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/documentation/articles/partner-xamarin-mobile-services-android-validate-modify-data-server-scripts "Xamarin.Android"
  [添加字符串长度验证]: #string-length-validation
  [更新客户端以支持验证]: #update-client-validation
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-ios
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-services-selection.png
  [1]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-portal-data-tables.png
  [2]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-insert-script-users.png
  [3]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-quickstart-data-error-ios.png
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-ios
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-ios
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-ios
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
