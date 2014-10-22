<properties linkid="develop-notificationhubs-tutorials-send-breaking-news-windowsdotnet" urlDisplayName="Breaking News" pageTitle="Notification Hubs Breaking News Tutorial" metaKeywords="" description="Learn how to use Azure Service Bus Notification Hubs to send breaking news notifications." metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="" title="Use Notification Hubs to send breaking news" authors="ricksal" solutions="" manager="" editor="" />

# 使用通知中心发送突发新闻

<div class="dev-center-tutorial-selector sublanding"> 
<a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/" title="Windows 应用商店 C#" class="current">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-windows-phone-send-breaking-news/" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-send-breaking-news/" title="iOS" class="current">iOS</a>
</div>

本主题演示如何使用 Azure 通知中心将突发新闻通知广播到 Windows 应用商店应用程序。完成时，你可以注册感兴趣的突发新闻类别并仅接收这些类别的推送通知。此方案对于很多应用程序来说是常见模式，在其中必须将通知发送到以前声明过对它们感兴趣的一组用户，这样的应用程序有 RSS 阅读器、针对音乐迷的应用程序等。

在通知中心创建注册时通过包括一个或多个*标签*来启用广播方案。将通知发送到标签时，已注册该标签的所有设备将接收通知。因为标签是简单的字符串，它们不必提前设置。有关标签的更多信息，请参见[通知中心指南][通知中心指南]。

本教程将指导你完成启用此方案的以下基本步骤：

1.  [向应用程序中添加类别选择][向应用程序中添加类别选择]
2.  [注册通知][注册通知]
3.  [从后端发送通知][从后端发送通知]
4.  [运行应用程序并生成通知][运行应用程序并生成通知]

本主题以你在[通知中心入门][通知中心入门]中创建的应用程序为基础。在开始本教程之前，必须先阅读[通知中心入门][通知中心入门]。

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
            <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top"/>
            <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="1" Grid.Column="0"/>
            <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="2" Grid.Column="0"/>
            <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="3" Grid.Column="0"/>
            <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="1" Grid.Column="1"/>
            <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="2" Grid.Column="1"/>
            <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="3" Grid.Column="1"/>
            <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
        </Grid>

2.  在该项目中，创建名为 **Notifications** 的新类，向类定义添加 **public** 修饰符，然后将以下 **using** 语句添加到新的代码文件：

        using Windows.Networking.PushNotifications;
        using Microsoft.WindowsAzure.Messaging;
        using Windows.Storage;

3.  将以下代码添加到新的 **Notifications** 类：

        private NotificationHub hub;

        public Notifications()
        {
            hub = new NotificationHub("<hub name>", "<connection string with listen access>");
        }

        public async Task StoreCategoriesAndSubscribe(IEnumerable<string> categories)
        {
            ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
            await SubscribeToCategories(categories);
        }

        public async Task SubscribeToCategories(IEnumerable<string> categories)
        {
            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();
            await hub.RegisterNativeAsync(channel.Uri, categories);
        }

    此类使用本地存储区存储此设备必须接收的新闻类别。它还包含注册这些类别所用的方法。

4.  在上述代码中，将`<hub name>` 和 `<connection string with listen access>` 占位符替换为通知中心名称和前面获取的 *DefaultListenSharedAccessSignature* 的连接字符串。

    <div class="dev-callout"><strong>说明</strong> 
<p>由于使用客户端应用程序分发的凭据通常是不安全的，你只应使用客户端应用程序分发具有侦听访问权限的密钥。侦听访问权限允许应用程序注册通知，但是无法修改现有注册，也无法发送通知。在受保护的后端服务中使用完全访问权限密钥，以便发送通知和更改现有注册。</p>
</div>

5.  在 App.xaml.cs 项目文件中，将以下属性添加到 **App** 类：

        public Notifications notifications = new Notifications();

    此属性用于创建和访问 **Notifications** 实例。

6.  在 MainPage.xaml.cs 中，添加以下行：

        using Windows.UI.Popups;

7.  在 MainPage.xaml.cs 项目文件中，添加以下方法：

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

    此方法创建一个类别列表并使用 **Notifications** 类将该列表存储在本地存储区中，将相应的标签注册到你的通知中心。更改类别时，使用新类别重新创建注册。

你的应用程序现在可以将一组类别存储在设备的本地存储区中了，每当用户更改所选类别时，会将这些类别注册到通知中心。

## <a name="register"></a>注册通知

这些步骤用于在启动时将在本地存储区中存储的类别注册到通知中心。

<div class="dev-callout"><strong>说明</strong> 
<p>由于 Windows 通知服务 (WNS) 分配的通道 URI 随时可能更改，因此你应该经常注册通知以避免通知失败。此示例在每次应用程序启动时注册通知。对于经常运行（一天一次以上）的应用程序，如果每次注册间隔时间不到一天，你可以跳过注册来节省带宽。</p>
</div>

1.  将以下代码添加到 **Notifications** 类：

        public IEnumerable<string> RetrieveCategories()
        {
            var categories = (string) ApplicationData.Current.LocalSettings.Values["categories"];
            return categories != null ? categories.Split(','): new string[0];
        }

    这返回在类中定义的类别。

2.  打开 App.xaml.cs 文件并将 **async** 修饰符添加到 **OnLaunched** 方法。

3.  在 **OnLaunched** 方法中，找到并使用以下代码行替换对 **InitNotificationsAsync** 方法的现有调用：

        await notifications.SubscribeToCategories(notifications.RetrieveCategories());

    这确保每次应用程序启动时，它从本地存储区检索类别并请求注册这些类别。**InitNotificationsAsync** 方法已在 [通知中心入门] 教程中创建，但是在本主题中不需要它。

4.  在 MainPage.xaml.cs 项目文件的 *OnNavigatedTo* 方法中添加以下代码：

        var categories = ((App)Application.Current).notifications.RetrieveCategories();

        if (categories.Contains("World")) WorldToggle.IsOn = true;
        if (categories.Contains("Politics")) PoliticsToggle.IsOn = true;
        if (categories.Contains("Business")) BusinessToggle.IsOn = true;
        if (categories.Contains("Technology")) TechnologyToggle.IsOn = true;
        if (categories.Contains("Science")) ScienceToggle.IsOn = true;
        if (categories.Contains("Sports")) SportsToggle.IsOn = true;

    这基于以前保存的类别状态更新主页。

应用程序现在已完成，可以在设备的本地存储区中存储一组类别了，每当用户更改所选类别时将使用这些类别注册到通知中心。接下来，我们将定义一个后端，它可将类别通知发送到此应用程序。

## <a name="send"></a><span class="short-header">发送通知</span>从后端发送通知

[WACOM.INCLUDE [notification-hubs-back-end][notification-hubs-back-end]]

## <a name="test-app"></a>运行应用程序并生成通知

1.  在 Visual Studio 中，按 F5 编译并启动应用程序。

    ![][]

    请注意，应用程序 UI 提供了一组开关，你可以使用它们选择要订阅的类别。

2.  启用一个或多个类别开关，然后单击“订阅”。

    应用程序将所选类别转换为标签并针对所选标签从通知中心请求注册新设备。返回注册的类别并显示在对话框中。

    ![][1]

3.  使用以下方式之一从后端发送新通知：

    -   **控制台应用程序：**启动控制台应用程序。

    -   **移动服务：**单击“计划程序”选项卡，单击该作业，然后单击“运行一次”。

    所选类别的通知作为 toast 通知显示。

    ![][2]

## <a name="next-steps"> </a>后续步骤

在本教程中，我们了解了如何按类别广播突发消息。请考虑学习侧重说明其他高级通知中心方案的以下教程之一：

-   [使用通知中心广播本地化的突发新闻][使用通知中心广播本地化的突发新闻]

    了解如何扩展突发新闻应用程序以允许发送本地化的通知。

-   [使用通知中心通知用户][使用通知中心通知用户]

    了解如何将通知推送到经过身份验证的特定用户。这是仅将通知发送到特定用户的好的解决方案。

<!-- Anchors. -->  

<!-- URLs.-->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/ "Windows 应用商店 C#"
  [Windows Phone]: /zh-cn/documentation/articles/notification-hubs-windows-phone-send-breaking-news/ "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-send-breaking-news/ "iOS"
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
  [向应用程序中添加类别选择]: #adding-categories
  [注册通知]: #register
  [从后端发送通知]: #send
  [运行应用程序并生成通知]: #test-app
  [通知中心入门]: /en-us/manage/services/notification-hubs/getting-started-windows-dotnet/
  [notification-hubs-back-end]: ../includes/notification-hubs-back-end.md

  [使用通知中心广播本地化的突发新闻]: /en-us/manage/services/notification-hubs/breaking-news-localized-dotnet/
  [使用通知中心通知用户]: /en-us/manage/services/notification-hubs/notify-users

<!-- Images. -->
  []: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-breakingnews-win1.png
  [1]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-reg-2.png
  [2]: ./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-toast-2.png



