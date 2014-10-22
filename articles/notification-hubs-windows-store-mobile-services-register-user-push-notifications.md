<properties linkid="notification-hubs-how-to-guides-howto-register-user-with-mobile-service-windowsphonedotnet" urlDisplayName="Notify Windows Store app users by using Mobile Services" pageTitle="Register the current user for push notifications by using a mobile service - Notification Hubs" metaKeywords="Azure registering application, Notification Hubs, Azure push notifications, push notification Windows Store app" description="Learn how to request push notification registration in a Windows Store app with Azure Notification Hubs when registeration is performed by Azure Mobile Services." metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="" title="Register the current user for push notifications by using a mobile service" authors="" solutions="" manager="" editor="" />

# 通过使用移动服务注册推送通知的当前用户

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/documentation/articles/notification-hubs-windows-store-mobile-services-register-user-push-notifications/" title="Windows 应用商店 C#" class="current">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-mobile-services-register-user-push-notifications/" title="iOS">iOS</a>
</div>

本主题说明在 Azure 移动服务执行注册时如何请求向 Azure 通知中心注册推送通知。它是对教程[使用通知中心通知用户][使用通知中心通知用户]的扩展。你必须在该教程中已完成创建经过身份验证的移动服务所需的步骤。有关通知用户方案的详细信息，请参阅[使用通知中心通知用户][使用通知中心通知用户]。

1.  在 Visual Studio 2012 Express for Windows 8 中，打开你在完成基础教程[身份验证入门][身份验证入门]时创建的项目。

2.  在解决方案资源管理器中，右键单击项目，单击“应用商店”，然后单击“将应用程序与应用商店关联...”。

    ![][]

    此时将显示“将应用程序与 Windows 应用商店关联”向导。

3.  在该向导中，单击“登录”，然后用你的 Microsoft 帐户登录。

4.  选择在[使用通知中心通知用户][使用通知中心通知用户]中注册的应用程序，单击“下一步”，然后单击“关联”。

    ![][1]

    这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

    <div class="dev-callout"><b>说明</b>
<p>这会将通知中心教程应用程序中的 Windows 应用商店注册重新用于此移动服务应用程序。这可能会阻止通知中心教程应用程序接收通知</p>
</div>

5.  在解决方案资源管理器中，双击 ackage.appxmanifest 项目文件以在 Visual Studio 编辑器中将其打开。

6.  向下滚动至“所有图像资产”，然后单击“徽章徽标”。在“通知”中，将“支持 Toast 通知”设置为“是”：

    ![][2]

    这允许此移动服务教程应用程序接收 toast 通知。

7.  在 Visual Studio 中，打开 MainPage.xaml.cs 文件并添加定义 **NotificationRequest** 和 **RegistrationResult** 类的以下代码：

        public class NotificationRequest
        {
            public string channelUri { get; set; }
            public string platform { get; set; }
        }

        public class RegistrationResult
        {
            public string RegistrationId { get; set; }
            public string ExpirationTime { get; set; }
        }

    这些类将分别保存请求主体和在调用自定义 API 时返回的注册 ID。

8.  在 **MainPage** 类中，添加以下方法：

        private async System.Threading.Tasks.Task RegisterNotification()
        {
            string message;

            // Get a channel for push notifications.
            var channel =
                await Windows.Networking.PushNotifications
                .PushNotificationChannelManager
                .CreatePushNotificationChannelForApplicationAsync();

            // Create the body of the request.
            var body = new NotificationRequest 
            {
                channelUri = channel.Uri, 
                platform = "win8" 
            }; 

            try
            {
                // Call the custom API POST method with the supplied body.
                var result = await App.MobileService
                    .InvokeApiAsync<NotificationRequest, 
                    RegistrationResult>("register_notifications", body,
                    System.Net.Http.HttpMethod.Post, null);

                // Set the response, which is the ID of the registration.
                message = string.Format("Registration ID: {0}", result.RegistrationId);
            }
            catch (MobileServiceInvalidOperationException ex)
            {
                message = ex.Message;
            }

            // Display a message dialog.
            var dialog = new MessageDialog(message);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }

    此方法为推送通知创建一个通道并将它与设备类型一起发送到在通知中心创建注册的自定义 API 方法。此自定义 API 已在[使用通知中心通知用户][使用通知中心通知用户]中定义。

9.  将以下代码行添加到 **OnNavigatedTo** 方法，就在调用 **Authenticate** 方法之后：

        await RegisterNotification();

    <div class="dev-callout"><b>说明</b>
<p>这可以确保每次加载页时都会请求注册。在应用程序中，你可能只需要定期执行此注册以确保注册是最新的。</p>
</div>

现在客户端应用程序已更新，请返回到[使用通知中心通知用户][使用通知中心通知用户]并更新移动服务以使用通知中心发送通知。

<!-- Anchors. --> 

<!-- URLs. -->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-mobile-services-register-user-push-notifications/ "Windows 应用商店 C#"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-mobile-services-register-user-push-notifications/ "iOS"
  [使用通知中心通知用户]: /en-us/manage/services/notification-hubs/notify-users
  [身份验证入门]: /en-us/develop/mobile/tutorials/get-started-with-users-dotnet/

<!-- Images. --> 

  []: ./media/notification-hubs-windows-store-mobile-services-register-user-push-notifications/mobile-services-select-app-name.png
  [1]: ./media/notification-hubs-windows-store-mobile-services-register-user-push-notifications/notification-hub-associate-win8-app.png
  [2]: ./media/notification-hubs-windows-store-mobile-services-register-user-push-notifications/notification-hub-win8-app-toast.png
