<properties
	pageTitle="通知中心本地化的突发新闻教程"
	description="了解如何使用 Azure 通知中心发送本地化的突发新闻通知。"
	services="notification-hubs"
	documentationCenter="windows"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="03/28/2016" 
	wacn.date="01/14/2016"/>

# 使用通知中心发送本地化的突发新闻

> [AZURE.SELECTOR]
- [Windows 应用商店 C#](/documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news/)
- [iOS](/documentation/articles/notification-hubs-ios-send-localized-breaking-news/)

##概述

本主题演示如何使用 Azure 通知中心的**模板**功能广播已按语言和设备本地化的突发新闻通知。在本教程中，你从在[使用通知中心发送突发新闻]中创建的 Windows 应用商店应用程序开始操作。完成时，你将可以注册感兴趣的突发新闻类别，指定要接收通知的语言并仅接收采用该语言的这些类别的推送通知。


此方案包含两个部分：

- Windows 应用商店应用程序允许客户端设备指定一种语言并订阅不同的突发新闻类别；

- 后端使用 Azure 通知中心的**标记**和**模板**功能广播通知。



##先决条件

你必须已完成学习[使用通知中心发送突发新闻][使用通知中心发送突发新闻]教程并具有可用的代码，因为本教程直接围绕该代码展开论述。

你还需要 Visual Studio 2012 或更高版本。


##模板概念

在[使用通知中心发送突发新闻]中，你构建了一个使用**标记**订阅不同新闻类别通知的应用程序。
但是，很多应用程序针对多个市场，需要本地化。这意味着通知内容本身必须本地化且传递到正确的设备组。
在本主题中，我们将演示如何使用通知中心的**模板**功能轻松传递本地化的突发新闻通知。

注意：发送本地化的通知的一种方式是创建每个标签的多个版本。例如，要支持英语、法语和汉语，我们需要三种不同的标签用于世界新闻：“world\_en”、“world\_fr”和“world\_ch”。我们然后必须将世界新闻的本地化版本分别发送到这些标签。在本主题中，我们使用模板来避免增生标签和发送多个消息的要求。

在较高级别上，模板是指定特定设备应如何接收通知的一种方法。模板通过引用作为你应用程序后端所发消息的一部分的属性，指定确切的负载格式。在我们的示例中，我们将发送包含所有支持的语言的区域设置未知的消息：

	{
		"News_English": "...",
		"News_French": "...",
		"News_Mandarin": "..."
	}

然后我们将确保设备注册到引用正确属性的模板。例如，要接收简单的 toast 消息的 Windows 应用商店应用将注册以下包含任何相应标记的模板：

	<toast>
	  <visual>
	    <binding template="ToastText01">
	      <text id="1">$(News_English)</text>
	    </binding>
	  </visual>
	</toast>



模板是很强大的功能，你可以在[模板](/documentation/articles/notification-hubs-templates/)一文中了解其更多信息。

##应用程序用户界面

我们现在将修改你在[使用通知中心发送突发新闻]主题中创建的“突发新闻”应用，以使用模板发送本地化的突发新闻。

在 Windows 应用商店应用程序中：

更改 MainPage.xaml 以包含区域设置组合框：

    <Grid Margin="120, 58, 120, 80"  
            Background="{StaticResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition />
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
        <ComboBox Name="Locale" HorizontalAlignment="Left" VerticalAlignment="Center" Width="200" Grid.Row="1" Grid.Column="0">
            <x:String>English</x:String>
            <x:String>French</x:String>
            <x:String>Mandarin</x:String>
        </ComboBox>
        <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="2" Grid.Column="0"/>
        <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="3" Grid.Column="0"/>
        <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="4" Grid.Column="0"/>
        <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="2" Grid.Column="1"/>
        <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="3" Grid.Column="1"/>
        <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="4" Grid.Column="1"/>
        <Button Content="Subscribe" HorizontalAlignment="Center" Grid.Row="5" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
    </Grid>

## <a name="building-client"></a>构建 Windows 应用商店客户端应用程序

1.  在 Notifications 类中，将一个区域设置参数添加到 *StoreCategoriesAndSubscribe* 和 *SubscribeToCateories* 方法。

        public async Task<Registration> StoreCategoriesAndSubscribe(string locale, IEnumerable<string> categories)
        {
            ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
            ApplicationData.Current.LocalSettings.Values["locale"] = locale;
            return await SubscribeToCategories(categories);
        }

        public async Task<Registration> SubscribeToCategories(string locale, IEnumerable<string> categories = null)
        {
            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

            if (categories == null)
            {
                categories = RetrieveCategories();
            }

            // Using a template registration. This makes supporting notifications across other platforms much easier.
            // Using the localized tags based on locale selected.
            string templateBodyWNS = String.Format("<toast><visual><binding template="ToastText01"><text id="1">$(News_{0})</text></binding></visual></toast>", locale);

            return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "localizedWNSTemplateExample", categories);
        }

	请注意，不是调用 RegisterNativeAsync 方法，我们调用的是 RegisterTemplateAsync：我们将注册特定的通知格式，在其中模板依赖于区域设置。我们还提供模板的名称（“localizedWNSTemplateExample”），因为我们可能要注册多个模板（例如一个用于 toast 通知，一个用于磁贴），需要命名它们以便可以更新或删除它们。

    请注意，如果一个设备使用同一标签注册多个模板，针对该标签的传入消息将导致多个通知发送到设备（每个通知对应一个模板）。当同一逻辑消息必须导致多个可视通知时，此行为很有用，例如在 Windows 应用商店应用程序显示徽章和 toast。

2.  添加以下方法来检索存储的区域设置：

        public string RetrieveLocale()
        {
            var locale = (string) ApplicationData.Current.LocalSettings.Values["locale"];
            return locale != null ? locale : "English";
        }

3. 在你的 MainPage.xaml.cs 中，通过检索“区域设置”组合框的当前值并将它提供给对 Notifications 类的调用，更新按钮单击处理程序，如下所示：

        private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
        {
            var locale = (string)Locale.SelectedItem;

         var categories = new HashSet<string>();
         if (WorldToggle.IsOn) categories.Add("World");
         if (PoliticsToggle.IsOn) categories.Add("Politics");
         if (BusinessToggle.IsOn) categories.Add("Business");
         if (TechnologyToggle.IsOn) categories.Add("Technology");
         if (ScienceToggle.IsOn) categories.Add("Science");
         if (SportsToggle.IsOn) categories.Add("Sports");

            var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(locale,
				 categories);

            var dialog = new MessageDialog("Locale: " + locale + " Subscribed to: " + 
				string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }


4. 最后，在 App.xaml.cs 文件中，确保更新 `InitNotificationsAsync` 方法以检索区域设置，并在订阅时使用该区域设置：

        private async void InitNotificationsAsync()
        {
            var result = await notifications.SubscribeToCategories(notifications.RetrieveLocale());

            // Displays the registration ID so you know it was successful
            if (result.RegistrationId != null)
            {
                var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }
        }

##从后端发送本地化的通知

[AZURE.INCLUDE [notification-hubs-localized-back-end](../includes/notification-hubs-localized-back-end.md)]






<!-- Anchors. -->
[Template concepts]: #concepts
[The app user interface]: #ui
[Building the Windows Store client app]: #building-client
[Send notifications from your back-end]: #send
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[Mobile Service]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/
[使用通知中心通知用户：ASP.NET]: /documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users/
[使用通知中心通知用户：移动服务]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/

[使用通知中心发送突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/

[Azure Management Portal]: https://manage.windowsazure.com/

[通知中心指南]: http://msdn.microsoft.com/library/jj927170.aspx
<!---HONumber=Mooncake_0503_2016-->