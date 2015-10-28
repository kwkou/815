<properties 
	pageTitle="向经过身份验证的 Windows Phone Silverlight 应用用户发送推送通知 | Windows Azure" 
	description="了解如何从 Azure 移动服务向 Windows Phone Silverlight 应用的特定用户发送推送通知。" 
	services="mobile-services,notification-hubs" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/04/2015" 
	wacn.date="10/22/2015"/>

#  向经过身份验证的用户发送推送通知

[AZURE.INCLUDE [mobile-services-selector-push-users](../includes/mobile-services-selector-push-users.md)]

本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[向应用程序添加推送通知]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将客户端注册到通知中心以发送推送通知。此外，你还要修改注册，以根据分配的用户 ID 添加标记。最后，服务器脚本会更新为仅对经过身份验证的用户 （而不是向所有注册用户）发送通知。

>[AZURE.NOTE]本教程支持 Windows Phone 8.0 和 Windows Phone 8.1 Silverlight 应用程序。对于 Windows Phone 8.1 应用商店应用程序，请参阅[本主题的 Windows 应用商店版本](/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-push-notifications-app-users)。

## 先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [向应用程序添加身份验证]<br/>向 TodoList 示例应用程序添加登录要求。

+ [向应用程序添加推送通知]<br/>配置 TodoList 示例应用程序，以使用通知中心发送推送通知。

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

## <a name="register"></a>更新服务以要求对注册进行身份验证

[AZURE.INCLUDE [mobile-services-javascript-backend-push-notifications-app-users](../includes/mobile-services-javascript-backend-push-notifications-app-users.md)]

<ol start="5"><li><p>将 insert 函数替换为以下代码，然后单击“保存”<strong></strong>：</p>
<pre><code>function insert(item, user, request) {
	// Define a payload for the Windows Phone toast notification.
	var payload = '&lt;?xml version="1.0" encoding="utf-8"?>' +
		'&lt;wp:Notification xmlns:wp="WPNotification">&lt;wp:Toast>' +
		'&lt;wp:Text1>New Item&lt;/wp:Text1>&lt;wp:Text2>' + item.text + 
		'&lt;/wp:Text2>&lt;/wp:Toast>&lt;/wp:Notification>';

	// Get the ID of the logged-in user.
	var userId = user.userId;		

	request.execute({
		success: function() {
			// If the insert succeeds, send a notification.
			push.mpns.send(userId, payload, 'toast', 22, {
				success: function(pushResponse) {
					console.log("Sent push:", pushResponse);
					request.respond();
					},             
					error: function (pushResponse) {
						console.log("Error Sending push:", pushResponse);
						request.respond(500, { error: pushResponse });
						}
					});
				}
			});      
}</code></pre>

<p>此插入脚本使用用户 ID 标记向已登录用户创建的所有 Windows Phone (MPNS) 应用程序注册发送推送通知（包括插入项的文本）。</p></li></ol>

## <a name="update-app"></a>更新应用程序以要求在注册之前登录

[AZURE.INCLUDE [mobile-services-windows-phone-push-notifications-app-users](/documentation/articles/mobile-services-windows-phone-push-notifications-app-users)]


## <a name="test"></a>测试应用程序

[AZURE.INCLUDE [mobile-services-windows-test-push-users](../includes/mobile-services-windows-test-push-users.md)]

<!-- Anchors. -->

[Updating the service to require authentication for registration]: #register
[Updating the app to log in before registration]: #update-app
[Testing the app]: #test
[Next Steps]: #next-steps


<!-- URLs. -->

[向应用程序添加身份验证]: /documentation/articles/mobile-services-windows-phone-get-started-users
[向应用程序添加推送通知]: /documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=74-->