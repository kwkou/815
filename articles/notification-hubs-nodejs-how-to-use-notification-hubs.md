<properties
	pageTitle="如何通过 Node.js 使用通知中心"
	description="了解如何使用通知中心从 Node.js 应用程序发送推送通知。"
	services="notification-hubs"
	documentationCenter="nodejs"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="06/16/2015"
	wacn.date="11/02/2015"/>
	
	
	
	
	
# 如何通过 Node.js 使用通知中心

> [AZURE.SELECTOR]
- [Java](/documentation/articles/notification-hubs-java-backend-how-to)
- [PHP](/documentation/articles/notification-hubs-php-backend-how-to)
- [Python](/documentation/articles/notification-hubs-python-backend-how-to)

##概述

本指南演示如何从 Node.js 应用程序使用通知中心。涉及的任务包括**将通知发送到 Android、iOS、Windows Phone 和 Windows 应用商店应用程序**。有关通知中心的详细信息，请参阅[后续步骤](#next)部分。

##什么是通知中心？



Azure 通知中心可提供用于向移动设备发送推送通知的易于使用、多平台且可缩放的基础结构。有关详细信息，请参阅 [Azure 通知中心](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj927170.aspx)。

##<a id="create"></a>创建 Node.js 应用程序

创建一个空的 Node.js 应用程序。有关创建 Node.js 应用程序的说明，请参阅[创建 Node.js 应用程序并将其部署到 Azure 网站][nodejswebsite]、[Node.js 云服务][Node.js Cloud Service]（使用 Windows PowerShell）或“使用 WebMatrix 构建网站”。

##<a id="config"></a>将应用程序配置为使用通知中心

要使用 Azure 通知中心，需要下载并使用 Node.js azure 包。其中包括一组便于与 REST 服务进行通信的库。

### 使用 Node 包管理器 (NPM) 可获取该程序包

1.  使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix) 等命令行界面导航到您在其中创建了示例应用程序的文件夹。

2.  在命令窗口中键入 **npm install azure**，这应会生成以下输出：

        azure@0.7.0 node_modules\azure
        |-- dateformat@1.0.2-1.2.3
        |-- xmlbuilder@0.4.2
        |-- node-uuid@1.2.0
        |-- mime@1.2.9
        |-- underscore@1.4.4
        |-- validator@0.4.28
        |-- tunnel@0.0.2
        |-- wns@0.5.3
        |-- xml2js@0.2.6 (sax@0.4.2)
        |-- request@2.16.6 (forever-agent@0.2.0, aws-sign@0.2.0, tunnel-agent@0.2.0, oauth-sign@0.2.0, json-stringify-safe@3.0.0, cookie-jar@0.2.0, node-uuid@1.4.0, qs@0.5.5, hawk@0.10.2, form-data@0.0.7)

3.  可以手动运行 **ls** 或 **dir** 命令来验证是否创建了 **node\_modules** 文件夹。在该文件夹中，找到 **azure** 包，其中包含访问通知中心所需的库。

### 导入模块

使用某一文本编辑器将以下内容添加到应用程序的 **server.js** 文件的顶部：

    var azure = require('azure');

### 设置 Azure 通知中心连接

可以通过 **NotificationHubService** 对象使用通知中心。以下代码为名为 **hubname** 的通知中心创建一个**NotificationHubService** 对象。将它添加到靠近 **server.js** 文件顶部、用于导入 azure 模块的语句之后的位置：

    var notificationHubService = azure.createNotificationHubService('hubname','connectionstring');

可通过执行以下步骤从 Azure 管理门户获取连接 **connectionstring** 值：

1.  在 Azure 管理门户中选择“服务总线”，然后选择包含通知中心的命名空间。

2.  选择“通知中心”，然后选择要使用的通知中心。

3.  从“速览”部分中选择“查看连接字符串”，并复制连接字符串值。

> [AZURE.NOTE]还可以使用 Azure PowerShell 提供的 **Get-AzureSbNamespace** cmdlet 或者在 Azure 命令行界面 (Azure CLI) 中使用 **azure sb namespace show** 命令检索连接字符串。

</div>

## <span id="send"></span></a>如何发送通知

**NotificationHubService** 对象将公开用于向特定设备和应用程序发送通知的以下对象实例：

-   **Android** - 使用可从 **notificationHubService.gcm** 中获取的 **GcmService** 对象
-   **iOS** - 使用可在 **notificationHubService.apns** 中访问的 **ApnsService** 对象
-   **Windows Phone** - 使用可从 **notificationHubService.mpns** 中获取的 **MpnsService** 对象
-   **Windows 应用商店应用程序** - 使用可从 **notificationHubService.wns** 中获取的 **WnsService** 对象

### 如何发送 Android 应用程序通知

**GcmService** 对象提供可用于将通知发送到 Android 应用程序的 **send** 方法。该 **send** 方法接受以下参数：

-   Tags - 标记标识符。如果没有提供任何标记，通知将发送给所有客户端。
-   Payload - 消息的 JSON 或字符串负载
-   Callback - 回调函数

有关负载格式的详细信息，请参阅[实施 GCM 服务器][实施 GCM 服务器]中的“负载”部分。

以下代码使用 **NotificationHubService** 公开的 **GcmService** 实例将一条消息发送到所有客户端。

    var payload = {
      data: {
        msg: 'Hello!'
      }
    };
    notificationHubService.gcm.send(null, payload, function(error){
      if(!error){
        //notification sent
      }
    });

### 如何发送 iOS 应用程序通知

**ApnsService** 对象提供可用于将通知发送到 iOS 应用程序的 **send** 方法。该 **send** 方法接受以下参数：

-   Tags - 标记标识符。如果没有提供任何标记，通知将发送给所有客户端。
-   Payload - 消息的 JSON 或字符串负载
-   Callback - 回调函数

有关负载格式的详细信息，请参阅[本地和推送通知编程指南][本地和推送通知编程指南]中的“通知负载”部分。

以下代码使用 **NotificationHubService** 公开的 **ApnsService** 实例将一条警报消息发送给所有客户端：

    var payload={ 
        alert: 'Hello!'
      };
    notificationHubService.apns.send(null, payload, function(error){
      if(!error){
        // notification sent
      }
    });

### 如何发送 Windows Phone 通知

**MpnsService** 对象提供可用于将通知发送到 Windows Phone 应用程序的 **send** 方法。该 **send** 方法接受以下参数：

-   Tags - 标记标识符。如果没有提供任何标记，通知将发送给所有客户端。
-   Payload - 消息的 XML 负载
-   TargetName -“toast”表示 toast 通知。“token”表示磁贴通知。
-   NotificationClass - 通知的优先级。有关有效值，请参阅[从服务器推送通知][从服务器推送通知]中的“HTTP 标头元素”部分。
-   Options - 可选的请求标头
-   Callback - 回调函数

有关有效的 TargetName、NotificationClass 和标头选项的列表，请参阅[从服务器推送通知][从服务器推送通知]。

以下代码使用 **NotificationHubService** 公开的 **MpnsService** 实例发送 toast 警报：

    var payload = '<?xml version="1.0" encoding="utf-8"?><wp:Notification xmlns:wp="WPNotification"><wp:Toast><wp:Text1>string</wp:Text1><wp:Text2>string</wp:Text2></wp:Toast></wp:Notification>';
    notificationHubService.mpns.send(null, payload, 'toast', 22, function(error){
      if(!error){
        //notification sent
      }
    });

### 如何发送 Windows 应用商店应用程序通知

**WnsService** 对象提供可用于将通知发送到 Windows 应用商店应用程序的 **send** 方法。该 **send** 方法接受以下参数：

-   Tags - 标记标识符。如果没有提供任何标记，通知将发送给所有客户端。
-   Payload - XML 消息负载
-   Type - 通知类型
-   Options - 可选的请求标头
-   Callback - 回调函数

有关有效类型和请求标头的列表，请参阅[推送通知服务请求和响应标头][推送通知服务请求和响应标头]。

以下代码使用 **NotificationHubService** 公开的 **WnsService** 实例发送 toast 警报：

    var payload = '<toast><visual><binding template="ToastText01"><text id="1">Hello!</text></binding></visual></toast>';
    notificationHubService.wns.send(null, payload , 'wns/toast', function(error){
      if(!error){
        // notification sent
      }
    });

## <span id="next"></span></a> 后续步骤

现在，你已了解有关通知中心的基础知识，请单击下面的链接了解更多信息。

-   请参阅 MSDN 参考：[Azure 通知中心](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj927170.aspx)。
-   访问 GitHub 上的 [Azure SDK for Node] 存储库。

  [后续步骤]: #next
  [什么是 服务总线 通知中心？]: #hub
  [创建 Node.js 应用程序]: #create
  [配置应用程序以使用 服务总线]: #config
  [如何：发送通知]: #send
  [Azure 服务总线 通知中心]: http://msdn.microsoft.com/zh-cn/library/azure/jj927170.aspx
  [创建 Node.js 应用程序并将其部署到 Azure 网站]: /documentation/articles/web-sites-nodejs-develop-deploy-mac
  [Node.js 云服务]: /documentation/articles/cloud-services-nodejs-develop-deploy-app
  [使用 WebMatrix 生成网站]: /documentation/articles/web-sites-nodejs-use-webmatrix
  [实施 GCM 服务器]: http://developer.android.com/google/gcm/server.html#payload
  [本地和推送通知编程指南]: http://developer.apple.com/library/ios/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/ApplePushService/ApplePushService.html
  [从服务器推送通知]: http://msdn.microsoft.com/library/hh221551.aspx
  [推送通知服务请求和响应标头]: http://msdn.microsoft.com/library/windows/apps/hh465435.aspx
  [Azure SDK for Node]: https://github.com/WindowsAzure/azure-sdk-for-node
