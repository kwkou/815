## 创建 WebAPI 项目

第一步是创建 ASP.NET WebAPI 项目。这是用于对客户端进行身份验证和生成通知的后端。

> [AZURE.NOTE] **重要提示**：在开始本教程之前，请确保已安装最新版本的 NuGet 程序包管理器。若要进行检查，请启动 Visual Studio。从"工具"菜单，单击"扩展和更新"。搜索"适用于 Visual Studio 2013 的 NuGet 程序包管理器"，并且确保具有版本 2.8.50313.46 或更高版本。否则，请卸载并重新安装 NuGet 程序包管理器。
> 
> ![][4]

1. 使用提升的权限启动 Visual Studio（以管理员身份运行）。
2. 在 Visual Studio 或 Visual Studio Express 中，依次单击"文件"、"新建"和"项目"，依次展开"模板"和"Visual C#"，然后依次单击"Web"和"ASP.NET Web 应用程序"，键入名称 **AppBackend**，然后单击"确定"。 
	
	![][1]

2. 在"新建 ASP.NET 项目"对话框中，单击"Web API"，然后单击"确定"。

	![][2]

3. 在"配置 Azure 站点"对话框中，选择要用于此项目的订阅、区域以及数据库。然后，单击"确定"以创建该项目。 

	![][5]

4. 在"解决方案资源管理器"中，右键单击"AppBackend"项目，然后单击"管理 NuGet 程序包"。

5. 在左侧，单击"联机"。

6. 在"搜索"框中，键入 **servicebus**。

7. 在结果列表中，单击"Windows Azure 服务总线"，然后单击"安装"。完成安装后，关闭"NuGet 程序包管理器"窗口。

	![][14]

8. 在"解决方案资源管理器"中，右键单击"Models"文件夹，然后依次单击"添加"和"类"。将新类命名为 **Notifications.cs**。单击"添加"以生成类。该模块表示将要发送的其他安全通知。在完整的实现中，这些通知存储在某个数据库中。在这种情况下，为了简便起见，我们将它们存储在内存中。

	![][6]

9. 将代码添加到 Notifications.cs，并将  `Notifications` 类定义替换为以下内容：

    	public class Notifications
    	{
        	public static Notifications Instance = new Notifications();
        
	        public NotificationHubClient Hub { get; set; }

	        private Notifications() {
	            Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}", "{hub name}");
	        }
	    }


10. 在文件顶部添加以下  `using` 语句：

		using Microsoft.ServiceBus.Notifications;

11. 在  `Notifications()` 方法中，将以下代码行中的两个占位符分别替换为通知中心的连接字符串（具有完全访问权限）和通知中心名称。可以从 [Azure 管理门户](http://manage.windowsazure.com)中获取这些值:

		Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}", "{hub name}");

12. 在"解决方案资源管理器"中，右键单击"AppBackend"项目，然后依次单击"添加"和"类"。将新类命名为 **AuthenticationTestHandler.cs**。单击"添加"以生成类。通过此类可使用  *Basic Authentication* 对用户进行身份验证。请注意，您的应用可以使用所有身份验证方案。

13. 将代码添加到 AuthenticationTestHandler.cs，并将  `AuthenticationTestHandler` 类定义替换为以下内容：

		public class AuthenticationTestHandler : DelegatingHandler
	    {
	        protected override Task<HttpResponseMessage> SendAsync(
	        HttpRequestMessage request, CancellationToken cancellationToken)
	        {
	            var authorizationHeader = request.Headers.GetValues("Authorization").First();
	
	            if (authorizationHeader != null && authorizationHeader
	                .StartsWith("Basic ", StringComparison.InvariantCultureIgnoreCase))
	            {
	                string authorizationUserAndPwdBase64 =
	                    authorizationHeader.Substring("Basic ".Length);
	                string authorizationUserAndPwd = Encoding.Default
	                    .GetString(Convert.FromBase64String(authorizationUserAndPwdBase64));
	                string user = authorizationUserAndPwd.Split(':')[0];
	                string password = authorizationUserAndPwd.Split(':')[1];
	
	                if (verifyUserAndPwd(user, password))
	                {
	                    // Attach the new principal object to the current HttpContext object
	                    HttpContext.Current.User =
	                        new GenericPrincipal(new GenericIdentity(user), new string[0]);
	                    System.Threading.Thread.CurrentPrincipal =
	                        System.Web.HttpContext.Current.User;
	                }
	                else return Unauthorised();
	            }
	            else return Unauthorised();
	
	            return base.SendAsync(request, cancellationToken);
	        }
	
	        private bool verifyUserAndPwd(string user, string password)
	        {
	            // This is not a real authentication scheme.
	            return user == password;
	        }
	
	        private Task<HttpResponseMessage> Unauthorised()
	        {
	            var response = new HttpResponseMessage(HttpStatusCode.Forbidden);
	            var tsc = new TaskCompletionSource<HttpResponseMessage>();
	            tsc.SetResult(response);
	            return tsc.Task;
	        }
	    }

	> [AZURE.NOTE] **安全说明**： `AuthenticationTestHandler` 类不提供真正的身份验证。它仅用于模拟基本身份验证并返回一个主体。需要该用户名来创建通知中心注册。以上实现是不安全的。您必须在生产应用程序和服务中实现安全的身份验证机制。

14. 在 AuthenticationTestHandler.cs 文件的顶部添加以下  `using` 语句：

		using System.Net.Http;
		using System.Threading.Tasks;
		using System.Threading;
		using System.Text;
		using System.Security.Principal;
		using System.Net;				

14. 在 **App_Start/WebApiConfig.cs** 类中的  `Register` 方法的末尾添加以下代码：

		config.MessageHandlers.Add(new AuthenticationTestHandler());

15. 在"解决方案资源管理器"中，右键单击"Controllers"文件夹，然后依次单击"添加"和"控制器"。单击"Web API 2 Controller -- Empty"项目，然后单击"添加"。 

	![][7]

16. 将新类命名为 **RegisterController**，然后再次单击"添加"以生成控制器。

	![][8]
	  
16. 在  `RegisterController` 类定义中添加以下代码：

		private NotificationHubClient hub;

        public RegisterController()
        {
            hub = Notifications.Instance.Hub;
        }

        public class DeviceRegistration
        {
            public string Platform { get; set; }
            public string Handle { get; set; }
            public string[] Tags { get; set; }
        }

        // POST api/register
        // This creates a registration id
        public async Task<string> Post(string handle = null)
        {
            // make sure there are no existing registrations for this push handle (used for iOS and Android)
            string newRegistrationId = null;
            
            if (handle != null)
            {
                var registrations = await hub.GetRegistrationsByChannelAsync(handle, 100);

                foreach (RegistrationDescription registration in registrations)
                {
                    if (newRegistrationId == null)
                    {
                        newRegistrationId = registration.RegistrationId;
                    }
                    else
                    {
                        await hub.DeleteRegistrationAsync(registration);
                    }
                }
            }

            if (newRegistrationId == null) newRegistrationId = await hub.CreateRegistrationIdAsync();

            return newRegistrationId;
        }

        // PUT api/register/5
        // This creates or updates a registration (with provided channelURI) at the specified id
        public async Task<HttpResponseMessage> Put(string id, DeviceRegistration deviceUpdate)
        {
            RegistrationDescription registration = null;
            switch (deviceUpdate.Platform)
            {
                case "mpns":
                    registration = new MpnsRegistrationDescription(deviceUpdate.Handle);
                    break;
                case "wns":
                    registration = new WindowsRegistrationDescription(deviceUpdate.Handle);
                    break;
                case "apns":
                    registration = new AppleRegistrationDescription(deviceUpdate.Handle);
                    break;
                case "gcm":
                    registration = new GcmRegistrationDescription(deviceUpdate.Handle);
                    break;
                default:
                    throw new HttpResponseException(HttpStatusCode.BadRequest);
            }

            registration.RegistrationId = id;
            var username = HttpContext.Current.User.Identity.Name;

            // add check if user is allowed to add these tags
            registration.Tags = new HashSet<string>(deviceUpdate.Tags);
            registration.Tags.Add("username:" + username);

            try
            {
                await hub.CreateOrUpdateRegistrationAsync(registration);
            }
            catch (MessagingException e)
            {
                ReturnGoneIfHubResponseIsGone(e);
            }

            return Request.CreateResponse(HttpStatusCode.OK);
        }

        // DELETE api/register/5
        public async Task<HttpResponseMessage> Delete(string id)
        {
            await hub.DeleteRegistrationAsync(id);
            return Request.CreateResponse(HttpStatusCode.OK);
        }

        private static void ReturnGoneIfHubResponseIsGone(MessagingException e)
        {
            var webex = e.InnerException as WebException;
            if (webex.Status == WebExceptionStatus.ProtocolError)
            {
                var response = (HttpWebResponse)webex.Response;
                if (response.StatusCode == HttpStatusCode.Gone)
                    throw new HttpRequestException(HttpStatusCode.Gone.ToString());
            }
        }

17. 在 RegisterController.cs 文件的顶部添加以下  `using` 语句：

		using Microsoft.ServiceBus.Notifications;
		using SecurePush.Models;
		using System.Threading.Tasks;
		using Microsoft.ServiceBus.Messaging;
		using System.Web;
		
18. 请注意，在该代码中，我们为已经过处理程序验证的用户添加了用户标记。还可以通过添加可选复选框来验证用户是否有权注册以获取请求标记。

19. 在"解决方案资源管理器"中，右键单击"Controllers"文件夹，然后依次单击"添加"和"控制器"。单击"Web API 2 Controller -- Empty"项目，然后单击"添加"。将新类命名为 **NotificationsController**，然后再次单击"添加"以生成控制器。该组件为设备提供一种安全检索通知的方法，还向用户提供了一种方法来为其设备触发安全推送（用于本教程的教学目的）。请注意，在向通知中心发送通知时，我们将发送一个仅具有通知 ID 的原始通知（没有实际的消息内容）。

20. 在 **NotificationsController** 类定义中添加以下代码：

        public async Task<HttpResponseMessage> Post()
        {
            var user = HttpContext.Current.User.Identity.Name;
            var userTag = "username:"+user;
            var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">Hello, " + user + "</text></binding></visual></toast>";
            await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

			return Request.CreateResponse(HttpStatusCode.OK);

        }

21. 在 NotificationsController.cs 文件的顶部添加以下  `using` 语句：

		using SecurePush.Models;
		using System.Threading.Tasks;
		using System.Web;

22. 按 **F5** 以运行应用程序并确保到目前为止工作的准确性。该应用应启动 Web 浏览器，然后显示 ASP.NET 主页。 

23. 现在，我们将此应用部署到 Azure 网站，以便可以从任意设备访问它。右键单击"AppBackend"项目，然后选择"发布"。

24. 选择 Azure 网站作为发布目标。

	![][B15]

25. 使用您的 Azure 帐户登录，然后选择一个现有的或新的网站。

	![][B16]

26. 记下"连接"选项卡中的"目标 URL"属性。在本教程后面部分，我们将此 URL 称为  *backend endpoint*。单击"发布"。

	![][B18]


[1]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push1.png
[2]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push2.png
[3]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push3.png
[4]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push4.png
[5]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push5.png
[6]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push6.png
[7]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push7.png
[8]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push8.png
[14]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push14.png

[B15]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users15.PNG
[B16]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users16.PNG
[B18]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users18.PNG
<!--HONumber=41-->
