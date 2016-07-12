<properties
	pageTitle="使用通知中心发送突发新闻 (Windows Phone)"
	description="通过 Azure 通知中心使用注册中的标记将突发新闻发送到 Windows Phone 应用。"
	services="notification-hubs"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="03/28/2016" 
	wacn.date="05/31/2016" />

# 使用通知中心发送突发新闻

[AZURE.INCLUDE [notification-hubs-selector-breaking-news](../includes/notification-hubs-selector-breaking-news.md)]


##概述

本主题演示如何使用 Azure 通知中心将突发新闻通知广播到 Windows Phone 8.0/8.1 Silverlight 应用。如果你要以 Windows 应用商店或 Phone 8.1 应用为目标，请参阅 [Windows Universal](/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/) 版本。完成时，你可以注册感兴趣的突发新闻类别并仅接收这些类别的推送通知。此方案对于很多应用程序来说是常见模式，在其中必须将通知发送到以前声明过对它们感兴趣的一组用户，这样的应用程序有 RSS 阅读器、针对音乐迷的应用程序等。

在创建通知中心的注册时，通过加入一个或多个标记来启用广播方案。将通知发送到标签时，已注册该标签的所有设备将接收通知。因为标签是简单的字符串，它们不必提前设置。有关标记的详细信息，请参阅[通知中心路由和标记表达式](/documentation/articles/notification-hubs-routing-tag-expressions/)。

##先决条件

本主题以你在[通知中心入门]中创建的应用程序为基础。在开始本教程之前，必须先阅读[通知中心入门]。

##<a name="adding-categories"></a>向应用添加类别选择

第一步是向现有主页添加 UI 元素，这些元素允许用户选择要注册的类别。用户选择的类别存储在设备上。应用程序启动时，使用所选类别作为标签在你的通知中心创建设备注册。

1. 打开 MainPage.xaml 项目文件，然后使用以下代码替换名为 `TitlePanel` 和 `ContentPanel` 的 **Grid** 元素：
			
        <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28">
            <TextBlock Text="Breaking News" Style="{StaticResource PhoneTextNormalStyle}" Margin="12,0"/>
            <TextBlock Text="Categories" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/>
        </StackPanel>

        <Grid Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
            <Grid.RowDefinitions>
                <RowDefinition Height="auto"/>
                <RowDefinition Height="auto" />
                <RowDefinition Height="auto" />
                <RowDefinition Height="auto" />
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition />
                <ColumnDefinition />
            </Grid.ColumnDefinitions>
            <CheckBox Name="WorldCheckBox" Grid.Row="0" Grid.Column="0">World</CheckBox>
            <CheckBox Name="PoliticsCheckBox" Grid.Row="1" Grid.Column="0">Politics</CheckBox>
            <CheckBox Name="BusinessCheckBox" Grid.Row="2" Grid.Column="0">Business</CheckBox>
            <CheckBox Name="TechnologyCheckBox" Grid.Row="0" Grid.Column="1">Technology</CheckBox>
            <CheckBox Name="ScienceCheckBox" Grid.Row="1" Grid.Column="1">Science</CheckBox>
            <CheckBox Name="SportsCheckBox" Grid.Row="2" Grid.Column="1">Sports</CheckBox>
            <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="3" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
        </Grid>

2. 在该项目中，创建名为 **Notifications** 的新类，向类定义添加 **public** 修饰符，然后将以下 **using** 语句添加到新的代码文件：

		using Microsoft.Phone.Notification;
		using Microsoft.WindowsAzure.Messaging;
		using System.IO.IsolatedStorage;
		using System.Windows;

3. 将以下代码添加到新的 **Notifications** 类：

        private NotificationHub hub;

        // Registration task to complete registration in the ChannelUriUpdated event handler
        private TaskCompletionSource<Registration> registrationTask;

        public Notifications(string hubName, string listenConnectionString)
        {
            hub = new NotificationHub(hubName, listenConnectionString);
        }

        public IEnumerable<string> RetrieveCategories()
        {
            var categories = (string)IsolatedStorageSettings.ApplicationSettings["categories"];
            return categories != null ? categories.Split(',') : new string[0];
        }

        public async Task<Registration> StoreCategoriesAndSubscribe(IEnumerable<string> categories)
        {
            var categoriesAsString = string.Join(",", categories);
            var settings = IsolatedStorageSettings.ApplicationSettings;
            if (!settings.Contains("categories"))
            {
                settings.Add("categories", categoriesAsString);
            }
            else
            {
                settings["categories"] = categoriesAsString;
            }
            settings.Save();

            return await SubscribeToCategories();
        }

        public async Task<Registration> SubscribeToCategories()
        {
            registrationTask = new TaskCompletionSource<Registration>();

            var channel = HttpNotificationChannel.Find("MyPushChannel");

            if (channel == null)
            {
                channel = new HttpNotificationChannel("MyPushChannel");
                channel.Open();
                channel.BindToShellToast();
                channel.ChannelUriUpdated += channel_ChannelUriUpdated;

				// This is optional, used to receive notifications while the app is running.
                channel.ShellToastNotificationReceived += channel_ShellToastNotificationReceived;
            }

            // If channel.ChannelUri is not null, we will complete the registrationTask here.  
			// If it is null, the registrationTask will be completed in the ChannelUriUpdated event handler.
            if (channel.ChannelUri != null)
            {
                await RegisterTemplate(channel.ChannelUri);
            }
            
            return await registrationTask.Task;
        }

        async void channel_ChannelUriUpdated(object sender, NotificationChannelUriEventArgs e)
        {
            await RegisterTemplate(e.ChannelUri);
        }

        async Task<Registration> RegisterTemplate(Uri channelUri)
        {
            // Using a template registration to support notifications across platforms.
            // Any template notifications that contain messageParam and a corresponding tag expression
            // will be delivered for this registration.

            const string templateBodyMPNS = "<wp:Notification xmlns:wp="WPNotification">" +
                                                "<wp:Toast>" +
                                                    "<wp:Text1>$(messageParam)</wp:Text1>" +
                                                "</wp:Toast>" +
                                            "</wp:Notification>";

			// The stored categories tags are passed with the template registration.

            registrationTask.SetResult(await hub.RegisterTemplateAsync(channelUri.ToString(), 
				templateBodyMPNS, "simpleMPNSTemplateExample", this.RetrieveCategories()));

            return await registrationTask.Task;
        }

		// This is optional. It is used to receive notifications while the app is running.
        void channel_ShellToastNotificationReceived(object sender, NotificationEventArgs e)
        {
            StringBuilder message = new StringBuilder();
            string relativeUri = string.Empty;

            message.AppendFormat("Received Toast {0}:\n", DateTime.Now.ToShortTimeString());

            // Parse out the information that was part of the message.
            foreach (string key in e.Collection.Keys)
            {
                message.AppendFormat("{0}: {1}\n", key, e.Collection[key]);

                if (string.Compare(
                    key,
                    "wp:Param",
                    System.Globalization.CultureInfo.InvariantCulture,
                    System.Globalization.CompareOptions.IgnoreCase) == 0)
                {
                    relativeUri = e.Collection[key];
                }
            }

            // Display a dialog of all the fields in the toast.
            System.Windows.Deployment.Current.Dispatcher.BeginInvoke(() => 
            { 
                MessageBox.Show(message.ToString()); 
            });
        }


    此类使用隔离存储区存储此设备要接收的新闻类别。它还包含用于通过[模板](/documentation/articles/notification-hubs-templates/)通知注册来注册这些类别的方法。


4. 在 App.xaml.cs 项目文件中，将以下属性添加到 **App** 类：将 `<hub name>` 和 `<connection string with listen access>` 占位符替换为通知中心名称和前面获取的 *DefaultListenSharedAccessSignature* 的连接字符串。

		public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");

	> [AZURE.NOTE]由于使用客户端应用程序分发的凭据通常是不安全的，你只应使用客户端应用程序分发具有侦听访问权限的密钥。侦听访问权限允许应用程序注册通知，但是无法修改现有注册，也无法发送通知。在受保护的后端服务中使用完全访问权限密钥，以便发送通知和更改现有注册。

5. 在 MainPage.xaml.cs 中，添加以下行：

		using Windows.UI.Popups;

6. 在 MainPage.xaml.cs 项目文件中，添加以下方法：

		private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
		{
		  var categories = new HashSet<string>();
		  if (WorldCheckBox.IsChecked == true) categories.Add("World");
		  if (PoliticsCheckBox.IsChecked == true) categories.Add("Politics");
		  if (BusinessCheckBox.IsChecked == true) categories.Add("Business");
		  if (TechnologyCheckBox.IsChecked == true) categories.Add("Technology");
		  if (ScienceCheckBox.IsChecked == true) categories.Add("Science");
		  if (SportsCheckBox.IsChecked == true) categories.Add("Sports");
	
		  var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);
	
		  MessageBox.Show("Subscribed to: " + string.Join(",", categories) + " on registration id : " +
			 result.RegistrationId);
		}

	此方法创建一个类别列表并使用 **Notifications** 类将该列表存储在本地存储区中，将相应的标签注册到你的通知中心。更改类别时，使用新类别重新创建注册。

你的应用程序现在可以将一组类别存储在设备的本地存储区中了，每当用户更改所选类别时，会将这些类别注册到通知中心。

##<a name="register"></a>注册通知

这些步骤用于在启动时将在本地存储区中存储的类别注册到通知中心。

> [AZURE.NOTE]由于 Microsoft 推送通知服务 (MPNS) 分配的通道 URI 随时可能更改，因此你应该经常注册通知以避免通知失败。此示例在每次应用程序启动时注册通知。对于经常运行（一天一次以上）的应用程序，如果每次注册间隔时间不到一天，你可以跳过注册来节省带宽。


1. 打开 App.xaml.cs 文件，将 **async** 修饰符添加到 **Application\_Launching** 方法，并将你在[通知中心入门]中添加的通知中心注册代码替换为以下代码：

        private async void Application_Launching(object sender, LaunchingEventArgs e)
        {
            var result = await notifications.SubscribeToCategories();

            if (result != null)
                System.Windows.Deployment.Current.Dispatcher.BeginInvoke(() =>
                {
                    MessageBox.Show("Registration Id :" + result.RegistrationId, "Registered", MessageBoxButton.OK);
                });
        }

	这确保每次应用程序启动时，它从本地存储区检索类别并请求注册这些类别。

2. 在 MainPage.xaml.cs 项目文件中，添加实现 **OnNavigatedTo** 方法的以下代码：

		protected override void OnNavigatedTo(NavigationEventArgs e)
		{
		    var categories = ((App)Application.Current).notifications.RetrieveCategories();
		
		    if (categories.Contains("World")) WorldCheckBox.IsChecked = true;
		    if (categories.Contains("Politics")) PoliticsCheckBox.IsChecked = true;
		    if (categories.Contains("Business")) BusinessCheckBox.IsChecked = true;
		    if (categories.Contains("Technology")) TechnologyCheckBox.IsChecked = true;
		    if (categories.Contains("Science")) ScienceCheckBox.IsChecked = true;
		    if (categories.Contains("Sports")) SportsCheckBox.IsChecked = true;
		}

	这基于以前保存的类别状态更新主页。

应用程序现在已完成，可以在设备的本地存储区中存储一组类别了，每当用户更改所选类别时将使用这些类别注册到通知中心。接下来，我们将定义一个后端，它可将类别通知发送到此应用程序。

##发送带标记的通知

[AZURE.INCLUDE [notification-hubs-send-categories-template](../includes/notification-hubs-send-categories-template.md)]

##<a name="test-app"></a>运行应用并生成通知

1. 在 Visual Studio 中，按 F5 编译并启动应用程序。

	![][1]

	请注意，应用程序 UI 提供了一组开关，你可以使用它们选择要订阅的类别。

2. 启用一个或多个类别开关，然后单击“订阅”。

	应用程序将所选类别转换为标签并针对所选标签从通知中心请求注册新设备。返回注册的类别并显示在对话框中。

	![][2]

3. 收到类别订阅已完成的确认消息后，运行控制台应用以发送每个类别的通知。确认你只会收到订阅的类别的通知。

	![][3]

你已完成本主题。


<!-- Anchors. -->
[向应用程序中添加类别选择]: #adding-categories
[注册通知]: #register
[从后端发送通知]: #send
[运行应用并生成通知]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->
[1]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-breakingnews.png
[2]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-registration.png
[3]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-toast.png



<!-- URLs.-->
[通知中心入门]: /documentation/articles/notification-hubs-windows-phone-get-started/
[Notify users with Notification Hubs]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/
[Mobile Service]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/
[通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx

[Azure Management Portal]: https://manage.windowsazure.cn/





<!---HONumber=Mooncake_0523_2016-->