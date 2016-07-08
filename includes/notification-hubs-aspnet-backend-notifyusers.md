## 创建 WebAPI 项目

新的 ASP.NET WebAPI 后端将会在后续部分中创建，该后端有三个主要用途：

1. **对客户端进行身份验证**：稍后将会添加消息处理程序，以便对客户端请求进行身份验证，并将用户与请求相关联。
2. **客户端通知注册**：稍后，你将要添加一个控制器来处理新的注册，使客户端设备能够接收通知。经过身份验证的用户名将作为[标记](https://msdn.microsoft.com/library/azure/dn530749.aspx)自动添加到注册。
3. **将通知发送到客户端**：稍后，你还要添加一个控制器，以便用户对与标记关联的设备和客户端触发安全推送。 

以下步骤说明了如何创建新的 ASP.NET WebAPI 后端：


> [AZURE.NOTE]**重要提示**：在开始本教程之前，请确保已安装最新版本的 NuGet 程序包管理器。若要进行检查，请启动 Visual Studio。从“工具”菜单，单击“扩展和更新”。搜索“适用于 Visual Studio 2013 的 NuGet 程序包管理器”，并且确保具有版本 2.8.50313.46 或更高版本。否则，请卸载并重新安装 NuGet 程序包管理器。
> 
> ![][B4]

> [AZURE.NOTE]请确保已安装 Visual Studio [Azure SDK](/zh-cn/downloads/) 以便进行 Web 应用部署。

1. 启动 Visual Studio 或 Visual Studio Express。单击“服务器资源管理器”并登录到你的 Azure 帐户。Visual Studio 需要你登录才能在你的帐户中创建 Web 应用资源。
2. 在 Visual Studio 中，依次单击“文件”、“新建”和“项目”，依次展开“模板”和“Visual C#”，然后依次单击“Web”和“ASP.NET Web 应用程序”，键入名称 **AppBackend**，然后单击“确定”。 
	
	![][B1]

3. 在“新建 ASP.NET 项目”对话框中，单击“Web API”，然后单击“确定”。

	![][B2]

4. 在“配置 Azure Web 应用”对话框中，选择订阅和你已创建的 **App Service 计划**。你也可以选择“创建新的 App Service 计划”，并通过对话框创建一个计划。在本教程中，你不需要使用数据库。选择 App Service 计划后，单击“确定”以创建项目。

	![][B5]



## 在 WebAPI 后端上对客户端进行身份验证

在本部分，你将为新的后端创建名为 **AuthenticationTestHandler** 的新消息处理程序类。此类衍生自 [DelegatingHandler](https://msdn.microsoft.com/library/system.net.http.delegatinghandler.aspx) 并已添加为消息处理程序，以便处理传入后端的所有请求。



1. 在“解决方案资源管理器”中，右键单击“AppBackend”项目，单击“添加”，然后单击“类”。将新类命名为 **AuthenticationTestHandler.cs**，然后单击“添加”以生成该类。通过此类可简单地使用*基本身份验证* 对用户进行身份验证。请注意，您的应用可以使用所有身份验证方案。

2. 在 AuthenticationTestHandler.cs 中，添加以下 `using` 语句：

        using System.Net.Http;
        using System.Threading;
        using System.Security.Principal;
        using System.Net;
        using System.Web;

3. 在 AuthenticationTestHandler.cs 中，将 `AuthenticationTestHandler` 类定义替换为以下代码。

	当以下三个条件都成立时，此处理程序将授权给请求：
	* 请求包含 *Authorization* 标头。
	* 请求使用*基本* 身份验证。
	* 用户名字符串和密码字符串是相同的字符串。

	否则，将会拒绝该请求。这不是真正的身份验证和授权方法。它只是本教程中一个非常简单的示例。

	如果请求消息已经过 `AuthenticationTestHandler` 的身份验证和授权，则基本身份验证用户将附加到 [HttpContext](https://msdn.microsoft.com/library/system.web.httpcontext.current.aspx) 上的当前请求。然后，另一个控制器 (RegisterController) 会使用 HttpContext 中的用户信息，将[标记](https://msdn.microsoft.com/library/azure/dn530749.aspx)添加到通知注册请求。

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
	                else return Unauthorized();
	            }
	            else return Unauthorized();
	
	            return base.SendAsync(request, cancellationToken);
	        }
	
	        private bool verifyUserAndPwd(string user, string password)
	        {
	            // This is not a real authentication scheme.
	            return user == password;
	        }
	
	        private Task<HttpResponseMessage> Unauthorized()
	        {
	            var response = new HttpResponseMessage(HttpStatusCode.Forbidden);
	            var tsc = new TaskCompletionSource<HttpResponseMessage>();
	            tsc.SetResult(response);
	            return tsc.Task;
	        }
	    }

	> [AZURE.NOTE]**安全说明**：`AuthenticationTestHandler` 类不提供真正的身份验证。它仅用于模拟基本身份验证并且是不安全的。您必须在生产应用程序和服务中实现安全的身份验证机制。

4. 在 **App\_Start/WebApiConfig.cs** 类中 `Register` 方法的末尾添加以下代码，以注册消息处理程序：

		config.MessageHandlers.Add(new AuthenticationTestHandler());

5. 保存所做更改。

## 使用 WebAPI 后端注册通知

在本部分，我们要将新的控制器添加到 WebAPI 后端来处理请求，以使用通知中心的客户端库为用户和设备注册通知。控制器将为已由 `AuthenticationTestHandler` 验证并已附加到 HttpContext 的用户添加用户标记。该标记采用以下字符串格式：`"username:<actual username>"`。


 

1. 在“解决方案资源管理器”中，右键单击“AppBackend”项目，然后单击“管理 NuGet 程序包”。

2. 在左侧，单击“联机”，然后在“搜索”框中搜索 **Microsoft.Azure.NotificationHubs**。

3. 在结果列表中，单击“Azure 通知中心”，然后单击“安装”。完成安装后，关闭“NuGet 程序包管理器”窗口。

	这将使用 <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet 程序包</a>添加对 Azure 通知中心 SDK 的引用。

4. 现在，我们要创建新的类文件，用于表示所要发送的不同安全通知。在完整的实现中，这些通知存储在某个数据库中。为简单起见，本教程将这些通知存储在内存中。在“解决方案资源管理器”中，右键单击“Models”文件夹，单击“添加”，然后单击“类”。将新类命名为 **Notifications.cs**，然后单击“添加”以生成该类。

	![][B6]

5. 在 Notifications.cs 中，在文件顶部添加以下 `using` 语句：

        using Microsoft.Azure.NotificationHubs;

6. 将 `Notifications` 类定义替换为以下内容并确保将两个占位符替换为通知中心的连接字符串（具有完全访问权限）和中心名称（可在 [Azure 经典管理门户](http://manage.windowsazure.cn)中找到）：

		public class Notifications
        {
            public static Notifications Instance = new Notifications();
        
            public NotificationHubClient Hub { get; set; }

            private Notifications() {
                Hub = NotificationHubClient.CreateClientFromConnectionString("<your hub's DefaultFullSharedAccessSignature>", 
																			 "<hub name>");
            }
        }



7. 接下来，我们将创建一个名为 **RegisterController** 的新控制器。在“解决方案资源管理器”中，右键单击“Controllers”文件夹，然后依次单击“添加”和“控制器”。单击“Web API 2 Controller -- Empty”项目，然后单击“添加”。将新类命名为 **RegisterController**，然后再次单击“添加”以生成该控制器。

	![][B7]

	![][B8]

8. 在 RegisterController.cs 中，添加以下 `using` 语句：

        using Microsoft.Azure.NotificationHubs;
		using Microsoft.Azure.NotificationHubs.Messaging;
        using AppBackend.Models;
        using System.Threading.Tasks;
        using System.Web;

9. 在 `RegisterController` 类定义中添加以下代码：请注意，在此代码中，我们将为已附加到 HttpContext 的用户添加用户标记。添加的消息筛选器 `AuthenticationTestHandler` 将对该用户进行身份验证并将其附加到 HttpContext。还可以通过添加可选复选框来验证用户是否有权注册以获取请求标记。

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
            string newRegistrationId = null;
            
            // make sure there are no existing registrations for this push handle (used for iOS and Android)
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

            if (newRegistrationId == null) 
				newRegistrationId = await hub.CreateRegistrationIdAsync();

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

10. 保存所做更改。

## 从 WebAPI 后端发送通知

在本部分，你将添加新的控制器，以便客户端设备使用 ASP.NET WebAPI 后端中的 Azure 通知中心服务管理库根据用户名标记发送通知。


1. 创建另一个名为 **NotificationsController** 的新控制器。以你在上一节中创建 **RegisterController** 的相同方式来创建新控制器。

2. 在 NotificationsController.cs 中，添加以下 `using` 语句：

        using AppBackend.Models;
        using System.Threading.Tasks;
        using System.Web;

3. 在 **NotificationsController** 类中添加以下方法。

	此代码将会根据平台通知服务 (PNS) `pns` 参数发送相应类型的通知。`to_tag` 的值用于设置消息中的 *username* 标记。此标记必须与活动的通知中心注册的用户名标记相匹配。将从 POST 请求正文提取通知消息，并根据目标 PNS 将其格式化。

	根据受支持设备用来接收通知的平台通知服务 (PNS)，支持使用不同的格式接收不同的通知。例如，在 Windows 设备上，可以使用其他 PNS 不能直接支持的 [toast 通知和 WNS](https://msdn.microsoft.com/library/windows/apps/br230849.aspx)。因此，后端需要将通知格式化为你打算使用的设备 PNS 所支持的通知。然后，对 [NotificationHubClient 类](https://msdn.microsoft.com/library/azure/microsoft.azure.notificationhubs.notificationhubclient_methods.aspx)使用相应的 send API

        public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag)
        {
            var user = HttpContext.Current.User.Identity.Name;
            string[] userTag = new string[2];
            userTag[0] = "username:" + to_tag;
            userTag[1] = "from:" + user;

            Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;
            HttpStatusCode ret = HttpStatusCode.InternalServerError;

            switch (pns.ToLower())
            {
                case "wns":
                    // Windows 8.1 / Windows Phone 8.1
                    var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" + 
                                "From " + user + ": " + message + "</text></binding></visual></toast>";
                    outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
                    break;
                case "apns":
                    // iOS
                    var alert = "{"aps":{"alert":"" + "From " + user + ": " + message + ""}}";
                    outcome = await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(alert, userTag);
                    break;
                case "gcm":
                    // Android
                    var notif = "{ "data" : {"message":"" + "From " + user + ": " + message + ""}}";
                    outcome = await Notifications.Instance.Hub.SendGcmNativeNotificationAsync(notif, userTag);
                    break;
            }

            if (outcome != null)
            {
                if (!((outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Abandoned) ||
                    (outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Unknown)))
                {
                    ret = HttpStatusCode.OK;
                }
            }

            return Request.CreateResponse(ret);
        }


4. 按 **F5** 运行应用程序并确保到目前为止操作的准确性。该应用应启动 Web 浏览器，然后显示 ASP.NET 主页。

##发布新的 WebAPI 后端

1. 现在，我们将此应用部署到 Azure Web 应用，以便可以从任意设备访问它。右键单击 **AppBackend** 项目，然后选择“发布”。

2. 选择“Azure Web Apps”作为发布目标。

    ![][B15]

3. 使用你的 Azure 帐户登录，然后选择一个现有的或新的 Web 应用。

    ![][B16]

4. 记下“连接”选项卡中的“目标 URL”属性。在本教程后面的部分中，我们将此 URL 称为*后端终结点*。单击“发布”。

    ![][B18]


[B1]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push1.png
[B2]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push2.png
[B3]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push3.png
[B4]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push4.png
[B5]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push5.png
[B6]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push6.png
[B7]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push7.png
[B8]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push8.png
[B14]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push14.png
[B15]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users15.PNG
[B16]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users16.PNG
[B18]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users18.PNG

<!---HONumber=Mooncake_0104_2016-->