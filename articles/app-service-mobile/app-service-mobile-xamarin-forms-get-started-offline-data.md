<properties
    pageTitle="为 Azure 移动应用启用脱机同步 (Xamarin.Forms) | Azure"
    description="了解如何在 Xamarin.Forms 应用程序中使用应用服务移动应用缓存和同步脱机数据"
    documentationCenter="xamarin"
    authors="ggailey777"
    manager="erikre"
    editor=""
    services="app-service\mobile"/>

<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-xamarin-ios"
    ms.devlang="dotnet"
    ms.topic="article"
	ms.date="10/04/2016"
	wacn.date="09/26/2016"
    ms.author="adrianha"/>

# 为 Xamarin.Forms 移动应用启用脱机同步

[AZURE.INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## 概述
本教程介绍适用于 Xamarin.Forms 的 Azure 移动应用的脱机同步功能。脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。更改存储在本地数据库中；设备重新联机后，这些更改会与远程服务同步。

本教程基于在完成教程[创建 Xamarin iOS 应用]时创建的移动应用的 Xamarin.Forms 快速入门解决方案。Xamarin.Forms 的快速入门解决方案包含用于支持脱机同步的代码，只需启用即可使用。在本教程中，将更新快速入门解决方案，以打开 Azure 移动应用的脱机功能。还将重点介绍该应用中的脱机特定代码。如果不使用下载的快速入门解决方案，必须将数据访问扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

若要了解有关脱机同步功能的详细信息，请参阅主题 [Azure 移动应用中的脱机数据同步]。

## 启用快速入门解决方案中的脱机同步功能

脱机同步代码使用 C# 预处理器指令，包含在项目中。定义 **OFFLINE\_SYNC\_ENABLED** 符号时，这些代码路径包含在生成中。对于 Windows 应用，还必须安装 SQLite 平台。

1. 在 Visual Studio 中，右键单击解决方案 >“管理解决方案的 NuGet 程序包…”，然后在解决方案的所有项目中搜索并安装 **Microsoft.Azure.Mobile.Client.SQLiteStore** NuGet 包。

2. 在解决方案资源管理器中，从名称中包含 **Portable** 的项目（该项目是可移植类库项目）中打开 TodoItemManager.cs 文件，然后取消注释以下预处理器指令：

		#define OFFLINE_SYNC_ENABLED

3. 在解决方案资源管理器中，从 **Portable** 项目打开 TodoList.xaml.cs 文件，找到 **OnAppearing** 方法，并确保在调用 **RefreshItems** 时为 *syncItems* 传递了 `true`，如下所示：

		await RefreshItems(true, syncItems: true);
	这意味着该应用在启动时将尝试与后端进行同步。
 
4. （可选）如果要支持 iOS 设备，请从 **iOS** 项目打开 AppDelegate.cs 文件，然后在 **FinishedLaunching** 方法中取消注释以下代码行：

		SQLitePCL.CurrentPlatform.Init();

	此代码在 iOS 设备上初始化 SQLite 存储。剩余任务仅在要支持 Windows 设备时才需要。

5. （可选）若要支持 Windows 设备，请安装以下 SQLite 运行时包之一：

	* **Windows 8.1 运行时：**安装 [SQLite for Windows 8.1](http://go.microsoft.com/fwlink/p/?LinkID=716919)。
    * **Windows Phone 8.1：**安装 [SQLite for Windows Phone 8.1](http://go.microsoft.com/fwlink/p/?LinkID=716920)。
    * **通用 Windows 平台** 安装[适用于通用 Windows 平台的 SQLite](http://sqlite.org/2016/sqlite-uwp-3120200.vsix)。快速入门目前不包括通用 Windows 平台 (UWP) 项目。

6. （可选）在每个 Windows 应用项目中，右键单击“引用”>“添加引用...”，展开 **Windows** 文件夹 >“扩展”，然后启用相应的 **SQLite for Windows Runtime** 和 **Visual C++ 2013 Runtime for Windows** SDK。请注意，每个 Windows 平台的 SQLite SDK 名称略有不同。对于 UWP 项目，安装**适用于通用 Windows 平台应用的 Visual C++ 2015 运行时** SDK。


## 查看客户端同步代码

下面简要概述了在教程代码的 `#if OFFLINE_SYNC_ENABLED` 指令中已包含的内容。请注意，脱机同步功能在可移植类库项目的 TodoItemManager.cs 项目文件中。有关功能的概念性概述，请参阅 [Azure 移动应用中的脱机数据同步]。

* 表操作之前，必须初始化本地存储区。在 **TodoItemManager** 类构造函数中使用以下代码初始化本地存储数据库：

	    var store = new MobileServiceSQLiteStore("localstore.db");
        store.DefineTable<TodoItem>();

        //Initializes the SyncContext using the default IMobileServiceSyncHandler.
        this.client.SyncContext.InitializeAsync(store);

        this.todoTable = client.GetSyncTable<TodoItem>();

	此代码使用 **MobileServiceSQLiteStore** 类创建一个新的本地 SQLite 数据库。**DefineTable** 方法在本地存储中创建一个与所提供类型（在本例中为 `ToDoItem`）的字段匹配的表。该类型无需包括远程数据库中的所有列。可以只存储列的子集。

* **TodoItemManager** 中的 **todoTable** 字段是 **IMobileServiceSyncTable** 类型，而不是 **IMobileServiceTable**。此类使用本地数据库进行所有创建、读取、更新和删除 (CRUD) 表操作。用户决定何时通过对客户端使用的 **IMobileServiceSyncContext** 调用 **PushAsync** 将这些更改推送到移动应用后端。对于调用 **PushAsync** 时客户端应用修改的所有表，此同步上下文通过跟踪和推送这些表中的更改来帮助保持表关系。

	将调用以下 **SyncAsync** 方法来与移动应用后端进行同步：

		public async Task SyncAsync()
        {
            ReadOnlyCollection<MobileServiceTableOperationError> syncErrors = null;

            try
            {
                await this.client.SyncContext.PushAsync();

                await this.todoTable.PullAsync(
                    "allTodoItems",
                    this.todoTable.CreateQuery());
            }
            catch (MobileServicePushFailedException exc)
            {
                if (exc.PushResult != null)
                {
                    syncErrors = exc.PushResult.Errors;
                }
            }

            // Simple error/conflict handling. 
            if (syncErrors != null)
            {
                foreach (var error in syncErrors)
                {
                    if (error.OperationKind == MobileServiceTableOperationKind.Update && error.Result != null)
                    {
                        //Update failed, reverting to server's copy.
                        await error.CancelAndUpdateItemAsync(error.Result);
                    }
                    else
                    {
                        // Discard local change.
                        await error.CancelAndDiscardItemAsync();
                    }

                    Debug.WriteLine(@"Error executing sync operation. Item: {0} ({1}). Operation discarded.",
						error.TableName, error.Item["id"]);
                }
            }
        }

	此示例使用默认同步处理程序的简单错误处理。实际应用程序将使用自定义的 **IMobileServiceSyncHandler** 实现处理各种错误，如网络状况、服务器冲突等。

##脱机同步注意事项

在此示例中，仅在启动时和专门请求同步时才调用 **SyncAsync** 方法。若要在 Android 或 iOS 应用中启动同步，请下拉项目列表；对于 Windows，请使用“同步”按钮。在实际应用程序中，还可以在网络状态发生更改时触发此同步功能。

对具有由上下文跟踪的未完成本地更新的表执行拉取操作时，该拉取操作将自动触发在前面执行的上下文推送操作。在此示例中刷新、添加和完成项目时，可省略显式 **PushAsync** 调用，因为它是冗余的。

在所提供的代码中，将查询远程 TodoItem 表中的所有记录，但它还可以筛选记录，只需将查询 ID 和查询传递给 **PushAsync** 即可。有关详细信息，请参阅 [Azure 移动应用中的脱机数据同步]中的*增量同步*部分。


## 运行客户端应用

现已启用脱机同步，现在可以在每个平台上至少运行一次客户端应用程序，以填充本地存储数据库。稍后，将模拟脱机情况，并在应用处于脱机状态时修改本地存储中的数据。

## 更新客户端应用的同步行为

在本部分中，将修改客户端项目，通过对后端使用无效的应用程序 URL 来模拟脱机情况。添加或更改数据项时，这些更改将保存在本地存储中，但在重新建立连接之前，不会同步到后端数据存储中。

2. 在解决方案资源管理器中，从 **Portable** 项目打开 Constants.cs 项目文件，更改 `ApplicationURL` 的值使其指向无效的 URL，如下所示：

        publis static string ApplicationURL = @"https://your-service.azurewebsites.xxx/";


2. 从 **Portable** 项目打开 TodoItemManager.cs 文件，然后在 **SyncAsync** 的 **try...catch** 块中为 **Exception** 基类添加一个额外的 **catch**。此 **catch** 块会将异常消息写入控制台，如下所示：

            catch (Exception ex)
            {
                Console.Error.WriteLine(@"Exception: {0}", ex.Message);
            }

3. 生成并运行客户端应用，添加一些新项目，并请注意每次尝试与后端同步时，都会在控制台中记录异常。这些新项目在推送到移动后端之前，只存在于本地存储中。客户端应用的行为就像它已连接到支持所有创建、读取、更新、删除 (CRUD) 操作的后端一样。

4. 关闭应用程序并重新启动它，以验证你创建的新项目是否已永久保存到本地存储中。

5. （可选）使用 Visual Studio 查看 Azure SQL 数据库表，以了解后端数据库中的数据是否未更改。

	在 Visual Studio 中，打开“服务器资源管理器”。导航到“Azure”->“SQL 数据库”中的数据库。右键单击数据库并选择“在 SQL Server 对象资源管理器中打开”。现在便可以浏览 SQL 数据库表及其内容。


## 更新客户端应用以重新连接移动后端

在本部分中，会将应用重新连接到移动后端，以模拟重新回到联机状态的应用。执行刷新手势时，数据将同步到移动后端。

1. 重新打开 Constants.cs，然后更正 `applicationURL` 以指向正确的 URL。

2. 重新生成并运行客户端应用。该应用在启动后将尝试与移动应用后端进行同步。验证调试控制台中是否未记录任何异常。

3. （可选）使用 SQL Server 对象资源管理器或 Fiddler 之类的 REST 工具查看更新后的数据。请注意，数据已在后端数据库和本地存储之间进行同步。

    请注意，数据已在数据库和本地存储之间进行同步，并包含在应用断开连接时添加的项目。

## 其他资源

* [Azure 移动应用中的脱机数据同步]

<!-- ##Summary



<!-- Images -->

<!-- URLs. -->
[创建 Xamarin iOS 应用]: /documentation/articles/app-service-mobile-xamarin-ios-get-started/
[Azure 移动应用中的脱机数据同步]: /documentation/articles/app-service-mobile-offline-data-sync/
[How to use the Xamarin Component client for Azure Mobile Services]: /documentation/articles/partner-xamarin-mobile-services-how-to-use-client-library/

<!---HONumber=Mooncake_0919_2016-->