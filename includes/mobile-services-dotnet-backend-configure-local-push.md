
您发布到 Azure 之前，可以（可选）使用在本地计算机或虚拟机上运行的移动服务来测试推送通知。为此，您必须在 web.config 文件中设置您的应用使用的通知中心的相关信息。仅在本地运行来连接到通知中心时才使用此信息；在发布到 Azure 时，则忽略它。

1. 返回移动服务的“推送”选项卡，然后单击“通知中心”链接。

	![](./media/mobile-services-dotnet-backend-configure-local-push/link-to-notification-hub.png)

	这会浏览到您的移动服务使用的通知中心。

2. 在通知中心页中，记下你的通知中心名称，然后单击“查看连接字符串”。

	![](./media/mobile-services-dotnet-backend-configure-local-push/notification-hub-page.png)

3. 在“访问连接信息”中，复制 **DefaultFullSharedAccessSignature** 连接字符串。

	![](./media/mobile-services-dotnet-backend-configure-local-push/notification-hub-connection-string.png)

4. 在 Visual Studio 中的移动服务项目内，打开服务的 Web.config 文件并在 **connectionStrings** 中，将 **MS_NotificationHubConnectionString** 的连接字符串替换为来自上一步的连接字符串。

5. 在 **appSettings** 中，将 **MS_NotificationHubName** 应用设置的值替换为通知中心的名称。

现在，移动服务项目配置为在本地运行时连接到 Azure 中的通知中心。请注意，重要的是使用与门户相同的通知中心名称和连接字符串，因为在 Azure 中运行时，这些 Web.config 项目设置被门户设置覆盖。

<!---HONumber=71-->