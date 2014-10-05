<properties linkid="develop-mobile-tutorials-optimistic-concurrent-data-dotnet" urlDisplayName="Optimistic concurrency" pageTitle="Handle database write conflicts with optimistic concurrency (Windows Store) | Mobile Dev Center" metaKeywords="" description="Learn how to handle database write conflicts on both the server and in your Windows Store application." metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Handling database write conflicts" authors="wesmc" />

# 处理数据库写入冲突

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/develop/mobile/tutorials/handle-database-write-conflicts-dotnet/" title="Windows Store C#" class="current">Windows 应用商店 C\#</a>
<a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-handle-database-conflicts/" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
<a href="/zh-cn/develop/mobile/tutorials/handle-database-write-conflicts-wp8/" title="Windows Phone">Windows Phone</a></div>	

本教程旨在帮助你更好地理解在两个或两个以上客户端写入 Windows 应用商店应用程序中的同一条数据库记录时，如何处理发生的冲突。在某些情况下，两个或两个以上客户端可能会同时将更改写入同一项目。如果没有任何冲突检测，则最后一次写入会覆盖任何以前的更新，即使这并不是所需要的结果。Azure 移动服务为检测和解决这些冲突提供支持。本主题将指导你完成用于处理服务器上和应用程序中数据库写入冲突的步骤。

在本教程中，你将向快速入门应用程序添加功能以处理更新 TodoItem 数据库时发生的争用。本教程将指导你完成以下基本步骤：

1.  [更新应用程序以允许更新][]
2.  [在应用程序中启用冲突检测][]
3.  [测试应用程序中的数据库写入冲突][]
4.  [使用服务器脚本自动解决冲突][]

本教程需要的内容如下：

-   Microsoft Visual Studio 2012 Express for Windows 或更高版本。
-   本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]。
-   [Azure 帐户][]
-   Azure 移动服务 NuGet 包 1.1.0 或更高版本。若要获取最新版本，请执行以下步骤：

    1.  在 Visual Studio 中，打开项目并右键单击解决方案资源管理器中的项目，然后单击“管理 Nuget 包” 。

        ![][]

    2.  展开“联机” 并单击“Microsoft 和 .NET” 。在“搜索”文本框中输入“Azure 移动服务” 。针对 Azure 移动服务  NuGet 包单击“安装” 。

        ![][1]

<a name="uiupdate"></a>
## 更新 UI更新应用程序以允许更新

在本节中，你将更新 TodoList 用户界面，以便更新列表框控件中每个项目的文本。列表框将包含针对数据库表中每个项目的复选框和文本框控件。你将能够更新 TodoItem 的文本字段。应用程序将处理文本框的 `LostFocus` 事件以更新数据库中的项目。

1.  在 Visual Studio 中，打开你在[移动服务入门][]教程中下载的 TodoList 项目。
2.  在 Visual Studio 解决方案资源管理器中，打开 MainPage.xaml，将 `ListView` 定义替换为下面所示的 `ListView` 并保存更改。

        <ListView Name="ListItems" Margin="62,10,0,0" Grid.Row="1">
        <ListView.ItemTemplate>
        <DataTemplate>
        <StackPanel Orientation="Horizontal">
        <CheckBox Name="CheckBoxComplete" IsChecked="{Binding Complete, Mode=TwoWay}" Checked="CheckBoxComplete_Checked" Margin="10,5" VerticalAlignment="Center"/>
        <TextBox x:Name="ToDoText" Height="25" Width="300" Margin="10" Text="{Binding Text, Mode=TwoWay}" AcceptsReturn="False" LostFocus="ToDoText_LostFocus"/>
        </StackPanel>
        </DataTemplate>
        </ListView.ItemTemplate>
        </ListView>

3.  在 MainPage.xaml.cs 中，将以下 `using` 指令添加到页面顶部。

        using System.Threading.Tasks;

4.  在 Visual Studio 解决方案资源管理器中，打开 MainPage.xaml.cs。向 MainPage 添加 `LostFocus` 事件的事件处理程序，如下所示。

        private async void ToDoText_LostFocus(object sender, RoutedEventArgs e)
        {
        TextBox tb = (TextBox)sender;
        TodoItem item = tb.DataContext as TodoItem;
        //let's see if the text changed
        if (item.Text != tb.Text)
            {
        item.Text = tb.Text;
        await UpdateToDoItem(item);
            }
        }

5.  在 MainPage.xaml.cs 中，添加事件处理程序中引用的 MainPage `UpdateToDoItem()` 方法的定义，如下所示。

        private async Task UpdateToDoItem(TodoItem item)
        {
        Exception exception = null;         
        try
            {
        //update at the remote table
        await todoTable.UpdateAsync(item);
            }
        catch (Exception ex)
            {
        exception = ex;
            }           
        if (exception != null)
            {
        await new MessageDialog(exception.Message, "Update Failed").ShowAsync();
            }
        }

现在，当文本框失去焦点时，应用程序将对每个项目的文本更改写回到数据库。

<a name="enableOC"></a>
## 启用乐观并发在应用程序中启用冲突检测

在某些情况下，两个或两个以上客户端可能会同时将更改写入同一项目。如果没有任何冲突检测，则最后一次写入会覆盖任何以前的更新，即使这并不是所需要的结果。[乐观并发控制][]假定每个事务均可以提交，因此不使用任何资源锁定。提交事务之前，乐观并发控制将验证是否没有其他事务修改了数据。如果数据已修改，则将回滚正在提交的事务。Azure 移动服务通过使用添加到每个表的 `__version` 系统属性列跟踪对每个项目的更改，来支持乐观并发控制。在本节中，我们将允许应用程序通过 `__version` 系统属性检测这些写入冲突。如果记录自上次查询以来已更改，则应用程序在更新尝试期间将收到 `MobileServicePreconditionFailedException` 发出的通知。然后，它将能够选择是将其更改提交到数据库，还是保持对数据库的上次更改不变。有关移动服务的系统属性的详细信息，请参阅[系统属性][]。

1.  在 MainPage.xaml.cs 中，使用以下代码更新 "TodoItem" 类定义以包括 \*\*\_\_version\*\* 系统属性，从而启用对写入冲突检测的支持。

        public class TodoItem
        {
        public string Id { get; set; }          
        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }            
        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }          
        [JsonProperty(PropertyName = "__version")]
        public string Version { set; get; }
        }

	<div class="dev-callout"><b>说明</b>

    <p>使用非类型表时，请通过将 Version 标志添加到表的 SystemProperties 来启用乐观并发。</p>

	<pre><code>//Enable optimistic concurrency by retrieving __version
todoTable.SystemProperties |= MobileServiceSystemProperties.Version;
</code></pre>
	</div>

2.  将 `Version` 属性添加到 `TodoItem` 类后，如果记录自上次查询以来已更改，则在更新期间将通过 `MobileServicePreconditionFailedException` 异常通知应用程序。此异常包括服务器中该项目的最新版本。在 MainPage.xaml.cs 中，添加以下代码以在 `UpdateToDoItem()` 方法中处理异常。

        private async Task UpdateToDoItem(TodoItem item)
        {
        Exception exception = null;         
        try
            {
        //update at the remote table
        await todoTable.UpdateAsync(item);
            }
        catch (MobileServicePreconditionFailedException<TodoItem> writeException)
            {
        exception = writeException;
            }
        catch (Exception ex)
            {
        exception = ex;
            }           
        if (exception != null)
            {
        if (exception is MobileServicePreconditionFailedException)
                {
        //Conflict detected, the item has changed since the last query
        //Resolve the conflict between the local and server item
        await ResolveConflict(item, ((MobileServicePreconditionFailedException<TodoItem>)exception).Item);
                }
        else
                {
        await new MessageDialog(exception.Message, "Update Failed").ShowAsync();
                }
            }
        }

3.  在 MainPage.xaml.cs 中，添加 `UpdateToDoItem()` 中引用的 `ResolveConflict()` 方法的定义。请注意，为了解决冲突，在提交用户的决定之前，请将本地项目的版本设置为服务器中的已更新版本。否则，你将不断遇到冲突。

        private async Task ResolveConflict(TodoItem localItem, TodoItem serverItem)
        {
        //Ask user to choose the resolution between versions
        MessageDialog msgDialog = new MessageDialog(String.Format("Server Text:\"{0}\" \nLocal Text:\"{1}\"\n", 
        serverItem.Text, localItem.Text), 
        "CONFLICT DETECTED - Select a resolution:");
        UICommand localBtn = new UICommand("Commit Local Text");
        UICommand ServerBtn = new UICommand("Leave Server Text");
        msgDialog.Commands.Add(localBtn);
        msgDialog.Commands.Add(ServerBtn);          
        localBtn.Invoked = async (IUICommand command) =>
            {
        // To resolve the conflict, update the version of the 
        // item being committed.Otherwise, you will keep
        // catching a MobileServicePreConditionFailedException.
        localItem.Version = serverItem.Version;             
        // Updating recursively here just in case another 
        // change happened while the user was making a decision
        await UpdateToDoItem(localItem);
            };          
        ServerBtn.Invoked = async (IUICommand command) =>
            {
        RefreshTodoItems();
            };          
        await msgDialog.ShowAsync();
        }

<a name="test-app"></a>
## 测试应用程序测试应用程序中的数据库写入冲突

在本节中，你将生成 Windows 应用商店应用程序包，以便在第二台计算机或虚拟机上安装应用程序。然后，你将在这两台计算机上运行应用程序，并生成写入冲突以测试代码。应用程序的两个实例将尝试更新同一项目的 `text` 属性，因此需要用户解决该冲突。

1.  创建 Windows 应用商店应用程序包，以便在第二台计算机或虚拟机上进行安装。为此，请在 Visual Studio 中单击“项目” -\>“存储” -\>“创建应用程序包” 。

    ![][2]

2.  在“创建包”屏幕上，单击“否” ，因为此包将不会上载到 Windows 应用商店。然后单击"“下一步”"。

    ![][3]

3.  在“选择和配置包”屏幕上，接受默认设置，然后单击“创建” 。

    ![][4]

4.  在“已创建包”屏幕上，单击“输出位置” 链接以打开包位置。

    ![][5]

5.  将包文件夹“todolist\_1.0.0.0\_AnyCPU\_Debug\_Test”复制到第二台计算机。在该计算机上，打开包文件夹并右键单击 "Add-AppDevPackage.ps1" PowerShell 脚本，然后单击“使用 PowerShell 运行” ，如下所示。按照提示操作以安装应用程序。

    ![][6]

6.  通过单击“调试” -\>“启动调试” 在 Visual Studio 中运行应用程序的第 1 个实例。在第二台计算机的“开始”屏幕上，单击向下箭头以查看“按名称排列的应用程序”。然后单击“Todolist” 应用程序以运行应用程序的第 2 个实例。

    应用程序实例 1
    ![][7]

    应用程序实例 2
    ![][7]

7.  在应用程序的第 1 个实例中，将最后一个项目的文本更新为“Test Write 1” ，然后单击另一个文本框，以使 `LostFocus` 事件处理程序更新数据库。下面的屏幕快照显示了一个示例。

    应用程序实例 1
    ![][8]

    应用程序实例 2
    ![][7]

8.  此时，应用程序的第 2 个实例中的相应项目具有该项目的旧版本。在该应用程序实例中，针对 `text` 属性输入“Test Write 2” 。然后单击另一个文本框，`LostFocus` 事件处理程序便会尝试使用旧 `_version` 属性更新数据库。

    应用程序实例 1
    ![][9]

    应用程序实例 2
    ![][10]

9.  由于用于更新尝试的 `__version` 值与服务器 `__version` 值不匹配，因此移动服务 SDK 将引发 `MobileServicePreconditionFailedException`，以便让应用程序来解决此冲突。若要解决此冲突，可以单击“提交本地文本” 以从实例 2 提交值，也可以单击“保留服务器文本” 以放弃实例 2 中的值，让应用程序的实例 1 提交相关值。

    应用程序实例 1
    ![][9]

    应用程序实例 2
    ![][11]

<a name="scriptsexample"></a>
## 使用脚本处理冲突使用服务器脚本自动解决冲突

可以使用服务器脚本检测和解决写入冲突。当你可以使用脚本逻辑而不是用户交互来解决冲突时，这是可行的。在本节中，你将向应用程序的 TodoItem 表添加服务器端脚本。该脚本将用于解决冲突的逻辑如下所示：

-   如果 TodoItem 的 `complete` 字段设为 true，则可以认为该项已完成并且无法再更改 `text`。
-   如果 TodoItem 的 `complete` 字段仍为 false，则将提交更新 `text` 的尝试。

以下步骤将指导你完成添加服务器更新脚本并对其进行测试的过程。

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][12]

2.  单击“数据” 选项卡，然后单击 TodoItem  表。

    ![][13]

3.  单击“脚本” ，然后选择“更新” 操作。

    ![][14]

4.  将现有脚本替换为以下函数，然后单击“保存” 。

        function update(item, user, request) { 
        request.execute({ 
        conflict:function (serverRecord) {
        // Only committing changes if the item is not completed.
        if (serverRecord.complete === false) {
        //write the updated item to the table
        request.execute();
                    }
        else
                    {
        request.respond(statusCodes.FORBIDDEN, 'The item is already completed.');
                    }
                }
            }); 
        }   

5.  在两台计算机上运行 "Todolist" 应用程序。在实例 2 中更改最后一个项目的 TodoItem `text`，然后单击另一个文本框，以使 `LostFocus` 事件处理程序更新数据库。

    应用程序实例 1
    ![][9]

    应用程序实例 2
    ![][10]

6.  在应用程序的实例 1 中，针对最后一个 text 属性输入不同的值。然后单击另一个文本框，`LostFocus` 事件处理程序便会尝试使用不正确的 `_version` 属性更新数据库。

    应用程序实例 1
    ![][15]

    应用程序实例 2
    ![][16]

7.  请注意，由于该项目未标记为“完成”而允许更新，服务器脚本解决了冲突，因此在应用程序中未遇到异常。若要查看更新是否确实成功，请单击实例 2 中的“刷新” 以重新查询数据库。

    应用程序实例 1
    ![][17]

    应用程序实例 2
    ![][17]

8.  在实例 1 中，单击复选框以完成最后一个 Todo 项目。

    应用程序实例 1
    ![][18]

    应用程序实例 2
    ![][17]

9.  在实例 2 中，尝试更新最后一个 TodoItem 的文本，触发 `LostFocus` 事件。在对冲突进行响应时，由于该项目已完成，此脚本通过拒绝更新解决了冲突。

    应用程序实例 1
    ![][19]

    应用程序实例 2
    ![][20]

<a name="next-steps"> </a>
## 后续步骤

本教程演示了在处理移动服务中的数据时，如何让 Windows 应用商店应用程序处理写入冲突。接下来，请考虑完成数据系列中的以下教程之一：

-   [使用脚本验证和修改数据][]
    了解更多有关使用移动服务中的服务器脚本验证和更改从应用程序发送的数据的信息。

-   [使用分页优化查询][]
    了解如何使用查询中的分页控制单个请求中处理的数据量。

完成了数据系列后，你还可以尝试以下 Windows 应用商店教程之一：

-   [身份验证入门][]
    了解如何对应用程序用户进行身份验证。

-   [推送通知入门][]
    了解如何使用移动服务将非常基本的推送通知发送到应用程序。

  [Windows 应用商店 C\#]: /zh-cn/develop/mobile/tutorials/handle-database-write-conflicts-dotnet/ "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-handle-database-conflicts/ "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/develop/mobile/tutorials/handle-database-write-conflicts-wp8/ "Windows Phone"
  [更新应用程序以允许更新]: #uiupdate
  [在应用程序中启用冲突检测]: #enableOC
  [测试应用程序中的数据库写入冲突]: #test-app
  [使用服务器脚本自动解决冲突]: #scriptsexample
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started
  [Azure 帐户]: http://www.windowsazure.com/zh-cn/pricing/free-trial/
  []: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/mobile-manage-nuget-packages-VS.png
  [1]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/mobile-manage-nuget-packages-dialog.png
  [乐观并发控制]: http://go.microsoft.com/fwlink/?LinkId=330935
  [系统属性]: http://go.microsoft.com/fwlink/?LinkId=331143
  [2]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-create-app-package1.png
  [3]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-create-app-package2.png
  [4]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-create-app-package3.png
  [5]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-create-app-package4.png
  [6]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-install-app-package.png
  [7]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-app1.png
  [8]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-app1-write1.png
  [9]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-app1-write2.png
  [10]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-app2-write2.png
  [11]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-app2-write2-conflict.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [12]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/mobile-services-selection.png
  [13]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/mobile-portal-data-tables.png
  [14]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/mobile-insert-script-users.png
  [15]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-app1-write3.png
  [16]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-app2-write3.png
  [17]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-write3.png
  [18]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-checkbox.png
  [19]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-2-items.png
  [20]: ./media/mobile-services-windows-store-dotnet-handle-database-conflicts/Mobile-oc-store-already-complete.png
  [使用脚本验证和修改数据]: /zh-cn/develop/mobile/tutorials/validate-modify-and-augment-data-dotnet
  [使用分页优化查询]: /zh-cn/develop/mobile/tutorials/add-paging-to-data-dotnet
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet
  [推送通知入门]: /zh-cn/develop/mobile/tutorials/get-started-with-push-dotnet
