<properties
	pageTitle="在移动服务中使用脱机数据 (Xamarin Android) | Azure"
	description="了解如何使用 Azure 移动服务向 Xamarin.android 应用程序中的缓存和同步离线数据"
	documentationCenter="xamarin"
	authors="lindydonna"
	editor="wesmc"
	manager="dwrede"
	services="mobile-services"/>

<tags
	ms.service="mobile-services"
	ms.date="03/16/2016"
	wacn.date="05/23/2016"/>

#  在移动服务中使用脱机数据同步

[AZURE.INCLUDE [mobile-services-selector-offline](../includes/mobile-services-selector-offline.md)]


本主题将指导你通过 Azure 移动服务的脱机同步功能在 todo 列表快速入门应用程序中。脱机同步可轻松地创建应用程序即使在最终用户不具有任何网络访问权限时才可用。

脱机同步具有几种可能的用法：

* 通过缓存在设备上的本地服务器数据来提高应用程序响应能力
* 使应用程序可灵活地应对间歇性网络连接
* 允许最终用户创建和修改数据，甚至在没有网络访问权限，并支持方案具有很少或没有连接时
* 跨多个设备同步数据和同一个记录修改由两个设备时检测冲突

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并取得多达 10 个免费的移动服务，即使在试用期结束之后仍可继续使用这些服务。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)</a>。
>
> 如果这是你第一次体验移动服务，你应首先完成[移动服务入门]。

本教程将指导你完成以下基本步骤：

1. [查看移动服务同步代码]
2. [更新应用程序的同步行为]
3. [更新应用程序以重新连接移动服务]

本教程需要的内容如下：

* Windows 上的 Visual Studio with Xamarin，或 Mac OS X 上的 Xamarin Studio。[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx) 中提供了完整的安装说明。
* 完成 [Get started with Mobile Services（移动服务入门）]教程。
 
## <a name="review-offline"></a>查看移动服务同步代码

Azure 移动服务脱机同步允许最终用户在无法访问网络时与本地数据库交互。若要在你的应用程序中使用这些功能，请将 `MobileServiceClient.SyncContext` 初始化到本地存储。然后，通过 `IMobileServiceSyncTable` 接口引用你的表。 
本部分将指导完成脱机同步 `ToDoActivity.cs` 中的相关代码。

1. 在 Visual Studio 或 Xamarin Studio 中，打开你在完成[移动服务入门]教程后创建的项目。打开 `ToDoActivity.cs` 文件。

2. 请注意成员 `toDoTable` 的类型是 `IMobileServiceSyncTable`。脱机同步使用此同步表接口而不是 `IMobileServiceTable`。当使用同步表时，所有操作转到本地存储区，并仅同步与远程服务与显式的推和请求操作。

    若要获取对同步表的引用，请使用 `GetSyncTable()` 方法。若要删除脱机同步功能，应改用 `GetTable()`。

3. 表操作之前，必须初始化本地存储区。这可以在 `InitLocalStoreAsync` 方法中完成：

        private async Task InitLocalStoreAsync()
        {
            // new code to initialize the SQLite store
            string path = Path.Combine(System.Environment.GetFolderPath(System.Environment.SpecialFolder.Personal), localDbFilename);

            if (!File.Exists(path))
            {
                File.Create(path).Dispose();
            }

            var store = new MobileServiceSQLiteStore(path);
            store.DefineTable<ToDoItem>();

            // Uses the default conflict handler, which fails on conflict
            await client.SyncContext.InitializeAsync(store);
        }

    这将使用移动服务 SDK 中提供的类 `MobileServiceSQLiteStore` 创建本地存储。你还可以通过实现 `IMobileServiceLocalStore` 提供不同的本地存储实现。

    `DefineTable` 方法与所提供的类型中的字段相匹配的本地存储中创建一个表 `ToDoItem` 这种情况下。该类型无需包括所有的列，同时在远程数据库中就可以存储列的子集。

    此重载 `InitializeAsync` 使用默认冲突处理程序，只要有冲突，则将失败。若要提供自定义冲突处理程序，请参阅教程[处理脱机支持的移动服务与冲突]。

4. 方法 `SyncAsync` 触发实际同步操作：

        private async Task SyncAsync()
        {
            await client.SyncContext.PushAsync();
            await toDoTable.PullAsync("allTodoItems", toDoTable.CreateQuery()); // query ID is used for incremental sync
        }

    首先，将调用 `IMobileServiceSyncContext.PushAsync()`。此方法属于 `IMobileServicesSyncContext` 而不是同步表，因为它会将更改推送到所有表中。只有已在本地以某种方式修改（通过 CUD 操作来完成）的记录才会发送到服务器。

    接下来，该方法调用 `IMobileServiceSyncTable.PullAsync()` 向应用程序的服务器上表中提取数据。请注意是否有任何挂起的更改的同步上下文中，请求始终先发出推送操作。这是为了确保一致的本地存储区以及关系中的所有表。在这种情况下，我们必须显式调用推送。

    在此示例中，我们检索远程中的所有记录 `TodoItem` 表中，但它也可能是要作为筛选依据传递查询的记录。`PullAsync()` 的第一个参数是用于增量同步的查询 ID；增量同步使用 `UpdatedAt` 时间戳以仅获取自上次同步以来修改的那些记录。查询 ID 应对于你的应用程序中的每个逻辑查询都是唯一的描述性字符串。若选择不要增量同步，请传递 `null` 作为查询 ID。此命令会检索每个请求的操作，这是可能效率低下上的所有记录。

    >[AZURE.NOTE]若要从设备本地存储区中删除已在移动设备数据库中删除的记录，应启用[软删除]。否则，你的应用程序应定期调用 `IMobileServiceSyncTable.PurgeAsync()` 以清除本地存储。

    请注意，推送和请求操作可能会发生 `MobileServicePushFailedException`。
    
    下一篇教程[使用移动服务脱机支持处理冲突]说明了如何处理这些同步相关的异常。

5. 在 `ToDoActivity` 类中，`SyncAsync()` 方法之后修改数据的操作，将调用 `AddItem()` 和 `CheckItem()`。它也称为从 `OnRefreshItemsSelected()`，以便用户获取最新数据，只要它们推送“刷新”按钮。该应用程序还执行同步启动，因为 `ToDoActivity.OnCreate()` 调用 `OnRefreshItemsSelected()`。

    因为 `SyncAsync()` 只要修改数据时，就将调用此应用程序假定用户处于联机状态，只要他们正在编辑的数据。在下一部分中，我们将更新应用程序，以便用户可以编辑甚至当它们处于脱机状态。

##  <a name="update-sync"></a>更新应用程序的同步行为

在本部分中，将修改应用程序，以便它不会同步应用程序启动或 insert 和 update 操作，而只推送刷新按钮。然后，将中断与移动服务以模拟脱机情况下的应用程序连接。在添加数据项时，它们将保存在本地存储区，但不是会立即同步到移动服务。

1. 在 `ToDoActivity` 类中，编辑方法 `AddItem()` 和 `CheckItem()`，并注释掉对 `SyncAsync()` 的调用。

2. 在 `ToDoActivity` 中，注释掉成员 `applicationURL` 和 `applicationKey` 的定义。添加以下行，通过引用无效的移动服务 URL：

        const string applicationURL = @"https://your-mobile-service.azure-mobile.xxx/";
        const string applicationKey = @"AppKey";

3. 在 `ToDoActivity.OnCreate()` 中，删除对 `OnRefreshItemsSelected()` 的调用并将其替换为：

        // Load the items from the Mobile Service
        // OnRefreshItemsSelected (); // don't sync on app launch
        await RefreshItemsFromTableAsync(); // load UI only

4. 构建并运行应用程序。添加一些新的 todo 项。新的 Todo 项目在推送到移动服务之前，只存在于本地存储中。客户端应用程序的行为就像它已连接到支持所有创建、读取、更新、删除 (CRUD) 操作的移动服务一样。

5. 关闭应用程序并重新启动它，以验证你创建的新项目是否已永久保存到本地存储中。

##  <a name="update-online-app"></a>更新应用程序以重新连接移动服务

在本节中，你会将应用程序重新连接到移动服务。这模拟的是通过移动服务从脱机状态转为联机状态的应用程序。推送“刷新”按钮时，数据将同步到你的移动服务。

1. 打开 `ToDoActivity.cs`。删除无效的移动服务 URL，并添加回正确的 URL 和应用程序密钥。

2. 构建并运行应用程序。请注意，数据看上去与脱机情况下相同，即使应用程序现已连接到移动服务。这是因为此应用程序始终使用指向本地存储的 `IMobileServiceSyncTable`。

3. 登录到 [Azure 管理门户]，查看你的移动服务数据库。如果服务使用 JavaScript 后端，则你可以从移动服务的“数据”选项卡浏览数据。

    如果将 .NET 后端用于移动服务，请在 Visual Studio 中，转到“服务器资源管理器”>“Azure”>“SQL 数据库”。右键单击数据库并选择“在 SQL Server 对象资源管理器中打开”。

    请注意，数据尚未在数据库和本地存储之间同步。

4. 在应用程序中，按刷新按钮。这将调用 `OnRefreshItemsSelected()`，从而又会调用 `SyncAsync()`。这将执行推送和请求的操作，首先将本地存储项发送到移动服务中，然后从服务中检索新数据。

5. 请检查你的移动服务以确认更改都已同步的数据库。

## 摘要

[AZURE.INCLUDE [mobile-services-offline-summary-csharp](../includes/mobile-services-offline-summary-csharp.md)]

##  后续步骤

* [使用移动服务脱机支持处理冲突]


<!-- Anchors. -->
[查看移动服务同步代码]: #review-offline
[更新应用程序的同步行为]: #update-sync
[更新应用程序以重新连接移动服务]: #update-online-app

<!-- Images -->


<!-- URLs. -->
[使用移动服务脱机支持处理冲突]: /documentation/articles/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/
[处理脱机支持的移动服务与冲突]: /documentation/articles/mobile-services-windows-store-dotnet-handling-conflicts-offline-data/
[Get started with Mobile Services（移动服务入门）]: /documentation/articles/partner-xamarin-mobile-services-android-get-started/
[移动服务入门]: /documentation/articles/partner-xamarin-mobile-services-android-get-started/
[软删除]: /documentation/articles/mobile-services-using-soft-delete/
[Mobile Services SDK Nuget]: http://www.nuget.org/packages/WindowsAzure.MobileServices/1.3.0
[SQLite store nuget]: http://www.nuget.org/packages/WindowsAzure.MobileServices.SQLiteStore/1.0.0
[Azure 管理门户]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0118_2016-->