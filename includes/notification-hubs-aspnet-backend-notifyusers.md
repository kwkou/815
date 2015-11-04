## 创建 WebAPI 项目

按照以下步骤创建新的 ASP.NET WebAPI 后端以对客户端进行身份验证并生成通知，或者修改以前的项目或[向经过身份验证的用户发送推送通知](/zh-cn/documentation/articles/mobile-services-dotnet-backend-ios-push-notifications-app-users/)教程中的现有后端。

> [AZURE.NOTE]**重要提示**：在开始本教程之前，请确保已安装最新版本的 NuGet 包管理器。若要进行检查，请启动 Visual Studio。从“工具”菜单，单击“扩展和更新”。搜索“适用于 Visual Studio 2013 的 NuGet 包管理器”，并且确保拥有版本 2.8.50313.46 或更高版本。否则，请卸载并重新安装 NuGet 程序包管理器。
> 
> ![][4]

> [AZURE.NOTE]请确保已安装 Visual Studio [Azure SDK](/zh-cn/downloads/) 以便进行网站部署。

1. 启动 Visual Studio 或 Visual Studio Express。
2. 在 Visual Studio 中，依次单击“文件”、“新建”和“项目”，依次展开“模板”和“Visual C#”，然后依次单击“Web”和“ASP.NET Web 应用程序”，键入名称 **AppBackend**，然后单击“确定”。 
	
	![][1]

3. 在“新建 ASP.NET 项目”对话框中，单击“Web API”，然后单击“确定”。

	![][2]

4. 在“配置 Azure 站点”对话框中，选择要用于此项目的订阅、区域以及数据库。然后，单击“确定”以创建该项目。

	![][5]

5. 在“解决方案资源管理器”中，右键单击“AppBackend”项目，然后单击“管理 NuGet 包”。

6. 在左侧，单击“联机”，然后在“搜索”框中搜索 **servicebus**。

7. 在结果列表中，单击“Microsoft Azure 服务总线”，然后单击“安装”。完成安装后，关闭“NuGet 程序包管理器”窗口。

	![][14]

8. 现在将创建一个新类 **Notifications.cs**。转到“解决方案资源管理器”，右键单击 **Models** 文件夹，单击“添加”，然后单击“类”。将新类命名为 **Notifications.cs** 后，单击“添加”以生成该类。该模块表示将要发送的其他安全通知。在完整的实现中，这些通知存储在某个数据库中。为简单起见，本教程将这些通知存储在内存中。

	![][6]

9. 在 Notifications.cs 中，在文件顶部添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;

10. 然后，将 `Notifications` 类定义替换为以下内容并确保将两个占位符替换为通知中心的连接字符串（具有完全访问权限）和中心名称（可在 [Azure 管理门户](http://manage.windowsazure.cn)中找到）：

    	public class Notifications
    	{
        	public static Notifications Instance = new Notifications();
        
	        public NotificationHubClient Hub { get; set; }

	        private Notifications() {
	            Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}", "{hub name}");
	        }
	    }

11. 然后，我们将创建一个新类 **AuthenticationTestHandler.cs**。在“解决方案资源管理器”中，右键单击“AppBackend”项目，然后依次单击“添加”和“类”。将新类命名 **AuthenticationTestHandler.cs**，然后单击“添加”以生成该类。通过此类可使用*基本身份验证*对用户进行身份验证。请注意，您的应用可以使用所有身份验证方案。

12. 在 AuthenticationTestHandler.cs 中，添加以下 `using` 语句：

        using System.Net.Http;
        using System.Threading.Tasks;
        using System.Threading;
        using System.Text;
        using System.Security.Principal;
        using System.Net;

13. 在 AuthenticationTestHandler.cs 中，将 `AuthenticationTestHandler` 类定义替换为以下内容：

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

	> [AZURE.NOTE]**安全说明**：`AuthenticationTestHandler` 类不提供真正的身份验证。它仅用于模拟基本身份验证并且是不安全的。您必须在生产应用程序和服务中实现安全的身份验证机制。

14. 在 **App\_Start/WebApiConfig.cs** 类的 `Register` 方法末尾添加以下代码：

		config.MessageHandlers.Add(new AuthenticationTestHandler());

15. 接下来，我们将创建一个新的控制器 **RegisterController**。在“解决方案资源管理器”中，右键单击“Controllers”文件夹，然后依次单击“添加”和“控制器”。单击“Web API 2 Controller -- Empty”项目，然后单击“添加”。将新类命名为 **RegisterController**，然后再次单击“添加”以生成该控制器。

	![][7]

	![][8]

16. 在 RegiterController.cs 中，添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;
        using AppBackend.Models;
        using System.Threading.Tasks;
        using Microsoft.ServiceBus.Messaging;
        using System.Web;

17. 在 `RegisterController` 类定义中添加以下代码：请注意，在该代码中，我们为已经过处理程序验证的用户添加了用户标记。还可以通过添加可选复选框来验证用户是否有权注册以获取请求标记。

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

18. 按照我们创建 **RegisterController** 的方式，创建新的控制器 **NotificationsController**。该组件为设备提供一种安全检索通知的方法，还向用户提供了一种方法来为设备触发安全推送。请注意，在向通知中心发送通知时，我们将发送一个仅具有通知 ID 的原始通知（没有实际的消息内容）。

19. 在 NotificationsController.cs 中，添加以下 `using` 语句：

        using AppBackend.Models;
        using System.Threading.Tasks;
        using System.Web;

20. 在 **NotificationsController** 类定义中添加以下代码，并确保注释掉未使用的平台的代码段。

        public async Task<HttpResponseMessage> Post()
        {
            var user = HttpContext.Current.User.Identity.Name;
            var userTag = "username:"+user;


            // windows
            var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">Hello, " + user + "</text></binding></visual></toast>";
            await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);


            // apns
            var alert = "{"aps":{"alert":"Hello"}}";
            await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(alert, userTag);


            // gcm
            var notif = "{ "data" : {"msg":"Hello"}}";
            await Notifications.Instance.Hub.SendGcmNativeNotificationAsync(notif, userTag);


            return Request.CreateResponse(HttpStatusCode.OK);
        }

21. 按 **F5** 以运行应用程序并确保到目前为止操作的准确性。该应用应启动 Web 浏览器，然后显示 ASP.NET 主页。

22. 现在，我们将此应用部署到 Azure 网站，以便可以从任意设备访问它。右键单击 **AppBackend** 项目，然后选择“发布”。

23. 选择 Azure 网站作为发布目标。

    ![][B15]

24. 使用您的 Azure 帐户登录，然后选择一个现有的或新的网站。

    ![][B16]

25. 记下“连接”选项卡中的“目标 URL”属性。在本教程后面的部分中，我们将此 URL 称为*后端终结点*。单击“发布”。

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

<!---HONumber=76-->