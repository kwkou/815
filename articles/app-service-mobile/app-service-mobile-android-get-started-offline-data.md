<properties
	pageTitle="为 Azure 移动应用启用脱机同步 (Android)"
	description="了解如何在 Android 应用程序中使用应用服务移动应用来缓存和同步脱机数据"
	documentationCenter="android"
	authors="RickSaling"
	manager="erikre"
	services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-android"
	ms.devlang="java"
	ms.topic="article"
	ms.date="10/01/2016"
	wacn.date="09/26/2016"
	ms.author="yuaxu"/>

# 为 Android 移动应用启用脱机同步

[AZURE.INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## 概述

本教程介绍适用于 Android 的 Azure 移动应用的脱机同步功能。脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。更改存储在本地数据库中；设备重新联机后，这些更改会与远程后端同步。

对于首次体验 Azure 移动应用的读者，请先完成 [Create an Android App]（创建 Android 应用）教程。如果不使用下载的快速入门服务器项目，必须将数据访问扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

若要了解有关脱机同步功能的详细信息，请参阅 [Offline Data Sync in Azure Mobile Apps]（Azure 移动应用中的脱机数据同步）主题。

## 更新应用以支持脱机同步

借助脱机同步，可从 *同步表* 读取和写入（使用 *IMobileServiceSyncTable* 接口），该表是设备上 **SQLite** 数据库的一部分。

若要在设备与 Azure 移动服务之间推送和拉取更改，你可以使用*同步上下文* ( *MobileServiceClient.SyncContext* )，该上下文借助本地存储数据时所用的本地数据库进行初始化。

1. 在 `TodoActivity.java` 中，注释掉 `mToDoTable` 的现有定义，取消注释同步表版本：

	    private MobileServiceSyncTable<ToDoItem> mToDoTable;

2. 在 `onCreate` 方法中，注释掉 `mToDoTable` 的现有初始化，取消注释以下定义：

        mToDoTable = mClient.getSyncTable("ToDoItem", ToDoItem.class);

3. 在 `refreshItemsFromTable` 中，注释掉 `results` 的定义，取消注释以下定义：

		// Offline Sync
		final List<ToDoItem> results = refreshItemsFromMobileServiceTableSyncTable();

4. 注释掉 `refreshItemsFromMobileServiceTable` 的定义。

5. 取消注释 `refreshItemsFromMobileServiceTableSyncTable` 的定义：

	    private List<ToDoItem> refreshItemsFromMobileServiceTableSyncTable() throws ExecutionException, InterruptedException {
	        //sync the data
	        sync().get();
	        Query query = QueryOperations.field("complete").
	                eq(val(false));
	        return mToDoTable.read(query).get();
	    }

6. 取消注释 `sync` 的定义：

	    private AsyncTask<Void, Void, Void> sync() {
	        AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
	            @Override
	            protected Void doInBackground(Void... params) {
	                try {
	                    MobileServiceSyncContext syncContext = mClient.getSyncContext();
	                    syncContext.push().get();
	                    mToDoTable.pull(null).get();
	                } catch (final Exception e) {
	                    createAndShowDialogFromTask(e, "Error");
	                }
	                return null;
	            }
	        };
	        return runAsyncTask(task);
	    }

## 测试应用程序

在此部分中，你将在启用 WiFi 的情况下测试行为，然后关闭 WiFi 以创建脱机方案。

添加数据项时，它们保存在本地 SQLite 存储中，但直到按下“刷新”按钮才同步到移动服务。根据数据需要同步的时间，其他应用可能具有不同的要求，但出于演示目的，本教程让用户显式请求它。

按下此按钮之后，将启动新的后台任务，并且先通过使用同步上下文推送对本地存储所做的所有更改，然后将更改的所有数据从 Azure 拉取到本地表。

### 脱机测试

1. 将设备或模拟器置于 *飞行模式* 中。这将创建脱机方案。

2. 添加一些 *ToDo* 项或将一些项标记为“完成”。退出设备或模拟器（或强制关闭应用），然后重新启动。验证所做更改是否保存在设备上，因为本地 SQLite 存储已保存这些更改。

3. 使用 SQL 工具（如 *SQL Server Management Studio* ）或 REST 客户端（如 *Fiddler* 或 *Postman* ）查看 Azure *TodoItem* 表的内容。验证新项是否_未_同步到服务器

   	+ 对于 Node.js 后端，请转到 [Azure 门户预览](https://portal.azure.cn/)，在移动应用后端中单击“简易表”>“TodoItem”，查看 `TodoItem` 表的内容。
   	+ 对于 .NET 后端，请使用 SQL 工具（如 *SQL Server Management Studio*）或 REST 客户端（如 *Fiddler* 或 *Poistman*）查看表内容。

4. 在设备或模拟器中打开 WiFi。接下来，按“刷新”按钮。

5. 在 Azure 门户中再次查看 TodoItem 数据。新的和更改的 TodoItem 现在应会出现。

## 其他资源

* [Azure 移动应用中的脱机数据同步]


<!-- URLs. -->

[Offline Data Sync in Azure Mobile Apps]: /documentation/articles/app-service-mobile-offline-data-sync/
[Azure 移动应用中的脱机数据同步]: /documentation/articles/app-service-mobile-offline-data-sync/

[Create an Android App]: /documentation/articles/app-service-mobile-android-get-started/

<!---HONumber=Mooncake_0919_2016-->