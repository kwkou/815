<properties linkid="manage-services-notification-hubs-notify-users-xplat-aspnet" urlDisplayName="Notify Users xplat aspnet" pageTitle="Send cross-platform notifications to users with Notification Hubs (ASP.NET)" metaKeywords="" description="Learn how to use Notification Hubs templates to send, in a single request, a platform-agnostic notification that targets all platforms." metaCanonical="" services="notification-hubs" documentationCenter="" title="Send cross-platform notifications to users with Notification Hubs" authors="glenga" solutions="" manager="" editor="" />

# 使用通知中心向用户发送跨平台通知

<div class="dev-center-tutorial-selector sublanding">
<a href="/en-us/documentation/articles/notification-hubs-mobile-services-cross-platform-notify-users/" title="移动服务">移动服务</a><a href="/en-us/documentation/articles/notification-hubs-aspnet-cross-platform-notify-users/" title="ASP.NET" class="current">ASP.NET</a>
</div>

在上一教程[使用通知中心通知用户][使用通知中心通知用户]中，你了解了如何将通知推送到经过身份验证的特定用户所注册的所有设备。在该教程中，需要使用多个请求将通知发送到每个支持的客户端平台。通知中心支持模板，这允许你指定特定设备要如何接收通知。这简化了发送跨平台通知。本主题演示如何利用模板在单个请求中发送针对所有平台的平台未知通知。有关模板的更多详细信息，请参见 [Azure 通知中心概述][Azure 通知中心概述]。

<div class="dev-callout"><b>说明</b>
<p>通知中心允许设备使用同一标记注册多个模板。在这种情况下，针对该标签的传入消息将导致多个通知发送到设备（每个通知对应一个模板）。这允许你在多个可视通知中显示同一消息，如作为 Windows 应用商店应用程序中的徽章和 toast 通知。</p>
</div>

完成以下步骤来使用模板发送跨平台通知：

1.  在 Visual Studio 的解决方案资源管理器中，展开“控制器”文件夹，然后打开 RegisterController.cs 文件。

2.  在 **Post** 方法中查找当`updated` 值为 **false** 时创建新注册的代码块，并将它替换为以下代码：

        if (!updated)
        {
            switch (platform)
            {
                case "windows":
                    var template = @"<toast><visual><binding template=""ToastText01""><text id=""1"">$(message)</text></binding></visual></toast>";
                    await hubClient.CreateWindowsTemplateRegistrationAsync(channelUri, template, new string[] { instId, userTag });
                    break;
                case "ios":
                    template = "{\"aps\":{\"alert\":\"$(message)\"}, \"inAppMessage\":\"$(message)\"}";
                    await hubClient.CreateAppleTemplateRegistrationAsync(deviceToken, template, new string[] { instId, userTag });
                    break;
            } 
        }

    此代码调用平台特定的方法来创建模板注册而非本机注册。不需要修改现有注册，因为模板注册派生自本机注册。

3.  将 **sendNotification** 方法替换为以下代码：

        // Send a cross-plaform notification by using templates. 
        private async Task sendNotification(string notificationText, string tag)
        {           
                var notification = new Dictionary<string, string> { { "message", "Hello, " + tag } };
                await hubClient.SendTemplateNotificationAsync(notification, tag);        
        }

    此代码将通知同时发送到所有平台，而不必指定本机负载。通知中心使用提供的*标签*值（在注册的模板中指定）生成正确的负载并将它传递到每个设备。

4.  再次运行客户端应用程序并验证注册成功。

5.  （可选）将客户端应用程序部署到第二个设备，然后运行该应用程序。

    请注意通知将显示在每个设备上。

## 后续步骤

现在，你已完成本教程，可以查看以下主题了解有关通知中心和模板的更多信息：

-   **使用通知中心发送突发新闻（[Windows 应用商店 C\#][Windows 应用商店 C\#] / [iOS][Windows 应用商店 C\#]）**
    演示使用模板的另一方案

-   **[Azure 通知中心概述][Azure 通知中心概述]**
    “概述”主题提供有关模板的更多详细信息。

-   **[针对 Windows 应用商店的通知中心操作指南][针对 Windows 应用商店的通知中心操作指南]**
    包含模板表示语言参考。

<!-- Anchors. --> 

<!-- Images. --> 

<!-- URLs. -->

  [移动服务]: /en-us/documentation/articles/notification-hubs-mobile-services-cross-platform-notify-users/ "移动服务"
  [ASP.NET]: /en-us/documentation/articles/notification-hubs-aspnet-cross-platform-notify-users/ "ASP.NET"
  [使用通知中心通知用户]: /en-us/manage/services/notification-hubs/notify-users-aspnet
  [Azure 通知中心概述]: http://go.microsoft.com/fwlink/p/?LinkId=317339
  [Windows 应用商店 C\#]: /en-us/manage/services/notification-hubs/breaking-news-dotnet
  [针对 Windows 应用商店的通知中心操作指南]: http://msdn.microsoft.com/zh-cn/library/azure/jj927172.aspx
