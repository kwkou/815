<properties 
	pageTitle="在通用 Windows 应用中使用脱机数据 | Microsoft Azure" 
	description="了解如何在通用 Windows 应用中使用 Azure 移动服务缓存和同步脱机数据" 
	documentationCenter="mobile-services" 
	authors="lindydonna" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/11/2016"
	wacn.date="03/21/2016"/>

# 在移动服务中使用脱机数据同步



[AZURE.INCLUDE [mobile-services-selector-offline](../includes/mobile-services-selector-offline.md)]

本教程说明如何使用 Azure 移动服务向通用 Windows 应用商店应用程序添加脱机支持。脱机支持将允许您在应用程序脱机的情况下与本地数据库交互。应用程序与后端数据库联机后，将使用脱机功能同步本地更改。



在本教程中，你将更新[移动服务入门]教程中的通用应用项目，以支持 Azure 移动服务的脱机功能。随后，你将在断开连接的脱机情况下添加数据，将这些项目同步到联机数据库，然后登录到 [Azure 管理门户]，查看在运行应用程序时对数据所做的更改。

>[AZURE.NOTE]本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 在 Windows 应用商店应用程序中存储和检索数据。如果这是你第一次体验移动服务，则应先完成[移动服务入门]教程。

##先决条件 

本教程需要的内容如下：

* 在 Windows 8.1 上运行的 Visual Studio 2013。
* 完成[移动服务入门]。
* [Azure 移动服务 SDK 版本 1.3.0（或更高版本）][Mobile Services SDK Nuget]
* [Azure 移动服务 SQLite Store 版本 1.0.0（或更高版本）][SQLite store nuget]
* [SQLite for Windows 8.1](http://www.sqlite.org/download.html)
* 一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并取得多达 10 个免费的移动服务，即使在试用期结束之后仍可继续使用这些服务。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。 

## <a name="enable-offline-app"></a>更新应用程序以支持脱机功能

当你的移动服务处于脱机情况时，可使用 Azure 移动服务脱机功能与本地数据库交互。若要在你的应用中使用这些功能，请将 `MobileServiceClient.SyncContext` 初始化到本地存储。然后，通过 `IMobileServiceSyncTable` 接口引用你的表。在本教程中，我们要将 SQLite 用于本地存储。

>[AZURE.NOTE]你可以跳过此部分，直接从移动服务的 GitHub 示例存储库中获取已有脱机支持的示例项目。启用了脱机支持的示例项目位于此处：[TodoList 脱机示例]。

1. 安装适用于 Windows 8.1 和 Windows Phone 8.1 的 SQLite 运行时。 

    * **Windows 8.1 运行时：**安装 [SQLite for Windows 8.1]。
    * **Windows Phone 8.1：**安装 [SQLite for Windows Phone 8.1]。

    >[AZURE.NOTE]如果你使用的是 Internet Explorer，则单击用于安装 SQLite 的链接可能会提示你下载 .zip 文件形式的 .vsix。使用 .vsix 扩展名而不是 .zip 将文件保存到硬盘上的某一位置。在 Windows 资源管理器中双击 .vsix 文件以运行安装程序。

2. 在 Visual Studio 中，打开在[移动服务入门]教程中完成的项目。安装适用于 Windows 8.1 运行时和 Windows Phone 8.1 项目的 **WindowsAzure.MobileServices.SQLiteStore** NuGet 包。

    * **Windows 8.1：**在解决方案资源管理器中，右键单击 Windows 8.1 项目，然后单击“管理 Nuget 包”以运行 NuGet 包管理器。搜索 **SQLiteStore** 以安装 `WindowsAzure.MobileServices.SQLiteStore` 包。
    * **Windows Phone 8.1：**右键单击 Windows Phone 8.1 项目，然后单击“管理 Nuget 包”以运行 NuGet 包管理器。搜索 **SQLiteStore** 以安装 `WindowsAzure.MobileServices.SQLiteStore` 包。

    >[AZURE.NOTE]如果安装过程中创建了对较旧版本的 SQLite 的引用，可以直接删除该重复引用。

    ![][2]

3. 在解决方案资源管理器中，右键单击适用于 Windows 8.1 运行时和 Windows Phone 8.1 平台项目的“引用”，并确保存在对位于“扩展”部分的 SQLite 的引用。

    ![][1] </br>

    **Windows 8.1 运行时**

    ![][11] </br>

    **Windows Phone 8.1**

4. SQLite 运行时要求你将所生成的项目的处理器体系结构更改为“x86”、“x64”或“ARM”。不支持“任何 CPU”。在解决方案资源管理器中，单击顶部的“解决方案”，然后将处理器体系结构下拉框更改为要测试的受支持设置之一。

    ![][13]

5. 在解决方案资源管理器中，在共享项目中打开 MainPage.cs 文件。使用文件顶部的语句取消注释以下内容：

        using Microsoft.WindowsAzure.MobileServices.SQLiteStore;  // offline sync
        using Microsoft.WindowsAzure.MobileServices.Sync;         // offline sync

6. 在 MainPage.cs 中，注释 `todoTable` 的定义，取消注释以下调用 `MobileServicesClient.GetSyncTable()` 的行上的定义：

        //private IMobileServiceTable<TodoItem> todoTable = App.MobileService.GetTable<TodoItem>();
        private IMobileServiceSyncTable<TodoItem> todoTable = App.MobileService.GetSyncTable<TodoItem>(); // offline sync


7. 在 MainPage.cs 中，在标记为 `Offline sync` 的区域中取消注释方法 `InitLocalStoreAsync` 和 `SyncAsync`。方法 `InitLocalStoreAsync` 使用 SQLite 存储初始化客户端同步上下文。

        private async Task InitLocalStoreAsync()
        {
            if (!App.MobileService.SyncContext.IsInitialized)
            {
                var store = new MobileServiceSQLiteStore("localstore.db");
                store.DefineTable<TodoItem>();
                await App.MobileService.SyncContext.InitializeAsync(store);
            }

            await SyncAsync();
        }

        private async Task SyncAsync()
        {
            await App.MobileService.SyncContext.PushAsync();
            await todoTable.PullAsync("todoItems", todoTable.CreateQuery());
        }

8. 在 `OnNavigatedTo` 事件处理程序中，取消注释对 `InitLocalStoreAsync` 的调用：

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            await InitLocalStoreAsync(); // offline sync
            await RefreshTodoItems();
        }

9. 在方法 `InsertTodoItem`、`UpdateCheckedTodoItem` 和 `ButtonRefresh_Click` 中取消注释对 `SyncAsync` 的 3 次调用：

        private async Task InsertTodoItem(TodoItem todoItem)
        {
            await todoTable.InsertAsync(todoItem);
            items.Add(todoItem);

            await SyncAsync(); // offline sync
        }

        private async Task UpdateCheckedTodoItem(TodoItem item)
        {
            await todoTable.UpdateAsync(item);
            items.Remove(item);
            ListItems.Focus(Windows.UI.Xaml.FocusState.Unfocused);

            await SyncAsync(); // offline sync
        }

        private async void ButtonRefresh_Click(object sender, RoutedEventArgs e)
        {
            ButtonRefresh.IsEnabled = false;

            await SyncAsync(); // offline sync
            await RefreshTodoItems();

            ButtonRefresh.IsEnabled = true;
        }

10. 在 `SyncAsync` 方法中添加异常处理程序：

        private async Task SyncAsync()
        {
            String errorString = null;

            try
            {
                await App.MobileService.SyncContext.PushAsync();
                await todoTable.PullAsync("todoItems", todoTable.CreateQuery()); // first param is query ID, used for incremental sync
            }

            catch (MobileServicePushFailedException ex)
            {
                errorString = "Push failed because of sync errors: " +
                  ex.PushResult.Errors.Count + " errors, message: " + ex.Message;
            }
            catch (Exception ex)
            {
                errorString = "Pull failed: " + ex.Message +
                  "\n\nIf you are still in an offline scenario, " +
                  "you can try your Pull again when connected with your Mobile Serice.";
            }

            if (errorString != null)
            {
                MessageDialog d = new MessageDialog(errorString);
                await d.ShowAsync();
            }
        }

    在此示例中，我们将检索远程 `todoTable` 中的所有记录，但也可以通过传递查询来筛选记录。`PullAsync` 的第一个参数是用于增量同步的查询 ID；增量同步使用 `UpdatedAt` 时间戳以仅获取自上次同步以来修改的记录。查询 ID 应对于你的应用程序中的每个逻辑查询都是唯一的描述性字符串。若选择不要增量同步，请传递 `null` 作为查询 ID。此命令会检索每个请求的操作，这是可能效率低下上的所有记录。

    >[AZURE.NOTE]* 若要从设备本地存储中删除已在移动设备数据库中删除的记录，应启用“[软删除]”。否则，你的应用程序应定期调用 `IMobileServiceSyncTable.PurgeAsync()` 以清除本地存储。

    请注意，推送和请求操作可能会发生 `MobileServicePushFailedException`。由于拉取操作会内部执行推送来确保所有表及所有关系都一致，因此也可能发生该调用。下一篇教程[使用移动服务脱机支持处理冲突]说明了如何处理这些同步相关的异常。

11. 在 Visual Studio 中，按 **F5** 键重新生成并运行应用程序。应用程序的行为将与其脱机同步更改前相同，因为它对插入、更新和刷新操作执行了同步操作。

## <a name="update-sync"></a>更新应用程序的同步行为

在本部分中，你将修改应用，使其不对插入和更新操作同步，而仅在按“刷新”按钮时同步。然后，将中断与移动服务以模拟脱机情况下的应用程序连接。在添加数据项时，数据项将保存在本地存储区中，而不同步到移动服务。

1. 打开共享项目中的 MainPage.cs。编辑方法 `InsertTodoItem` 和 `UpdateCheckedTodoItem` 以注释掉对 `SyncAsync` 的调用。

2. 编辑共享项目中的 App.xaml.cs。注释掉 **MobileServiceClient** 的初始化并添加使用无效移动服务 URL 的以下行：

         public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://your-mobile-service.azure-mobile.xxx/",
            "AppKey"
        );

3. 在 `InitLocalStoreAsync()` 中，注释掉对 `SyncAsync()` 的调用，使应用不在启动时执行同步。

4. 按 **F5** 生成并运行应用。输入一些新的 todo 项目，然后为每个项目单击“保存”。新的 Todo 项目在推送到移动服务之前，只存在于本地存储中。客户端应用程序的行为就像它已连接到支持所有创建、读取、更新、删除 (CRUD) 操作的移动服务一样。

5. 关闭应用程序并重新启动它，以验证你创建的新项目是否已永久保存到本地存储中。

## <a name="update-online-app"></a>更新应用程序以重新连接移动服务

在本节中，你会将应用程序重新连接到移动服务。这模拟的是通过移动服务从脱机状态转为联机状态的应用程序。当您按“刷新”按钮时，数据将同步到您的移动服务。

1. 打开共享项目中的 App.xaml.cs。取消注释之前的 `MobileServiceClient` 初始化，重新添加正确的移动服务 URL 和应用密钥。

2. 按 **F5** 键重新生成并运行应用。请注意，数据看上去与脱机情况下相同，即使应用程序现已连接到移动服务。这是因为此应用程序始终使用指向本地存储区的 `IMobileServiceSyncTable`。

3. 登录到 [Azure 经典门户]，查看你的移动服务数据库。如果你的服务将 JavaScript 后端用于移动服务，则可以通过移动服务的“数据”选项卡来游览数据。

    如果将 .NET 后端用于移动服务，请在 Visual Studio 中，转到“服务器资源管理器”->“Azure”->“SQL 数据库”。右键单击数据库并选择“在 SQL Server 对象资源管理器中打开”。

    请注意，数据尚未在数据库和本地存储之间进行同步。

    ![][6]

4. 在应用中，按“刷新”按钮。这将导致应用调用 `PushAsync` 和 `PullAsync`。此推送操作会将本地存储项发送到移动服务，然后从移动服务检索新数据。

    推送操作从 `MobileServiceClient.SyncContext` 而非 `IMobileServicesSyncTable` 执行，并将更改推送到与该同步上下文关联的所有表中。这是为了应对表之间存在关系的情况。

    ![][7]

5. 在应用程序中，单击要在本地存储区中完成的几个项旁边的复选框。

    ![][8]

6. 再次按“刷新”按钮，这将导致调用 `SyncAsync`。`SyncAsync` 同时调用推送和拉取，但在本例中，我们可能已删除了对 `PushAsync` 的调用。这是因为“拉取时始终先执行推送操作”。这是为了确保本地存储中的所有表以及关系都保持一致。

    ![][10]
  

##摘要

[AZURE.INCLUDE [mobile-services-offline-summary-csharp](../includes/mobile-services-offline-summary-csharp.md)]

## 后续步骤

* [使用移动服务脱机支持处理冲突]

* [使用移动服务中的软删除](/documentation/articles/mobile-services-using-soft-delete/)

<!-- Anchors. -->
[Update the app to support offline features]: #enable-offline-app
[Update the sync behavior of the app]: #update-sync
[Update the app to reconnect your mobile service]: #update-online-app
[Next Steps]: #next-steps

<!-- Images -->
[1]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-add-reference-sqlite-dialog.png
[2]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-sqlitestore-nuget.png
[6]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-data-browse.png
[7]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-data-browse2.png
[8]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-online-app-run2.png
[10]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-data-browse3.png
[11]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/mobile-services-add-wp81-reference-sqlite-dialog.png
[12]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/new-synchandler-class.png
[13]: ./media/mobile-services-windows-store-dotnet-get-started-offline-data/cpu-architecture.png


<!-- URLs. -->
[使用移动服务脱机支持处理冲突]: /documentation/articles/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/
[TodoList 脱机示例]: http://go.microsoft.com/fwlink/?LinkId=394777
[Get started with Mobile Services]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/#create-new-service
[Getting Started]: /documentation/articles/mobile-services-dotnet-backend-windows-phone-get-started/
[移动服务入门]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started/
[SQLite for Windows 8.1]: http://go.microsoft.com/fwlink/?LinkId=394776
[SQLite for Windows Phone 8.1]: http://go.microsoft.com/fwlink/?LinkId=397953
[软删除]: /documentation/articles/mobile-services-using-soft-delete/



[Mobile Services SDK Nuget]: http://www.nuget.org/packages/WindowsAzure.MobileServices/1.3.0
[SQLite store nuget]: http://www.nuget.org/packages/WindowsAzure.MobileServices.SQLiteStore/1.0.0
[Azure 经典门户]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0118_2016-->