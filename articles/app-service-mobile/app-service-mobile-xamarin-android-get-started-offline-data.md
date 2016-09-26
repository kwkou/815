<properties
    pageTitle="为 Azure 移动应用启用脱机同步 (Xamarin Android)"
    description="了解如何在 Xamarin Android 应用程序中使用应用服务移动应用缓存和同步脱机数据"
    documentationCenter="xamarin"
    authors="wesmc7777"
    manager="dwrede"
    editor=""
    services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.date="05/05/2016"
	wacn.date="09/26/2016"/>

# 为 Xamarin.Android 移动应用启用脱机同步

[AZURE.INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## 概述

本教程介绍适用于 Xamarin.Android 的 Azure 移动应用的脱机同步功能。脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。更改存储在本地数据库中；设备重新联机后，这些更改会与远程服务同步。

在本教程中，将更新[创建 Xamarin Android 应用]教程中的客户端项目，以支持 Azure 移动应用的脱机功能。如果不使用下载的快速入门服务器项目，必须将数据访问扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

若要了解有关脱机同步功能的详细信息，请参阅主题 [Azure 移动应用中的脱机数据同步]。

## 查看客户端同步代码

完成教程[创建 Xamarin Android 应用]时下载的 Xamarin 客户端项目已包含使用本地 SQLite 数据库支持脱机同步的代码。下面简要概述了在教程代码中已包含的内容。有关功能的概念性概述，请参阅 [Azure 移动应用中的脱机数据同步]。

* 表操作之前，必须初始化本地存储区。`ToDoActivity.OnCreate()` 执行 `ToDoActivity.InitLocalStoreAsync()` 时，对本地存储数据库进行初始化。这将使用 Azure 移动应用客户端 SDK 提供的 `MobileServiceSQLiteStore` 类创建一个新的本地 SQLite 数据库。

	`DefineTable` 方法与所提供的类型中的字段相匹配的本地存储中创建一个表 `ToDoItem` 这种情况下。该类型无需包括远程数据库中的所有列。可以只存储列的子集。

		// ToDoActivity.cs
        private async Task InitLocalStoreAsync()
        {
            // new code to initialize the SQLite store
            string path = Path.Combine(System.Environment
				.GetFolderPath(System.Environment.SpecialFolder.Personal), localDbFilename);

            if (!File.Exists(path))
            {
                File.Create(path).Dispose();
            }

            var store = new MobileServiceSQLiteStore(path);
            store.DefineTable<ToDoItem>();

            // Uses the default conflict handler, which fails on conflict
            // To use a different conflict handler, pass a parameter to InitializeAsync.
			// For more details, see http://go.microsoft.com/fwlink/?LinkId=521416.
            await client.SyncContext.InitializeAsync(store);
        }


* `ToDoActivity` 的 `toDoTable` 成员属于 `IMobileServiceSyncTable` 类型而不是 `IMobileServiceTable` 类型。这会将所有创建、读取、更新和删除 (CRUD) 表操作定向到本地存储数据库。

	使用客户端连接的同步上下文调用 `IMobileServiceSyncContext.PushAsync()` 即可决定何时将这些更改推送到 Azure 移动应用后端。对于调用 `PushAsync` 时客户端应用修改的所有表，此同步上下文通过跟踪和推送这些表中的更改来帮助保持表关系。

	每当刷新 todoitem 列表或者添加或完成 todoitem 时，所提供的代码便会调用 `ToDoActivity.SyncAsync()` 进行同步。因此，它在每次本地更改后进行同步，对同步上下文执行推送，对同步表执行拉取。但是，一定要认识到，如果对具有由上下文跟踪的未完成本地更新的表执行拉取操作，该拉取操作将自动先触发上下文推送操作。因此，在这些情况（刷新、添加和完成项目）下，可以省略显式 `PushAsync` 调用。它是冗余的。

    在所提供的代码中，将查询远程 `TodoItem` 表中的所有记录，但它还可以筛选记录，只需将查询 ID 和查询传递给 `PushAsync` 即可。有关详细信息，请参阅 [Azure 移动应用中的脱机数据同步]中的*增量同步*部分。

	<!-- Need updated conflict handling info : `InitializeAsync` uses the default conflict handler, which fails whenever there is a conflict. To provide a custom conflict handler, see the tutorial [Handling conflicts with offline support for Mobile Services].
	-->


		// ToDoActivity.cs
        private async Task SyncAsync()
        {
			try {
	            await client.SyncContext.PushAsync();
	            await toDoTable.PullAsync("allTodoItems", toDoTable.CreateQuery()); // query ID is used for incremental sync
			} catch (Java.Net.MalformedURLException) {
				CreateAndShowDialog (new Exception ("There was an error creating the Mobile Service. Verify the URL"), "Error");
			} catch (Exception e) {
				CreateAndShowDialog (e, "Error");
			}
        }


## 运行客户端应用

至少运行一次客户端应用程序，以填充本地存储数据库。在下一部分中，将模拟脱机情况，并在应用处于脱机状态时修改本地存储中的数据。

## 更新客户端应用的同步行为

在本部分中，将修改客户端应用，通过对后端使用无效的应用程序 URL 来模拟脱机情况。添加或更改数据项时，这些更改将保存在本地存储中，但在重新建立连接之前，不会同步到后端数据存储中。

1. 在 `ToDoActivity.cs` 的顶部，更改 `applicationURL` 的初始化，使其指向无效的 URL：

        const string applicationURL = @"https://your-service.azurewebsites.fail/";

	请注意，当应用还在使用身份验证时，这会导致登录失败。还可以通过在设备上禁用 wifi 和手机网络或使用飞行模式来演示脱机行为。

2. 更新 `ToDoActivity.SyncAsync` 以捕获到 `MobileServicePushFailedException`，只需忽略该异常来假定处于脱机状态。

        private async Task SyncAsync()
        {
			try {
	            await client.SyncContext.PushAsync();
	            await toDoTable.PullAsync("allTodoItems", toDoTable.CreateQuery()); // query ID is used for incremental sync
			} catch (Java.Net.MalformedURLException) {
				CreateAndShowDialog (new Exception ("There was an error creating the Mobile Service. Verify the URL"), "Error");
			} catch (MobileServicePushFailedException)
            {
                // Not reporting this exception. Assuming the app is offline for now
            }
            catch (Exception e) {
				CreateAndShowDialog (e, "Error");
			}
        }


3. 构建并运行应用程序。如果显示报告名称解析异常的异常对话框，请关闭该对话框。添加一些新的待办事项，并注意应用的行为就像它已连接，因为处理 `MobileServicePushFailedException` 时未显示对话框。

4. 添加的新项目在推送到移动后端之前，只存在于本地存储中。关闭应用程序并重新启动它，以验证你创建的新项目是否已永久保存到本地存储中。

5. （可选）使用 Visual Studio 查看 Azure SQL 数据库表，以了解后端数据库中的数据是否未更改。

   	在 Visual Studio 中，打开“服务器资源管理器”。导航到“Azure”->“SQL 数据库”中的数据库。右键单击数据库并选择“在 SQL Server 对象资源管理器中打开”。现在便可以浏览 SQL 数据库表及其内容。

6. （可选）通过 Fiddler 或 Postman 之类的 REST 工具使用 `https://your-mobile-app-backend-name.chinacloudsites.cn/tables/TodoItem` 格式的 GET 查询，查询移动后端。


## 更新客户端应用以重新连接移动后端

在本部分中，会将应用重新连接到移动后端，以模拟重新回到联机状态的应用。执行刷新手势时，数据将同步到移动后端。

1. 打开 `ToDoActivity.cs`。更正 `applicationURL`，使其指向正确的 URL。

2. 构建并运行应用程序。该应用在启动后将尝试与 Azure 移动应用后端进行同步。确认创建了无异常对话框。

3. （可选）使用 SQL Server 对象资源管理器或 Fiddler 之类的 REST 工具查看更新后的数据。请注意，数据已在 Azure 移动应用后端数据库和本地存储之间进行同步。

    请注意，数据已在数据库和本地存储之间进行同步，并包含在脱机情况下添加的项目。

## 其他资源

* [Azure 移动应用中的脱机数据同步]

<!-- ##Summary

[AZURE.INCLUDE [mobile-services-offline-summary-csharp](../../includes/mobile-services-offline-summary-csharp.md)]

## Next steps

* [Handling conflicts with offline support for Mobile Services]

* [How to use the Xamarin Component client for Azure Mobile Services]
 -->

<!-- Images -->

<!-- URLs. -->
[创建 Xamarin Android 应用]: /documentation/articles/app-service-mobile-xamarin-android-get-started/
[Azure 移动应用中的脱机数据同步]: /documentation/articles/app-service-mobile-offline-data-sync/

[How to use the Xamarin Component client for Azure Mobile Services]: /documentation/articles/partner-xamarin-mobile-services-how-to-use-client-library/

[Xamarin Studio]: http://xamarin.com/download
[Xamarin extension]: http://xamarin.com/visual-studio

<!---HONumber=Mooncake_0919_2016-->