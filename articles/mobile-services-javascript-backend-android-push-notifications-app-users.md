
<properties
	pageTitle="向经过身份验证的 Android 应用用户发送推送通知（JavaScript 后端）"
	description="了解如何使用具有 JavaScript 后端的移动服务向经过身份验证的特定 Android 应用用户发送推送通知。"
	services="mobile-services, notification-hubs"
	documentationCenter="android"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="09/03/2015" 
	wacn.date="10/22/2015"/>


# 向经过身份验证的 Android 应用用户发送推送通知

[AZURE.INCLUDE [mobile-services-selector-push-users](../includes/mobile-services-selector-push-users.md)]

## 概述

本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[推送通知][Get started with push notifications]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将客户端注册到通知中心以发送推送通知。此外，你还要修改注册，以根据分配的用户 ID 添加标记。最后，服务器脚本会更新为仅对经过身份验证的用户 （而不是向所有注册用户）发送通知。

本教程支持使用具有 JavaScript 后端的 Azure 移动服务的 Android 应用。

## 先决条件 

在开始本教程之前，必须已完成以下移动服务教程：

+ [向移动服务添加身份验证]<br/>向 TodoList 示例应用程序添加登录要求。

+ [推送通知入门]<br/>配置 TodoList 示例应用程序，以使用通知中心发送推送通知。

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

## 更新服务以要求对注册进行身份验证

[AZURE.INCLUDE [mobile-services-javascript-backend-push-notifications-app-users](../includes/mobile-services-javascript-backend-push-notifications-app-users.md)]

<ol start="5"><li><p>将 insert 函数替换为以下代码，然后单击“保存”<strong></strong>：</p>
<pre><code>function insert(item, user, request) {

    // Define a payload for the Google Cloud Messaging toast notification.
    var payload = 
        '{"data":{"message" : "Hello from Mobile Services! An Item was inserted"}}';

    // Get the ID of the logged-in user.
    var userId = user.userId;		

    request.execute({
        success: function() {
            // If the insert succeeds, send a notification to all devices 
            // registered to the logged-in user as a tag.
            push.gcm.send(userId, payload, {
                success: function(pushResponse) {
                    console.log("Sent push with " + userId + " tag:", pushResponse, payload);
	    			request.respond();
                    },             
                    error: function (pushResponse) {
                            console.log("Error Sending push:", pushResponse);
	    				request.respond(500, { error: pushResponse });
                        }
                    });
                },
                error: function(err) {
                    console.log("request.execute error", err)
                    request.respond();
                }
            });
}</code></pre>

<p>此插入脚本使用用户 ID 标记向已登录用户创建的所有 Google Cloud Messaging 注册发送推送通知（包括插入项的文本）。</p></li></ol>

## 更新应用程序以要求在注册之前登录

[AZURE.INCLUDE [mobile-services-android-push-notifications-app-users](/documentation/articles/mobile-services-android-push-notifications-app-users)]

## 测试应用程序

[AZURE.INCLUDE [mobile-services-android-test-push-users](../includes/mobile-services-android-test-push-users.md)]

<!---##Next steps

In the next tutorial, [Service-side authorization of Mobile Services users](/documentation/articles/mobile-services-javascript-backend-service-side-authorization), you will take the user ID value provided by Mobile Services based on an authenticated user and use it to filter the data returned by Mobile Services. Learn more about how to use Mobile Services with .NET in [Mobile Services .NET How-to Conceptual Reference]-->


<!-- URLs. -->
[向移动服务添加身份验证]: /documentation/articles/mobile-services-android-get-started-users
[Get started with push notifications]: /documentation/articles/mobile-services-javascript-backend-android-get-started-push
[推送通知入门]: /documentation/articles/mobile-services-javascript-backend-android-get-started-push
[Azure Management Portal]: https://manage.windowsazure.cn/
[Mobile Services .NET How-to Conceptual Reference]: /documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library

<!---HONumber=74-->