<properties pageTitle="向经过身份验证的用户发送推送通知" metaKeywords="push notifications, authentication, users, Notification Hubs, Mobile Services" description="了解如何将推送通知发送到特定 " metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="glenga" solutions="Mobile" manager="dwrede" editor="" />


# 向经过身份验证的用户发送推送通知

[WACOM.INCLUDE [mobile-services-selector-push-users](../includes/mobile-services-selector-push-users.md)]

本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[推送通知][推送通知入门]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将客户端注册到通知中心以发送推送通知。此外修改了注册以添加基于分配用户 ID 的标记。最后，服务器脚本会更新为仅对经过身份验证的用户 （而不是向所有注册用户）发送通知。

本教程将指导你完成以下过程：

+ [更新服务以要求对注册进行身份验证]
+ [更新应用程序以要求在注册之前登录]
+ [测试应用程序]
 
本教程支持 Windows Phone 8.0 和 Windows Phone 8.1 ("Silverlight") 应用程序。对于 Windows Phone 8.1 应用商店应用程序，请参阅[本主题的 Windows 应用商店版本](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-push-notifications-app-users)。

##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [身份验证入门]<br/>向 TodoList 示例应用程序添加登录要求。

+ [推送通知入门]<br/>使用通知中心中配置推送通知的 TodoList 示例应用程序。 

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

##<a name="register"></a>更新服务以要求对注册进行身份验证

[WACOM.INCLUDE [mobile-services-javascript-backend-push-notifications-app-users](../includes/mobile-services-javascript-backend-push-notifications-app-users.md)] 

<ol start="5"><li><p>将 insert 函数替换为以下代码，然后单击 <strong>保存</strong>:</p>
<pre><code>function insert(item, user, request) {
	// Define a payload for the Windows Phone toast notification.
	var payload = '&lt;?xml version="1.0" encoding="utf-8"?&gt;' +
		'&lt;wp:Notification xmlns:wp="WPNotification"&gt;&lt;wp:Toast&gt;' +
		'&lt;wp:Text1&gt;New Item&lt;/wp:Text1&gt;&lt;wp:Text2&gt;' + item.text + 
		'&lt;/wp:Text2&gt;&lt;/wp:Toast&gt;&lt;/wp:Notification&gt;';

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

##<a name="update-app"></a>更新应用程序以要求在注册之前登录

[WACOM.INCLUDE [mobile-services-windows-phone-push-notifications-app-users](../includes/mobile-services-windows-phone-push-notifications-app-users.md)] 


##<a name="test"></a>测试应用

[WACOM.INCLUDE [mobile-services-windows-test-push-users](../includes/mobile-services-windows-test-push-users.md)] 

<!---## <a name="next-steps"> </a>后续步骤

在下一教程[移动服务用户的服务端授权][使用脚本进行用户授权]中，您将使用移动服务根据已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。请在[移动服务 .NET 操作方法概念性参考]中了解有关如何将移动服务与 .NET 一起使用的详细信息。-->

<!-- Anchors. -->
[更新服务以要求对注册进行身份验证]: #register
[更新应用程序以要求在注册之前登录]: #update-app
[测试应用程序]: #test
[后续步骤]:#next-steps


<!-- URLs. -->
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-phone-get-started-push/

[Azure 管理门户]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
