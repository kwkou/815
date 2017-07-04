<properties
    pageTitle="使用 Azure 通知中心和 Node.js 发送推送通知"
    description="了解如何使用通知中心从 Node.js 应用程序发送推送通知。"
    keywords="推送通知,push notification,node.js 推送,ios 推送"
    services="notification-hubs"
    documentationcenter="nodejs"
    author="ysxu"
    manager="dwrede"
    editor="" />  

    
<tags
    ms.assetid="ded4749c-6c39-4ff8-b2cf-1927b3e92f93"
    ms.service="notification-hubs"
    ms.workload="mobile"
    ms.tgt_pltfrm="na"
    ms.devlang="javascript"
    ms.topic="article"
    ms.date="10/25/2016"
    wacn.date="12/13/2016"
    ms.author="yuaxu" />  


# 使用 Azure 通知中心和 Node.js 发送推送通知
[AZURE.INCLUDE [notification-hubs-backend-how-to-selector](../../includes/notification-hubs-backend-how-to-selector.md)]

## 概述

> [AZURE.IMPORTANT] 若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 1 元试用](/pricing/1rmb-trial/)。


本指南将演示如何借助 Azure 通知中心，直接从 Node.js 应用程序发送推送通知。

涵盖的方案包括在下列平台将推送通知发送到应用程序：

- Android
- iOS
- Windows Phone
- 通用 Windows 平台

有关通知中心的详细信息，请参阅[后续步骤](#Next_Steps)部分。

## 什么是通知中心？
Azure 通知中心提供用于向移动设备发送推送通知的易于使用、多平台且可缩放的基础结构。有关服务基础结构的详细信息，请参阅 [Azure 通知中心](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj927170.aspx)页。

## 创建 Node.js 应用程序
本教程的第一步是创建新的空白 Node.js 应用程序。有关创建 Node.js 应用程序的说明，请参阅 [Node.js 云服务][Node.js Cloud Service]（使用 Windows PowerShell）。

## 将应用程序配置为使用通知中心
若要使用 Azure 通知中心，请下载和使用 Node.js [azure 包](https://www.npmjs.com/package/azure)，其中包括一组可与推送通知 REST 服务通信的内置帮助程序库。

### 使用 Node 包管理器 (NPM) 可获取包
1. 使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Linux) 等命令行接口导航到你在其中创建了空白应用程序的文件夹。
2. 在命令窗口中键入 **npm install azure-sb**。
3. 可以手动运行 **ls** 或 **dir** 命令来验证是否创建了 **node\_modules** 文件夹。在该文件夹中，找到 **azure** 包，其中包含访问通知中心所需的库。

>[AZURE.NOTE] 可在官方 [NPM 博客](http://blog.npmjs.org/post/85484771375/how-to-install-npm)中了解有关安装 NPM 的详细信息。

### 导入模块
使用某一文本编辑器将以下内容添加到应用程序的 **server.js** 文件的顶部：

    var azure = require('azure');

### 设置 Azure 通知中心连接
可以通过 **NotificationHubService** 对象使用通知中心。以下代码为名为 **hubname** 的通知中心创建一个 **NotificationHubService** 对象。将它添加到靠近 **server.js** 文件顶部、用于导入 azure 模块的语句之后的位置：

    var notificationHubService = azure.createNotificationHubService('hubname','connectionstring');

可通过执行以下步骤从 [Azure 门户]获取连接 **connectionstring** 值：

1. 在左侧导航窗格中，单击“浏览”。
2. 选择“通知中心”，然后找到要用于示例的中心。如果在创建新通知中心时需要获得帮助，可参阅 [Windows 应用商店入门教程](/documentation/articles/notification-hubs-windows-store-dotnet-get-started-wns-push-notification/)。
3. 选择“设置”。
4. 单击“访问策略”。你会看到共享连接字符串和完全访问连接字符串。

![Azure 门户 — 通知中心](./media/notification-hubs-nodejs-how-to-use-notification-hubs/notification-hubs-portal.png)  


> [AZURE.NOTE] 还可使用 [Azure PowerShell](/documentation/articles/powershell-install-configure/) 提供的 **Get-AzureSbNamespace** cmdlet，或者在 [Azure 命令行接口 (Azure CLI)](/documentation/articles/xplat-cli-install/) 中使用 **azure sb namespace show** 命令检索连接字符串。

## 一般体系结构
**NotificationHubService** 对象将公开用于向特定设备和应用程序发送推送通知的以下对象实例：

- **Android** - 使用可从 **notificationHubService.gcm** 中获取的 **GcmService** 对象
- **iOS** - 使用可在 **notificationHubService.apns** 中访问的 **ApnsService** 对象
- **Windows Phone** - 使用可从 **notificationHubService.mpns** 中获取的 **MpnsService** 对象
- **通用 Windows 平台** - 使用可从 **notificationHubService.wns** 中获取的 **WnsService** 对象

### 如何：向 Android 应用程序发送推送通知
**GcmService** 对象提供可用于将推送通知发送到 Android 应用程序的 **send** 方法。该 **send** 方法接受以下参数：

- **Tags** - 标记标识符。如果没有提供任何标记，通知将发送给所有客户端。
- **Payload** - 消息的 JSON 或原始字符串的有效负载。
- **Callback** - 回调函数。

有关有效负载格式的详细信息，请参阅[实施 GCM 服务器](http://developer.android.com/google/gcm/server.html#payload)文档中的“有效负载”部分。

以下代码使用 **NotificationHubService** 公开的 **GcmService** 实例将一条推送通知发送到所有已注册的客户端。

    var payload = {
      data: {
        message: 'Hello!'
      }
    };
    notificationHubService.gcm.send(null, payload, function(error){
      if(!error){
        //notification sent
      }
    });

### 如何：向 iOS 应用程序发送推送通知
与上述 Android 应用程序一样，**ApnsService** 对象提供可用于将推送通知发送到 iOS 应用程序的 **send** 方法。该 **send** 方法接受以下参数：

- **Tags** - 标记标识符。如果没有提供任何标记，通知将发送给所有客户端。
- **Payload** - 消息的 JSON 或字符串的有效负载。
- **Callback** - 回调函数。

有关有效负载格式的详细信息，请参阅[本地和推送通知编程指南](http://developer.apple.com/library/ios/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/ApplePushService/ApplePushService.html)文档中的“通知有效负载”部分。

以下代码使用 **NotificationHubService** 公开的 **ApnsService** 实例将一条警报消息发送给所有客户端：

    var payload={
        alert: 'Hello!'
      };
    notificationHubService.apns.send(null, payload, function(error){
      if(!error){
         // notification sent
      }
    });

### 如何：向 Windows Phone 应用程序发送推送通知
**MpnsService** 对象提供可用于将推送通知发送到 Windows Phone 应用程序的 **send** 方法。该 **send** 方法接受以下参数：

- **Tags** - 标记标识符。如果没有提供任何标记，通知将发送给所有客户端。
- **Payload** - 消息的 XML 有效负载。
- **TargetName** - `toast` 表示 toast 通知。`token` 表示磁贴通知。
- **NotificationClass** - 通知的优先级。有关有效值，请参阅[从服务器推送通知](http://msdn.microsoft.com/zh-cn/library/hh221551.aspx)文档中的“HTTP 标头元素”部分。
- **Options** - 可选的请求标头。
- **Callback** - 回调函数。

若要了解有效的 **TargetName**、**NotificationClass** 和标头选项的列表，请查看[从服务器推送通知](http://msdn.microsoft.com/zh-cn/library/hh221551.aspx)页面。

以下示例代码使用 **NotificationHubService** 公开的 **MpnsService** 实例发送 toast 推送通知：

    var payload = '<?xml version="1.0" encoding="utf-8"?><wp:Notification xmlns:wp="WPNotification"><wp:Toast><wp:Text1>string</wp:Text1><wp:Text2>string</wp:Text2></wp:Toast></wp:Notification>';
    notificationHubService.mpns.send(null, payload, 'toast', 22, function(error){
      if(!error){
        //notification sent
      }
    });

### 如何：向通用 Windows 平台 (UWP) 应用程序发送推送通知
**WnsService** 对象提供可用于将推送通知发送到通用 Windows 平台应用程序的 **send** 方法。该 **send** 方法接受以下参数：

- **Tags** - 标记标识符。如果没有提供任何标记，通知将发送给所有已注册的客户端。
- **Payload** - XML 消息有效负载。
- **Type** - 通知类型。
- **Options** - 可选的请求标头。
- **Callback** - 回调函数。

若要了解有效的类型和请求标头的列表，请参阅[推送通知服务请求和响应头](http://msdn.microsoft.com/zh-cn/library/windows/apps/hh465435.aspx)。

以下代码使用 **NotificationHubService** 公开的 **WnsService** 实例将 toast 推送通知发送到 UWP 应用：

    var payload = '<toast><visual><binding template="ToastText01"><text id="1">Hello!</text></binding></visual></toast>';
    notificationHubService.wns.send(null, payload , 'wns/toast', function(error){
      if(!error){
         // notification sent
      }
    });

## <a name="Next_Steps"></a>后续步骤

使用上述示例代码段，你可以轻松地构建服务基础结构，将推送通知传递到各种设备。了解使用通知中心和 node.js 的基础知识之后，请参考下列链接以了解有关如何进一步扩展这些功能的详细信息。

- 请参阅 [Azure 通知中心](https://msdn.microsoft.com/zh-cn/library/azure/jj927170.aspx)的 MSDN 参考。
- 请访问 GitHub 上的[用于 Node 的 Azure SDK] 存储库以获取更多示例和实施详细信息。

[用于 Node 的 Azure SDK]: https://github.com/WindowsAzure/azure-sdk-for-node
[Next Steps]: #Next_Steps
[What are Service Bus Topics and Subscriptions?]: #what-are-service-bus-topics
[Create a Service Namespace]: #create-a-service-namespace
[Obtain the Default Management Credentials for the Namespace]: #obtain-default-credentials
[Create a Node.js Application]: #Create_a_Nodejs_Application
[Configure Your Application to Use Service Bus]: #Configure_Your_Application_to_Use_Service_Bus
[How to: Create a Topic]: #How_to_Create_a_Topic
[How to: Create Subscriptions]: #How_to_Create_Subscriptions
[How to: Send Messages to a Topic]: #How_to_Send_Messages_to_a_Topic
[How to: Receive Messages from a Subscription]: #How_to_Receive_Messages_from_a_Subscription
[How to: Handle Application Crashes and Unreadable Messages]: #How_to_Handle_Application_Crashes_and_Unreadable_Messages
[How to: Delete Topics and Subscriptions]: #How_to_Delete_Topics_and_Subscriptions
[1]: #Next_Steps
[Topic Concepts]: ./media/notification-hubs-nodejs-how-to-use-notification-hubs/sb-topics-01.png
[Azure Classic Management Portal]: http://manage.windowsazure.cn
[image]: ./media/notification-hubs-nodejs-how-to-use-notification-hubs/sb-queues-03.png
[2]: ./media/notification-hubs-nodejs-how-to-use-notification-hubs/sb-queues-04.png
[3]: ./media/notification-hubs-nodejs-how-to-use-notification-hubs/sb-queues-05.png
[4]: ./media/notification-hubs-nodejs-how-to-use-notification-hubs/sb-queues-06.png
[5]: ./media/notification-hubs-nodejs-how-to-use-notification-hubs/sb-queues-07.png
[SqlFilter.SqlExpression]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.servicebus.messaging.sqlfilter.sqlexpression.aspx
[Azure Service Bus Notification Hubs]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj927170.aspx
[SqlFilter]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.servicebus.messaging.sqlfilter.aspx
[Node.js Cloud Service]: /documentation/articles/cloud-services-nodejs-develop-deploy-app/
[Previous Management Portal]: ./media/notification-hubs-nodejs-how-to-use-notification-hubs/previous-portal.png

[Azure 门户]: https://portal.azure.cn

<!---HONumber=Mooncake_1205_2016-->
