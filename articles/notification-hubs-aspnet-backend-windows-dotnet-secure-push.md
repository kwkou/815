<properties 
	pageTitle="Azure 通知中心安全推送" 
	description="了解如何在 Azure 中发送安全推送通知。代码示例是使用 .NET API 通过 C# 编写的。" 
	documentationCenter="windows" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="" 
	services="notification-hubs"/>

<tags 
	ms.service="notification-hubs" 
	ms.date="02/29/2016"
	wacn.date="05/18/2016"/>

# Azure 通知中心安全推送

> [AZURE.SELECTOR]
- [Windows Universal](/documentation/articles/notification-hubs-windows-dotnet-secure-push/)
- [iOS](/documentation/articles/notification-hubs-aspnet-backend-ios-secure-push/)


## 概述

利用 Microsoft Azure 中的推送通知支持，你可以访问易于使用且横向扩展的多平台推送基础结构，这大大简化了为移动平台的使用者应用程序和企业应用程序实现推送通知的过程。

由于法规或安全约束，有时应用程序可能想要在通知中包含某些无法通过标准推送通知基础结构传输的内容。本教程介绍如何通过客户端设备和应用后端之间安全且经过验证的连接发送敏感信息，以便获得相同的体验。

在高级别中，此流程如下所示：

1. 应用后端：
	- 在后端数据库中存储安全有效负载。
	- 将此通知的 ID 发送到此设备（不发送任何安全信息）。
2. 此设备上的应用在接收通知时：
	- 此设备将联系请求安全有效负载的后端。
	- 此应用可以将有效负载显示为设备上的通知。

请务必注意，在之前的流程（以及本教程中）中，我们假设此设备会在用户登录后在本地存储中存储身份验证令牌。这可以保证完全无缝的体验，因为该设备可以使用此令牌检索通知的安全有效负载。如果您的应用程序未在设备上存储身份验证令牌，或者如果这些令牌可能已过期，此设备应用在收到通知时应显示提示用户启动应用的通用通知。然后，应用对用户进行身份验证并显示通知有效负载。

本安全推送教程演示如何安全地发送推送通知。本教程以[通知用户](/documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users/)教程为基础，因此您应该先完成该教程中的步骤。

> [AZURE.NOTE] 本教程假设您已根据[通知中心入门（Windows 应用商店）](/documentation/articles/notification-hubs-windows-store-dotnet-get-started/)中所述创建并配置了通知中心。
此外，请注意 Windows Phone 8.1 需要 Windows（而不是 Windows Phone）凭据，且后台任务无法在 Windows Phone 8.0 或 Silverlight 8.1 上正常运行。对于 Windows 应用商店应用程序，您只能在应用锁屏界面启用（单击 Appmanifest 中的复选框）的情况下，通过运行后台任务来接收通知。

[AZURE.INCLUDE [notification-hubs-aspnet-backend-securepush](../includes/notification-hubs-aspnet-backend-securepush.md)]

## 修改 Windows Phone 项目

1. 在 **NotifyUserWindowsPhone** 项目中，将以下代码添加到 App.xaml.cs 注册推送后台任务。在 `OnLaunched()` 方法的末尾添加以下代码行：

		RegisterBackgroundTask();

2. 仍在 App.xaml.cs 中，紧跟 `OnLaunched()` 方法添加以下代码：

		private async void RegisterBackgroundTask()
        {
            if (!Windows.ApplicationModel.Background.BackgroundTaskRegistration.AllTasks.Any(i => i.Value.Name == "PushBackgroundTask"))
            {
                var result = await BackgroundExecutionManager.RequestAccessAsync();
                var builder = new BackgroundTaskBuilder();

                builder.Name = "PushBackgroundTask";
                builder.TaskEntryPoint = typeof(PushBackgroundComponent.PushBackgroundTask).FullName;
                builder.SetTrigger(new Windows.ApplicationModel.Background.PushNotificationTrigger());
                BackgroundTaskRegistration task = builder.Register();
            }
        }

3. 在 App.xaml.cs 文件的顶部添加以下 `using` 语句：

		using Windows.Networking.PushNotifications;
		using Windows.ApplicationModel.Background;

4. 从 Visual Studio 的“文件”菜单中，单击“全部保存”。
		
## 创建推送背景组件

下一步是创建推送背景组件。

1. 在“解决方案资源管理器”中，右键单击解决方案的顶层节点（在本例中为 **Solution SecurePush**），然后依次单击“添加”和“新建项目”。

2. 展开“应用商店应用”，然后依次单击“Windows Phone 应用”和“Windows 运行时组件 (Windows Phone)”。将该项目命名为 **PushBackgroundComponent**，然后单击“确定”创建项目。

	![][12]

3. 在“解决方案资源管理器”中，右键单击 “PushBackgroundComponent (Windows Phone 8.1)” 项目，然后依次单击“添加”和“类”。将新类命名为 **PushBackgroundTask.cs**。单击“添加”生成类。

4. 将 **PushBackgroundComponent** 命名空间定义的整个内容替换为以下代码，将占位符 `{back-end endpoint}` 替换为部署后端时获取的后端终结点：

		public sealed class Notification
    		{
        		public int Id { get; set; }
        		public string Payload { get; set; }
        		public bool Read { get; set; }
    		}
    
		    public sealed class PushBackgroundTask : IBackgroundTask
    		{
        		private string GET_URL = "{back-end endpoint}/api/notifications/";
		
        		async void IBackgroundTask.Run(IBackgroundTaskInstance taskInstance)
		        {
        		    // Store the content received from the notification so it can be retrieved from the UI.
		            RawNotification raw = (RawNotification)taskInstance.TriggerDetails;
            		var notificationId = raw.Content;

            		// retrieve content
		            BackgroundTaskDeferral deferral = taskInstance.GetDeferral();
            		var httpClient = new HttpClient();
		            var settings = ApplicationData.Current.LocalSettings.Values;
		            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

		            var notificationString = await httpClient.GetStringAsync(GET_URL + notificationId);

            		var notification = JsonConvert.DeserializeObject<Notification>(notificationString);

		            ShowToast(notification);

		            deferral.Complete();
		        }

		        private void ShowToast(Notification notification)
		        {
		            ToastTemplateType toastTemplate = ToastTemplateType.ToastText01;
		            XmlDocument toastXml = ToastNotificationManager.GetTemplateContent(toastTemplate);
            		XmlNodeList toastTextElements = toastXml.GetElementsByTagName("text");
		            toastTextElements[0].AppendChild(toastXml.CreateTextNode(notification.Payload));
    	        	ToastNotification toast = new ToastNotification(toastXml);
		            ToastNotificationManager.CreateToastNotifier().Show(toast);
    		    }
    		}

5. 在“解决方案资源管理器”中，右键单击 “PushBackgroundComponent (Windows Phone 8.1)” 项目，然后单击“管理 NuGet 包”。

6. 在左侧单击“联机”。

7. 在“搜索”框中键入 **Http 客户端**。

8. 在结果列表中，单击“Microsoft HTTP 客户端库”，然后单击“安装”。完成安装。

9. 返回到 NuGet“搜索”框，键入 **Json.net**。安装 **Json.NET** 包，然后关闭“NuGet 包管理器”窗口。

10. 在 **PushBackgroundTask.cs** 文件的顶部添加以下 `using` 语句：

		using Windows.ApplicationModel.Background;
		using Windows.Networking.PushNotifications;
		using System.Net.Http;
		using Windows.Storage;
		using System.Net.Http.Headers;
		using Newtonsoft.Json;
		using Windows.UI.Notifications;
		using Windows.Data.Xml.Dom;

11. 在“解决方案资源管理器”的 **NotifyUserWindowsPhone (Windows Phone 8.1)** 项目中，右键单击“引用”，然后单击“添加引用...”。在“引用管理器”对话框中，选中 **PushBackgroundComponent** 旁边的复选框，然后单击“确定”。

12. 在“解决方案资源管理器”中，双击 **NotifyUserWindowsPhone (Windows Phone 8.1)** 项目中的“Package.appxmanifest”。在“通知”下，将“支持 Toast 通知”设置为“是”。

	![][3]

13. 仍在 **Package.appxmanifest** 中，单击顶部附近的“声明”菜单。在“可用声明”下拉列表中，单击“后台任务”，然后单击“添加”。
 
14. 在“属性”下的 **Package.appxmanifest** 中选中“推送通知”。

15. 在“应用设置”下的 **Package.appxmanifest** 中，在“入口点”字段中键入 **PushBackgroundComponent.PushBackgroundTask**。

	![][13]

16. 在“文件”菜单中，单击“全部保存”。

## 运行应用程序

若要运行应用程序，请执行以下操作：

1. 在 Visual Studio 中运行此 **AppBackend** Web API 应用程序。将显示 ASP.NET 网页。

2. 在 Visual Studio 中运行此 **NotifyUserWindowsPhone (Windows Phone 8.1)** Windows Phone 应用。Windows Phone 模拟器将自动运行并加载应用程序。

3. 在 **NotifyUserWindowsPhone** 应用 UI 中，输入用户名和密码。这些信息可以是任意字符串，但必须是相同的值。

4. 在 **NotifyUserWindowsPhone** 应用 UI 中，单击“登录并注册”。然后单击“发送推送”。

[3]: ./media/notification-hubs-aspnet-backend-windows-dotnet-secure-push/notification-hubs-secure-push3.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-secure-push/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-secure-push/notification-hubs-secure-push13.png

<!---HONumber=Mooncake_0503_2016-->