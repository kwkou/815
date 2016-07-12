

通常，推送通知是在后端服务（例如，移动服务，或者使用兼容库的 ASP.NET）中发送的。如果你的后端没有可用的库，则你也可以使用 REST API 直接发送通知消息。

下面是你可能想要查看的有关发送通知的其他教程列表：

- Azure 移动服务：有关如何从通知中心集成的 Azure 移动服务后端发送通知的示例，请参阅 [移动服务中的推送通知入门]。  
- ASP.NET：[使用通知中心将通知推送到用户]。
- Azure 通知中心 Java SDK：有关从 Java 发送通知的信息，请参阅[如何通过 Java 使用通知中心](/documentation/articles/notification-hubs-java-backend-how-to/)。这种方法已在 Eclipse for Android 开发环境中进行测试。
- PHP：[如何通过 PHP 使用通知中心](/documentation/articles/notification-hubs-php-backend-how-to/)。


在本教程的下一部分中，你将了解如何使用[通知中心 REST 接口](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn223264.aspx)直接在应用程序中发送通知消息。所有已注册的设备将接收任何设备发送的通知。

<!---HONumber=82-->