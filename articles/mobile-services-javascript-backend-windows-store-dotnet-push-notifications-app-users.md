<properties 
	pageTitle="向经过身份验证的通用l Windows 应用用户发送推送通知。" 
	description="了解如何从 Azure 移动服务向通用 Windows C# 应用的特定用户发送推送通知。" 
	services="mobile-services,notification-hubs" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/05/2016" 
	wacn.date="04/11/2016"/>


# 向经过身份验证的用户发送推送通知

[AZURE.INCLUDE [mobile-services-selector-push-users](../includes/mobile-services-selector-push-users.md)]

##概述
本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[向应用程序添加推送通知]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将客户端注册到通知中心以发送推送通知。此外，你还要修改注册，以根据分配的用户 ID 添加标记。最后，服务器脚本会更新为仅对经过身份验证的用户 （而不是向所有注册用户）发送通知。

本教程将指导你完成以下过程：

1. [更新服务以要求对注册进行身份验证]
2. [更新应用程序以要求在注册之前登录]
3. [测试应用程序]
 
本教程支持 Windows 应用商店和 Windows Phone 应用商店应用程序。

##先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [向应用程序添加身份验证]<br/>向 TodoList 示例应用程序添加登录要求。

+ [向应用程序添加推送通知]<br/>配置 TodoList 示例应用程序，以使用通知中心发送推送通知。

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

##<a name="register"></a>更新服务以要求对注册进行身份验证

[AZURE.INCLUDE [mobile-services-javascript-backend-push-notifications-app-users](../includes/mobile-services-javascript-backend-push-notifications-app-users.md)]

&nbsp;&nbsp;5.将 insert 函数替换为以下代码，然后单击“保存”：

	function insert(item, user, request) {
    // Define a payload for the Windows Store toast notification.
    var payload = '<?xml version="1.0" encoding="utf-8"?><toast><visual>' +    
    '<binding template="ToastText01"><text id="1">' +
    item.text + '</text></binding></visual></toast>';

    // Get the ID of the logged-in user.
    var userId = user.userId;		

    request.execute({
        success: function() {
            // If the insert succeeds, send a notification to all devices 
	    	// registered to the logged-in user as a tag.
            	push.wns.send(userId, payload, 'wns/toast', {
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
	}

&nbsp;&nbsp;此插入脚本使用用户 ID 标记向已登录用户创建的所有 Windows 应用商店应用注册发送推送通知（包含插入项的文本）。

##<a name="update-app"></a>更新应用程序以要求在注册之前登录

[AZURE.INCLUDE [mobile-services-windows-store-dotnet-push-notifications-app-users](../includes/mobile-services-windows-store-dotnet-push-notifications-app-users.md)]

##<a name="test"></a>测试应用程序

[AZURE.INCLUDE [mobile-services-windows-test-push-users](../includes/mobile-services-windows-test-push-users.md)]

<!-- Anchors. -->
[更新服务以要求对注册进行身份验证]: #register
[更新应用程序以要求在注册之前登录]: #update-app
[测试应用程序]: #test
[Next Steps]: #next-steps


<!-- URLs. -->
[向应用程序添加身份验证]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/
[向应用程序添加推送通知]: /documentation/articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-push/

[Azure Management portal]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->