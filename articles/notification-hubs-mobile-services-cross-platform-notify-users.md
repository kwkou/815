<properties linkid="manage-services-notification-hubs-notify-users-xplat-mobile-services" urlDisplayName="notify users xplat mobile services" pageTitle="Send cross-platform notifications to users with Notification Hubs (Mobile Services)" metaKeywords="" description="Learn how to use Notification Hubs templates to send, in a single request, a platform-agnostic notification that targets all platforms." metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="" title="Send cross-platform notifications to users with Notification Hubs" authors="glenga" solutions="" manager="" editor="" />

# 使用通知中心向用户发送跨平台通知

<div class="dev-center-tutorial-selector sublanding">
<a href="/zh-cn/documentation/articles/notification-hubs-mobile-services-cross-platform-notify-users/" title="移动服务" class="current">移动服务</a>
<a href="/zh-cn/documentation/articles/notification-hubs-aspnet-cross-platform-notify-users/" title="ASP.NET">ASP.NET</a>
</div>

在上一教程[使用通知中心通知用户][使用通知中心通知用户]中，你了解了如何将通知推送到经过身份验证的特定用户所注册的所有设备。在该教程中，需要使用多个请求将通知发送到每个支持的客户端平台。通知中心支持模板，这允许你指定特定设备要如何接收通知。这简化了发送跨平台通知。本主题演示如何利用模板在单个请求中发送针对所有平台的平台未知通知。有关模板的更多详细信息，请参见 [Azure 通知中心概述][Azure 通知中心概述]。

<div class="dev-callout"><b>说明</b>
<p>通知中心允许设备使用同一标记注册多个模板。在这种情况下，针对该标签的传入消息将导致多个通知发送到设备（每个通知对应一个模板）。这允许你在多个可视通知中显示同一消息，如作为 Windows 应用商店应用程序中的徽章和 toast 通知。</p>
</div>

完成以下步骤来使用模板发送跨平台通知：

1.  登录到 [Azure 管理门户][Azure 管理门户]，单击“移动服务”，然后单击你的应用程序。

    ![][]

2.  单击“API”选项卡，然后单击 API 表中的 **register\_notifications** 条目。

    ![][1]

3.  单击“脚本”选项卡，查找当 existingRegs 值为 **false** 时创建新注册的 **else** 块，并将它替换为以下代码：

        else {
            // Create a new registration.
            var template;
            if (platform === 'win8') {                
                template = { text1: '$(message)' };              
                hub.wns.createToastText01Registration(request.body.channelUri, 
                [userId, installationId], template, registrationComplete);
            } else if (platform === 'ios') {
                template = '{\"aps\":{\"alert\":\"$(message)\"}, \"inAppMessage\":\"$(message)\"}';
                hub.apns.createTemplateRegistration(request.body.deviceToken, 
                [userId, installationId], template, registrationComplete);
            } else {
                response.send(500, 'Unknown client.');
            }
        }

    此代码调用平台特定的方法来创建模板注册而非本机注册。不需要修改现有注册，因为模板注册派生自本机注册。

4.  单击“数据”选项卡，然后单击 **TodoItem** 表。

    ![][2]

5.  在 **todoitem** 中，单击“脚本”选项卡，选择“插入”，然后将现有 **insert** 函数替换为以下代码：

    ![][3]

    将显示当 **TodoItem** 表中发生插入时所调用的函数。

6.  将插入函数替换为以下代码：

        function insert(item, user, request) {
            var azure = require('azure');
            var hub = azure.createNotificationHubService('<NOTIFICATION_HUB_NAME>', 
            '<FULL_SAS_CONNECTION_STRING>');

            // Execute the request and send notifications.
            request.execute({
                success: function() {
                    // Write the default response and send a notification 
                    // to the user on all devices by using the userId tag.
                    request.respond();
                    // Create a template-based payload.
                    var payload = '{ "message" : "New item added: ' + item.text + '" }';            
                    // Send a notification to the current user on all platforms. 
                    hub.send(user.userId, payload,  
                    function(error, outcome){
                        // Do something here with the outcome or error.
                    });     
                }
            });
        }

    此代码将通知同时发送到所有平台，而不必指定本机负载。通知中心使用提供的*标签*值（在注册的模板中指定）生成正确的负载并将它传递到每个设备。

7.  更新脚本，以将 *`<NOTIFICATION_HUB_NAME>`* 和 *`<FULL_SAS_CONNECTION_STRING>`* 替换为通知中心的值，然后单击“保存”。

8.  停止设备模拟器或从设备卸载现有客户端应用程序。

    这确保新的安装 ID 由移动服务客户端生成。如果你没有这样做，服务将尝试使用现有的非模板注册。

9.  再次部署并运行客户端应用程序，验证注册成功且显示新的注册 ID。

10. 输入和以前一样的文本并添加新项。

    请注意，完成插入后，应用程序将收到来自通知中心的推送通知。

11. （可选）将客户端应用程序部署到第二个设备，然后运行该应用程序并插入文本。

    请注意通知将显示在每个设备上。

## 后续步骤

现在，你已完成本教程，可以查看以下主题了解有关通知中心和模板的更多信息：

-   **使用通知中心发送突发新闻（[Windows 应用商店 C\#][Windows 应用商店 C\#] / [iOS][iOS]）**
    演示使用模板的另一方案

-   **[Azure 通知中心概述][Azure 通知中心概述]**
    “概述”主题提供有关模板的更多详细信息。

-   **[针对 Windows 应用商店的通知中心操作指南][针对 Windows 应用商店的通知中心操作指南]**
    包含模板表示语言参考。

<!-- Anchors. -->  

  [移动服务]: /zh-cn/documentation/articles/notification-hubs-mobile-services-cross-platform-notify-users/ "移动服务"
  [ASP.NET]: /zh-cn/documentation/articles/notification-hubs-aspnet-cross-platform-notify-users/ "ASP.NET"
  [使用通知中心通知用户]: /en-us/manage/services/notification-hubs/notify-users
  [Azure 通知中心概述]: http://go.microsoft.com/fwlink/p/?LinkId=317339
  [Azure 管理门户]: https://manage.windowsazure.cn/

<!-- Images. -->
  []: ./media/notification-hubs-mobile-services-cross-platform-notify-users/mobile-services-selection.png
  [1]: ./media/notification-hubs-mobile-services-cross-platform-notify-users/mobile-custom-api-select.png
  [2]: ./media/notification-hubs-mobile-services-cross-platform-notify-users/mobile-portal-data-tables.png
  [3]: ./media/notification-hubs-mobile-services-cross-platform-notify-users/mobile-insert-script-push2.png

<!-- URLs. --> 
  [Windows 应用商店 C\#]: /en-us/manage/services/notification-hubs/breaking-news-dotnet
  [iOS]: /en-us/manage/services/notification-hubs/breaking-news-ios
  [针对 Windows 应用商店的通知中心操作指南]: http://msdn.microsoft.com/zh-cn/library/azure/jj927172.aspx
