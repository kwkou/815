<properties
	pageTitle="Azure 通知中心入门（Chrome 应用）| Windows Azure"
	description="在本教程中，你将了解如何使用 Azure 通知中心将通知推送到 Chrome 应用。"
	services="notification-hubs"
	documentationCenter=""
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>
<tags 
    ms.service="notification-hubs" 
    ms.date="09/03/2015"
    wacn.date="11/02/2015" />

# 通知中心入门

[AZURE.INCLUDE [notification-hubs-selector-get-started](../includes/notification-hubs-selector-get-started.md)]

本主题演示如何使用 Azure 通知中心将推送通知发送到 Chrome 应用。

使用 Chrome 应用通知的其中一个主要优势在于通知显示在 Google Chrome 浏览器的上下文中，你无需运行 Chrome 应用或在浏览器中打开（尽管必须运行 Chrome 浏览器本身）。此外，你还可以在 Chrome 通知窗口中获得所有通知的合并视图。

>[AZURE.NOTE]这不是泛型浏览器内推送通知，这是针对 Chrome 应用的通知。有关详细信息，请参阅 [Chrome 应用概述]。Chrome 应用以前被称为“封装应用”，与较为简单的“托管应用”不同。请参阅[可安装的 Web Apps]，了解不同之处。Chrome 应用还可以在使用 Apache Cordova 的移动设备（Android 和 iOS）上运行。请参阅[移动设备上的 Chrome 应用]，了解详细信息。

在本教程中，我们将创建一个空白 Chrome 应用，它使用 Google Cloud Messaging (GCM) 接收推送通知。完成之后，你将可以向安装此 Chrome 应用的所有 Chrome 用户广播推送通知。

本教程将指导你完成启用推送通知的以下基本步骤：

* [启用 Google Cloud Messaging (GCM)](#register)
* [配置通知中心](#configure-hub)
* [将你的 Chrome 应用连接到通知中心](#connect-app)
* [向你的 Chrome 应用发送通知](#send)
* [后续步骤](#next-steps)

本教程演示使用通知中心的简单广播方案。配置 GCM 和 Azure 通知中心类似于为 Android 进行配置，因为已弃用 [Google Cloud Messaging for Chrome]，而且相同的 GCM 现在支持 Android 设备和 Chrome 实例。

务必遵循后续步骤中的教程以了解如何使用通知中心来通知特定用户和设备组。

>[AZURE.NOTE]若要完成本教程，你必须有一个有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](http://www.windowsazure.cn/pricing/1rmb-trial/)。

##<a id="register"></a>启用 Google Cloud Messaging (GCM)

1. 导航到 [Google Cloud Console] 网站，使用你的 Google 帐户凭据登录，然后单击“创建项目”按钮。提供相应的**项目名称**，然后单击“创建”按钮。

   	![][1]

2. 在你刚才创建的项目的“项目”页面上记录**项目编号**。你将使用此编号作为 Chrome 应用中的 **GCM 发送器 ID**，以在 GCM 中进行注册。

   	![][2]

3. 在左侧窗格中，单击“API 和身份验证”，然后向下滚动并单击切换键以启用 **Google Cloud Messaging for Android**。你无需启用 *Google Cloud Messaging for Chrome*。有可能该名称在以后直接更改为 *Google Cloud Messaging*。

   	![][3]

4. 在左侧窗格中，单击“凭据”->“新建密钥”->“服务器密钥”->“创建”

   	![][4]

5. 记下服务器 **API 密钥**。你将在通知中心中配置此密钥，然后进行启用以将推送通知发送到 GCM。

   	![][5]

##<a id="configure-hub"></a>配置通知中心

1. 登录到 **[Azure 管理门户]**，然后单击屏幕左下角的“+新建”。

2. 单击“应用程序服务”->“服务总线”->“通知中心”->“快速创建”。键入通知中心的名称，选择所需的区域，然后单击“创建新的通知中心”。

   	![][6]

4. 转到你刚刚创建的通知中心。单击托管你的通知中心的命名空间（通常为***通知中心名称*-ns**）。

   	![][7]

5. 在顶部单击“通知中心”选项卡。

   	![][8]

6. 现在，单击顶部的“配置”选项卡。

   	![][9]

7. 在“配置”选项卡上，向下滚动到“Google Cloud Messaging 设置”，输入在上一部分中获取的“API 密钥”值，然后单击“保存”。

   	![][10]

8. 选择顶部的“仪表板”选项卡，然后单击底部的“连接信息”。

   	![][11]

9. 记下 **DefaultListenSharedAccessSignature**（在 Chrome 应用上注册通知中心时需要）和 **DefaultFullSharedAccessSignature**（发送通知时需要）

   	![][12]

你的通知中心现在已配置为使用 GCM，并且你有必要的连接字符串以进行进一步配置。

##<a id="connect-app"></a>将你的 Chrome 应用连接到通知中心

###新建 Chrome 应用
下文示例以 [Chrome App GCM 示例]为基础，并且使用推荐的方式来创建 Chrome 应用。在下文部分中，我们将重点突出 Azure 通知中心相关的步骤。我们建议你从 [Chrome 应用通知中心示例]中下载此 Chrome 应用的源。

你可以使用 JavaScript 创建 Chrome 应用并且可以使用首选的文字编辑器来进行创建。

1. 下文是此 Chrome 应用的大致外观。

   	![][15]

2. 创建一个文件夹并将其命名为 **ChromePushApp**。你可以对其进行任意命名。

3. 从此文件夹中的 *cryto-js 库*中下载 [crypto-js 库]。此库文件夹包含两个子文件夹：*组件*和*汇总*。

4. 创建一个 manifest.json 文件。所有 Chrome 应用都由一个清单文件提供支持，该清单文件描述应用元数据，尤其是适用于应用的特定权限的清单文件。

		{
		  "name": "NH-GCM Notifications",
		  "description": "Chrome platform app.",
		  "manifest_version": 2,
		  "version": "0.1",
		  "app": {
		    "background": {
		      "scripts": ["background.js"]
		    }
		  },
		  "permissions": ["gcm", "storage", "notifications", "https://*.servicebus.chinacloudapi.cn/*"],
		  "icons": { "128": "gcm_128.png" }
		}

	请注意，*permissions* 元素指定此 Chrome 应用可以从 GCM 中接收推送通知。此外，它还必须指定 Azure 通知中心 URI，其中 Chrome 应用进行 REST 调用以进行注册。这将使用图标文件 gcm\_128.png，该文件可在原始 GCM 示例中重复使用的源中找到。你可以使用任何你想要的图像。

5. 使用以下代码创建名为 background.js 的文件：

		// Returns a new notification ID used in the notification.
		function getNotificationId() {
		  var id = Math.floor(Math.random() * 9007199254740992) + 1;
		  return id.toString();
		}

		function messageReceived(message) {
		  // A message is an object with a data property that
		  // consists of key-value pairs.

		  // Concatenate all key-value pairs to form a display string.
		  var messageString = "";
		  for (var key in message.data) {
		    if (messageString != "")
		      messageString += ", "
		    messageString += key + ":" + message.data[key];
		  }
		  console.log("Message received: " + messageString);

		  // Pop up a notification to show the GCM message.
		  chrome.notifications.create(getNotificationId(), {
		    title: 'GCM Message',
		    iconUrl: 'gcm_128.png',
		    type: 'basic',
		    message: messageString
		  }, function() {});
		}

		var registerWindowCreated = false;

		function firstTimeRegistration() {
		  chrome.storage.local.get("registered", function(result) {

		    registerWindowCreated = true;
		    chrome.app.window.create(
		      "register.html",
		      {  width: 520,
		         height: 500,
		         frame: 'chrome'
		      },
		      function(appWin) {}
		    );
		  });
		}

		// Set up a listener for GCM message event.
		chrome.gcm.onMessage.addListener(messageReceived);

		// Set up listeners to trigger the first time registration.
		chrome.runtime.onInstalled.addListener(firstTimeRegistration);
		chrome.runtime.onStartup.addListener(firstTimeRegistration);

	这就是弹出 Chrome 应用窗口 html (*register.html*) 的文件，此文件同时也定义了处理程序 *messageReceived*，以处理即将传入的推送通知。

6. 创建名为 *register.html* 的文件，其定义了 Chrome 应用的 UI。请注意，此示例使用 *CryptoJS v3.1.2*。如果你已下载其他任何版本，则修改该脚本 src 路径。

		<html>

		<head>
		<title>GCM Registration</title>
		<script src="register.js"></script>
		<script src="CryptoJS v3.1.2/rollups/hmac-sha256.js"></script>
		<script src="CryptoJS v3.1.2/components/enc-base64-min.js"></script>
		</head>

		<body>

		Sender ID:<br/><input id="senderId" type="TEXT" size="20"><br/>
		<button id="registerWithGCM">Register with GCM</button>
		<br/>
		<br/>
		<br/>

		Notification Hub Name:<br/><input id="hubName" type="TEXT" style="width:400px"><br/><br/>
		Connection String:<br/><textarea id="connectionString" type="TEXT" style="width:400px;height:60px"></textarea>

		<br/>

		<button id="registerWithNH" disabled="true">Register with Azure Notification Hubs</button>

		<br/>
		<br/>

		<textarea id="console" type="TEXT" readonly style="width:500px;height:200px;background-color:#e5e5e5;padding:5px"></textarea>

		</body>

		</html>

7. 使用以下代码创建名为 *register.js* 的文件。此文件指定 *register.html* 背后的脚本。Chrome 应用不允许内联执行，因此需要为你的 UI 创建单独的备份脚本。

		var registrationId = "";
		var hubName        = "", connectionString = "";
		var originalUri    = "", targetUri = "", endpoint = "", sasKeyName = "", sasKeyValue = "", sasToken = "";

		window.onload = function() {
		   document.getElementById("registerWithGCM").onclick = registerWithGCM;  
		   document.getElementById("registerWithNH").onclick = registerWithNH;
		   updateLog("You have not registered yet. Please provider sender ID and register with GCM and then with  Notification Hubs.");
		}

		function updateLog(status) {
		  currentStatus = document.getElementById("console").innerHTML;
		  if (currentStatus != "") {
		    currentStatus = currentStatus + "\n\n";
		  }

		  document.getElementById("console").innerHTML = currentStatus  + status;
		}

		function registerWithGCM() {
		  var senderId = document.getElementById("senderId").value.trim();
		  chrome.gcm.register([senderId], registerCallback);

		  // Prevent register button from being clicked again before the registration finishes
		  document.getElementById("registerWithGCM").disabled = true;
		}

		function registerCallback(regId) {
		  registrationId = regId;
		  document.getElementById("registerWithGCM").disabled = false;

		  if (chrome.runtime.lastError) {
		    // When the registration fails, handle the error and retry the
		    // registration later.
		    updateLog("Registration failed: " + chrome.runtime.lastError.message);
		    return;
		  }

		  updateLog("Registration with GCM succeeded.");
		  document.getElementById("registerWithNH").disabled = false;

		  // Mark that the first-time registration is done.
		  chrome.storage.local.set({registered: true});
		}

		function registerWithNH() {
		  hubName = document.getElementById("hubName").value.trim();
		  connectionString = document.getElementById("connectionString").value.trim();
		  splitConnectionString();
		  generateSaSToken();
		  sendNHRegistrationRequest();
		}

		// From http://msdn.microsoft.com/zh-cn/library/dn495627.aspx
		function splitConnectionString()
		{
		  var parts = connectionString.split(';');
		  if (parts.length != 3)
		  throw "Error parsing connection string";

		  parts.forEach(function(part) {
		    if (part.indexOf('Endpoint') == 0) {
		    endpoint = 'https' + part.substring(11);
		    } else if (part.indexOf('SharedAccessKeyName') == 0) {
		    sasKeyName = part.substring(20);
		    } else if (part.indexOf('SharedAccessKey') == 0) {
		    sasKeyValue = part.substring(16);
		    }
		  });

		  originalUri = endpoint + hubName;
		}

		function generateSaSToken()
		{
		  targetUri = encodeURIComponent(originalUri.toLowerCase()).toLowerCase();
		  var expiresInMins = 10; // 10 minute expiration

		  // Set expiration in seconds
		  var expireOnDate = new Date();
		  expireOnDate.setMinutes(expireOnDate.getMinutes() + expiresInMins);
		  var expires = Date.UTC(expireOnDate.getUTCFullYear(), expireOnDate
		    .getUTCMonth(), expireOnDate.getUTCDate(), expireOnDate
		    .getUTCHours(), expireOnDate.getUTCMinutes(), expireOnDate
		    .getUTCSeconds()) / 1000;
		  var tosign = targetUri + '\n' + expires;

		  // using CryptoJS
		  var signature = CryptoJS.HmacSHA256(tosign, sasKeyValue);
		  var base64signature = signature.toString(CryptoJS.enc.Base64);
		  var base64UriEncoded = encodeURIComponent(base64signature);

		  // construct authorization string
		  sasToken = "SharedAccessSignature sr=" + targetUri + "&sig="
		                  + base64UriEncoded + "&se=" + expires + "&skn=" + sasKeyName;
		}

		function sendNHRegistrationRequest()
		{
		  var registrationPayload =
		  "<?xml version="1.0" encoding="utf-8"?>" +
		  "<entry xmlns="http://www.w3.org/2005/Atom">" +
		      "<content type="application/xml">" +
		          "<GcmRegistrationDescription xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/netservices/2010/10/servicebus/connect">" +
		              "<GcmRegistrationId>{GCMRegistrationId}</GcmRegistrationId>" +
		          "</GcmRegistrationDescription>" +
		      "</content>" +
		  "</entry>";

		  // Update the payload with the registration id obtained earlier
		  registrationPayload = registrationPayload.replace("{GCMRegistrationId}", registrationId);

		  var url = originalUri + "/registrations/?api-version=2014-09";
		  var client = new XMLHttpRequest();

		  client.onload = function () {
		    if (client.readyState == 4) {
		      if (client.status == 200) {
		        updateLog("Notification Hub Registration succesful!");
		        updateLog(client.responseText);
		      } else {
		        updateLog("Notification Hub Registration did not succeed!");
		        updateLog("HTTP Status: " + client.status + " : " + client.statusText);
		        updateLog("HTTP Response: " + "\n" + client.responseText);
		      }
		    }
		  };

		  client.onerror = function () {
		        updateLog("ERROR - Notification Hub Registration did not succeed!");
		  }

		  client.open("POST", url, true);
		  client.setRequestHeader("Content-Type", "application/atom+xml;type=entry;charset=utf-8");
		  client.setRequestHeader("Authorization", sasToken);
		  client.setRequestHeader("x-ms-version", "2014-09");

		  try {
		      client.send(registrationPayload);
		  }
		  catch(err) {
		      updateLog(err.message);
		  }
		}

	上述脚本具有以下特点：
	- *window.onload* 定义了 UI 上的两个按钮的按钮单击事件。一个按钮在 GCM 中注册，另一个按钮使用在 GCM 中注册后返回的注册 ID 在 Azure 通知中心中注册。
	- *updateLog* 函数定义简单的记录函数。
	- *registerWithGCM* 是第一个按钮单击处理程序，它对 GCM 进行 **chrome.gcm.register** 调用以注册此 Chrome 应用实例。
	- *registerCallback* 是回调函数，在上述 GCM 注册调用返回时获得调用。
	- *registerWithNH* 是第二个按钮单击处理程序，可注册到通知中心。它获取用户指定的 **hubName** 和 **connectionString** 并创建通知中心注册 REST API 调用。
	- *splitConnectionString* 和 *generateSaSToken* 是创建 SaS 令牌（必须在所有 REST API 调用中发送）的 JavaScript 实现。有关详细信息，请参阅[基本概念](http://msdn.microsoft.com/library/dn495627.aspx)。
	- *sendNHRegistrationRequest* 函数进行 HTTP REST 调用。
	- *registrationPayload* 定义注册 XML 负载。有关详细信息，请参阅[创建注册 NH REST API]。我们使用从 GCM 收到的信息更新其中的注册 ID。
	- *client* 是 **XMLHttpRequest** 的实例，我们使用它来发出 HTTP POST 请求。请注意，我们使用 **sasToken** 更新 **Authorization** 标头。成功完成此次调用将在 Azure 通知中心中注册此 Chrome 应用实例。


8. 你应在此实例的末尾看到文件夹的以下视图：![][21]

###设置并测试你的 Chrome 应用

1. 打开 Chrome 浏览器。打开“Chrome 扩展”并启用“开发人员模式”。

   	![][16]

2. 单击“加载解包扩展”，并导航到你创建文件的文件夹中。你还可以选择使用 **Chrome 应用和扩展开发人员工具**，它本身是 Chrome 应用（必须从 Chrome 网上应用店中进行安装）并为你的 Chrome 应用开发提供高级调试功能。

   	![][17]

3. 如果创建的 Chrome 应用没有任何错误，那么你将看到 Chrome 应用显示出来。

   	![][18]

4. 输入你先前从 **Google Cloud Console** 中获取的**项目编号**作为发送器 ID，然后单击“注册到 GCM”。你肯定会看到“已成功注册到 GCM”的消息。

   	![][19]

5. 输入你先前从 Azure 管理门户中获取的**通知中心名称**和 **DefaultListenSharedAccessSignature**，并单击“注册到 Azure 通知中心”。你肯定可以看到“通知中心注册成功!”的消息以及包含 Azure 通知中心注册 ID 的注册响应的详细信息。

   	![][20]

##<a name="send"></a>向你的 Chrome 应用发送通知

在本教程中，你使用 .NET 控制台应用程序发送通知，虽然你也可以使用通知中心从任何使用 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/dn223264.aspx">REST 接口</a>的后端发送通知，。

有关如何从与通知中心集成的 Azure 移动服务后端发送通知的示例，请参阅**移动服务中的推送通知入门**（[.NET 后端](/documentation/articles/mobile-services-javascript-backend-android-get-started-push) | [JavaScript 后端](/documentation/articles/mobile-services-javascript-backend-android-get-started-push)）。有关如何使用 REST API 发送通知的示例，请参阅**如何通过 Java/PHP/Python 使用通知中心** ([Java](/documentation/articles/notification-hubs-java-backend-how-to) | [PHP](/documentation/articles/notification-hubs-php-backend-how-to) | [Python](/documentation/articles/notification-hubs-python-backend-how-to))。

1. 在 Visual Studio 中，从“文件”菜单中依次选择“新建”和“项目...”，然后在 **Visual C#** 下依次单击 **Windows**、“控制台应用程序”和“确定”。这将创建一个新的控制台应用程序项目。

2. 在“工具”菜单中，单击“库包管理器”，然后单击“包管理器控制台”。这会显示包管理器控制台。

3. 在控制台窗口中，执行以下命令：

        Install-Package WindowsAzure.ServiceBus

   	这将使用 <a href="http://nuget.org/packages/  WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet 包</a>添加对 Azure 服务总线 SDK 的引用。

4. 打开文件 Program.cs 并添加以下 `using` 语句：

        using Microsoft.ServiceBus.Notifications;

5. 在 **Program** 类中，添加以下方法：

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            String message = "{"data":{"message":"Hello Chrome from Azure Notification Hubs"}}";
            await hub.SendGcmNativeNotificationAsync(message);
        }

   	确保将“中心名称”占位符替换为在门户的“通知中心”选项卡中显示的通知中心名称。此外，使用你在“配置通知中心”部分中获取的名称为 **DefaultFullSharedAccessSignature** 的连接字符串替换连接字符串占位符。

	>[AZURE.NOTE]确保你使用的是具有**完全**访问权限的连接字符串，而不是具有**监听**访问权限的连接字符串。监听访问字符串无权发送通知。

5. 在 **Main** 方法中添加下列行：

         SendNotificationAsync();
		 Console.ReadLine();

6. 请确保你的 Chrome 浏览器处于打开状态。你的 Chrome 应用无需为此打开。你应该查看桌面上的以下通知弹出窗口。

   	![][13]

7. 在 Chrome 运行时，还可以使用 Chrome 通知窗口（可从 Windows 上的任务栏进行访问）来查看你的所有通知。

   	![][14]

## <a name="next-steps"></a>后续步骤

在这个简单的示例中，你已将通知广播到所有 Chrome 应用。
在[通知中心概述]中，了解有关通知中心的详细信息。
若要针对特定客户，请参阅教程 [Azure 通知中心 - 通知用户]。如果要按兴趣组来划分用户，可以阅读 [Azure 通知中心突发新闻]。

<!-- Images. -->
[1]: ./media/notification-hubs-chrome-get-started/GoogleConsoleCreateProject.PNG
[2]: ./media/notification-hubs-chrome-get-started/GoogleProjectNumber.png
[3]: ./media/notification-hubs-chrome-get-started/EnableGCM.png
[4]: ./media/notification-hubs-chrome-get-started/CreateServerKey.png
[5]: ./media/notification-hubs-chrome-get-started/ServerKey.png
[6]: ./media/notification-hubs-chrome-get-started/CreateNH.png
[7]: ./media/notification-hubs-chrome-get-started/NHNamespace.png
[8]: ./media/notification-hubs-chrome-get-started/NamespaceConfigure.png
[9]: ./media/notification-hubs-chrome-get-started/NHConfigure.png
[10]: ./media/notification-hubs-chrome-get-started/NHConfigureGCM.png
[11]: ./media/notification-hubs-chrome-get-started/NHDashboard.png
[12]: ./media/notification-hubs-chrome-get-started/NHConnString.png
[13]: ./media/notification-hubs-chrome-get-started/ChromeNotification.png
[14]: ./media/notification-hubs-chrome-get-started/ChromeNotificationWindow.png
[15]: ./media/notification-hubs-chrome-get-started/ChromeApp.png
[16]: ./media/notification-hubs-chrome-get-started/ChromeExtensions.png
[17]: ./media/notification-hubs-chrome-get-started/ChromeLoadExtension.png
[18]: ./media/notification-hubs-chrome-get-started/ChromeAppLoaded.png
[19]: ./media/notification-hubs-chrome-get-started/ChromeAppGCM.png
[20]: ./media/notification-hubs-chrome-get-started/ChromeAppNH.png
[21]: ./media/notification-hubs-chrome-get-started/FinalFolderView.png

<!-- URLs. -->
[Chrome 应用通知中心示例]: http://google.com
[Google Cloud Console]: http://cloud.google.com/console
[Azure 管理门户]: https://manage.windowsazure.cn/
[通知中心概述]: http://msdn.microsoft.com/zh-cn/library/jj927170.aspx
[Chrome 应用概述]: https://developer.chrome.com/apps/about_apps
[Chrome App GCM 示例]: https://github.com/GoogleChrome/chrome-app-samples/tree/master/samples/gcm-notifications
[可安装的 Web Apps]: https://developers.google.com/chrome/apps/docs/
[移动设备上的 Chrome 应用]: https://developer.chrome.com/apps/chrome_apps_on_mobile
[创建注册 NH REST API]: http://msdn.microsoft.com/zh-cn/library/azure/dn223265.aspx
[crypto-js 库]: http://code.google.com/p/crypto-js/
[GCM with Chrome Apps]: https://developer.chrome.com/apps/cloudMessaging
[Google Cloud Messaging for Chrome]: https://developer.chrome.com/apps/cloudMessagingV1
[Azure 通知中心 - 通知用户]: /documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users
[Azure 通知中心突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news
 

<!---HONumber=76-->