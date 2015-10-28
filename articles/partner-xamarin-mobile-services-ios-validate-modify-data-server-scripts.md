<properties
	pageTitle="使用服务器脚本验证和修改数据 (Xamarin iOS) | 移动开发人员中心"
	description="了解如何验证和修改使用服务器脚本从 Xamarin iOS 应用程序发送的数据。"
	services="mobile-services"
	documentationCenter="xamarin"
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="04/24/2015"
	wacn.date="07/25/2015"/>

#  使用服务器脚本在移动服务中验证和修改数据

[AZURE.INCLUDE [mobile-services-selector-validate-modify-data](../includes/mobile-services-selector-validate-modify-data.md)]

本主题说明如何在 Azure 移动服务中利用服务器脚本。可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。在本教程中，将要定义并注册用于验证和修改数据的服务器脚本。由于服务器端脚本的行为往往会影响到客户端，因此你还要更新 iOS 应用程序以利用这些新行为。在 [ValidateModifyData 应用程序][GitHub]示例中提供完成的代码。

本教程将指导你完成以下基本步骤：

1. [添加字符串长度验证]
2. [更新客户端以支持验证]
3. [在插入操作中添加时间戳]
4. [更新客户端以显示时间戳]

本教程以前一教程[数据处理入门]中的步骤和示例应用程序为基础。在开始本教程之前，应先完成[数据处理入门]。

##  <a name="string-length-validation"></a>添加验证

验证用户提交的数据的长度总不失为一种良好做法。首先，您要注册一个脚本，用于验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

1. 登录到 [Azure 管理门户]，单击“移动服务”，然后单击你的应用程序。 

	![][0]

2. 单击“数据”选项卡，然后单击 **TodoItem** 表。

   	![][1]

3. 单击“脚本”，然后选择“插入”操作。

   	![][2]

4. 将现有脚本替换为以下函数，然后单击“保存”。

    	function insert(item, user, request) { if (item.text.length > 10) { request.respond(statusCodes.BAD_REQUEST, 'Text length must be 10 characters or less.'); } else { request.execute(); } }

    此脚本将检查 **text** 属性的长度，如果该长度超过 10 个字符，则发送错误响应。如果未超过 10 个字符，将调用 **execute** 方法以完成插入。

    > [AZURE.NOTE]在“脚本”选项卡中，依次单击“清除”和“保存”可以删除某个已注册的脚本。

##  <a name="update-client-validation"></a>更新客户端

移动服务会验证数据和发送错误响应，而你则需要更新你的应用程序，使之能够处理验证后生成的错误响应。

1. 在 Xamarin Studio 中，打开你在完成[数据处理入门]教程后修改的项目。

2. 按“运行”按钮生成项目并启动应用程序，在文本框中键入 10 个字符以上的文本，然后单击加号 (+) 图标。

	可以看到，由于移动服务返回了 400 响应（“错误的请求”），应用程序引发了一个未处理的错误。

3. 在 TodoService.cs 文件中，找到 **InsertTodoItemAsync** 方法中的当前 <code>try/catch</code> 异常处理，并将 <code>catch</code> 替换为以下代码：
    
    	catch (Exception ex) { var exDetail = (ex.InnerException.InnerException as MobileServiceInvalidOperationException); Console.WriteLine(exDetail.Message);
                                
        UIAlertView alert = new UIAlertView() { 
            	Title = "Error", 
            	Message = exDetail.Message
        } ;
        alert.AddButton("Ok");
        alert.Show();

        return -1;
		}

	此代码显示一个弹出窗口，在其中向用户显示错误信息。

4. 在 **TodoListViewController.cs** 中找到 **OnAdd** 方法。更新该方法以确保返回的 <code>index</code> 不是 <code>-1</code>，因为该值是在 **InsertTodoItemAsync** 中的异常处理部分返回的。在这种情况下，我们不需要向 <code>TableView</code> 中添加新行。

    	if (index != -1) { TableView.InsertRows(new { NSIndexPath.FromItemSection(index, 0) }, UITableViewRowAnimation.Top); itemText.Text = ""; }


5. 重新生成并启动应用程序。

	![][4]

	可以看到，错误已被处理，并且已向用户显示了错误消息。


##  <a name="next-steps"></a>后续步骤

本教程到此结束，建议你继续学习有关数据处理的系列教程中的最后一篇：[使用分页优化查询]。

在为用户授权以及发送推送通知时，也可以使用服务器脚本。有关详细信息，请参阅以下教程：

* [使用脚本为用户授权]<br/>了解如何基于某个已经过身份验证的用户的 ID 筛选数据。

* [推送通知入门]<br/>了解如何向应用程序发送一条很基本的推送通知。

* [移动服务服务器脚本参考 ]<br/>了解有关注册和使用服务器脚本的详细信息。

<!-- Anchors. -->
[添加字符串长度验证]: #string-length-validation
[更新客户端以支持验证]: #update-client-validation
[在插入操作中添加时间戳]: #add-timestamp
[更新客户端以显示时间戳]: #update-client-timestamp
[Next Steps]: #next-steps

<!-- Images. -->

[0]: ./media/partner-xamarin-mobile-services-ios-validate-modify-data-server-scripts/mobile-services-selection.png
[1]: ./media/partner-xamarin-mobile-services-ios-validate-modify-data-server-scripts/mobile-portal-data-tables.png
[2]: ./media/partner-xamarin-mobile-services-ios-validate-modify-data-server-scripts/mobile-insert-script-users.png

[4]: ./media/partner-xamarin-mobile-services-ios-validate-modify-data-server-scripts/mobile-quickstart-data-error-ios.png

<!-- URLs. -->
[移动服务服务器脚本参考 ]: http://go.microsoft.com/fwlink/?LinkId=262293
[Get started with Mobile Services]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started
[使用脚本为用户授权]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization
[使用分页优化查询]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library#paging
[数据处理入门]: 
documentation/articles/partner-xamarin-mobile-services-ios-get-started-data
[Get started with authentication]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started-users
[推送通知入门]: /documentation/articles/partner-xamarin-mobile-services-ios-get-started-push
[Management Portal]: https://manage.windowsazure.cn/
[Azure 管理门户]: https://manage.windowsazure.cn/
[GitHub]: http://go.microsoft.com/fwlink/p/?LinkId=331330

<!---HONumber=HO63-->