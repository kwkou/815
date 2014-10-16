<properties linkid="develop-mobile-tutorials-validate-modify-and-augment-data-wp8" urlDisplayName="Validate Data" pageTitle="Use server scripts to validate data (Windows Phone) | Mobile Dev Center" metaKeywords="" description="Learn how to validate and modify data sent using server scripts from your Windows Phone 8 app." metaCanonical="" services="" documentationCenter="Mobile" title="Validate and modify data in Mobile Services by using server scripts" authors="glenga" solutions="" manager="" editor="" />

# 使用服务器脚本在移动服务中验证和修改数据

<div class="dev-center-tutorial-selector sublanding"> 
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet" title="Windows Store C#">Windows 应用商店 C\#</a>
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-js" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-wp8" title="Windows Phone" class="current">Windows Phone</a>
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-ios" title="iOS">iOS</a>
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-android" title="Android">Android</a>
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-html" title="HTML">HTML</a>
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-ios" title="Xamarin.iOS">Xamarin.iOS</a>
<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-android" title="Xamarin.Android">Xamarin.Android</a>
</div>

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-validate-modify-data/" title=".NET backend">.NET 后端</a> | 
	<a href="/zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-wp8"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">

<p>本主题说明如何在 Azure 移动服务中利用服务器脚本。你可以在移动服务中注册服务器脚本，然后使用这些脚本对所要插入和更新的数据执行各种操作，包括验证和数据修改。在本教程中，你将要定义并注册用于验证和修改数据的服务器脚本。由于服务器端脚本的行为往往会影响到客户端，因此你还要更新 Windows Phone 8 应用程序以利用这些新行为。</p>
</div>

<div class="dev-onpage-video-wrapper"><a href="http://go.microsoft.com/fwlink/?LinkId=298629" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/mobile-wp8-validate-modify-data-180x120.png') !important;" href="http://go.microsoft.com/fwlink/?LinkId=298629" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">11:36</span></div>
</div>

本教程将指导你完成以下基本步骤：

1.  [添加字符串长度验证][]
2.  [更新客户端以支持验证][]
3.  [在插入操作中添加时间戳][]
4.  [更新客户端以显示时间戳][]

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
        request.respond(statusCodes.BAD_REQUEST, 'Text length must be under 10');
        } else {
        request.execute();
            }
        }

    此脚本将检查 "TodoItem.text" 属性的长度，如果该长度超过 10 个字符，则发送错误响应。如果未超过 10 个字符，将调用 "execute" 方法以完成插入。

    <div class="dev-callout"> <b>说明</b>

    <p>在“脚本”选项卡中，依次单击“清除”和“保存”可以删除某个已注册的脚本 。</p>
	</div>

<a name="update-client-validation"></a>
## 更新客户端

移动服务会验证数据和发送错误响应，而你则需要更新你的应用程序，使之能够处理验证后生成的错误响应。

1.  在 Visual Studio 2012 Express for Windows Phone 中，打开你在完成教程[数据处理入门][]后修改的项目。

2.  按 "F5" 键运行应用程序，然后在文本框中键入超过 10 个字符的文本，然后单击“保存” 。

    可以看到，由于移动服务返回了 400 响应（“错误的请求”），应用程序引发了一个未处理的 "MobileServiceInvalidOperationException"。

3.  打开文件 MainPage.xaml.cs，然后将现有 "InsertTodoItem" 方法替换为以下内容：

        private async void InsertTodoItem(TodoItem todoItem)
        {
        // This code inserts a new TodoItem into the database. 
        // When the operation completes and Mobile Services has 
        // assigned an Id, the item is added to the collection.
        try
            {
        await todoTable.InsertAsync(todoItem);
        items.Add(todoItem);
            }
        catch (MobileServiceInvalidOperationException e)
            {
        MessageBox.Show(e.Message,
        string.Format("{0} (HTTP {1})",
        e.Response.ReasonPhrase,
        (int)e.Response.StatusCode), 
        MessageBoxButton.OK);
            }
        }

    此方法版本包括针对 "MobileServiceInvalidOperationException" 的错误处理，可在消息框中显示错误响应。

<a name="add-timestamp"></a>
## 添加时间戳

前面的任务验证了某个插入操作，并已接受或拒绝该操作。现在，你要使用一个服务器脚本来更新已插入的数据，该脚本将在插入对象之前向其添加一个时间戳属性。

<div class="dev-callout"><b>说明</b>

<p>此处演示的 <b>createdAt</b> 时间戳属性目前是多余的。移动服务会自动为每个表创建一个 </b>__createdAt</b> 系统属性。你只需通过将以下成员添加到 TodoItem 类，即可在应用程序中使用此系统属性</p>

<pre><code>
[JsonProperty(PropertyName = "__createdAt")]
public DateTime createdAt { set; get; }
</code></pre>

</div>

1.  在[管理门户][Azure 管理门户]中的“脚本”选项卡上，将当前的 "Insert" 脚本替换为以下函数，然后单击“保存” 。

        function insert(item, user, request) {
        if (item.text.length > 10) {
        request.respond(statusCodes.BAD_REQUEST, 'Text length must be under 10');
        } else {
        item.createdAt = new Date();
        request.execute();
            }
        }

    在通过调用 "request"."execute" 插入对象之前，此函数会将一个新的 "createdAt" 时间戳属性添加到该对象，从而扩展前面的 insert 脚本。

    <div class="dev-callout"><b>说明</b>

    <p>首次运行此 insert 脚本时，必须启用动态架构。启用动态架构后，移动服务在首次执行时会自动将 <b>createdAt</b> 列添加到 <b>TodoItem</b> 表。默认情况下，将为新的移动服务启用动态架构，将应用程序发布到 Windows Phone 应用商店之前，应该禁用动态架构。</p>
	</div>

2.  在 Visual Studio 中，按 "F5" 键运行应用程序，然后在文本框中键入文本（少于 10 个字符），然后单击“保存” 。

    你会发现，新的时间戳并未显示在应用程序 UI 中。

3.  返回管理门户，在“todoitem”表中单击“浏览”选项卡 。

    可以看到，现在出现了一个“createdAt” 列，并且插入的新项带有一个时间戳值。

接下来，需要更新 Windows Phone 应用程序以显示这个新列。

<a name="update-client-timestamp"></a>
## 再次更新客户端

移动服务客户端将忽略响应中的、无法序列化成已定义类型上的属性的所有数据。最后一步是更新客户端以显示这些新数据。

1.  在 Visual Studio 中打开文件 MainPage.xaml.cs，然后将现有 "TodoItem" 类替换为以下定义：

        public class TodoItem
        {
        public string Id { get; set; }

        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }

        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }

        [JsonProperty(PropertyName = "createdAt")]
        public DateTime?CreatedAt { get; set; }
        }

    此新类定义包括新的时间戳属性，作为可以为 null 的 DateTime 类型。

    <div class="dev-callout"><b>说明</b>

    <p><b>DataMemberAttribute</b> 告知客户端要将应用程序中的新 <b>CreatedAt</b> 属性映射到 TodoItem 表中定义的具有不同大小写的 <b>createdAt</b> 列。如果你使用此属性，应用程序在对象上使用的属性名称可以不同于 SQL Database 中的列名称。如果不使用此属性，则会因大小写不同而发生错误。</p>
	</div>

2.  在 MainPage.xaml 文件中的 "CheckBoxComplete" 元素的正下方，添加以下 XAML 元素：

        <TextBlock Name="WhenCreated" Text="{Binding CreatedAt}" VerticalAlignment="Center"/>

    这将在文本框中显示新的 "CreatedAt" 属性。

3.  按 "F5" 键以运行应用程序。

    可以看到，只显示了你在更新 insert 脚本后插入的项的时间戳。

4.  将现有的 "RefreshTodoItems" 方法替换为以下代码：

        private async void RefreshTodoItems()
        {
        // This query filters out completed TodoItems and 
        // items without a timestamp. 
        items = await todoTable
        .Where(todoItem => todoItem.Complete == false
        && todoItem.CreatedAt != null)
        .ToCollectionAsync();

        ListItems.ItemsSource = items;
        }

    此方法将更新查询，以便同时筛选掉没有时间戳值的项。

5.  按 "F5" 键以运行应用程序。

    可以看到，创建的不带时间戳值的所有项已从 UI 中消失。

现在，你已完成了这篇数据处理教程。

<a name="next-steps"> </a>
## 后续步骤

现在你已完成本教程，建议你继续学习数据系列中的最后一篇教程：

在为用户授权以及发送推送通知时，也可以使用服务器脚本。有关详细信息，请参阅以下教程：[使用分页优化查询][]。

-   [使用脚本为用户授权][]
    了解如何基于某个已经过身份验证的用户的 ID 筛选数据。

-   [推送通知入门][]
    了解如何向应用程序发送一条非常简单的推送通知。

-   [移动服务服务器脚本参考][]
    了解有关注册和使用服务器脚本的详细信息。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-js "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-wp8 "Windows Phone"
  [iOS]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-ios "iOS"
  [Android]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-android "Android"
  [HTML]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-html "HTML"
  [Xamarin.iOS]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-ios "Xamarin.iOS"
  [Xamarin.Android]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-xamarin-android "Xamarin.Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-phone-validate-modify-data/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-wp8 "JavaScript 后端"
  [观看教程]: http://go.microsoft.com/fwlink/?LinkId=298629
  [添加字符串长度验证]: #string-length-validation
  [更新客户端以支持验证]: #update-client-validation
  [在插入操作中添加时间戳]: #add-timestamp
  [更新客户端以显示时间戳]: #update-client-timestamp
  [数据处理入门]: /zh-cn/develop/mobile/tutorials/get-started-with-data-wp8
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-windows-phone-validate-modify-data-server-scripts/mobile-services-selection.png
  [1]: ./media/mobile-services-windows-phone-validate-modify-data-server-scripts/mobile-portal-data-tables.png
  [2]: ./media/mobile-services-windows-phone-validate-modify-data-server-scripts/mobile-insert-script-users.png
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-wp8
  [使用脚本为用户授权]: /zh-cn/develop/mobile/tutorials/authorize-users-in-scripts-wp8
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-wp8
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
