<properties
	pageTitle="将脱机数据同步添加到 Android 移动服务应用 | Microsoft Azure"
	description="了解如何使用 Azure 移动服务在 Android 应用程序中缓存和同步脱机数据"
	documentationCenter="android"
	authors="RickSaling"
	manager="dwrede"
	editor=""
	services="mobile-services"/>

<tags
	ms.service="mobile-services"
	ms.date="02/07/2016"
	wacn.date="04/18/2016"/>

# 将脱机数据同步添加到 Android 移动服务应用

[AZURE.INCLUDE [mobile-services-selector-offline](../includes/mobile-services-selector-offline.md)]


## 摘要

移动到没有服务的区域或出现网络问题时，移动应用可能丢失网络连接。例如，在遥远施工现场上使用的建筑行业应用可能需要输入稍后要同步到 Azure 的计划数据。使用 Azure 移动服务脱机同步，可以在丢失网络连接时继续工作，这对许多移动应用至关重要。借助脱机同步，你可以使用 Azure SQL Server 表的本地副本并定期重新同步两者。

在本教程中，你将从[移动服务快速启动教程]更新应用，从而启用脱机同步，然后通过脱机添加数据、将这些项同步至联机数据库并验证 Azure 经典门户中的更改，来测试该应用。

无论你是脱机还是已连接，任何时候对数据作出多种更改都可能引发冲突。将来的教程将探索如何处理同步冲突，其中你将选择要接受的更改版本。在本教程中，我们假定没有同步冲突且对现有数据所做的任何更改都将直接应用到 Azure SQL Server。


## 入门所需操作

[AZURE.INCLUDE [mobile-services-android-prerequisites](../includes/mobile-services-android-prerequisites.md)]


## 更新应用以支持脱机同步

借助脱机同步，可从“同步表”读取和写入（使用 *IMobileServiceSyncTable* 接口），该表是设备上 **SQLite** 数据库的一部分。

若要在设备与 Azure 移动服务之间推送和拉取更改，你可以使用“同步上下文” (*MobileServiceClient.SyncContext*)，该上下文借助本地存储数据时所用的本地数据库进行初始化。

1. 通过将此代码添加到 *AndroidManifest.xml* 文件来添加可检查网络连接的权限：

	    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />


2. 将下列 **import** 语句添加到 *ToDoActivity.java*：

		import java.util.Map;
		
		import android.widget.Toast;
		
		import com.microsoft.windowsazure.mobileservices.table.query.Query; 
		import com.microsoft.windowsazure.mobileservices.table.sync.MobileServiceSyncContext; 
		import com.microsoft.windowsazure.mobileservices.table.sync.MobileServiceSyncTable; 
		import com.microsoft.windowsazure.mobileservices.table.sync.localstore.ColumnDataType; 
		import com.microsoft.windowsazure.mobileservices.table.sync.localstore.SQLiteLocalStore; 

3. 在 `ToDoActivity` 类的顶部附近，将 `mToDoTable` 变量的声明从 `MobileServiceTable<ToDoItem>` 类更改为 `MobileServiceSyncTable<ToDoItem>` 类。

		 private MobileServiceSyncTable<ToDoItem> mToDoTable;

	在此处定义同步表。

4. 若要处理本地存储的初始化，请在创建 `MobileServiceClient` 实例后以 `onCreate` 方法添加以下行：

			// Saves the query which will be used for pulling data
			mPullQuery = mClient.getTable(ToDoItem.class).where().field("complete").eq(false);

			SQLiteLocalStore localStore = new SQLiteLocalStore(mClient.getContext(), "ToDoItem", null, 1);
			SimpleSyncHandler handler = new SimpleSyncHandler();
			MobileServiceSyncContext syncContext = mClient.getSyncContext();

			Map<String, ColumnDataType> tableDefinition = new HashMap<String, ColumnDataType>();
			tableDefinition.put("id", ColumnDataType.String);
			tableDefinition.put("text", ColumnDataType.String);
			tableDefinition.put("complete", ColumnDataType.Boolean);
			tableDefinition.put("__version", ColumnDataType.String);

			localStore.defineTable("ToDoItem", tableDefinition);
			syncContext.initialize(localStore, handler).get();

			// Get the Mobile Service Table instance to use
			mToDoTable = mClient.getSyncTable(ToDoItem.class);

5. 接着 `try` 块中的上述代码，在 `MalformedURLException` 块后再添加一个 `catch` 块：

		} catch (Exception e) {
			Throwable t = e;
			while (t.getCause() != null) {
					t = t.getCause();
				}
			createAndShowDialog(new Exception("Unknown error: " + t.getMessage()), "Error");
		}

6. 添加此方法来验证网络连接：

		private boolean isNetworkAvailable() {
			ConnectivityManager connectivityManager
					= (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
			NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
			return activeNetworkInfo != null && activeNetworkInfo.isConnected();
		}


7. 添加此方法以本地 *SQL Light* 存储和 Azure SQL Server 之间进行同步：

		public void syncAsync(){
			if (isNetworkAvailable()) {
				new AsyncTask<Void, Void, Void>() {
	
					@Override
					protected Void doInBackground(Void... params) {
						try {
							mClient.getSyncContext().push().get();
							mToDoTable.pull(mPullQuery).get();

						} catch (Exception exception) {
							createAndShowDialog(exception, "Error");
						}
						return null;
					}
				}.execute();
			} else {
				Toast.makeText(this, "You are not online, re-sync later!" +
						"", Toast.LENGTH_LONG).show();
			}
		}


8. 在 `onCreate` 方法中，将此代码添加到紧接 `refreshItemsFromTable` 的调用之前，作为倒数第二行：

			syncAsync();

	这将导致该设备在启动时与 Azure 表进行同步。否则将显示本地存储的最后一批脱机内容。


 
9. 更新 `refreshItemsFromTable` 方法中的代码以使用此查询（`try` 块内的第一行代码）：

		final MobileServiceList<ToDoItem> result = mToDoTable.read(mPullQuery).get();

10. 在 `onOptionsItemSelected` 方法中，用以下代码替换 `if` 块的内容：

			syncAsync();
			refreshItemsFromTable();

	按下右上角的“刷新”按钮时，会运行此代码。除了在启动时同步外，这是将本地存储同步到 Azure 的主要方式。此方式鼓励同步更改的批处理，并且在来自 Azure 的拉取操作相对很昂贵时非常高效。如果你的应用需要此操作，你也可以通过将对 `syncAsync` 的调用添加到 `addItem` 和 `checkItem` 方法来设计此应用。


## 测试应用程序

在此部分中，你将在启用 WiFi 的情况下测试行为，然后关闭 WiFi 以创建脱机方案。

添加数据项时，它们保存在本地 SQL Light 存储中，但直到按下“刷新”按钮才同步到移动服务。根据数据需要同步的时间，其他应用可能具有不同的要求，但出于演示目的，本教程让用户显式请求它。

按下此按钮之后，将启动新的后台任务，并且先通过使用同步上下文推送对本地存储所做的所有更改，然后将更改的所有数据从 Azure 拉取到本地表。


### 联机测试

要测试以下方案。

1. 在你的设备上添加一些新项； 
2. 验证门户中未显示的项； 
3. 接下来，按“刷新”并验证它们随后是否显示。
4. 在门户中更改或添加项，然后按“刷新”并验证设备上是否显示所做更改。

### 脱机测试

<!-- Now if you run the app and tap the refresh button, you should see all the items from the server. At that point you should be able to turn off the networking from the device by placing it in *Airplane Mode*, and continue making changes – the app will work just fine. When it’s time to sync the changes to the server, turn the network back on, and tap the **Refresh** button again.

One thing which is important to point out: if there are pending changes in the local store, a pull operation will first push those changes to the server (so that if there are changes in the same row, the push operation will fail and the application has an opportunity to handle the conflicts appropriately). That means that the push call in the code above isn’t necessarily required, but I think it’s always a good practice to be explicit about what the code is doing.
-->

1. 将设备或模拟器置于“飞行模式”中。这将创建脱机方案。

2. 添加一些 *ToDo* 项或将一些项标记为“完成”。退出设备或模拟器（或强制关闭应用），然后重新启动。验证所做更改是否保存在设备上，因为本地 SQL Light 存储已保存这些更改。

3. 查看 Azure *TodoItem* 表的内容。验证新项是否“未”同步到服务器：

   - 对于 JavaScript 后端，请转到 Azure 管理门户，然后单击“数据”选项卡查看 `TodoItem` 表的内容。
   - 对于 .NET 后端，请使用 SQL 工具（如 *SQL Server Management Studio*）或 REST 客户端（如 *Fiddler* 或 *Poistman*）查看表内容。

4. 在设备或模拟器中打开 WiFi。接下来，按“刷新”按钮。

5. 在 Azure 管理门户中再次查看 TodoItem 数据。新的和更改的 TodoItem 现在应会出现。


## 后续步骤

* [使用移动服务中的软删除][Soft Delete]

## 其他资源

* [云覆盖：Azure 移动服务中的脱机同步]

* [Aazure Friday：Azure 移动服务中支持脱机的应用]（注意：演示针对 Windows，但功能讨论适用于所有平台）


<!-- URLs. -->

[Get the sample app]: #get-app
[Review the Core Data model]: #review-core-data
[Review the Mobile Services sync code]: #review-sync
[Change the sync behavior of the app]: #setup-sync
[Test the app]: #test-app


[Mobile Services sample repository on GitHub]: https://github.com/Azure/mobile-services-samples


[Get started with Mobile Services]: /documentation/articles/mobile-services-android-get-started/
[Handling Conflicts with Offline Support for Mobile Services]: /documentation/articles/mobile-services-android-handling-conflicts-offline-data/
[Soft Delete]: /documentation/articles/mobile-services-using-soft-delete/

[云覆盖：Azure 移动服务中的脱机同步]: http://channel9.msdn.com/Shows/Cloud+Cover/Episode-155-Offline-Storage-with-Donna-Malayeri
[Aazure Friday：Azure 移动服务中支持脱机的应用]: http://azure.microsoft.com/documentation/videos/azure-mobile-services-offline-enabled-apps-with-donna-malayeri/

[移动服务快速启动教程]: /documentation/articles/mobile-services-android-get-started/

<!---HONumber=Mooncake_0118_2016-->