<properties
	pageTitle="使用移动应用为通用 Windows 平台 (UWP) 应用启用脱机同步 | Azure 应用服务"
	description="了解如何在通用 Windows 平台 (UWP) 应用中使用 Azure 移动应用缓存和同步脱机数据。"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	editor=""
	services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.date="05/14/2016"
	wacn.date="10/17/2016"/>

# 为 Windows 应用启用脱机同步

[AZURE.INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## 概述

本教程演示如何使用 Azure 移动应用后端为通用 Windows 平台 (UWP) 应用添加脱机支持。脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。更改存储在本地数据库中；设备重新联机后，这些更改会与远程后端同步。

在本教程中，将更新[创建 Windows 应用]教程中的 UWP 应用项目，以支持 Azure 移动应用的脱机功能。如果不使用下载的快速入门服务器项目，必须将数据访问扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

若要了解有关脱机同步功能的详细信息，请参阅主题 [Azure 移动应用中的脱机数据同步]。

## 要求

本教程需要的内容如下：

* 在 Windows 8.1 上运行的 Visual Studio 2013。
* 完成[创建 Windows 应用][create a windows app]。
* [Azure 移动服务 SQLite Store][sqlite store nuget]
* [SQLite for Universal Windows Platform](http://www.sqlite.org/downloads)（适用于通用 Windows 平台的 SQLite）

## 更新客户端应用以支持脱机功能

脱机情况下，可使用 Azure 移动应用脱机功能与本地数据库交互。若要在应用中使用这些功能，请将 [SyncContext][synccontext] 初始化到本地存储。然后，通过 [IMobileServiceSyncTable][IMobileServiceSyncTable] 接口引用表。SQLite 在设备上用作本地存储。

1. 安装[适用于通用 Windows 平台的 SQLite 运行时](http://sqlite.org/2016/sqlite-uwp-3120200.vsix)。

2. 在 Visual Studio 中，打开在[创建 Windows 应用]教程中完成的 UWP 应用项目的 NuGet 包管理器，然后搜索并安装 **Microsoft.Azure.Mobile.Client.SQLiteStore** NuGet 包。

4. 在解决方案资源管理器中，右键单击“引用”>“添加引用...”>“通用 Windows”>“扩展”，然后同时启用“适用于通用 Windows 平台的 SQLite”和“适用于通用 Windows 平台应用的 Visual C++ 2015 运行时”。

    ![添加 SQLite UWP 引用][1]

5. 打开 MainPage.xaml.cs 文件，取消注释文件顶部的以下 `using` 语句：

        using Microsoft.WindowsAzure.MobileServices.SQLiteStore;  
        using Microsoft.WindowsAzure.MobileServices.Sync;         

6. 注释将 `todoTable` 初始化为 **IMobileServiceTable** 的代码行，然后取消注释将 `todoTable` 初始化为 **IMobileServiceSyncTable** 的代码行：

        //private IMobileServiceTable<TodoItem> todoTable = App.MobileService.GetTable<TodoItem>();
        private IMobileServiceSyncTable<TodoItem> todoTable = App.MobileService.GetSyncTable<TodoItem>(); 

7. 在 MainPage.xaml.cs 的 `Offline sync` 区域中，取消注释以下方法：

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

	在 `SyncAsync` 中从 [SyncContext][synccontext] 启动推送操作，后跟增量同步。同步上下文跟踪客户端对所有表进行的更改。有关此行为的详细信息，请参阅 [Azure 移动应用中的脱机数据同步]。

8. 在 `OnNavigatedTo` 事件处理程序中，取消注释对 `InitLocalStoreAsync` 的调用。如果已完成[身份验证教程](/documentation/articles/app-service-mobile-windows-store-dotnet-get-started-users/)，必须改为在 `AuthenticateAsync` 方法中执行此操作。

9. 在 `InsertTodoItem` 和 `UpdateCheckedTodoItem` 方法中取消注释对 [PushAsync] 的调用，然后在 `ButtonRefresh_Click` 方法中取消注释对 `SyncAsync` 的调用。

10. 在 `SyncAsync` 方法中，添加以下异常处理程序：

        private async Task SyncAsync()
        {
            String errorString = null;

            try
            {
                await App.MobileService.SyncContext.PushAsync();
                await todoTable.PullAsync("todoItems", todoTable.CreateQuery()); 
            }

            catch (MobileServicePushFailedException ex)
            {
                errorString = "Push failed because of sync errors. You may be offine.\nMessage: " +
                  ex.Message + "\nPushResult.Status: " + ex.PushResult.Status.ToString();
            }
            catch (Exception ex)
            {
                errorString = "Pull failed: " + ex.Message +
                  "\n\nIf you are still in an offline scenario, " +
                  "you can try your pull again when connected with your backend.";
            }

            if (errorString != null)
            {
                MessageDialog d = new MessageDialog(errorString);
                await d.ShowAsync();
            }
        }

	在脱机情况下，[PullAsync][PullAsync] 可能会导致 [MobileServicePushFailedException][MobileServicePushFailedException]，其 [PushResult.Status][Status] 为 [CancelledByNetworkError][CancelledByNetworkError]。当隐式推送尝试在拉取前推送未完成的更新并且失败时，会发生这种情况。有关详细信息，请参阅 [Azure 移动应用中的脱机数据同步]。

11. 在 Visual Studio 中，按 **F5** 键重新生成并运行客户端应用。应用的工作方式与启用脱机同步之前一样。但是，本地数据库中现在填充了可以在脱机方案中使用的数据。接下来，创建脱机方案，并使用本地存储的数据来启动应用。

## <a name="update-sync"></a>更新应用以与后端断开连接

在本部分中，将断开与移动应用后端的连接，以模拟脱机情况。添加数据项时，异常处理程序将指示该应用处于脱机模式。在此状态下，新项将添加到本地存储，下次以连接状态运行推送时，这些新项将同步到移动应用后端。

1. 编辑共享项目中的 App.xaml.cs。注释掉 **MobileServiceClient** 的初始化并添加使用无效移动应用 URL 的以下行：

         public static MobileServiceClient MobileService =
				new MobileServiceClient("https://your-service.azurewebsites.fail");

	请注意，当应用还在使用身份验证时，这会导致登录失败。还可以通过在设备上禁用 wifi 和手机网络或使用飞行模式来演示脱机行为。

2. 按 **F5** 生成并运行应用。请注意，在应用启动时，同步刷新将失败。
 
3. 输入新项，并注意每次单击“保存”时，推送将失败，并显示 [CancelledByNetworkError] 状态。但是，新的待办事项在推送到移动应用后端之前，存在于本地存储中。

	在生产应用中，如果取消显示这些异常，客户端应用的行为就像它仍连接到移动应用后端一样。

4. 关闭应用程序并重新启动它，以验证你创建的新项目是否已永久保存到本地存储中。

5. （可选）在 Visual Studio 中，打开“服务器资源管理器”。导航到“Azure”->“SQL 数据库”中的数据库。右键单击数据库并选择“在 SQL Server 对象资源管理器中打开”。现在便可以浏览 SQL 数据库表及其内容。验证后端数据库中的数据是否未更改。

6. （可选）通过 Fiddler 或 Postman 之类的 REST 工具使用 `https://<your-mobile-app-backend-name>.chinacloudsites.cn/tables/TodoItem` 格式的 GET 查询，查询移动后端。

## <a name="update-online-app"></a>更新应用以重新连接移动应用后端

在本部分中，会将应用重新连接到移动应用后端。这模拟的是通过移动应用后端从脱机状态转为联机状态的应用。首次运行该应用程序时，`OnNavigatedTo` 事件处理程序将调用 `InitLocalStoreAsync`。而此程序又将调用 `SyncAsync`，将本地存储与后端数据库同步。因此，该应用将尝试在启动时进行同步。

1. 在共享项目中，打开 App.xaml.cs，并取消注释 `MobileServiceClient` 之前的初始化，以使用正确的移动应用 URL。

2. 按 **F5** 键重新生成并运行应用。`OnNavigatedTo` 事件处理程序执行时，该应用将立即使用推送和拉取操作将本地更改与 Azure 移动应用后端同步。

3. （可选）使用 SQL Server 对象资源管理器或 Fiddler 之类的 REST 工具查看更新后的数据。请注意，数据已在 Azure 移动应用后端数据库和本地存储之间进行同步。

4. 在应用程序中，单击要在本地存储区中完成的几个项旁边的复选框。

  `UpdateCheckedTodoItem` 调用 `SyncAsync`，将每个已完成项与移动应用后端进行同步。`SyncAsync` 同时调用推送和拉取操作。但是，应注意**每当对客户端已更改的表执行拉取操作时，始终先对客户端同步上下文自动执行推送操作**。这是为了确保本地存储中的所有表以及关系都保持一致。因此，在这种情况下，我们可以删除对 `PushAsync` 的调用，因为它在执行拉取操作时自动执行。如果不注意它，此行为可能会导致意外的推送。有关此行为的详细信息，请参阅 [Azure 移动应用中的脱机数据同步]。


##API 摘要

为了支持移动服务的脱机功能，我们使用了 [IMobileServiceSyncTable] 接口，并使用本地 SQLite 数据库初始化了 [MobileServiceClient.SyncContext][synccontext]。脱机时，移动应用的普通 CRUD 操作执行起来就像此应用仍处于连接状态一样，但操作针对本地存储进行。以下方法用于将本地存储与服务器进行同步：

*  **[PushAsync]** 
   由于此方法是 [IMobileServicesSyncContext] 的成员，因此对所有表进行的更改将推送到后端。只有具有本地更改的记录将发送到服务器。

* **[PullAsync]** 
   从 [IMobileServiceSyncTable] 启动拉取操作。当表中存在被跟踪的更改时，会执行隐式推送操作以确保本地存储中的所有表以及关系都保持一致。*PushOtherTables* 参数控制在隐式推送操作中是否推送上下文中的其他表。*query* 参数使用 [IMobileServiceTableQuery&lt;U&gt;][IMobileServiceTableQuery] 或 OData 查询字符串来筛选返回的数据。*queryId* 参数用于定义增量同步。有关详细信息，请参阅 [Azure 移动应用中的脱机数据同步](/documentation/articles/app-service-mobile-offline-data-sync/#how-sync-works)。

* **[PurgeAsync]** 
应用应定期调用此方法，从本地存储中清除过时数据。需要清除尚未同步的任何更改时，请使用 *force* 参数。

有关这些概念的详细信息，请参阅 [Azure 移动应用中的脱机数据同步](/documentation/articles/app-service-mobile-offline-data-sync/#how-sync-works)。

## 更多信息

以下主题提供有关移动应用的脱机同步功能的更多背景信息：

* [Azure 移动应用中的脱机数据同步]

<!-- Anchors. -->
[Update the app to support offline features]: #enable-offline-app
[Update the sync behavior of the app]: #update-sync
[Update the app to reconnect your Mobile Apps backend]: #update-online-app
[Next Steps]: #next-steps

<!-- Images -->
[1]: ./media/app-service-mobile-windows-store-dotnet-get-started-offline-data/app-service-mobile-add-reference-sqlite-dialog.png
[11]: ./media/app-service-mobile-windows-store-dotnet-get-started-offline-data/app-service-mobile-add-wp81-reference-sqlite-dialog.png
[13]: ./media/app-service-mobile-windows-store-dotnet-get-started-offline-data/cpu-architecture.png


<!-- URLs. -->
[Azure 移动应用中的脱机数据同步]: /documentation/articles/app-service-mobile-offline-data-sync/
[create a windows app]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started/
[创建 Windows 应用]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started/
[SQLite for Windows 8.1]: http://go.microsoft.com/fwlink/?LinkID=716919
[SQLite for Windows Phone 8.1]: http://go.microsoft.com/fwlink/?LinkID=716920
[SQLite for Windows 10]: http://go.microsoft.com/fwlink/?LinkID=716921
[synccontext]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient.synccontext(v=azure.10).aspx
[sqlite store nuget]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client.SQLiteStore/
[IMobileServiceSyncTable]: https://msdn.microsoft.com/zh-cn/library/azure/mt691742(v=azure.10).aspx
[IMobileServiceTableQuery]: https://msdn.microsoft.com/zh-cn/library/azure/dn250631(v=azure.10).aspx
[IMobileServicesSyncContext]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.sync.imobileservicesynccontext(v=azure.10).aspx
[MobileServicePushFailedException]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.sync.mobileservicepushfailedexception(v=azure.10).aspx
[Status]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.sync.mobileservicepushcompletionresult.status(v=azure.10).aspx
[CancelledByNetworkError]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.sync.mobileservicepushstatus(v=azure.10).aspx
[PullAsync]: https://msdn.microsoft.com/zh-cn/library/azure/mt667558(v=azure.10).aspx
[PushAsync]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileservicesynccontextextensions.pushasync(v=azure.10).aspx
[PurgeAsync]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.sync.imobileservicesynctable.purgeasync(v=azure.10).aspx

<!---HONumber=Mooncake_0919_2016-->