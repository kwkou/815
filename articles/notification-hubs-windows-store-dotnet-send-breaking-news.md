<properties 
	pageTitle="使用通知中心发送突发新闻 (Windows Universal)" 
	description="结合注册中的标记使用 Azure 通知中心将突发新闻发送到通用 Windows 应用。" 
	services="notification-hubs" 
	documentationCenter="windows" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor=""/>


<tags
	ms.service="notification-hubs" 
	ms.date="09/24/2015"
	wacn.date="10/03/2015"/>

# 使用通知中心发送突发新闻


[AZURE.INCLUDE [notification-hubs-selector-breaking-news](../includes/notification-hubs-selector-breaking-news.md)]


##概述

本主题演示如何使用 Azure 通知中心将突发新闻通知广播到 Windows 应用商店应用程序。完成时，你可以注册感兴趣的突发新闻类别并仅接收这些类别的推送通知。此方案对于很多应用程序来说是常见模式，在其中必须将通知发送到以前声明过对它们感兴趣的一组用户，这样的应用程序有 RSS 阅读器、针对音乐迷的应用程序等。

在创建通知中心的注册时，通过加入一个或多个_标记_来启用广播方案。将通知发送到标签时，已注册该标签的所有设备将接收通知。因为标签是简单的字符串，它们不必提前设置。有关标记的详细信息，请参阅[通知中心指南]。

##先决条件

本主题以你在[通知中心入门][get-started]中创建的应用为基础。在开始本教程之前，必须先阅读[通知中心入门][get-started]。

## <a name="adding-categories"></a>向应用程序中添加类别选择

第一步是向现有主页添加 UI 元素，这些元素允许用户选择要注册的类别。用户选择的类别存储在设备上。应用程序启动时，使用所选类别作为标签在你的通知中心创建设备注册。

1.  打开 MainPage.xaml 项目文件，然后在 **Grid** 元素中复制以下代码：

        <Grid Margin="120, 58, 120, 80" >
            <Grid.RowDefinitions>
                <RowDefinition />
                <RowDefinition />
                <RowDefinition />
                <RowDefinition />
                <RowDefinition />
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition />
                <ColumnDefinition />
            </Grid.ColumnDefinitions>
            <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top" HorizontalAlignment="Center"/>
            <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="1" Grid.Column="0" HorizontalAlignment="Center"/>
            <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="2" Grid.Column="0" HorizontalAlignment="Center"/>
            <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="3" Grid.Column="0" HorizontalAlignment="Center"/>
            <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="1" Grid.Column="1" HorizontalAlignment="Center"/>
            <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="2" Grid.Column="1" HorizontalAlignment="Center"/>
            <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="3" Grid.Column="1" HorizontalAlignment="Center"/>
            <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click"/>
        </Grid>


2. 右键单击“共享”项目，添加名为 **Notifications** 的新类，向类定义添加 **public** 修饰符，然后将以下 **using** 语句添加到新的代码文件：

		using Windows.Networking.PushNotifications;
		using Microsoft.WindowsAzure.Messaging;
		using Windows.Storage;
		using System.Threading.Tasks;

3.  将以下代码添加到新的 **Notifications** 类：

        private NotificationHub hub;

        public Notifications(string hubName, string listenConnectionString)
        {
            hub = new NotificationHub(hubName, listenConnectionString);
        }

        public async Task StoreCategoriesAndSubscribe(IEnumerable<string> categories)
        {
            ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
            await SubscribeToCategories(categories);
        }

		public IEnumerable<string> RetrieveCategories()
        {
            var categories = (string) ApplicationData.Current.LocalSettings.Values["categories"];
            return categories != null ? categories.Split(','): new string[0];
        }

        public async Task SubscribeToCategories(IEnumerable<string> categories = null)
        {
            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

            if (categories == null)
            {
                IEnumerable<string> categories = RetrieveCategories();
            }

            await hub.RegisterNativeAsync(channel.Uri, categories);
        }

    此类使用本地存储区存储此设备必须接收的新闻类别。它还包含注册这些类别所用的方法。


4. 在 App.xaml.cs 项目文件中，将以下属性添加到 **App** 类：

		public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");

    此属性用于创建和访问 **Notifications** 实例。

	在上面的代码中，将 `<hub name>` 和 `<connection string with listen access>` 占位符替换为你的通知中心的名称和你之前获取的 *DefaultListenSharedAccessSignature* 的连接字符串。

	> [AZURE.NOTE]由于使用客户端应用程序分发的凭据通常是不安全的，你只应使用客户端应用程序分发具有侦听访问权限的密钥。侦听访问权限允许应用程序注册通知，但是无法修改现有注册，也无法发送通知。在受保护的后端服务中使用完全访问权限密钥，以便发送通知和更改现有注册。

5. 在 MainPage.xaml.cs 中，添加以下行：

        using Windows.UI.Popups;

6. 在 MainPage.xaml.cs 项目文件中，添加以下方法：

        private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
        {
            var categories = new HashSet<string>();
            if (WorldToggle.IsOn) categories.Add("World");
            if (PoliticsToggle.IsOn) categories.Add("Politics");
            if (BusinessToggle.IsOn) categories.Add("Business");
            if (TechnologyToggle.IsOn) categories.Add("Technology");
            if (ScienceToggle.IsOn) categories.Add("Science");
            if (SportsToggle.IsOn) categories.Add("Sports");

            await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);

            var dialog = new MessageDialog("Subscribed to: " + string.Join(",", categories));
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
	
	此方法将创建一个类别列表，使用 **Notifications** 类将该列表存储在本地存储中，并将相应的标记注册到你的通知中心。更改类别时，使用新类别重新创建注册。

你的应用程序现在可以将一组类别存储在设备的本地存储区中了，每当用户更改所选类别时，会将这些类别注册到通知中心。

## <a name="register"></a>注册通知

这些步骤用于在启动时将在本地存储区中存储的类别注册到通知中心。

> [AZURE.NOTE]由于 Windows 通知服务 (WNS) 分配的通道 URI 随时可能更改，因此你应该经常注册通知以避免通知失败。此示例在每次应用程序启动时注册通知。对于经常运行（一天一次以上）的应用程序，如果每次注册间隔时间不到一天，你可以跳过注册来节省带宽。

1. 打开 App.xaml.cs 文件并将 **async** 修饰符添加到 **OnLaunched** 方法。

2. 在 **OnLaunched** 方法中，找到并使用以下代码行替换对 **InitNotificationsAsync** 方法的现有调用：

		await notifications.SubscribeToCategories();

	这确保每次应用程序启动时，它从本地存储区检索类别并请求注册这些类别。**InitNotificationsAsync** 方法已作为 [通知中心入门] 教程的一部分创建，但是在本主题中不需要此方法。

3. 在 MainPage.xaml.cs 项目文件的 *OnNavigatedTo* 方法中添加以下代码：

		var categories = ((App)Application.Current).notifications.RetrieveCategories();

        if (categories.Contains("World")) WorldToggle.IsOn = true;
        if (categories.Contains("Politics")) PoliticsToggle.IsOn = true;
        if (categories.Contains("Business")) BusinessToggle.IsOn = true;
        if (categories.Contains("Technology")) TechnologyToggle.IsOn = true;
        if (categories.Contains("Science")) ScienceToggle.IsOn = true;
        if (categories.Contains("Sports")) SportsToggle.IsOn = true;

    这基于以前保存的类别状态更新主页。

应用程序现在已完成，可以在设备的本地存储区中存储一组类别了，每当用户更改所选类别时将使用这些类别注册到通知中心。接下来，我们将定义一个后端，它可将类别通知发送到此应用程序。

##从后端发送通知

[AZURE.INCLUDE [notification-hubs-back-end](../includes/notification-hubs-back-end.md)]

##运行应用并生成通知

1. 在 Visual Studio 中，按 F5 编译并启动应用程序。

	![][1]

	请注意，应用程序 UI 提供了一组开关，你可以使用它们选择要订阅的类别。

2. 启用一个或多个类别开关，然后单击“订阅”。

	应用程序将所选类别转换为标签并针对所选标签从通知中心请求注册新设备。返回注册的类别并显示在对话框中。

	![][19]

4. 使用以下方式之一从后端发送新通知：

	+ **控制台应用：**启动控制台应用。

	+ **Java/PHP：**运行你的应用/脚本。

	所选类别的通知作为 toast 通知显示。

	![][14]

##后续步骤

在本教程中，我们了解了如何按类别广播突发消息。请考虑学习侧重说明其他高级通知中心方案的以下教程之一：

+ [使用通知中心广播本地化的突发新闻]

	了解如何扩展突发新闻应用程序以允许发送本地化的通知。

+ [使用通知中心通知用户]

	了解如何将通知推送到经过身份验证的特定用户。这是仅将通知发送到特定用户的好的解决方案。


<!-- Anchors. -->
[Add category selection to the app]: #adding-categories
[Register for notifications]: #register
[Send notifications from your back-end]: #send
[Run the app and generate notifications]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->
[1]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-breakingnews-win1.png

[14]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-toast-2.png


[19]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-reg-2.png

<!-- URLs.-->
[get-started]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started
[使用通知中心广播本地化的突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news
[使用通知中心通知用户]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users
[通知中心指南]: http://msdn.microsoft.com/library/jj927170.aspx


<!---HONumber=71-->