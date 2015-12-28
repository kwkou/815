9
1. 在 Android Studio 的“项目资源管理器”中，打开 ToDoActivity.java 文件，然后添加以下 import 语句。

		import java.util.concurrent.ExecutionException;
		import java.util.concurrent.atomic.AtomicBoolean;

		import android.content.Context;
		import android.content.SharedPreferences;
		import android.content.SharedPreferences.Editor;

		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceAuthenticationProvider;
		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceUser;

2. 将以下方法添加到 **ToDoActivity** 类： 
	
		private void authenticate() {
	    // Login using the Google provider.
	    
			ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory);

    		Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
    			@Override
    			public void onFailure(Throwable exc) {
    				createAndShowDialog((Exception) exc, "Error");
    			}   		
    			@Override
    			public void onSuccess(MobileServiceUser user) {
    				createAndShowDialog(String.format(
                        "You are now logged in - %1$2s",
                        user.getUserId()), "Success");
    				createTable();	
    			}
    		});   	
		}


	这将会创建一个用于处理身份验证过程的新方法。将使用 Azure Active Directory 登录对用户进行身份验证。此时将出现一个对话框，其中显示了已经过身份验证的用户的 ID。如果未正常完成身份验证，您将无法继续操作。

    > [AZURE.NOTE]请根据你使用的身份提供者将传递给上述 **login** 方法的值更改为下列其中一项：MicrosoftAccount 或 WindowsAzureActiveDirectory。

3. 在 **OnCreate** 方法中，在实例化 `MobileServiceClient` 对象的代码后面添加以下代码行。

		authenticate();

	此调用启动身份验证过程。

4. 将 **onCreate** 方法中 `authenticate();` 后面的剩余代码移到新的 **createTable** 方法，如下所示：

		private void createTable() {
	
			// Get the Mobile Service Table instance to use
			mToDoTable = mClient.getTable(ToDoItem.class);
	
			mTextNewToDo = (EditText) findViewById(R.id.textNewToDo);
	
			// Create an adapter to bind the items with the view
			mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);
			ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
			listViewToDo.setAdapter(mAdapter);
	
			// Load the items from the Mobile Service
			refreshItemsFromTable();
		}

5. 然后，从“运行”菜单中单击“运行应用”以启动应用，并使用所选的标识提供者登录。

   	当您成功登录时，应用应该运行而不出现错误，您应该能够查询移动服务，并对数据进行更新。
<!---HONumber=71-->