<properties title="Azure Notification Hubs Notify Users" pageTitle="Azure 通知中心 - 通知用户" metaKeywords="Azure push notifications, Azure notification hubs" description="Learn how to send secure push notifications in Azure. Code samples written in C# using the .NET API." documentationCenter="" metaCanonical="" disqusComments="1" umbracoNaviHide="0" authors="glenga" manager="dwrede" services="notification-hubs" />

<tags ms.service="notification-hubs" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows" ms.devlang="dotnet" ms.topic="article" ms.date="11/22/2014" ms.author="glenga" />

#Azure 通知中心 - 通知用户

<div class="dev-center-tutorial-selector sublanding"> 
    	<a href="/zh-cn/documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users/" title="Windows Universal" class="current">Windows Universal</a><a href="/zh-cn/documentation/articles/notification-hubs-aspnet-backend-ios-notify-users/" title="iOS">iOS</a>
		<a href="/zh-cn/documentation/articles/notification-hubs-aspnet-backend-android-notify-users/" title="Android">Android</a>
</div>

利用 Azure 中的推送通知支持，你可以访问易于使用且向外扩展的多平台推送基础结构，这大大简化了为移动平台的使用者应用程序和企业应用程序实现推送通知的过程。本教程说明如何使用 Azure 通知中心将推送通知发送到特定设备上的特定应用程序用户。ASP.NET WebAPI 后端用于对客户端进行身份验证并生成通知，如主题[从应用程序后端注册](http://msdn.microsoft.com/zh-cn/library/dn743807.aspx)中所述。本教程以你在**通知中心入门**教程中创建的通知中心为基础。

此外，只有在学习本教程后，才可以学习**安全推送**教程。完成本**通知用户**教程中的步骤后，你可以继续学习**安全推送**教程，其中说明了如何修改**通知用户**代码以安全地发送推送通知。 

> [AZURE.NOTE]本教程假设你已根据[通知中心入门（Windows 应用商店）](/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/)中所述创建并配置了通知中心。
> 如果你使用移动服务作为后端服务，请参阅本教程的[移动服务版本](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-push-notifications-app-users/)。
>另请注意，在本教程中，你将要创建一个 Windows Phone 应用商店 8.1 应用程序。可以使用相同的代码来创建 Windows 应用商店和 Windows Universal 应用程序。所有这些应用程序都必须使用 Windows（而不是 Windows Phone）凭据。

## 创建并配置通知中心

在开始本教程之前，你必须保留一个应用程序名称，然后创建 Azure 通知中心并将其连接到该应用程序。请遵循[通知中心入门（Windows 应用商店）](/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/)中的步骤；具体而言，请遵循[在 Windows 应用商店中注册你的应用程序](/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/#register)和[配置通知中心](/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/#configure-hub)部分中的步骤。请务必确保已在门户中你的通知中心的**"配置"**选项卡上输入了**"程序包 SID"**和**"客户端机密"**值。[配置通知中心](/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/#configure-hub)部分中介绍了此配置过程。这个步骤非常重要：如果门户上的凭据与针对所选应用程序名称指定的凭据不匹配，推送通知将不会成功。

[WACOM.INCLUDE [notification-hubs-aspnet-backend-notifyusers](../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## 创建 Windows Phone 项目

下一步是创建 Windows Phone 应用程序。若要将此项目添加到当前解决方案，请执行以下操作：

1. 在解决方案资源管理器中，右键单击解决方案的顶层节点（在本例中为 **Solution NotifyUsers**），单击**"添加"**，然后单击**"新建项目"**。

2. 展开**"应用商店应用程序"**，单击**"Windows Phone 应用程序"**，然后单击**"空白应用程序(Windows Phone)"**。

	![][9]

3. 在**"名称"**框中，键入 **NotifyUserWindowsPhone**，然后单击**"确定"**以生成项目。

 
4. 将此应用程序与 Windows Phone 应用商店相关联：在解决方案资源管理器，右键单击**"NotifyUserWindowsPhone (Windows Phone 8.1)"**，单击**"应用商店"**，然后单击**"将应用程序与应用商店关联..."**。

	![][10]
 
5. 遵循向导中的步骤登录，并将应用程序与应用商店相关联。

	![][11]
	
	> [AZURE.NOTE]请务必记下你在执行此过程期间选择的应用程序名称。你必须在门户中使用从 [Windows 开发人员中心](http://go.microsoft.com/fwlink/p/?linkid=266582&clcid=0x409)获取的凭据针对这个特定的保留应用程序名称配置通知中心。[配置通知中心](/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started/#configure-hub)中介绍了此配置过程。这个步骤非常重要：如果门户上的凭据与针对所选应用程序名称指定的凭据不匹配，推送通知将不会成功。

6. 在解决方案资源管理器中，右键单击**"NotifyUserWindowsPhone (Windows Phone 8.1)"**项目，然后单击**"管理 NuGet 包"**。

7. 在左侧单击**"联机"**。

8. 在**"搜索"**框中键入**"Http 客户端"**。

9. 在结果列表中，单击**"Microsoft HTTP 客户端库"**，然后单击**"安装"**。完成安装。

10. 返回到 NuGet**"搜索"**框，键入 **Json.net**。安装 **Json.NET** 包，然后关闭"NuGet 包管理器"窗口。

11. 在解决方案资源管理器中的 **NotifyUserWindowsPhone (Windows Phone 8.1)** 项目内，双击**"MainPage.xaml"**在 Visual Studio 编辑器中打开它。

12. 在 **MainPage.xaml** XML 代码中，将 `<Grid>` 节替换为以下代码：

		<Grid>
	        <Grid.RowDefinitions>
	            <RowDefinition Height="Auto"/>
	            <RowDefinition Height="*"/>
	        </Grid.RowDefinitions>

	        <TextBlock Grid.Row="0" Text="Secure Push" HorizontalAlignment="Center" FontSize="48"/>

        	<StackPanel Grid.Row="1" VerticalAlignment="Center">
        	    <Grid>
        	        <Grid.RowDefinitions>
        	            <RowDefinition Height="Auto"/>
        	            <RowDefinition Height="Auto"/>
        	            <RowDefinition Height="Auto"/>
        	            <RowDefinition Height="Auto"/>
        	            <RowDefinition Height="Auto"/>
        	            <RowDefinition Height="*"/>
        	        </Grid.RowDefinitions>
            	    <TextBlock Grid.Row="0" Text="Username" FontSize="24" Margin="20,0,20,0"/>
            	    <TextBox Name="UsernameTextBox" Grid.Row="1" Margin="20,0,20,0"/>
            	    <TextBlock Grid.Row="2" Text="Password" FontSize="24" Margin="20,0,20,0" />
            	    <PasswordBox Name="PasswordTextBox" Grid.Row="3" Margin="20,0,20,0"/>
	
            	    <Button Grid.Row="4" HorizontalAlignment="Center" VerticalAlignment="Center" Content="1. Login and register" Click="LoginAndRegisterClick" />

            	    <Button Grid.Row="5" HorizontalAlignment="Center" VerticalAlignment="Center" Content="2. Send push" Click="PushClick" />
            	</Grid>
        	</StackPanel>
    	</Grid>


13. 在解决方案资源管理器中，右键单击**"NotifyUserWindowsPhone (Windows Phone 8.1)"**项目，单击**"添加"**，然后单击**"类"**。将类命名为 **RegisterClient.cs**，然后单击**"确定"**以生成该类。此组件将实现所需的 REST 调用，以便能够联系应用程序后端来注册推送通知。它还会在本地存储通知中心创建的 *registrationIds*（[从应用程序后端注册](http://msdn.microsoft.com/zh-cn/library/dn743807.aspx)中提供了详细信息）。请注意，该组件使用当你单击**"登录并注册"**按钮时存储在本地存储中的授权令牌。

14. 在 `RegisterClient` 类定义中添加以下代码。请务必将 `{backend endpoint}` 替换为在上一部分中获取的后端终结点：

		private string POST_URL = "{backend endpoint}/api/register";

        private class DeviceRegistration
        {
            public string Platform { get; set; }
            public string Handle { get; set; }
            public string[] Tags { get; set; }
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
                        throw new Exception();
                    }
                }
            }
            return (string)settings["__NHRegistrationId"];

        }


15. 在 RegisterClient.cs 文件的顶部添加以下 `using` 语句：

		using Windows.Storage;
		using System.Net;
		using System.Net.Http;
		using System.Net.Http.Headers;
		using Newtonsoft.Json;
		
16. 在 MainPage.xaml.cs 中添加按钮的代码。对**"登录并注册"**的回调会将基本身份验证令牌存储在本地存储中（请注意，这表示你的身份验证方案使用的任何令牌），然后使用 `RegisterClient` 调用后端。对 **AppBackend** 的回调会调用后端，以触发向此用户的所有设备发送安全通知。 

在 MainPage.xaml.cs 文件中 `OnNavigatedTo()` 方法的后面添加以下代码。请务必将 `{backend endpoint}` 替换为在上一部分中获取的后端终结点：

		private async void PushClick(object sender, RoutedEventArgs e)
        {
            var POST_URL = "{backend endpoint}/api/notifications";

            using (var httpClient = new HttpClient())
            {
                var settings = ApplicationData.Current.LocalSettings.Values;
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string) settings["AuthenticationToken"]);

                await httpClient.PostAsync(POST_URL, new StringContent(""));
            }
        }

        private async void LoginAndRegisterClick(object sender, RoutedEventArgs e)
        {
            SetAuthenticationTokenInLocalStorage();
            
            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();
            await new RegisterClient().RegisterAsync(channel.Uri, new string[] { "myTag" });

            var dialog = new MessageDialog("Registered as: " + UsernameTextBox.Text);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }

        private void SetAuthenticationTokenInLocalStorage()
        {
            string username = UsernameTextBox.Text;
            string password = PasswordTextBox.Password;
            
            var token = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(username + ":" + password));
            ApplicationData.Current.LocalSettings.Values["AuthenticationToken"] = token;
        }


17. 在 MainPage.xaml.cs 文件的顶部添加以下 `using` 语句：

		using System.Net.Http;
		using Windows.Storage;
		using System.Net.Http.Headers;
		using Windows.Networking.PushNotifications;
		using Windows.UI.Popups;

## 运行应用程序

若要运行应用程序，请执行以下操作：

1. 在 Visual Studio 中，运行 **NotifyUserWindowsPhone (Windows Phone 8.1)** Windows Phone 应用程序。Windows Phone 模拟器将自动运行并加载应用程序。

2. 在 **NotifyUserWindowsPhone** 应用程序 UI 中，输入用户名和密码。这些信息可以是任意字符串，但必须是相同的值。

3. 在 **NotifyUserWindowsPhone** 应用程序 UI 中，单击**"登录并注册"**。然后单击**"发送推送通知"**。


[9]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push9.png
[10]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push10.png
[11]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push11.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push13.png
