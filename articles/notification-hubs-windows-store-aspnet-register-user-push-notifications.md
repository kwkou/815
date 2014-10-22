<properties linkid="notification-hubs-how-to-guides-howto-register-user-with-aspnet-webapi-windowsphonedotnet" urlDisplayName="Notify Windows Store app users by using Web API" pageTitle="Register the current user for push notifications by using Web API - Notification Hubs" metaKeywords="Azure registering application, Notification Hubs, Azure push notifications, push notification Windows Store app" description="Learn how to request push notification registration in a Windows Store app with Azure Notification Hubs when registeration is performed by ASP.NET Web API." metaCanonical="" services="notification-hubs" documentationCenter="" title="Register the current user for push notifications by using ASP.NET" authors="" solutions="" manager="" editor="" />

# 通过使用 ASP.NET 注册推送通知的当前用户

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/documentation/articles/notification-hubs-windows-store-aspnet-register-user-push-notifications/" title="Windows 应用商店 C#" class="current">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-aspnet-register-user-push-notifications/" title="iOS">iOS</a>
</div>

本主题演示在 ASP.NET Web API 执行注册时如何请求向 Azure 通知中心注册推送通知。它是对教程[使用通知中心通知用户][使用通知中心通知用户]的扩展。你必须在该教程中已完成创建经过身份验证的移动服务所需的步骤。有关通知用户方案的详细信息，请参阅[使用通知中心通知用户][使用通知中心通知用户]。

1.  在 Visual Studio 2012 中，在你完成基础教程[使用通知中心通知用户][使用通知中心通知用户]时创建的项目中打开 app.xaml.cs 文件。

2.  找到 **InitNotificationsAsync** 方法并在方法中注释掉该代码。

    你将改用 ASP.NET Web API 来注册通知。

3.  在“解决方案资源管理器”中，右键单击该项目名称，然后选择“管理 NuGet 包”。

4.  在左窗格中，选择“联机”类别，搜索`json.net`，单击 **Json.NET** 包对应的“安装”，然后接受许可协议。

    ![][]

    这会向你的项目添加第三方 Newtonsoft.Json.dll 程序集。

5.  在解决方案资源管理器中，右键单击该项目，依次单击“添加”和“类”，键入 `LocalStorageManager` 并单击“添加”。

    ![][1]

    这会将 **LocalStorageManager** 类的代码文件添加到该项目。

6.  打开新的 LocalStorageManager.cs 项目文件并添加以下 **using** 语句：

        using Windows.Storage;

7.  将 **LocalStorageManager** 类定义替换为以下代码：

        class LocalStorageManager
        {
            private static ApplicationDataContainer GetLocalStorageContainer()
            {
                if (!ApplicationData.Current.LocalSettings
                    .Containers.ContainsKey("InstallationContainer"))
                {
                    ApplicationData.Current.LocalSettings
                        .CreateContainer("InstallationContainer",                                                            
                        ApplicationDataCreateDisposition.Always);
                }
                return ApplicationData.Current.LocalSettings
                    .Containers["InstallationContainer"];
            }

            public static string GetInstallationId()
            {
                var container = GetLocalStorageContainer();
                if (!container.Values.ContainsKey("InstallationId"))
                {
                    container.Values["InstallationId"] = Guid.NewGuid().ToString();
                }
                return (string)container.Values["InstallationId"];
            }
        }

    此代码创建和存储设备特定的安装 ID，在创建通知时该 ID 将作为标记。如果存在安装 ID，则使用它。

8.  打开 MainPage.xaml 项目文件，然后使用以下 XAML 代码替换根 **Grid** 元素：

        <Grid Background="{StaticResource ApplicationPageBackgroundThemeBrush}">
            <Grid Margin="120, 58, 120, 80" >
                <Grid.RowDefinitions>
                    <RowDefinition />
                    <RowDefinition/>
                    <RowDefinition/>
                    <RowDefinition/>
                    <RowDefinition/>
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition MaxWidth="200"/>
                    <ColumnDefinition/>
                </Grid.ColumnDefinitions>
                <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  
                               TextWrapping="Wrap" Text="Push To Users with Notification Hubs" 
                               FontSize="42"  VerticalAlignment="Top"/>
                <TextBlock Grid.Row="1" Grid.Column="0" Text="Installation Id:" 
                               FontSize="24" VerticalAlignment="Center" />
                <TextBlock Grid.Row="1" Grid.Column="1" Name="InstId" FontSize="24" 
                               VerticalAlignment="Center"/>
                <TextBlock Grid.Row="2" Grid.Column="0" Text="Username:" FontSize="24" 
                               VerticalAlignment="Center"/>
                <TextBox Grid.Row="2" Grid.Column="1" Name="User" FontSize="24" 
                             Width="300" Height="40" HorizontalAlignment="Left"/>
                <TextBlock Grid.Row="3" Grid.Column="0" Text="Password:" FontSize="24" 
                               VerticalAlignment="Center" />
                <PasswordBox Grid.Row="3" Grid.Column="1" Name="Password" FontSize="24" 
                                 Width="300" Height="40" HorizontalAlignment="Left"/>
                <Button Name="Login" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" 
                            Content="Login and Register for Push" FontSize="24" Click="Login_Click" />
            </Grid>
        </Grid>

    此代码创建一个 UI 用于收集用户名和密码并显示安装 ID。

9.  返回到 MainPage.xaml.cs 文件，添加以下 **using** 指令：

        using Newtonsoft.Json;
        using System.Net.Http;
        using System.Text;
        using Windows.Networking.PushNotifications;
        using Windows.UI.Popups;

10. 在 **MainPage** 类中添加以下定义：

        private string registerUri = "https://<SERVICE_ROOT_URL>/api/register";

    将 *`<SERVICE_ROOT_URL>`* 替换为你在**使用通知中心通知用户**中创建的 Web API 的根 URI。

11. 将以下方法添加到 **MainPage** 类：

        private async void Login_Click(object sender, RoutedEventArgs e)
        {
            // Get the info that we need to request registration.
            var installationId = InstId.Text = LocalStorageManager.GetInstallationId();
            var channel = await PushNotificationChannelManager
                .CreatePushNotificationChannelForApplicationAsync();
            var user = User.Text;
            var pwd = Password.Password;

            // Define a registration to pass in the body of the POST request.
            var registration = new Dictionary<string, string>()
                                   {{"platform", "windows"},
                                   {"instId", installationId.ToString()},
                                   {"channelUri", channel.Uri}};

            // Create a client to send the HTTP registration request.
            var client = new HttpClient();
            var request =
                new HttpRequestMessage(HttpMethod.Post, new Uri(registerUri));

            // Add the Authorization header with the basic login credentials.
            var auth = "Basic " +
                Convert.ToBase64String(Encoding.UTF8.GetBytes(user + ":" + pwd));
            request.Headers.Add("Authorization", auth);
            request.Content = new StringContent(JsonConvert.SerializeObject(registration),
                Encoding.UTF8, "application/json");

            string message;

            try
            {
                // Send the registration request.
                HttpResponseMessage response = await client.SendAsync(request);

                // Get and display the response, either the registration ID
                // or an error message.
                message = await response.Content.ReadAsStringAsync();
                message = string.Format("Registration ID: {0}", message);
            }
            catch (Exception ex)
            {
                message = ex.Message;
            }

            // Display a message dialog.
            var dialog = new MessageDialog(message);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }

    此方法获取一个安装 ID 和用于推送通知的通道并将它与设备类型一起发送到在通知中心创建注册的已经身份验证的 Web API 方法。此 Web API 已在[使用通知中心通知用户][使用通知中心通知用户]中定义。

现在客户端应用程序已更新，请返回到[使用通知中心通知用户][使用通知中心通知用户]并更新移动服务以使用通知中心发送通知。

<!-- Anchors. --> 

<!-- URLs. -->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-aspnet-register-user-push-notifications/ "Windows 应用商店 C#"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-aspnet-register-user-push-notifications/ "iOS"
  [使用通知中心通知用户]: /en-us/manage/services/notification-hubs/notify-users-aspnet

<!-- Images. --> 
  []: ./media/notification-hubs-windows-store-aspnet-register-user-push-notifications/notification-hub-add-nuget-package-json.png
  [1]: ./media/notification-hubs-windows-store-aspnet-register-user-push-notifications/notification-hub-create-aspnet-class.png
