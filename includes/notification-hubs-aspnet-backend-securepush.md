## WebAPI 项目

1. 在 Visual Studio 中，打开您在**通知用户**教程中创建的 **AppBackend** 项目。
2. 在 Notifications.cs 中，将整个 **Notifications** 类替换为以下代码。请确保将占位符替换为通知中心的连接字符串（具有完全访问权限）和中心名称。可以从 [Azure 经典管理门户](http://manage.windowsazure.cn)中获取这些值。现在，该模块将表示要发送的其他安全通知。在完整的实现中，通知将存储在数据库中；为简单起见，在此示例中我们将它们存储在内存中。

		public class Notification
	    {
	        public int Id { get; set; }
	        public string Payload { get; set; }
	        public bool Read { get; set; }
	    }
    
    
	    public class Notifications
	    {
	        public static Notifications Instance = new Notifications();
	        
	        private List<Notification> notifications = new List<Notification>();
	
	        public NotificationHubClient Hub { get; set; }
	
	        private Notifications() {
	            Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}", 	"{hub name}");
	        }

	        public Notification CreateNotification(string payload)
	        {
	            var notification = new Notification() {
                Id = notifications.Count,
                Payload = payload,
                Read = false
            	};

            	notifications.Add(notification);

            	return notification;
	        }

	        public Notification ReadNotification(int id)
	        {
	            return notifications.ElementAt(id);
	        }
	    }

20. 在 NotificationsController.cs 中，将 **NotificationsController** 类定义中的代码替换为以下代码。该组件为设备实现了一种安全检索通知的方法，还提供了一种方法来触发到设备的安全推送（用于本教程的教学目的）。请注意，在向通知中心发送通知时，我们将只发送一个包含通知 ID（且没有实际的消息内容）的原始通知：

		public NotificationsController()
        {
            Notifications.Instance.CreateNotification("This is a secure notification!");
        }

        // GET api/notifications/id
        public Notification Get(int id)
        {
            return Notifications.Instance.ReadNotification(id);
        }

        public async Task<HttpResponseMessage> Post()
        {
            var secureNotificationInTheBackend = Notifications.Instance.CreateNotification("Secure confirmation.");
            var usernameTag = "username:" + HttpContext.Current.User.Identity.Name;

            // windows
            var rawNotificationToBeSent = new Microsoft.Azure.NotificationHubs.WindowsNotification(secureNotificationInTheBackend.Id.ToString(),
                            new Dictionary<string, string> {
                                {"X-WNS-Type", "wns/raw"}
                            });
            await Notifications.Instance.Hub.SendNotificationAsync(rawNotificationToBeSent, usernameTag);

            // apns
            await Notifications.Instance.Hub.SendAppleNativeNotificationAsync("{"aps": {"content-available": 1}, "secureId": "" + secureNotificationInTheBackend.Id.ToString() + ""}", usernameTag);

            // gcm
            await Notifications.Instance.Hub.SendGcmNativeNotificationAsync("{"data": {"secureId": "" + secureNotificationInTheBackend.Id.ToString() + ""}}", usernameTag);


            return Request.CreateResponse(HttpStatusCode.OK);
        }


请注意，`Post` 方法现在不发送 toast 通知。它将发送只包含通知 ID 且没有任何敏感内容的原始通知。另外，请确保注释在通知中心上未配置其凭据的平台的发送操作，因为它们会导致错误。

21. 现在，我们将此应用重新部署到 Azure Web 应用，以便可以从所有设备对其进行访问。右键单击 **AppBackend** 项目，然后选择“发布”。

24. 选择 Azure Web 应用作为发布目标。使用您的 Azure 帐户登录，选择一个现有的或新的 Web 应用，并记下“连接”选项卡中的**目标 URL** 属性。在本教程后面的部分中，我们将此 URL 称为*后端终结点*。单击“发布”。

<!---HONumber=82-->