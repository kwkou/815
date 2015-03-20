<properties linkid="develop-mobile-tutorials-validate-modify-and-augment-data-ios" urlDisplayName="验证数据" pageTitle="使用服务器脚本验证和修改数据 (iOS) |移动开发人员中心" metaKeywords="" description="了解如何验证和修改使用服务器脚本从 iOS 应用程序发送的数据。" metaCanonical="" services="" documentationCenter="Mobile" title="Validate and modify data in Mobile Services by using server scripts" authors="" solutions="" manager="" editor="" />

# 使用服务器脚本在移动服务中验证和修改数据

[WACOM.INCLUDE [mobile-services-selector-validate-modify-data](../includes/mobile-services-selector-validate-modify-data.md)]

本主题说明如何在 Azure 移动服务中利用服务器脚本。可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。在本教程中，将要定义并注册用于验证和修改数据的服务器脚本。由于服务器端脚本的行为往往会影响到客户端，因此你还要更新 iOS 应用程序以利用这些新行为。

本教程将指导您完成以下基本步骤：

1. [添加字符串长度验证]
2. [更新客户端以支持验证]


本教程以前一教程[数据处理入门]中的步骤和示例应用程序为基础。在开始本教程之前，必须先完成[数据处理入门]。  

## <a name="string-length-validation"></a>添加验证

验证用户提交的数据的长度总不失为一种良好做法。首先，您要注册一个脚本，用于验证发送到移动服务的字符串数据长度，并拒绝过长（在本例中为 10 个字符以上）的字符串。

1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的应用。

   	![][0]

2. 单击"数据"选项卡，然后单击 **TodoItem **表。

   	![][1]

3. 单击"脚本"，然后选择"插入"操作。

   	![][2]

4. 将现有脚本替换为以下函数，然后单击"保存"。

        function insert(item, user, request) {
            if (item.text.length > 10) {
                request.respond(statusCodes.BAD_REQUEST, 'Text length must be 10 characters or less.');
            } else {
                request.execute();
            }
        }

    此脚本将检查**文本**属性的长度，如果该长度超过 10 个字符，则发送错误响应。如果未超过 10 个字符，将调用 **execute** 方法以完成插入。

    <div class="dev-callout">
	<b>注意</b>
	<p>您可以在 <strong>脚本</strong> 选项卡上删除注册的脚本，方法是：单击 <strong>"清除"，</strong> 然后单击 <strong>保存</strong>。</p></div>

## <a name="update-client-validation"></a>更新客户端

移动服务会验证数据和发送错误响应，而你则需要更新你的应用程序，使之能够处理验证后生成的错误响应。

1. 在 Xcode 中，打开你在完成[数据处理入门]教程后修改的项目。

2. 按**运行**按钮 (Command + R) 生成项目并启动应用程序，在文本框中键入 10 个字符以上的文本，然后单击加号 (**+**) 图标。

   可以看到，由于移动服务返回了 400 响应（"错误的请求"），应用程序引发了一个未处理的错误。

3. 在 QSTodoService.m 文件中，在**addItem**方法中找到以下代码行：
    
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

4. 重新生成并启动应用程序。 

   	![][4]

  	可以看到，错误已被处理，并且已向用户显示了错误消息。

<!--## <a name="add-timestamp"></a>添加时间戳

前面的任务验证了某个插入操作，并接受或拒绝了该操作。现在，要使用一个服务器脚本来更新已插入的数据，该脚本将在插入对象之前向其添加一个时间戳属性。

1. 在[管理门户]中的"脚本"****选项卡上，将当前的 **Insert** 脚本替换为以下函数，然后单击"保存"****。

        function insert(item, user, request) {
            if (item.text.length > 10) {
                request.respond(statusCodes.BAD_REQUEST, 'Text length must be under 10');
            } else {
                item.createdAt = new Date();
                request.execute();
            }
        }

    在通过调用 **request**.**execute** 插入对象之前，此函数会将一个新的 **createdAt** 时间戳属性添加到该对象，从而扩展前面的 insert 脚本。 

    <div class="dev-callout"><b>注意</b>
	<p>首次运行此 insert 脚本时，必须启用动态架构。启用动态架构后，移动服务会在第一次执行时自动将 <strong>createdAt</strong> 列添加到 <strong>TodoItem</strong> 表。默认情况下，将为新的移动服务启用动态架构，发布应用程序之前应该禁用动态架构。</p>
    </div>

2. 在 Visual Studio 中，按 F5 ****键运行应用程序，然后在"插入 TodoItem"****中键入文本（少于 10 个字符），然后单击"保存"****。

   	你会发现，新的时间戳并未显示在应用程序 UI 中。

3. 返回管理门户，在"todoitem"****表中单击"浏览"****选项卡。
   
   	可以看到，现在出现了一个"createdAt"****列，并且插入的新项带有一个时间戳值。
  
接下来，需要更新 iOS 应用程序以显示这个新列。

## <a name="update-client-timestamp"></a>再次更新客户端

移动服务客户端将忽略响应中的、无法序列化成已定义类型上的属性的所有数据。最后一步是更新客户端以显示这些新数据。

1. 在 Visual Studio 中打开文件 MainPage.xaml.cs，然后将现有 **TodoItem** 类替换为以下定义：

	    public class TodoItem
	    {
	        public int Id { get; set; }
          
            [DataMember(Name="text")]
	        public string Text { get; set; }

            [DataMember(Name="complete")]
	        public bool Complete { get; set; }
	        
            [DataMember(Name="createdAt")]
	        public DateTime? CreatedAt { get; set; }
	    }
	
    此新类定义包括新的时间戳属性，作为可以为 null 的 DateTime 类型。
  
    <div class="dev-callout"><b>注意</b>
	<p>该 <strong>DataMemberAttribute</strong> 会让客户端将应用程序中的新 <strong>CreatedAt</strong> 属性映射到 TodoItem 表中定义的 <strong>createdAt</strong> 列（具有不同的大小写）。如果你使用此属性，应用程序在对象上使用的属性名称可以不同于 SQL Database 中的列名称。如果不使用此属性，则会因大小写不同而发生错误。</p>
    </div>

5. 在 MainPage.xaml 文件中的 **CheckBoxComplete** 元素的正下方，添加以下 XAML 元素：
	      
        <TextBlock Name="WhenCreated" Text="{Binding CreatedAt}" VerticalAlignment="Center"/>

   	这将在文本框中显示新的 **CreatedAt** 属性。 
	
6. 按 **F5** 键以运行应用。 

   可以看到，只显示了你在更新 insert 脚本后插入的项的时间戳。

7. 将现有的 **RefreshTodoItems** 方法替换为以下代码：
 
        private void RefreshTodoItems()
        { 
            // This query filters out completed TodoItems and 
            // items without a timestamp. 
            items = todoTable
               .Where(todoItem => todoItem.Complete == false
                   && todoItem.CreatedAt != null)
               .ToCollectionView();

            ListItems.ItemsSource = items;
        }

   	此方法将更新查询，以便同时筛选掉没有时间戳值的项。
	
8. 按 **F5** 键以运行应用。

   	可以看到，创建的不带时间戳值的所有项已从 UI 中消失。

现在，已完成了这篇数据处理教程。-->

## <a name="next-steps"> </a>后续步骤

现在已经完成了本教程，建议您继续数据系列中的最后一个教程：[使用分页优化查询]。

在为用户授权以及发送推送通知时，也可以使用服务器脚本。有关详细信息，请参阅以下教程：

* [使用脚本为用户授权]
  <br/>了解如何根据经过身份验证的用户的 ID 筛选数据。

* [推送通知入门] 
  <br/>了解如何将非常简单的推送通知发送到您的应用程序。

* [移动服务服务器脚本参考]
  <br/>了解有关注册和使用服务器脚本的详细信息。

<!-- Anchors. -->
[添加字符串长度验证]: #string-length-validation
[更新客户端以支持验证]: #update-client-validation
[在插入操作中添加时间戳]: #add-timestamp
[更新客户端以显示时间戳]: #update-client-timestamp
[后续步骤]: #next-steps

<!-- Images. -->
[0]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-services-selection.png
[1]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-portal-data-tables.png
[2]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-insert-script-users.png

[4]: ./media/mobile-services-ios-validate-modify-data-server-scripts/mobile-quickstart-data-error-ios.png

<!-- URLs. -->
[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-ios
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-ios-authorize-users-in-scripts
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-ios-add-paging-data
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-data-ios
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-ios
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-ios

[管理门户]: https://manage.windowsazure.cn/
[Azure 管理门户]: https://manage.windowsazure.cn/
