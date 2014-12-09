<properties linkid="notification-hubs-how-to-guides-howto-notify-users-aspnet" urlDisplayName="Notify Users" pageTitle="使用通知中心通知用户 ASP.NET 服务事件" metaKeywords="" description="遵循本教程来注册到通知中心以从 ASP.NET 服务接收通知" metaCanonical="" services="notification-hubs" documentationCenter="" title="使用通知中心通知用户" authors="glenga" solutions="" manager="" editor="" />

# 使用通知中心通知用户

<div class="dev-center-tutorial-selector sublanding">

[移动服务][移动服务][ASP.NET][ASP.NET]

</div>

本教程演示如何使用 Azure 通知中心将推送通知发送到特定设备上的特定应用程序用户。使用 ASP.NET Web API 后端对客户端进行身份验证和生成通知。本教程以您在前面的**通知中心入门**教程中创建的通知中心为基础。将通知注册代码从客户端移到后端服务。这确保仅在客户端已经过服务验证后才完成注册。它还表示通知中心凭据不随客户端应用程序一起分发。服务还控制在注册期间请求的标签。

本教程将指导您完成以下基本步骤：

-   [创建具有身份验证功能的 ASP.NET 应用程序][创建具有身份验证功能的 ASP.NET 应用程序]
-   [更新 ASP.NET 应用程序以注册通知][更新 ASP.NET 应用程序以注册通知]
-   [更新应用程序以登录和请求注册][更新应用程序以登录和请求注册]

## 先决条件

-   Visual Studio 2012。您还可以使用 Visual Studio Express 2012 for Web 和 Visual Studio Express 2012 for Windows 8 来分别创建 ASP.NET 应用程序和 Windows 应用商店应用程序。
-   本教程以您在**通知中心入门**中创建的应用程序和通知中心为基础。在您开始本教程之前，必须先完成**通知中心入门**教程（[Windows 应用商店 C#][Windows 应用商店 C#]/[iOS][iOS]/[Android][Android]）。

<div class="dev-callout">

**说明**
您在本教程中创建的 ASP.NET Web API 项目将在本地计算机上运行。还可以将 ASP.NET Web API 项目发布到 Azure。有关更多信息，请参见[使用 ASP.NET Web API 和 SQL Database 创建具有良好移动性的 REST 服务][使用 ASP.NET Web API 和 SQL Database 创建具有良好移动性的 REST 服务]。

</div>

## <a name="create-application"></a><span class="short-header">创建 ASP.NET 应用程序</span>创建具有身份验证功能的 ASP.NET 应用程序

首先您将创建一个 ASP.NET Web API 应用程序。此后端服务将对客户端进行身份验证，代表经过身份验证的用户注册推送通知以及发送通知。

1.  在 Visual Studio 或 Visual Studio Express 2012 for Web 中，单击“文件”，然后从“文件”菜单中单击“新建项目”，展开“模板”和“Visual C#”，然后单击“Web”和“ASP.NET MVC 4 Web 应用程序”，输入名称 *NotificationService*，然后单击“确定”。

    ![][0]

2.  在“新建 ASP.NET MVC 项目”对话框中，单击“空”，然后单击“确定”。

    这将创建一个 ASP.NET MVC 项目。

3.  在解决方案资源管理器中，右键单击该项目，依次单击“添加”和“类”，然后键入 `AuthenticationTestHandler`，并单击“添加”。

    ![][1]

    这会将 **AuthenticationTestHandler** 类的代码文件添加到该项目。

4.  打开新的 AuthenticationTestHandler.cs 项目文件，将类定义替换为以下代码：

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Net;
        using System.Net.Http;
        using System.Threading;
        using System.Threading.Tasks;
        using System.Security.Principal;
        using System.Text;
        using System.Web;

        namespace NotificationService
        {
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
                    } else return Unauthorised();

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
        } 

    <div class="dev-callout">

    **安全说明**
    **AuthenticationTestHandler** 类不提供真正的身份验证。它仅用于模拟基本身份验证并返回一个主体。需要该用户名来创建通知中心注册。以上实现是不安全的。您必须在生产应用程序和服务中实现安全的身份验证机制。

    </div>

5.  展开 **App\_Start** 文件夹，打开 WebApiConfig.cs 文件，并在 **Register** 方法末尾添加以下代码行：

        config.MessageHandlers.Add(new AuthenticationTestHandler());

    这要求对 ASP.NET 应用程序的请求包含“授权”标头。

现在我们已创建了具有模拟身份验证方案的基本应用程序，它为我们提供一个用户名。

## <a name="register-notification"></a><span class="short-header">注册通知</span>更新 ASP.NET 应用程序以注册通知

下一步是通过创建新的 **Registration** 控制器来向 ASP.NET 应用程序添加通知中心的注册逻辑。

1.  登录到 [Azure 管理门户][Azure 管理门户]，依次单击“Service Bus”、您的命名空间和“通知中心”，然后选择您的通知中心并单击“连接信息”。

    ![][2]

2.  记录通知中心的名称并复制 **DefaultFullSharedAccessSignature** 的连接字符串。

    ![][3]

    您将使用此连接字符串和通知中心名称来注册和发送通知。

3.  在“解决方案资源管理器”中，右键单击该项目名称，然后选择“管理 NuGet 包”。

4.  在左窗格中，选择“联机”类别，搜索 `WindowsAzure.ServiceBus`，在 **Azure Service Bus** 包上单击“安装”，然后接受许可协议。

    ![][4]

    这会向您的项目添加 Microsoft.ServiceBus.dll 程序集。

5.  在解决方案资源管理器中，右键单击 **Controllers** 文件夹，依次单击“添加”和“控制器...”，然后键入 `RegisterController` 作为**控制器名称**，选择“空 API 控制器”并单击“添加”。

    ![][5]

    这会向项目添加一个控制器类。此控制器在被调用时将设备注册到通知中心。

6.  打开新的 RegisterController.cs 项目文件并添加以下 **using** 语句。

        using Microsoft.ServiceBus.Notifications;
        using Newtonsoft.Json.Linq;
        using System.Threading.Tasks;
        using System.Web;

7.  将以下代码添加到新的 RegisterController 类：

        // Define the Notification Hubs client.
        private NotificationHubClient hubClient;

        // Create the client in the constructor.
        public RegisterController()
        {
            var cn = "<FULL_SAS_CONNECTION_STRING>";
            hubClient = NotificationHubClient
                .CreateClientFromConnectionString(cn, "<NOTIFICATION_HUB_NAME>");
        }

8.  更新构造函数中的代码以将 *`<NOTIFICATION_HUB_NAME>`* 和 *`<FULL_SAS_CONNECTION_STRING>`* 替换为用于您通知中心的值，然后单击“保存”。

    <div class="dev-callout">

    **说明**
    确保将 **DefaultFullSharedAccessSignature** 用于 *`<FULL_SAS_CONNECTION_STRING>`*。此声明允许您的 Web API 创建和更新注册。

    </div>

9.  将以下 Post 方法代码添加到 RegisterController 类：

        public async Task<RegistrationDescription> Post([FromBody]JObject registrationCall)
        {
            // Get the registration info that we need from the request. 
            var platform = registrationCall["platform"].ToString();
            var installationId = registrationCall["instId"].ToString();
            var channelUri = registrationCall["channelUri"] != null ? 
                registrationCall["channelUri"].ToString() : null;
            var deviceToken = registrationCall["deviceToken"] != null ? 
                registrationCall["deviceToken"].ToString() : null;       
            var userName = HttpContext.Current.User.Identity.Name;

            // Get registrations for the current installation ID.
            var regsForInstId = await hubClient.GetRegistrationsByTagAsync(installationId, 100);

            bool updated = false;
            bool firstRegistration = true;
            RegistrationDescription registration = null;

            // Check for existing registrations.
            foreach (var registrationDescription in regsForInstId)
            {
                if (firstRegistration)
                {
                    // Update the tags.
                    registrationDescription.Tags = new HashSet<string>() { installationId, userName };

                    // We need to handle each platform separately.
                    switch (platform)
                    {
                        case "windows":
                            var winReg = registrationDescription as WindowsRegistrationDescription;
                            winReg.ChannelUri = new Uri(channelUri);
                            registration = await hubClient.UpdateRegistrationAsync(winReg);                            
                            break;
                        case "ios":
                            var iosReg = registrationDescription as AppleRegistrationDescription;
                            iosReg.DeviceToken = deviceToken;
                            registration = await hubClient.UpdateRegistrationAsync(iosReg);
                            break;
                    }
                    updated = true;
                    firstRegistration = false;
                }
                else
                {
                    // We shouldn't have any extra registrations; delete if we do.
                    await hubClient.DeleteRegistrationAsync(registrationDescription);
                }
            }

            // Create a new registration.
            if (!updated)
            {
                switch (platform)
                {
                    case "windows":
                        registration = await hubClient.CreateWindowsNativeRegistrationAsync(channelUri, 
                            new string[] { installationId, userName });
                        break;
                    case "ios":
                        registration = await hubClient.CreateAppleNativeRegistrationAsync(deviceToken, 
                            new string[] { installationId, userName });
                        break;
                }
            }

            // Send out a test notification.
            sendNotification(string.Format("Test notification for {0}", userName), userName);

            return registration;
        }

    此代码由 POST 请求调用，以从消息正文获取平台和设备 ID 信息。此数据和请求标头中的安装 ID 以及登录用户的用户 ID 一起用于更新注册。如果注册不存在，将创建一个新注册。此注册使用用户 ID 和安装 ID 进行标记。

10. 添加以下 sendNotification 方法：

        // Basic implementation that sends a not ification to Windows Store and iOS app clients.
        private async Task sendNotification(string notificationText, string tag)
        {
            try
            {
                // Create notifications for both Windows Store and iOS platforms.
                var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
                    notificationText + "</text></binding></visual></toast>";
                var alert = "{\"aps\":{\"alert\":\"" + notificationText + 
                    "\"}, \"inAppMessage\":\"" + notificationText + "\"}";

                // Send a notification to the logged-in user on both platforms.
                await hubClient.SendWindowsNativeNotificationAsync(toast, tag);
                await hubClient.SendAppleNativeNotificationAsync(alert, tag);
            }
            catch(ArgumentException ex)
            {
                // This is expected when an APNS registration doesn't exist.
                Console.WriteLine(ex.Message);
            }
        }

    调用此方法以在注册完成时立即发送通知。

接下来，我们将更新您在完成**通知中心入门**教程时创建的客户端应用程序。

## <a name="update-app"></a><span class="short-header">更新应用程序</span>更新应用程序以登录和请求注册

您在完成**通知中心入门**教程时所创建的应用程序会直接从通知中心请求注册。您将从客户端应用程序中删除此注册代码并将其替换为对 ASP.NET Web API 应用程序中新注册 API 的调用。

1.  按 F5 以启动 ASP.NET 应用程序并尝试加载默认页。

    显示浏览器时，记下所请求的网站的主机名。在更新客户端应用程序时，您将需要此根 URL。

    <div class="dev-callout">

    **说明**
    使用本地 IIS Web 服务器或 Visual Studio Development Server 时，还必须指定端口号。请注意，将返回一个 404 错误，因为我们在此应用程序中未实现默认页。

    </div>

2.  根据您的客户端平台，按**通过使用 ASP.NET Web API 注册推送通知的当前用户**的以下版本之一中的步骤操作：

    -   [Windows 应用商店 C# 版本][Windows 应用商店 C# 版本]
    -   [iOS 版本][iOS 版本]

3.  运行更新的应用程序，使用用于用户名和密码的同一字符串登录服务，然后验证是否显示分配给通知的注册 ID。

    您还将接收一个推送通知。

    <div class="dev-callout">

    **说明**
    当缺少注册的平台（请求将通知发送到该平台）时，会在后端引发错误。在本示例中，可以忽略此错误。若要了解如何使用模板来避免此情况，请参见[使用通知中心向用户发送跨平台通知][使用通知中心向用户发送跨平台通知]。

    </div>

4.  （可选）将客户端应用程序部署到第二个设备，然后运行该应用程序并插入文本。

    通知将显示在每个设备上。

## <a name="next-steps"> </a> 后续步骤

现在您已完成本教程，请考虑继续学习以下教程：

-   **使用通知中心发送突发新闻（[Windows 应用商店 C#][6] / [iOS][6]）**
    此平台特定的教程演示如何使用标签来允许用户订阅他们感兴趣的通知类型。

-   **[使用通知中心向用户发送跨平台通知][使用通知中心向用户发送跨平台通知]**
    此教程是对当前**使用通知中心通知用户**教程的扩展，以使用平台特定的模板来注册通知。这允许您使用服务器端代码中的单个方法来发送通知。

有关通知中心的更多信息，请参见 [Azure 通知中心][Azure 通知中心]。

<!-- Anchors. --> <!-- Images. --> <!-- URLs. -->

  [移动服务]: /zh-cn/documentation/articles/notification-hubs-mobile-services-cross-platform-notify-users/ "移动服务"
  [ASP.NET]: /zh-cn/documentation/articles/notification-hubs-aspnet-cross-platform-notify-users/ "ASP.NET"
  [创建具有身份验证功能的 ASP.NET 应用程序]: #create-application
  [更新 ASP.NET 应用程序以注册通知]: #register-notification
  [更新应用程序以登录和请求注册]: #update-app
  [Windows 应用商店 C#]: /zh-cn/manage/services/notification-hubs/getting-started-windows-dotnet
  [iOS]: /zh-cn/manage/services/notification-hubs/get-started-notification-hubs-ios
  [Android]: /zh-cn/manage/services/notification-hubs/get-started-notification-hubs-android
  [使用 ASP.NET Web API 和 SQL Database 创建具有良好移动性的 REST 服务]: /zh-cn/develop/net/tutorials/rest-service-using-web-api/
  [0]: ./media/notification-hubs-aspnet-notify-users/notification-hub-create-mvc-app.png
  [1]: ./media/notification-hubs-aspnet-notify-users/notification-hub-create-aspnet-class.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [2]: ./media/notification-hubs-aspnet-notify-users/notification-hub-select-hub-connection.png
  [3]: ./media/notification-hubs-aspnet-notify-users/notification-hub-connection-strings.png
  [4]: ./media/notification-hubs-aspnet-notify-users/notification-hub-add-nuget-package.png
  [5]: ./media/notification-hubs-aspnet-notify-users/notification-hub-add-register-controller2.png
  [Windows 应用商店 C# 版本]: /zh-cn/manage/services/notification-hubs/register-users-aspnet-dotnet
  [iOS 版本]: /zh-cn/manage/services/notification-hubs/howto-register-user-with-aspnet-ios
  [使用通知中心向用户发送跨平台通知]: /zh-cn/manage/services/notification-hubs/notify-users-xplat-aspnet
  [6]: /zh-cn/manage/services/notification-hubs/breaking-news-dotnet
  [Azure 通知中心]: http://msdn.microsoft.com/zh-cn/library/azure/jj927170.aspx
