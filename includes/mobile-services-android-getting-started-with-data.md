将移动服务准备就绪后，你可以更新应用程序，以便在移动服务而不是本地集合中存储项。

1.  如果你还没有[移动服务 Android SDK][]，请立即下载并展开压缩的文件。

2.  将 `.jar` 文件从 SDK 的 `mobileservices` 文件夹复制到 GetStartedWithData 项目的 `libs` 文件夹中。

3.  在 Eclipse 中的“包资源管理器”中，右键单击 `libs` 文件夹，单击“刷新” ，复制的 jar 文件将会出现

    这样可将移动服务 SDK 引用添加到工作区。

4.  打开 AndroidManifest.xml 文件并添加以下代码行，以便应用程序能够访问 Azure 中的移动服务。

        <uses-permission android:name="android.permission.INTERNET" />

5.  在“包资源管理器”中，打开位于 com.example.getstartedwithdata 包中的 TodoActivity.java 文件，并且取消注释以下代码行：

        import com.microsoft.windowsazure.mobileservices.MobileServiceClient;
        import com.microsoft.windowsazure.mobileservices.MobileServiceTable;
        import com.microsoft.windowsazure.mobileservices.NextServiceFilterCallback;
        import com.microsoft.windowsazure.mobileservices.ServiceFilter;
        import com.microsoft.windowsazure.mobileservices.ServiceFilterRequest;
        import com.microsoft.windowsazure.mobileservices.ServiceFilterResponse;
        import com.microsoft.windowsazure.mobileservices.ServiceFilterResponseCallback;
        import com.microsoft.windowsazure.mobileservices.TableOperationCallback;
        import com.microsoft.windowsazure.mobileservices.TableQueryCallback;

        import java.net.MalformedURLException;

6.  我们将删除应用程序当前使用的内存中列表，因此可将其替换为移动服务。在 "ToDoActivity" 类中，注释掉以下定义现有 "toDoItemList" 列表的代码行。

        public List<ToDoItem> toDoItemList = new ArrayList<ToDoItem>();

7.  保存文件，项目将指示生成错误。搜索剩余三个使用 `toDoItemList` 变量的位置，并注释掉所指示的部分。还要删除 `import java.util.ArrayList`。这样可以完全删除内存中列表。

8.  现在，我们将添加移动服务。取消注释以下代码行：

        private MobileServiceClient mClient;
        private private MobileServiceTable<ToDoItem> mToDoTable;

9.  找到文件底部的 ProgressFilter 类，并取消其注释。当 MobileServiceClient 运行网络操作时，此类显示“正在加载”指示器。

10. 在管理门户中单击“移动服务”，然后单击你刚刚创建的移动服务 。

11. 单击“仪表板”选项卡并记下“站点 URL”中的值，然后单击“管理密钥”并记下“应用程序密钥”中的值 。

    ![][0]

    从应用程序代码访问移动服务时，你需要使用这些值。

12. 在 "onCreate" 方法中，取消注释以下定义 "MobileServiceClient" 变量的代码行：

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
        createAndShowDialog(new Exception("There was an error creating the Mobile Service.Verify the URL"), "Error");
        }

    这将创建用于访问移动服务的 MobileServiceClient 的新实例。它还将创建用于代理移动服务中的数据存储的 MobileServiceTable 实例。

13. 在上面的代码中，请将 `MobileServiceUrl` 和 `AppKey` 依次替换为移动服务的 URL 和应用程序密钥。

14. 取消注释 "checkItem" 方法的以下行：

        mToDoTable.update(item, new TableOperationCallback<ToDoItem>() {    
        public void onCompleted(ToDoItem entity, Exception exception,
        ServiceFilterResponse response) {
        if(exception == null){
        if (entity.isComplete()) {
        mAdapter.remove(entity);
                    }
        } else {
        createAndShowDialog(exception, "Error");    
                }
            }
        });

    这会将项目更新发送到移动服务，并从适配器中删除已选中的项目。

15. 取消注释 "addItem" 方法的以下行：

        mToDoTable.insert(item, new TableOperationCallback<ToDoItem>() {

        public void onCompleted(ToDoItem entity, Exception exception,
        ServiceFilterResponse response) {
        if(exception == null){
        if (!entity.isComplete()) {
        mAdapter.add(entity);
                    }
        } else {
        createAndShowDialog(exception, "Error");
                }               
            }
        });

    此代码将创建一个新项目并将其插入到远程移动服务的表中。

16. 取消注释 "refreshItemsFromTable" 方法的以下行：

        mToDoTable.where().field("complete").eq(false)
        .execute(new TableQueryCallback<ToDoItem>() {
        public void onCompleted(List<ToDoItem> result, 
        int count, Exception exception, 
        ServiceFilterResponse response) {

        if(exception == null){
        mAdapter.clear();

        for (ToDoItem item :result) {
        mAdapter.add(item);
                            }
        } else {
        createAndShowDialog(exception, "Error");
                        }
                    }
                }); 

    这将查询移动服务，并返回未标记为“完成”的所有项目。这些项目将添加到用于绑定的适配器。

  [移动服务 Android SDK]: http://go.microsoft.com/fwlink/p/?LinkID=280126
  [0]: ./media/download-android-sample-code/mobile-dashboard-tab.png
