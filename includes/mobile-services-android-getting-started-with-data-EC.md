将移动服务准备就绪后，您可以更新应用，以便在移动服务而不是本地集合中存储项。 

1. 如果您还没有[移动服务 Android SDK]，请立即下载并展开压缩的文件。

2. 将  `.jar` 文件从 SDK 的  `mobileservices` 文件夹复制到 GetStartedWithData 项目的  `libs` 文件夹中。

3. 在 Eclipse 中的“程序包资源管理器”中，右键单击 `libs` 文件夹，单击“刷新”，复制的 jar 文件将会出现

  	这样可将移动服务 SDK 引用添加到工作区。

4. 打开 AndroidManifest.xml 文件并添加以下代码行，以便应用能够访问 Azure 中的移动服务。

		<uses-permission android:name="android.permission.INTERNET" />

5. 在“程序包资源管理器”中，打开位于 com.example.getstartedwithdata 程序包中的 TodoActivity.java 文件，并且取消注释以下代码行：

		import java.net.MalformedURLException;
		import android.os.AsyncTask;
		import com.google.common.util.concurrent.FutureCallback;
		import com.google.common.util.concurrent.Futures;
		import com.google.common.util.concurrent.ListenableFuture;
		import com.microsoft.windowsazure.mobileservices.MobileServiceClient;
		import com.microsoft.windowsazure.mobileservices.MobileServiceList;
		import com.microsoft.windowsazure.mobileservices.http.NextServiceFilterCallback;
		import com.microsoft.windowsazure.mobileservices.http.ServiceFilter;
		import com.microsoft.windowsazure.mobileservices.http.ServiceFilterRequest;
		import com.microsoft.windowsazure.mobileservices.http.ServiceFilterResponse;
		import com.microsoft.windowsazure.mobileservices.table.MobileServiceTable;

 
6. 取消注释以下行：

		import java.util.ArrayList;
		import java.util.List;

7. 我们将删除应用当前使用的内存中列表，因此可将其替换为移动服务。在 **ToDoActivity** 类中，注释掉以下定义现有 **toDoItemList** 列表的代码行。

		public List<ToDoItem> toDoItemList = new ArrayList<ToDoItem>();

8. 保存文件，项目将指示生成错误。搜索剩余三个使用  `toDoItemList` 变量的位置，并取消注释所指示的部分。这样可以完全删除内存中列表。 

9. 现在，我们将添加移动服务。取消注释以下代码行：

		private MobileServiceClient mClient;
		private private MobileServiceTable<ToDoItem> mToDoTable;

10. 找到文件底部的 *ProgressFilter* 类，并取消其注释。当 *MobileServiceClient* 运行网络操作时，此类显示“正在加载”指示器。


11. 在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务。

12. 单击“仪表板”选项卡并记下“站点 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值。

   	![](./media/download-android-sample-code/mobile-dashboard-tab.png)

  	从应用代码访问移动服务时，您需要使用这些值。

13. 在 **onCreate** 方法中，取消注释以下定义 **MobileServiceClient** 变量的代码行：

		try {
		// Create the Mobile Service Client instance, using the provided
		// Mobile Service URL and key
			mClient = new MobileServiceClient(
					"MobileServiceUrl",
					"AppKey", 
					this).withFilter(new ProgressFilter());

			// Get the Mobile Service Table instance to use
			mToDoTable = mClient.getTable(ToDoItem.class);
		} catch (MalformedURLException e) {
			createAndShowDialog(new Exception("There was an error creating the Mobile Service. Verify the URL"), "Error");
		}

  	这将创建用于访问移动服务的  *MobileServiceClient* 的新实例。它还将创建用于代理移动服务中的数据存储的  *MobileServiceTable* 实例。

14. 在上面的代码中，请将  `MobileServiceUrl` 和  `AppKey` 依次替换为移动服务的 URL 和应用程序密钥。



15. 取消注释 **checkItem** 方法的以下行：

	    new AsyncTask<Void, Void, Void>() {
	        @Override
	        protected Void doInBackground(Void... params) {
	            try {
	                mToDoTable.update(item).get();
	                runOnUiThread(new Runnable() {
	                    public void run() {
	                        if (item.isComplete()) {
	                            mAdapter.remove(item);
	                        }
	                        refreshItemsFromTable();
	                    }
	                });
	            } catch (Exception exception) {
	                createAndShowDialog(exception, "Error");
	            }
	            return null;
	        }
	    }.execute();

   	这会将项更新发送到移动服务，并从适配器中删除已选中的项。
    
16. 取消注释 **addItem** 方法的以下行：
	
		// Insert the new item
		new AsyncTask<Void, Void, Void>() {
	        @Override
	        protected Void doInBackground(Void... params) {
	            try {
	                mToDoTable.insert(item).get();
	                if (!item.isComplete()) {
	                    runOnUiThread(new Runnable() {
	                        public void run() {
	                            mAdapter.add(item);
	                        }
	                    });
	                }
	            } catch (Exception exception) {
	                createAndShowDialog(exception, "Error");
	            }
	            return null;
	        }
	    }.execute();
		

  	此代码将创建一个新项目并将其插入到远程移动服务的表中。

17. 取消注释 **refreshItemsFromTable** 方法的以下行：

		// Get the items that weren't marked as completed and add them in the adapter
	    new AsyncTask<Void, Void, Void>() {
	        @Override
	        protected Void doInBackground(Void... params) {
	            try {
	                final MobileServiceList<ToDoItem> result = mToDoTable.where().field("complete").eq(false).execute().get();
	                runOnUiThread(new Runnable() {
	                    @Override
	                    public void run() {
	                        mAdapter.clear();

	                        for (ToDoItem item : result) {
	                            mAdapter.add(item);
	                        }
	                    }
	                });
	            } catch (Exception exception) {
	                createAndShowDialog(exception, "Error");
	            }
	            return null;
	        }
	    }.execute();

	这将查询移动服务，并返回未标记为“完成”的所有项。这些项目将添加到用于绑定的适配器。
		

<!-- URLs. -->
[移动服务 Android SDK]: https://zumo.blob.core.windows.net/sdk/azuresdk-android-2.0.2-beta2.zip

<!---HONumber=71-->