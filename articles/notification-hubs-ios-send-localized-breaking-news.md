<properties linkid="develop-notificationhubs-tutorials-send-localized-breaking-news-iOS" urlDisplayName="Localized Breaking News" pageTitle="Notification Hubs Localized Breaking News Tutorial for iOS" metaKeywords="" description="Learn how to use Azure Service Bus Notification Hubs to send localized breaking news notifications (iOS)." metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="" title="Use Notification Hubs to send localized breaking news to iOS devices" authors="ricksal" solutions="" manager="" editor="" />

# 使用通知中心将本地化的突发新闻发送到 iOS 设备

<div class="dev-center-tutorial-selector sublanding"> 
<a href="/zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news/" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/notification-hubs-ios-send-localized-breaking-news/" title="iOS" class="current">iOS</a>
</div>

本主题演示如何使用 Azure 通知中心的**模板**功能广播已按语言和设备本地化的突发新闻通知。在本教程中，你从在[使用通知中心发送突发新闻][使用通知中心发送突发新闻]中创建的 Windows 应用商店应用程序开始操作。完成时，你将可以注册感兴趣的突发新闻类别，指定要接收通知的语言并仅接收采用该语言的这些类别的推送通知。

本教程将指导你完成启用此方案的以下基本步骤：

1.  [模板概念][模板概念]
2.  [应用程序用户界面][应用程序用户界面]
3.  [构建 iOS 应用程序][构建 iOS 应用程序]
4.  [从后端发送通知][从后端发送通知]

此方案包含两个部分：

-   iOS 应用程序允许客户端设备指定一种语言并订阅不同的突发新闻类别；

-   后端使用 Azure 通知中心的**标记**和**模板**功能广播通知。

## 先决条件

你必须已完成学习[使用通知中心发送突发新闻][使用通知中心发送突发新闻]教程并具有可用的代码，因为本教程直接围绕该代码展开论述。

你还需要 Visual Studio 2012。

## <a name="concepts"></a><span class="short-header">概念</span>模板概念

在[使用通知中心发送突发新闻][使用通知中心发送突发新闻]中，你生成了一个使用**标记**订阅不同新闻类别通知的应用程序。但是，很多应用程序针对多个市场，需要本地化。这意味着通知内容本身必须本地化且传递到正确的设备组。在本主题中，我们将演示如何使用通知中心的**模板**功能轻松传递本地化的突发新闻通知。

注意：发送本地化的通知的一种方式是创建每个标签的多个版本。例如，要支持英语、法语和汉语，我们需要三种不同的标签用于世界新闻：“world\_en”、“world\_fr”和“world\_ch”。我们然后必须将世界新闻的本地化版本分别发送到这些标签。在本主题中，我们使用模板来避免增生标签和发送多个消息的要求。

在较高级别上，模板是指定特定设备应如何接收通知的一种方法。模板通过引用作为你应用程序后端所发消息的一部分的属性，指定确切的负载格式。在我们的示例中，我们将发送包含所有支持的语言的区域设置未知的消息：

    {
        "News_English": "...",
        "News_French": "...",
        "News_Mandarin": "..."
    }

然后我们将确保设备注册到引用正确属性的模板。例如，要注册法语新闻的 iOS 应用程序将注册以下模板：

    {
        aps:{
            alert: "$(News_French)"
        }
    }

模板是很强大的功能，你可以在[通知中心指南][通知中心指南]一文中了解其更多信息。一个模板表达语言的参考是 [操作指南：Service Bus 通知中心（iOS 应用程序）][操作指南：Service Bus 通知中心（iOS 应用程序）]。

## <a name="ui"></a><span class="short-header">应用程序 UI</span>应用程序用户界面

我们现在将修改你在[使用通知中心发送突发新闻][使用通知中心发送突发新闻]主题中创建的“突发新闻”应用程序，以使用模板发送本地化的突发新闻。

在 MainStoryboard\_iPhone.storyboard 中，添加具有我们支持的三种语言的分段控件：英语、法语和汉语。

![][]

然后确保在 ViewController.h 中添加 IBOutlet，如下所示：

![][1]

## <a name="building-client"></a><span class="building app">应用程序 UI</span>生成 iOS 应用程序

为了改编你的客户端应用程序以接收本地化的消息，你必须使用模板注册替换*本机*注册（即未指定模板的注册）。

1.  在 Notification.h 中，添加 *retrieveLocale* 方法，然后修改存储区和 subscribe 方法，如下所示：

        - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet*) categories completion: (void (^)(NSError* error))completion;

        - (void) subscribeWithLocale:(int) locale categories:(NSSet*) categories completion:(void (^)(NSError *))completion;

        - (NSSet*) retrieveCategories;

        - (int) retrieveLocale;

    在 Notification.m 中，通过添加区域设置参数并将它存储在用户默认值中，修改 *storeCategoriesAndSubscribe* 方法：

        - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion {
            NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
            [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];
            [defaults setInteger:locale forKey:@"BreakingNewsLocale"];
            [defaults synchronize];

            [self subscribeWithLocale: locale categories:categories completion:completion];
        }

    然后修改 *subscribe* 方法以包括该区域设置：

        - (void) subscribeWithLocale: (int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion{
            SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:@"<connection string>" notificationHubPath:@"<hub name>"];

            NSString* localeString;
            switch (locale) {
                case 0:
                    localeString = @"English";
                    break;
                case 1:
                    localeString = @"French";
                    break;
                case 2:
                    localeString = @"Mandarin";
                    break;
            }

            NSString* template = [NSString stringWithFormat:@"{\"aps\":{\"alert\":\"$(News_%@)\"},\"inAppMessage\":\"$(News_%@)\"}", localeString, localeString];

            [hub registerTemplateWithDeviceToken:self.deviceToken name:@"newsTemplate" jsonBodyTemplate:template expiryTemplate:@"0" tags:categories completion:completion];
        }

    请注意，我们现在使用的是 *registerTemplateWithDeviceToken* 方法而非 *registerNativeWithDeviceToken*。当我们注册一个模板时，必须提供 json 模板，还要指定其名称（因为我们的应用程序可能要注册不同的模板）。确保将类别作为标记注册，因为我们要确保接收有关这些新闻的通知。

    最后，添加一个方法来从用户默认设置检索区域设置：

        - (int) retrieveLocale {
            NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

            int locale = [defaults integerForKey:@"BreakingNewsLocale"];

            return locale < 0?0:locale;
        }

2.  现在我们修改了 Notifications 类，必须确保 ViewController 使用新的 UISegmentControl。在 *viewDidLoad* 方法中添加以下行，以确保显示当前选择的区域设置：

        self.Locale.selectedSegmentIndex = [notifications retrieveLocale];

    然后，在 *subscribe* 方法中，将对 *storeCategoriesAndSubscribe* 的调用更改为：

        [notifications storeCategoriesAndSubscribeWithLocale: self.Locale.selectedSegmentIndex categories:[NSSet setWithArray:categories] completion: ^(NSError* error) {
            if (!error) {
                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                                      @"Subscribed!" delegate:nil cancelButtonTitle:
                                      @"OK" otherButtonTitles:nil, nil];
                [alert show];
            } else {
                NSLog(@"Error subscribing: %@", error);
            }
        }];

3.  最后，你必须在 AppDelegate.m 中更新 *didRegisterForRemoteNotificationsWithDeviceToken* 方法，以便在应用程序启动时正确刷新你的注册信息。将对通知的 *subscribe* 方法的调用更改为：

        NSSet* categories = [notifications retrieveCategories];
        int locale = [notifications retrieveLocale];
        [notifications subscribeWithLocale: locale categories:categories completion:^(NSError* error) {
            if (error != nil) {
                NSLog(@"Error registering for notifications: %@", error);
            }
        }];

## <a name="send"></a><span class="short-header">发送本地化的通知</span>从后端发送本地化的通知

[WACOM.INCLUDE [notification-hubs-localized-back-end][notification-hubs-localized-back-end]]

## 后续步骤

有关使用模板的详细信息，请参阅：

-   [使用通知中心通知用户：ASP.NET][使用通知中心通知用户：ASP.NET]
-   [使用通知中心通知用户：移动服务][使用通知中心通知用户：移动服务]
-   [通知中心指南][通知中心指南]

[适用于 iOS 的通知中心操作方法][操作指南：Service Bus 通知中心（iOS 应用程序）]中提供了模板表达式语言的参考信息。



<!-- Anchors. -->
  [模板概念]: #concepts
  [应用程序用户界面]: #ui
  [构建 iOS 应用程序]: #building-client
  [从后端发送通知]: #send 

<!-- Images. --> 

<!-- URLs. -->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news/ "Windows 应用商店 C#"
  [iOS]: /zh-cn/documentation/articles/notification-hubs-ios-send-localized-breaking-news/ "iOS"
  [使用通知中心发送突发新闻]: /en-us/manage/services/notification-hubs/breaking-news-ios
  [通知中心指南]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
  [操作指南：Service Bus 通知中心（iOS 应用程序）]: http://msdn.microsoft.com/zh-cn/library/jj927168.aspx
  []: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized1.png
  [1]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized2.png
  [notification-hubs-localized-back-end]: ../includes/notification-hubs-localized-back-end.md
  [使用通知中心通知用户：ASP.NET]: /en-us/manage/services/notification-hubs/notify-users-aspnet
  [使用通知中心通知用户：移动服务]: /en-us/manage/services/notification-hubs/notify-users
