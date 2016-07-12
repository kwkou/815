<properties 
	pageTitle="如何结合使用通知中心与 Java" 
	description="了解如何从 Java 后端使用 Azure 通知中心。" 
	services="notification-hubs" 
	documentationCenter="" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor=""/>

<tags    	
	ms.service="notification-hubs" 
	ms.date="03/28/2016" 
	wacn.date="05/09/2016"/>

# 如何通过 Java 使用通知中心
[AZURE.INCLUDE [notification-hubs-backend-how-to-selector](../includes/notification-hubs-backend-how-to-selector.md)]
		
本主题将向你介绍完全受支持的全新官方 Azure 通知中心 Java SDK 的主要功能。 
这是一个开源项目，你可以在 [Java SDK] 查看完整的 SDK 代码。 

通常情况下，如 MSDN 主题[通知中心 REST API](http://msdn.microsoft.com/zh-cn/library/dn223264.aspx) 中所述，你可以使用通知中心 REST 接口从 Java/PHP/Python/Ruby 后端访问所有通知中心功能。此 Java SDK 在以 Java 形式表示的 REST 接口上提供瘦包装器。

SDK 当前支持：

- 通知中心上的 CRUD 
- 注册上的 CRUD
- 安装管理
- 导入/导出注册
- 常规发送
- 计划发送
- 通过 Java NIO 的异步操作
- 支持的平台：APNS (iOS)、GCM (Android)、WNS（Windows 应用商店应用）、MPNS (Windows Phone)、ADM (Amazon Kindle Fire)、百度（没有 Google 服务的 Android） 

## SDK 用法

### 编译和生成

使用 [Maven]

生成：

	mvn package

## 代码

### 通知中心 CRUD

**创建命名空间管理器：**
	
	NamespaceManager namespaceManager = new NamespaceManager("connection string")

**创建通知中心：**
	
	NotificationHubDescription hub = new NotificationHubDescription("hubname");
	hub.setWindowsCredential(new WindowsCredential("sid","key"));
	hub = namespaceManager.createNotificationHub(hub);
	
 或者

	hub = new NotificationHub("connection string", "hubname");

**获取通知中心：**
	
	hub = namespaceManager.getNotificationHub("hubname");

**更新通知中心：**
	
	hub.setMpnsCredential(new MpnsCredential("mpnscert", "mpnskey"));
	hub = namespaceManager.updateNotificationHub(hub);

**删除通知中心：**
	
	namespaceManager.deleteNotificationHub("hubname");

### 注册 CRUD
**创建通知中心客户端：**

	hub = new NotificationHub("connection string", "hubname");

**创建 Windows 注册：**

	WindowsRegistration reg = new WindowsRegistration(new URI(CHANNELURI));
	reg.getTags().add("myTag");
	reg.getTags().add("myOtherTag");    
	hub.createRegistration(reg);

**创建 iOS 注册：**

	AppleRegistration reg = new AppleRegistration(DEVICETOKEN);
	reg.getTags().add("myTag");
	reg.getTags().add("myOtherTag");
	hub.createRegistration(reg);

同样，可以针对 Android (GCM)、Windows Phone (MPNS) 和 Kindle Fire (ADM) 创建注册。

**创建模板注册：**

	WindowsTemplateRegistration reg = new WindowsTemplateRegistration(new URI(CHANNELURI), WNSBODYTEMPLATE);
	reg.getHeaders().put("X-WNS-Type", "wns/toast");
	hub.createRegistration(reg);

**使用“创建 registrationid + upsert 模式”创建注册**

如果在设备上存储注册 ID，请删除重复项以防出现任何响应丢失：

	String id = hub.createRegistrationId();
	WindowsRegistration reg = new WindowsRegistration(id, new URI(CHANNELURI));
	hub.upsertRegistration(reg);

**更新注册：**
	
	hub.updateRegistration(reg);

**删除注册：**
	
	hub.deleteRegistration(regid);

**查询注册：**

* 	**获取单个注册：**
	
		hub.getRegistration(regid);
	
* 	**获取中心的所有注册：**
	
		hub.getRegistrations();
	
* 	**获取具有标记的注册：**
	
		hub.getRegistrationsByTag("myTag");
	
* 	**按渠道获取注册：**
	
		hub.getRegistrationsByChannel("devicetoken");

所有集合查询都支持 $top 和继续标记。

### 安装 API 用法
安装 API 是一种注册管理的替代机制。其现在可以使用“单个”安装对象，而不是维护着多个注册，后者不但工作量较大，而且容易出错且效率低下。安装包含你所需的一切：推送通道（设备标记）、标记、模板、辅助磁贴（用于 WNS 和 APNS）。你不必再调用该服务以获取 ID - 只需生成 GUID 或任何其他标识符，将其保存在设备上并与推送通道（设备标记）一起发送到你的后端即可。在后端，你应当只做一个调用：CreateOrUpdateInstallation，其完全是幂等的，因此，如果需要，可随时重试。

针对 Amazon Kindle Fire，示例如下：

	Installation installation = new Installation("installation-id", NotificationPlatform.Adm, "adm-push-channel");
	hub.createOrUpdateInstallation(installation);

如果你希望进行更新：

	installation.addTag("foo");
	installation.addTemplate("template1", new InstallationTemplate("{"data":{"key1":"$(value1)"}}","tag-for-template1"));
	installation.addTemplate("template2", new InstallationTemplate("{"data":{"key2":"$(value2)"}}","tag-for-template2"));
	hub.createOrUpdateInstallation(installation);

对于高级方案，我们提供有部分更新功能，以允许仅修改安装对象的特定属性。基本上，部分更新是你针对安装对象运行 JSON Patch 操作的子集。

	PartialUpdateOperation addChannel = new PartialUpdateOperation(UpdateOperationType.Add, "/pushChannel", "adm-push-channel2");
	PartialUpdateOperation addTag = new PartialUpdateOperation(UpdateOperationType.Add, "/tags", "bar");
	PartialUpdateOperation replaceTemplate = new PartialUpdateOperation(UpdateOperationType.Replace, "/templates/template1", new InstallationTemplate("{"data":{"key3":"$(value3)"}}","tag-for-template1")).toJson());
	hub.patchInstallation("installation-id", addChannel, addTag, replaceTemplate);

删除安装：

	hub.deleteInstallation(installation.getInstallationId());

CreateOrUpdate、Patch 和 Delete 最终与 Get 一致。你请求的操作会在调用期间进入系统队列并在后台执行。请注意，Get 并不适用于主运行时方案，只适用于调试和故障排除，其会受到服务的严密限制。

安装的发送流与注册的一样。我们只是引入了一个选项以将通知锁定至特定安装 - 仅使用了标记 "InstallationId:{desired-id}"。对于上述情况，其如下所示：

	Notification n = Notification.createWindowsNotification("WNS body");
	hub.sendNotification(n, "InstallationId:{installation-id}");

为多个模板之一：

	Map<String, String> prop =  new HashMap<String, String>();
	prop.put("value3", "some value");
	Notification n = Notification.createTemplateNotification(prop);
	hub.sendNotification(n, "InstallationId:{installation-id} && tag-for-template1");

### 计划通知（适用于标准层）

与常规发送相同，但多了一个参数 - scheduledTime，表示通知应传递的时间。服务接受现在 + 5 分钟与现在 + 7 天之间的任何时间点。

**计划 Windows 本机通知：**

	Calendar c = Calendar.getInstance();
	c.add(Calendar.DATE, 1);    
	Notification n = Notification.createWindowsNotification("WNS body");
	hub.scheduleNotification(n, c.getTime());

### 导入/导出（可用于标准层）
有时需要针对注册执行批量操作。通常这是为了与另一个系统集成，或只是一个大规模修复以更新标记。如果涉及到数以千计的注册，强烈不建议使用 Get/Update 流。导入/导出功能专门针对以下方案设计。基本上，你会在存储帐户下提供对一些 BLOB 容器的访问权限作为传入数据的源和输出的位置。

**提交导出作业：**

	NotificationHubJob job = new NotificationHubJob();
	job.setJobType(NotificationHubJobType.ExportRegistrations);
	job.setOutputContainerUri("container uri with SAS signature");
	job = hub.submitNotificationHubJob(job);


**提交导入作业：**

	NotificationHubJob job = new NotificationHubJob();
	job.setJobType(NotificationHubJobType.ImportCreateRegistrations);
	job.setImportFileUri("input file uri with SAS signature");
	job.setOutputContainerUri("container uri with SAS signature");
	job = hub.submitNotificationHubJob(job);

**等待直到作业完成：**

	while(true){
	    Thread.sleep(1000);
	    job = hub.getNotificationHubJob(job.getJobId());
	    if(job.getJobStatus() == NotificationHubJobStatus.Completed)
	        break;
	}       

**获取所有作业：**

	List<NotificationHubJob> jobs = hub.getAllNotificationHubJobs();

**使用 SAS 签名的 URI：**
这是某些 BLOB 文件或 BLOB 容器的 URL，加上一组参数（例如权限和到期日期），再加上使用帐户的 SAS 密钥生成的所有这些内容的签名。Azure 存储 Java SDK 具有丰富的功能，包括创建这种类型的 URI。作为简单的替代，你可以考虑使用 ImportExportE2E 测试类（来自 github 位置），其具有非常基本、精简的签名算法。

###发送通知
通知对象只有带有标头的正文，一些实用工具方法可帮助你构建本机和模板通知对象。

* **Windows 应用商店和 Windows Phone 8.1（非 Silverlight）**

		String toast = "<toast><visual><binding template="ToastText01"><text id="1">Hello from Java!</text></binding></visual></toast>";
		Notification n = Notification.createWindowsNotification(toast);
		hub.sendNotification(n);

* **iOS**

		String alert = "{"aps":{"alert":"Hello from Java!"}}";
		Notification n = Notification.createAppleNotification(alert);
		hub.sendNotification(n);

* **Android**

		String message = "{"data":{"msg":"Hello from Java!"}}";
		Notification n = Notification.createGcmNotification(message);
		hub.sendNotification(n);

* **Windows Phone 8.0 和 8.1 Silverlight**

		String toast = "<?xml version="1.0" encoding="utf-8"?>" +
			        "<wp:Notification xmlns:wp="WPNotification">" +
			           "<wp:Toast>" +
			                "<wp:Text1>Hello from Java!</wp:Text1>" +
			           "</wp:Toast> " +
			        "</wp:Notification>";
		Notification n = Notification.createMpnsNotification(toast);
		hub.sendNotification(n);

* **Kindle Fire**

		String message = "{"data":{"msg":"Hello from Java!"}}";
		Notification n = Notification.createAdmNotification(message);
		hub.sendNotification(n);

* **发送到标记**

		Set<String> tags = new HashSet<String>();
		tags.add("boo");
		tags.add("foo");
		hub.sendNotification(n, tags);

* **发送到标记表达式**

		hub.sendNotification(n, "foo && ! bar");

* **发送模板通知**

		Map<String, String> prop =  new HashMap<String, String>();
		prop.put("prop1", "v1");
		prop.put("prop2", "v2");
		Notification n = Notification.createTemplateNotification(prop);
		hub.sendNotification(n);

运行 Java 代码，现在应该生成显示在目标设备上的通知。


##<a name="next-steps"></a>后续步骤
在本主题中，我们介绍了如何为通知中心创建简单的 Java REST 客户端。从这里你可以：

* 下载完整的 [Java SDK]，其中包含整个 SDK 代码。 
* 播放示例：
	- [通知中心入门]
	- [发送突发新闻]
	- [发送当地的突发新闻]
	- [发送通知到经身份验证的用户]
	- [发送跨平台通知到经身份验证的用户]

[Java SDK]: https://github.com/Azure/azure-notificationhubs-java-backend
[Get started tutorial]: /documentation/articles/notification-hubs-ios-get-started/
[通知中心入门]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[发送突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/
[发送当地的突发新闻]: /documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news/
[发送通知到经身份验证的用户]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/
[发送跨平台通知到经身份验证的用户]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-push-notifications-app-users/
[Maven]: http://maven.apache.org/
 
<!---HONumber=Mooncake_0503_2016-->