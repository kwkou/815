
1. 在 Visual Studio 的解决方案资源管理器中，展开 App\_Start 文件夹，然后打开 WebApiConfig.cs 项目文件。

2. 将以下代码行添加到 Register 方法的 **ConfigOptions** 定义后面：

        options.PushAuthorization = 
            Microsoft.WindowsAzure.Mobile.Service.Security.AuthorizationLevel.User;
 
	这会强制用户在注册推送通知之前进行身份验证。

3. 右键单击该项目，单击“添加”，然后单击“类...”。

4. 将新的空类命名为 `PushRegistrationHandler`，然后单击“添加”。

5. 在代码页的顶部，添加以下 **using** 语句：

		using System.Threading.Tasks; 
		using System.Web.Http; 
		using System.Web.Http.Controllers; 
		using Microsoft.WindowsAzure.Mobile.Service; 
		using Microsoft.WindowsAzure.Mobile.Service.Notifications; 
		using Microsoft.WindowsAzure.Mobile.Service.Security; 

6. 将现有的 **PushRegistrationHandler** 类替换为以下代码：
 
	    public class PushRegistrationHandler : INotificationHandler
	    {
	        public Task Register(ApiServices services, HttpRequestContext context,
            NotificationRegistration registration)
        {
            try
            {
                // Perform a check here for user ID tags, which are not allowed.
                if(!ValidateTags(registration))
                {
                    throw new InvalidOperationException(
                        "You cannot supply a tag that is a user ID.");                    
                }

                // Get the logged-in user.
                var currentUser = context.Principal as ServiceUser;

                // Add a new tag that is the user ID.
                registration.Tags.Add(currentUser.Id);

                services.Log.Info("Registered tag for userId: " + currentUser.Id);
            }
            catch(Exception ex)
            {
                services.Log.Error(ex.ToString());
            }
                return Task.FromResult(true);
        }

        private bool ValidateTags(NotificationRegistration registration)
        {
            // Create a regex to search for disallowed tags.
            System.Text.RegularExpressions.Regex searchTerm =
            new System.Text.RegularExpressions.Regex(@"microsoftaccount:",
                System.Text.RegularExpressions.RegexOptions.IgnoreCase);

            foreach (string tag in registration.Tags)
            {
                if (searchTerm.IsMatch(tag))
                {
                    return false;
                }
            }
            return true;
        }
	
        public Task Unregister(ApiServices services, HttpRequestContext context, 
            string deviceId)
        {
            // This is where you can hook into registration deletion.
            return Task.FromResult(true);
        }
    	}

	在注册期间将调用 **Register** 方法。这样，你便可以向注册添加一个标记（已登录用户的 ID）。将验证提供的标记以防止用户注册其他用户的 ID。通知发送给该用户后，用户可在此设备和用户所注册的其他任何设备上接收通知。

7. 展开 Controllers 文件夹、打开 TodoItemController.cs 项目文件、找到 **PostTodoItem** 方法，然后将调用 **SendAsync** 的代码行替换为以下代码：

        // Get the logged-in user.
		var currentUser = this.User as ServiceUser;
		
		// Use a tag to only send the notification to the logged-in user.
        var result = await Services.Push.SendAsync(message, currentUser.Id);

8. 重新发布移动服务项目。

现在，服务将使用该用户 ID 标记向已登录用户创建的所有注册发送推送通知（包括插入项的文本）。
 

<!---HONumber=74-->