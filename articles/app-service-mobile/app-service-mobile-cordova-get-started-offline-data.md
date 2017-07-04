<properties
    pageTitle="为 Azure 移动应用启用脱机同步 (Cordova) | Azure"
    description="了解如何在 Cordova 应用程序中使用应用服务移动应用来缓存和同步脱机数据"
    documentationCenter="cordova"
    authors="adrianhall"
    manager="erikre"
    editor=""
    services="app-service\mobile"/>  


<tags
    ms.service="app-service-mobile"
    ms.workload="mobile"
    ms.tgt_pltfrm="mobile-cordova-ios"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="10/30/2016"
    wacn.date="01/23/2017"
    ms.author="adrianha"/>  


# 为 Cordova 移动应用启用脱机同步

[AZURE.INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

本教程介绍适用于 Cordova 的 Azure 移动应用的脱机同步功能。脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。在本地数据库中存储更改。设备重新联机后，这些更改会与远程服务同步。

本教程基于完成教程 [Apache Cordova 快速入门]时创建的移动应用的 Cordova 快速入门解决方案。在本教程中，将更新快速入门解决方案，以添加 Azure 移动应用的脱机功能。我们还重点介绍了该应用中的特定于脱机的代码。

若要了解有关脱机同步功能的详细信息，请参阅主题 [Azure 移动应用中的脱机数据同步]。若要深入了解 API 用法，请参阅 [API 文件](https://azure.github.io/azure-mobile-apps-js-client)。

## 在快速入门解决方案中添加脱机同步功能

应用中必须添加脱机同步代码。脱机同步功能需要 cordova-sqlite-storage 插件，该插件会在项目中包含 Azure 移动应用时自动添加到应用中。快速入门项目包含上述两个插件。

1. 在 Visual Studio 解决方案资源管理器中，打开 index.js，然后将以下代码

        var client,            // Connection to the Azure Mobile App backend
          todoItemTable;      // Reference to a table endpoint on backend

    替换为此代码：

        var client,            // Connection to the Azure Mobile App backend
           todoItemTable,      // Reference to a table endpoint on backend
           syncContext;        // Reference to offline data sync context

2. 接下来，将以下代码：

        client = new WindowsAzure.MobileServiceClient('http://yourmobileapp.azurewebsites.cn');

	替换为此代码：

        client = new WindowsAzure.MobileServiceClient('http://yourmobileapp.azurewebsites.cn');

        var store = new WindowsAzure.MobileServiceSqliteStore('store.db');

        store.defineTable({
          name: 'todoitem',
          columnDefinitions: {
              id: 'string',
              text: 'string',
              complete: 'boolean',
              version: 'string'
          }
        });

        // Get the sync context from the client
        syncContext = client.getSyncContext();

    前面增加的代码会初始化本地存储，并定义与 Azure 后端中使用的列值匹配的本地表。（无需在此代码中包含所有列值。） `version` 字段由移动后端维护并用于冲突解决方案。

    调用 **getSyncContext** 可获取对同步上下文的引用。对于调用 `.push()` 时客户端应用修改的所有表，此同步上下文通过跟踪和推送这些表中的更改来帮助保持表关系。

3. 将应用程序 URL 更新为移动应用的应用程序 URL。

4. 接下来，将此代码：

        todoItemTable = client.getTable('todoitem'); // todoitem is the table name

    替换为此代码：

        // Initialize the sync context with the store
        syncContext.initialize(store).then(function () {

        // Get the local table reference.
        todoItemTable = client.getSyncTable('todoitem');

        syncContext.pushHandler = {
            onConflict: function (pushError) {
                // Handle the conflict.
                console.log("Sync conflict! " + pushError.getError().message);
                // Update failed, revert to server's copy.
                pushError.cancelAndDiscard();
              },
              onError: function (pushError) {
                  // Handle the error
                  // In the simulated offline state, you get "Sync error! Unexpected connection failure."
                  console.log("Sync error! " + pushError.getError().message);
              }
        };

        // Call a function to perform the actual sync
        syncBackend();

        // Refresh the todoItems
        refreshDisplay();

        // Wire up the UI Event Handler for the Add Item
        $('#add-item').submit(addItemHandler);
        $('#refresh').on('click', refreshDisplay);

    前面的代码会初始化同步上下文，然后调用 getSyncTable（而不是 getTable）来获取对本地表的引用。

    此代码使用本地数据库进行所有创建、读取、更新和删除 (CRUD) 表操作。

    此示例会对同步冲突执行简单的错误处理。实际应用程序会处理各种错误，例如网络状况、服务器冲突等。有关代码示例，请参阅[脱机同步示例]。

5. 接下来，添加此函数以执行实际同步操作。

        function syncBackend() {

          // Sync local store to Azure table when app loads, or when login complete.
          syncContext.push().then(function () {
              // Push completed

          });

          // Pull items from the Azure table after syncing to Azure.
          syncContext.pull(new WindowsAzure.Query('todoitem'));
        }

    通过调用 **syncContext.push()**，决定何时将更改推送到移动应用后端。例如，在绑定到同步按钮的按钮事件处理程序中，可调用 **syncBackend**。

## 脱机同步注意事项

在示例中，**syncContext** 的 **push** 方法仅会在应用启动时在登录的回调函数中调用。在实际应用程序中，还可以手动或在网络状态发生更改时触发此同步功能。

如果对一个表执行拉取操作，并且该表具有由上下文跟踪的未完成的本地更新，那么该拉取操作将自动触发推送操作。在此示例中刷新、添加和完成项时，可省略显式 **push** 调用，因为它可能是冗余的。

在所提供的代码中，将查询远程 todoItem 表中的所有记录，也可以筛选记录，只需将查询 ID 和查询传递给 **push** 即可。有关详细信息，请参阅 [Azure 移动应用中的脱机数据同步]中的 *增量同步* 部分。

## （可选）禁用身份验证

如果不想在测试脱机同步功能之前设置身份验证，请注释禁止登录的回调函数，但让回调函数内部的代码保持取消注释状态。注释禁止登录行后，该代码应如下所示：

      // Login to the service.
      // client.login('twitter')
      //    .then(function () {
        syncContext.initialize(store).then(function () {
          // Leave the rest of the code in this callback function  uncommented.
                ...
        });
      // }, handleError);

现在，应用将在运行应用时与 Azure 后端同步。

## 运行客户端应用
启用脱机同步后，可在每个平台上至少运行一次客户端应用程序，以填充本地存储数据库。稍后，模拟脱机场景，并在应用处于脱机状态时修改本地存储中的数据。

## （可选）测试同步行为
本节对客户端项目进行修改，通过对后端使用无效的应用程序 URL 来模拟脱机场景。添加或更改数据项时，这些更改保存在本地存储中，但在重新建立连接之前，这些更改不会同步到后端数据存储中。

1. 在解决方案资源管理器中，打开 index.js 项目文件，然后更改应用程序 URL，使其指向无效的 URL，如以下代码所示：

        client = new WindowsAzure.MobileServiceClient('http://yourmobileapp.azurewebsites.net-fail');

2. 在 index.html 中，使用同一无效的 URL 更新 CSP `<meta>` 元素。

        <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: http://yourmobileapp.azurewebsites.net-fail; style-src 'self'; media-src *">

3. 生成并运行客户端应用，请注意，如果应用在登录后尝试与后端同步，则会在控制台中记录异常。添加的任何新项在推送到移动后端之前，只存在于本地存储中。客户端应用的行为就像它已连接到后端一样。

4. 关闭应用程序并重新启动它，以验证你创建的新项目是否已永久保存到本地存储中。

5. （可选）使用 Visual Studio 查看 Azure SQL 数据库表，以了解后端数据库中的数据是否未更改。

    在 Visual Studio 中，打开“服务器资源管理器”。导航到“Azure”->“SQL 数据库”中的数据库。右键单击数据库并选择“在 SQL Server 对象资源管理器中打开”。现在便可以浏览 SQL 数据库表及其内容。

## （可选）测试与移动后端的重新连接

本节将应用重新连接到移动后端，可模拟重新回到联机状态的应用。登录时，数据将同步到移动后端。

1. 重新打开 index.js 并还原应用程序 URL。
2. 重新打开 index.html，更正 CSP `<meta>` 元素中的应用程序 URL。
3. 重新生成并运行客户端应用。应用在登录后尝试与移动应用后端进行同步。验证调试控制台中是否未记录任何异常。
4. （可选）使用 SQL Server 对象资源管理器或 Fiddler 之类的 REST 工具查看更新后的数据。请注意，数据已在后端数据库和本地存储之间进行同步。

    请注意，数据已在数据库和本地存储之间进行同步，并包含在应用断开连接时添加的项目。

## 其他资源
* [Azure 移动应用中的脱机数据同步]
* [用于 Apache Cordova 的 Visual Studio 工具]

## 后续步骤
* 在[脱机同步示例]中，查看更高级的脱机同步功能，例如冲突解决
* 在 [API 文档](https://azure.github.io/azure-mobile-apps-js-client)中，查看脱机同步 API 参考。

<!-- ##Summary -->


<!-- Images -->

<!-- URLs. -->

[Apache Cordova 快速入门]: /documentation/articles/app-service-mobile-cordova-get-started/
[脱机同步示例]: https://github.com/Azure-Samples/app-service-mobile-cordova-client-conflict-handling
[Offline Data Sync in Azure Mobile Apps]: /documentation/articles/app-service-mobile-offline-data-sync/
[Azure 移动应用中的脱机数据同步]: /documentation/articles/app-service-mobile-offline-data-sync/
[Adding Authentication]: /documentation/articles/app-service-mobile-cordova-get-started-users/
[authentication]: /documentation/articles/app-service-mobile-cordova-get-started-users/
[Work with the .NET backend server SDK for Azure Mobile Apps]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Visual Studio Community 2015]: http://www.visualstudio.com/
[用于 Apache Cordova 的 Visual Studio 工具]: https://www.visualstudio.com/vs/cordova/
[Apache Cordova SDK]: /documentation/articles/app-service-mobile-cordova-how-to-use-client-library/
[ASP.NET Server SDK]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Node.js Server SDK]: /documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description:update wording and code-->