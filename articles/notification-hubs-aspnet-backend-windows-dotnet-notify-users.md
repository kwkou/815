<properties 
	pageTitle="Azure 通知中心 - 使用 .NET 后端通知用户"
	description="了解如何在 Azure 中发送安全推送通知。代码示例是使用 .NET API 通过 C# 编写的。"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	services="notification-hubs"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="02/29/2016"
	wacn.date="05/30/2016"/>

#Azure 通知中心 - 使用 .NET 后端通知用户

[AZURE.INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]


##概述

利用 Azure 中的推送通知支持，你可以访问易于使用且向外扩展的多平台推送基础结构，这大大简化了为移动平台的使用者应用程序和企业应用程序实现推送通知的过程。本教程说明如何使用 Azure 通知中心将推送通知发送到特定设备上的特定应用程序用户。ASP.NET WebAPI 后端用于对客户端进行身份验证。后端使用经过身份验证的客户端用户自动将标记添加通知注册。后端将使用此标记为特定的用户生成通知。有关使用应用后端注册通知的详细信息，请参阅指南主题[从应用后端注册](http://msdn.microsoft.com/library/dn743807.aspx)。本教程以你在[通知中心入门]教程中创建的通知中心和项目为基础。

此外，只有在学习本教程后，才可以学习[安全推送]教程。完成本教程中的步骤后，你可以继续学习[安全推送]教程，其中说明了如何修改本教程中的代码以安全地发送推送通知。





## 开始之前

我们非常重视你的反馈。如果你在完成本主题的过程中遇到任何难题，或者在改善内容方面有任何建议，请在页面底部提供反馈，我们将不胜感激。

可以在 GitHub 上的[此处](https://github.com/Azure/azure-notificationhubs-samples/tree/master/dotnet/NotifyUsers)找到本教程的已完成代码。



##先决条件

在开始本教程之前，必须已完成以下移动服务教程：

+ [通知中心入门]<br/>在此教程中，你创建通知中心，保留应用名称，然后注册以接收通知。本教程假设已完成这些步骤。请遵循[通知中心入门（Windows 应用商店）](/documentation/articles/notification-hubs-windows-store-dotnet-get-started/)中的步骤；具体而言，请遵循[在 Windows 应用商店中注册你的应用](/documentation/articles/notification-hubs-windows-store-dotnet-get-started/#register-your-app-for-the-windows-store)和[配置通知中心](/documentation/articles/notification-hubs-windows-store-dotnet-get-started/#configure-your-notification-hub)部分中的步骤。请务必确保已在门户中你的通知中心的“配置”选项卡上输入了“程序包 SID”和“客户端机密”值。[配置通知中心](/documentation/articles/notification-hubs-windows-store-dotnet-get-started/#configure-your-notification-hub)部分中介绍了此配置过程。这个步骤非常重要：如果门户上的凭据与针对所选应用程序名称指定的凭据不匹配，推送通知将不会成功。




> [AZURE.NOTE]如果你使用移动服务作为后端服务，请参阅本教程的[移动服务版本](/documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-push/)。

[AZURE.INCLUDE [notification-hubs-aspnet-backend-notifyusers](../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## 更新客户端项目的代码

在本部分中，你将更新你在[通知中心入门]教程中完成的项目中的代码。这些代码应该已与应用商店关联，并已针对通知中心进行配置。在本部分中，你将添加代码以调用新的 WebAPI 后端，并使用该后端来注册和发送通知。

1. 在 Visual Studio 中，打开为[通知中心入门]教程创建的解决方案。

2. 在“解决方案资源管理器”中，右键单击“(Windows 8.1)”项目，然后单击“管理 NuGet 包”。

3. 在左侧单击“联机”。

4. 在“搜索”框中键入 **Http 客户端**。

5. 在结果列表中，单击“Microsoft HTTP 客户端库”，然后单击“安装”。完成安装。

6. 返回到 NuGet“搜索”框，键入 **Json.net**。安装 **Json.NET** 包，然后关闭“NuGet 包管理器”窗口。

7. 针对“(Windows Phone 8.1)”项目重复上述步骤，以安装 Windows Phone 项目的 **JSON.NET** NuGet 包。


8. 在解决方案资源管理器中的“(Windows 8.1)”项目内，双击“MainPage.xaml”在 Visual Studio 编辑器中打开它。

9. 在 **MainPage.xaml** XML 代码中，将 `<Grid>` 节替换为以下代码：此代码将添加用户用来进行身份验证的用户名和密码文本框。它还会添加通知消息的文本框，以及应接收通知的用户名标记：

        <Grid>
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
            </Grid.RowDefinitions>

            <TextBlock Grid.Row="0" Text="Notify Users" HorizontalAlignment="Center" FontSize="48"/>

            <StackPanel Grid.Row="1" VerticalAlignment="Center">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="Auto"/>
                    </Grid.RowDefinitions>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition></ColumnDefinition>
                        <ColumnDefinition></ColumnDefinition>
                        <ColumnDefinition></ColumnDefinition>
                    </Grid.ColumnDefinitions>
                    <TextBlock Grid.Row="0" Grid.ColumnSpan="3" Text="Username" FontSize="24" Margin="20,0,20,0"/>
                    <TextBox Name="UsernameTextBox" Grid.Row="1" Grid.ColumnSpan="3" Margin="20,0,20,0"/>
                    <TextBlock Grid.Row="2" Grid.ColumnSpan="3" Text="Password" FontSize="24" Margin="20,0,20,0" />
                    <PasswordBox Name="PasswordTextBox" Grid.Row="3" Grid.ColumnSpan="3" Margin="20,0,20,0"/>

                    <Button Grid.Row="4" Grid.ColumnSpan="3" HorizontalAlignment="Center" VerticalAlignment="Center"
                                Content="1. Login and register" Click="LoginAndRegisterClick" Margin="0,0,0,20"/>

                    <ToggleButton Name="toggleWNS" Grid.Row="5" Grid.Column="0" HorizontalAlignment="Right" Content="WNS" IsChecked="True" />
                    <ToggleButton Name="toggleGCM" Grid.Row="5" Grid.Column="1" HorizontalAlignment="Center" Content="GCM" />
                    <ToggleButton Name="toggleAPNS" Grid.Row="5" Grid.Column="2" HorizontalAlignment="Left" Content="APNS" />

                    <TextBlock Grid.Row="6" Grid.ColumnSpan="3" Text="Username Tag To Send To" FontSize="24" Margin="20,0,20,0"/>
                    <TextBox Name="ToUserTagTextBox" Grid.Row="7" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                    <TextBlock Grid.Row="8" Grid.ColumnSpan="3" Text="Enter Notification Message" FontSize="24" Margin="20,0,20,0"/>
                    <TextBox Name="NotificationMessageTextBox" Grid.Row="9" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                    <Button Grid.Row="10" Grid.ColumnSpan="3" HorizontalAlignment="Center" Content="2. Send push" Click="PushClick" Name="SendPushButton" />
                </Grid>
            </StackPanel>
        </Grid>


10. 在“解决方案资源管理器”的“(Windows Phone 8.1)”项目中，打开 **MainPage.xaml**，并将 Windows Phone 8.1 `<Grid>` 节替换为上述相同的代码。界面看起来应如下所示。

	![][13]

11. 在“解决方案资源管理器”中，打开“(Windows 8.1)”和“(Windows 8.1)”项目的 **MainPage.xaml.cs** 文件。在这两个文件顶部添加以下 `using` 语句：

		using System.Net.Http;
		using Windows.Storage;
		using System.Net.Http.Headers;
		using Windows.Networking.PushNotifications;
		using Windows.UI.Popups;
		using System.Threading.Tasks;

12. 在“(Windows 8.1)”和“(Windows 8.1)”项目的 **MainPage.xaml.cs** 中，将以下成员添加 `MainPage` 类。确保使用前面获取的实际后端终结点来替换 `<Enter Your Backend Endpoint>`。例如，`http://mybackend.azurewebsites.net`。

        private static string BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";



13. 将以下代码添加到“(Windows 8.1)”和“(Windows Phone 8.1)”项目的 **MainPage.xaml.cs** 中的 MainPage 类。

	`PushClick` 方法是“发送推送”按钮的单击处理程序。它调用后端以触发向用户名标记与 `to_tag` 参数匹配的所有设备发送通知。通知消息作为请求正文中的 JSON 内容发送。

	`LoginAndRegisterClick` 方法是“登录和注册”按钮的单击处理程序。它在本地存储中存储基本身份验证令牌（请注意，这代表身份验证方案使用的任何令牌），然后使用 `RegisterClient` 来通过后端注册通知。


        private async void PushClick(object sender, RoutedEventArgs e)
        {
            if (toggleWNS.IsChecked.Value)
            {
                await sendPush("wns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
            }
            if (toggleGCM.IsChecked.Value)
            {
                await sendPush("gcm", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
            }
            if (toggleAPNS.IsChecked.Value)
            {
                await sendPush("apns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);

            }
        }

        private async Task sendPush(string pns, string userTag, string message)
        {
            var POST_URL = BACKEND_ENDPOINT + "/api/notifications?pns=" +
                pns + "&to_tag=" + userTag;

            using (var httpClient = new HttpClient())
            {
                var settings = ApplicationData.Current.LocalSettings.Values;
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                try
                {
                    await httpClient.PostAsync(POST_URL, new StringContent(""" + message + """,
                        System.Text.Encoding.UTF8, "application/json"));
                }
                catch (Exception ex)
                {
                    MessageDialog alert = new MessageDialog(ex.Message, "Failed to send " + pns + " message");
                    alert.ShowAsync();
                }
            }
        }

        private async void LoginAndRegisterClick(object sender, RoutedEventArgs e)
        {
            SetAuthenticationTokenInLocalStorage();

            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

            // The "username:<user name>" tag gets automatically added by the message handler in the backend.
            // The tag passed here can be whatever other tags you may want to use.
            try
            {
				// The device handle used will be different depending on the device and PNS. 
				// Windows devices use the channel uri as the PNS handle.
                await new RegisterClient(BACKEND_ENDPOINT).RegisterAsync(channel.Uri, new string[] { "myTag" });

                var dialog = new MessageDialog("Registered as: " + UsernameTextBox.Text);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
                SendPushButton.IsEnabled = true;
            }
            catch (Exception ex)
            {
                MessageDialog alert = new MessageDialog(ex.Message, "Failed to register with RegisterClient");
                alert.ShowAsync();
            }
        }

        private void SetAuthenticationTokenInLocalStorage()
        {
            string username = UsernameTextBox.Text;
            string password = PasswordTextBox.Password;

            var token = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(username + ":" + password));
            ApplicationData.Current.LocalSettings.Values["AuthenticationToken"] = token;
        }



14. 在“解决方案资源管理器”中的“共享”项目下，打开 **App.xaml.cs** 文件。在 `OnLaunched()` 事件处理程序中，查找对 `InitNotificationsAsync()` 的调用。注释掉或删除对 `InitNotificationsAsync()` 的调用。上面添加的按钮处理程序将初始化通知注册。


        protected override void OnLaunched(LaunchActivatedEventArgs e)
        {
            //InitNotificationsAsync();


15. 在“解决方案资源管理器”中，右键单击“共享”项目，然后依次单击“添加”和“类”。将类命名为 **RegisterClient.cs**，然后单击“确定”以生成该类。
	
	此类将包装所需的 REST 调用，以便能够联系应用程序后端来注册推送通知。它还会在本地存储通知中心创建的 *registrationIds*（从[应用后端注册](http://msdn.microsoft.com/library/dn743807.aspx)中提供了详细信息）。请注意，该组件使用当你单击“登录并注册”按钮时存储在本地存储中的授权令牌。


16. 在 RegisterClient.cs 文件的顶部添加以下 `using` 语句：

		using Windows.Storage;
		using System.Net;
		using System.Net.Http;
		using System.Net.Http.Headers;
		using Newtonsoft.Json;
		using System.Threading.Tasks;
		using System.Linq;

17. 在 `RegisterClient` 类定义中添加以下代码：

		private string POST_URL;

        private class DeviceRegistration
        {
            public string Platform { get; set; }
            public string Handle { get; set; }
            public string[] Tags { get; set; }
        }

        public RegisterClient(string backendEndpoint)
        {
            POST_URL = backendEndpoint + "/api/register";
        }

        public async Task RegisterAsync(string handle, IEnumerable<string> tags)
        {
            var regId = await RetrieveRegistrationIdOrRequestNewOneAsync();

            var deviceRegistration = new DeviceRegistration
            {
                Platform = "wns",
                Handle = handle,
                Tags = tags.ToArray<string>()
            };

            var statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);

            if (statusCode == HttpStatusCode.Gone)
            {
                // regId is expired, deleting from local storage & recreating
                var settings = ApplicationData.Current.LocalSettings.Values;
                settings.Remove("__NHRegistrationId");
                regId = await RetrieveRegistrationIdOrRequestNewOneAsync();
                statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);
            }

            if (statusCode != HttpStatusCode.Accepted)
            {
                // log or throw
				throw new System.Net.WebException(statusCode.ToString());
            }
        }

        private async Task<HttpStatusCode> UpdateRegistrationAsync(string regId, DeviceRegistration deviceRegistration)
        {
            using (var httpClient = new HttpClient())
            {
                var settings = ApplicationData.Current.LocalSettings.Values;
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string) settings["AuthenticationToken"]);

                var putUri = POST_URL + "/" + regId;

                string json = JsonConvert.SerializeObject(deviceRegistration);
                                var response = await httpClient.PutAsync(putUri, new StringContent(json, Encoding.UTF8, "application/json"));
                return response.StatusCode;
            }
        }

        private async Task<string> RetrieveRegistrationIdOrRequestNewOneAsync()
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            if (!settings.ContainsKey("__NHRegistrationId"))
            {
                using (var httpClient = new HttpClient())
                {
                    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                    var response = await httpClient.PostAsync(POST_URL, new StringContent(""));
                    if (response.IsSuccessStatusCode)
                    {
                        string regId = await response.Content.ReadAsStringAsync();
                        regId = regId.Substring(1, regId.Length - 2);
                        settings.Add("__NHRegistrationId", regId);
                    }
                    else
                    {
						throw new System.Net.WebException(response.StatusCode.ToString());
                    }
                }
            }
            return (string)settings["__NHRegistrationId"];

        }

18. 保存所有更改。
		

## 测试应用程序

1. 在 Windows 8.1 和 Windows Phone 8.1 上启动应用程序。对于 Windows Phone 8.1，可以在模拟器或实际设备中运行实例。

2. 在应用的 Windows 8.1 实例中，输入“用户名”和“密码”，如以下屏幕中所示。这应该与在 Windows Phone 上输入的用户名和密码不同。


3. 单击“登录和注册”，然后验证是否会确认显示你已登录的对话框。这也会启用“发送推送”按钮。

    ![][14]

4. 在 Windows Phone 8.1 实例上，于“用户名”和“密码”字段中输入用户名字符串，然后单击“登录和注册”。
5. 然后，在“接收方用户名标记”字段中，输入在 Windows 8.1 上注册的用户名。输入通知消息，然后单击“发送推送”。

    ![][16]

6. 只有已使用匹配用户名标记进行注册的设备才会收到通知消息。

	![][15]

## 后续步骤

* 如果要按兴趣组划分用户，可以阅读[使用通知中心发送突发新闻]。
* 若要了解有关如何使用通知中心的详细信息，请参阅[通知中心指南]。



[9]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push9.png
[10]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push10.png
[11]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push11.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-ui.png
[14]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-windows-instance-username.png
[15]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-notification-received.png
[16]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-send-message.png



<!-- URLs. -->
[通知中心入门]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[安全推送]: /documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-secure-push/
[使用通知中心发送突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/
[通知中心指南]: http://msdn.microsoft.com/library/jj927170.aspx
<!---HONumber=Mooncake_0523_2016-->