<properties linkid="develop-mobile-tutorials-optimistic-concurrent-data-javascript" urlDisplayName="Optimistic concurrency" pageTitle="Handle database write conflicts with optimistic concurrency (Windows Store) | Mobile Dev Center" metaKeywords="" writer="wesmc" description="Learn how to handle database write conflicts on both the server and in your Windows Store application." metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Handling database write conflicts" authors="wesmc" />

# 处理数据库写入冲突

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/develop/mobile/tutorials/handle-database-write-conflicts-dotnet/" title="Windows Store C#">Windows 应用商店 C\#</a>
<a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-handle-database-conflicts/" title="Windows Store JavaScript" class="current">Windows 应用商店 JavaScript</a>
<a href="/zh-cn/develop/mobile/tutorials/handle-database-write-conflicts-wp8/" title="Windows Phone">Windows Phone</a></div>	

本教程旨在帮助你更好地理解在两个或两个以上客户端写入 Windows 应用商店应用程序中的同一条数据库记录时，如何处理发生的冲突。在某些情况下，两个或两个以上客户端可能会同时将更改写入同一项目。如果没有任何冲突检测，则最后一次写入会覆盖任何以前的更新，即使这并不是所需要的结果。Azure 移动服务为检测和解决这些冲突提供支持。本主题将指导你完成用于处理服务器上和应用程序中数据库写入冲突的步骤。

在本教程中，你将向快速入门应用程序添加功能以处理更新 TodoItem 数据库时发生的争用。本教程将指导你完成以下基本步骤：

1.  [更新应用程序以允许更新][]
2.  [在应用程序中启用冲突检测][]
3.  [测试应用程序中的数据库写入冲突][]
4.  [使用服务器脚本自动解决冲突][]

本教程需要的内容如下：

-   Microsoft Visual Studio 2013 Express for Windows 或更高版本。
-   本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][]，并下载初学者项目的 JavaScript 语言版本。
-   [Azure 帐户][]
-   Windows Azure 移动服务 NuGet 包 1.1.5 或更高版本。若要获取最新版本，请执行以下步骤：

    1.  在 Visual Studio 中，打开项目并右键单击解决方案资源管理器中的项目，然后单击“管理 Nuget 包” 。

    2.  展开“联机” 并单击“Microsoft 和 .NET” 。在搜索文本框中输入“WindowsAzure.MobileServices.WinJS” 。针对 Windows Azure Mobile Services for WinJS  NuGet 包单击“安装” 。

        ![][]

<a name="uiupdate"></a>
## 更新 UI更新应用程序以允许更新

在本节中，你将更新用户界面，以便更新每个项目的文本。绑定模板将针对数据库表中的每个项目提供一个复选框和文本类控件。你将能够更新 TodoItem 的文本字段。应用程序将处理 `keydown` 事件，以便按 "Enter" 键更新项目。

1.  在 Visual Studio 中，打开你在[移动服务入门][]教程中下载的 TodoList 项目的 JavaScript 语言版本。
2.  在 Visual Studio 解决方案资源管理器中，打开 default.html，将 `TemplateItem` div 标记定义替换为下面显示的 div 标记并保存更改。这将添加一个文本框控件，以便编辑 TodoItem 的文本。

        <div id="TemplateItem" data-win-control="WinJS.Binding.Template">
        <div style="display:-ms-grid; -ms-grid-columns:auto 1fr">
        <input class="itemCheckbox" type="checkbox" data-win-bind="checked:complete; dataContext:this" />
        <div style="-ms-grid-column:2; -ms-grid-row-align:center; margin-left:5px">
        <input id="modifytextbox" class="text win-interactive" data-win-bind="value:text; dataContext:this" />
        </div>
        </div>
        </div>

3.  在 Visual Studio 的解决方案资源管理器中，展开“js” 文件夹。打开 default.js 文件，将 `updateTodoItem` 函数替换为下面的定义，该定义将不会从用户界面中删除已更新的项目。

        var updateTodoItem = function (todoItem) {
        // This code takes a freshly completed TodoItem and updates the database. 
        todoTable.update(todoItem);
          };

4.  在 default.js 文件中，添加 `keydown` 事件的以下事件处理程序，以便按 "Enter" 键更新项目。

        listItems.onkeydown = function (eventArgs) {
        if (eventArgs.key == "Enter") {
        var todoItem = eventArgs.target.dataContext.backingData;
        todoItem.text = eventArgs.target.value;
        updateTodoItem(todoItem);
            }
          };

现在，当按 "Enter" 键时，应用程序将对每个项目的文本更改写回到数据库。

<a name="enableOC"></a>
## 启用乐观并发在应用程序中启用冲突检测

Azure 移动服务通过使用添加到每个表的 `__version` 系统属性列跟踪对每个项目的更改，来支持乐观并发控制。在本节中，我们将允许应用程序通过 `__version` 系统属性检测这些写入冲突。在 todoTable 上启用此系统属性后，如果记录自上次查询以来已更改，则应用程序在更新尝试期间将收到 `MobileServicePreconditionFailedException` 发出的通知。然后，应用程序将能够选择是将其更改提交到数据库，还是保持对数据库的上次更改不变。有关移动服务的系统属性的详细信息，请参阅[系统属性][]。

1.  在 default.js 文件中的 `todoTable` 变量声明下，添加代码以包括 \*\*\_\_version\*\* 系统属性，从而启用对写入冲突检测的支持。

        var todoTable = client.getTable('TodoItem');
        todoTable.systemProperties |= WindowsAzure.MobileServiceTable.SystemProperties.Version;

2.  将 `Version` 系统属性添加到该表的系统属性后，如果记录自上次查询以来已更改，则在更新期间将通过 `MobileServicePreconditionFailedException` 异常通知应用程序。此异常将在 JavaScript 中通过错误函数捕获。此错误包含服务器中用于解决冲突的项目的最新版本。在 default.js 中，更新 `updateTodoItem` 函数以捕获错误并调用 `resolveDatabaseConflict` 函数。

        var updateTodoItem = function (todoItem) {
        // This code takes a freshly completed TodoItem and updates the database. 
        // If the server version of the record has been updated, we get the updated
        // record from the Precondition Failed error in order to resolve the conflict.
        var serverItem = null;
        todoTable.update(todoItem).then(null, function (error) {
        if (error.message == "Precondition Failed") {
        serverItem = error.serverInstance;
            }
        else {
        var msgDialog =
        new Windows.UI.Popups.MessageDialog(error.request.responseText,"Update Failed");
        msgDialog.showAsync();
            }
        }).done(function () {
        if (serverItem != null)
        resolveDatabaseConflict(todoItem, serverItem);
          });
        };

3.  在 default.js 中，添加 `updateTodoItem` 函数中引用的 `resolveDatabaseConflict()` 函数的定义。请注意，为了解决冲突，在更新数据库中的项目之前，请将本地项目的版本设置为服务器中的已更新版本。否则，你将不断遇到冲突。

        var resolveDatabaseConflict = function (localItem, serverItem) {
        var content = "This record has been changed as follows on the server already..\n\n" +
        "id :" + serverItem.id + "\n" +
        "text :" + serverItem.text + "\n" +
        "complete :" + serverItem.complete + "\n\n" +
        "Do you want to overwrite the server instance with your data?";
        var msgDialog = new Windows.UI.Popups.MessageDialog(content, "Resolve Database Conflict");
        msgDialog.commands.append(new Windows.UI.Popups.UICommand("Yes"));
        msgDialog.commands.append(new Windows.UI.Popups.UICommand("No"));
        msgDialog.showAsync().done(function (command) {
        if (command.label == "Yes") {
        localItem.__version = serverItem.__version;
        updateTodoItem(localItem);
              }
          });
        }

<a name="test-app"></a>
## 测试应用程序测试应用程序中的数据库写入冲突

在本节中，你将生成 Windows 应用商店应用程序包，以便在第二台计算机或虚拟机上安装应用程序。然后，你将在这两台计算机上运行应用程序，并生成写入冲突以测试代码。应用程序的两个实例将尝试更新同一项目的 `text` 属性，因此需要用户解决该冲突。

1.  创建 Windows 应用商店应用程序包，以便在第二台计算机或虚拟机上进行安装。为此，请在 Visual Studio 中单击“项目” -\>“存储” -\>“创建应用程序包” 。

    ![][1]

2.  在“创建包”屏幕上，单击“否” ，因为此包将不会上载到 Windows 应用商店。然后单击"“下一步”"。

    ![][2]

3.  在“选择和配置包”屏幕上，接受默认设置，然后单击“创建” 。

    ![][3]

4.  在“已创建包”屏幕上，单击“输出位置” 链接以打开包位置。

    ![][4]

5.  将包文件夹“todolist\_1.0.0.0\_AnyCPU\_Debug\_Test”复制到第二台计算机。在该计算机上，打开包文件夹并右键单击 "Add-AppDevPackage.ps1" PowerShell 脚本，然后单击“使用 PowerShell 运行” ，如下所示。按照提示操作以安装应用程序。

    ![][5]

6.  通过单击“调试” -\>“启动调试” 在 Visual Studio 中运行应用程序的第 1 个实例。在第二台计算机的“开始”屏幕上，单击向下箭头以查看“按名称排列的应用程序”。然后单击“Todolist” 应用程序以运行应用程序的第 2 个实例。

    应用程序实例 1
    ![][6]

    应用程序实例 2
    ![][6]

7.  在应用程序的第 1 个实例中，将最后一个项目的文本更新为“Test Write 1” ，然后按 "Enter" 键以更新数据库。下面的屏幕快照显示了一个示例。

    应用程序实例 1
    ![][7]

    应用程序实例 2
    ![][6]

8.  此时，应用程序的第 2 个实例中的最后一个项目具有该项目的旧版本。在该应用程序实例中，针对最后一个项目的 `text` 属性输入“Test Write 2” ，然后按 "Enter" 以使用旧的 `_version` 属性更新数据库。

    应用程序实例 1
    ![][8]

    应用程序实例 2
    ![][9]

9.  由于用于更新尝试的 `__version` 值与服务器 `__version` 值不匹配，因此移动服务 SDK 将引发 `MobileServicePreconditionFailedException` 作为 `updateTodoItem` 函数中的错误，以便让应用程序来解决此冲突。若要解决此冲突，可以单击“是” 以从实例 2 提交值，也可以单击“否” 以放弃实例 2 中的值，让应用程序的实例 1 提交相关值。

    应用程序实例 1
    ![][8]

    应用程序实例 2
    ![][10]

<a name="scriptsexample"></a>
## 使用脚本处理冲突使用服务器脚本自动解决冲突

可以使用服务器脚本检测和解决写入冲突。当你可以使用脚本逻辑而不是用户交互来解决冲突时，这是可行的。在本节中，你将向应用程序的 TodoItem 表添加服务器端脚本。该脚本将用于解决冲突的逻辑如下所示：

-   如果 TodoItem 的 `complete` 字段设为 true，则可以认为该项已完成并且无法再更改 `text`。
-   如果 TodoItem 的 `complete` 字段仍为 false，则将提交更新 `text` 的尝试。

以下步骤将指导你完成添加服务器更新脚本并对其进行测试的过程。

1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][11]

2.  单击“数据” 选项卡，然后单击 TodoItem  表。

    ![][12]

3.  单击“脚本” ，然后选择“更新” 操作。

    ![][13]

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

5.  在两台计算机上运行 "Todolist" 应用程序。在实例 2 中更改最后一个项目的 TodoItem `text`，按 "Enter" 以使应用程序更新数据库。

    应用程序实例 1
    ![][8]

    应用程序实例 2
    ![][9]

6.  在应用程序的实例 1 中，针对最后一个 text 属性输入不同的值，然后按 "Enter"。应用程序将尝试使用不正确的 `__version` 属性更新数据库。

    应用程序实例 1
    ![][14]

    应用程序实例 2
    ![][15]

7.  请注意，由于该项目未标记为“完成”而允许更新，服务器脚本解决了冲突，因此在应用程序中未遇到异常。若要查看更新是否确实成功，请单击实例 2 中的“刷新” 以重新查询数据库。

    应用程序实例 1
    ![][16]

    应用程序实例 2
    ![][16]

8.  在实例 1 中，单击复选框以完成最后一个 Todo 项目。

    应用程序实例 1
    ![][17]

    应用程序实例 2
    ![][16]

9.  在实例 2 中，尝试更新最后一个 TodoItem 的文本并按 "Enter"，这将导致冲突，因为已更新该项目并将“完成”字段设为 true。在对冲突进行响应时，由于该项目已完成，此脚本通过拒绝更新解决了冲突。此脚本在响应中提供了一条消息。

    应用程序实例 1
    ![][18]

    应用程序实例 2
    ![][19]

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
  [0]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/mobile-manage-nuget-packages-dialog.png
  [系统属性]: http://go.microsoft.com/fwlink/?LinkId=331143
  [1]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-create-app-package1.png
  [2]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-create-app-package2.png
  [3]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-create-app-package3.png
  [4]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-create-app-package4.png
  [5]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-install-app-package.png
  [6]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-app1.png
  [7]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-app1-write1.png
  [8]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-app1-write2.png
  [9]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-app2-write2.png
  [10]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-app2-write2-conflict.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [11]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/mobile-services-selection.png
  [12]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/mobile-portal-data-tables.png
  [13]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/mobile-insert-script-users.png
  [14]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-app1-write3.png
  [15]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-app2-write3.png
  [16]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-write3.png
  [17]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-checkbox.png
  [18]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-2-items.png
  [19]: ./media/mobile-services-windows-store-javascript-handle-database-conflicts/Mobile-oc-store-already-complete.png
  [使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-validate-modify-data-server-scripts/
  [使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-add-paging-data/
  [身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users/
  [推送通知入门]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push/
