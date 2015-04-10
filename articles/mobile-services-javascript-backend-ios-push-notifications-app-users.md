
<properties pageTitle="向经过身份验证的用户发送推送通知" metaKeywords="push notifications, authentication, users, Notification Hubs, Mobile Services" description="了解如何将推送通知发送到特定 " metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="krisragh" solutions="Mobile" manager="dwrede" editor="" />


<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-ios" ms.devlang="objective-c" ms.topic="article" ms.date="10/10/2014" ms.author="krisragh" />

# 向经过身份验证的用户发送推送通知

[WACOM.INCLUDE [mobile-services-selector-push-users](../includes/mobile-services-selector-push-users.md)]

本主题说明如何向任何已注册设备上已经过身份验证的用户发送推送通知。与前面的[推送通知][推送通知入门]教程不同，本教程将指导你更改移动服务，以要求先对用户进行身份验证，然后，才可以将 iOS 客户端注册到通知中心以发送推送通知。此外修改了注册以添加基于分配用户 ID 的标记。最后，服务器脚本会更新为仅对经过身份验证的用户 （而不是向所有注册用户）发送通知。

本教程将指导你完成以下过程：

+ [更新服务以要求对注册进行身份验证]
+ [更新应用程序以要求在注册之前登录]
+ [测试应用程序]

##先决条件

在开始本教程之前，必须已完成以下移动服务教程：

+ [身份验证入门]<br/>向 TodoList 示例应用程序添加登录要求。

+ [推送通知入门]<br/>使用通知中心中配置推送通知的 TodoList 示例应用程序。

完成这两篇教程后，你可以防止未经身份验证的用户从你的移动服务注册推送通知。

##<a name="register"></a>更新服务以要求对注册进行身份验证

[WACOM.INCLUDE [mobile-services-javascript-backend-push-notifications-app-users](../includes/mobile-services-javascript-backend-push-notifications-app-users.md)]

<ol start="5"><li><p>将 insert 函数替换为以下代码，然后单击 <strong>保存</strong>:</p>
<pre><code>function insert(item, user, request) {

        function insert(item, user, request) {
            request.execute();
            setTimeout(function() {
                push.apns.send(null, {
                    alert: "Alert: " + item.text,
                    payload: {
                        "Hey, a new item arrived: '" + item.text + "'"
                    }
                });
            }, 2500);
        }

}</code></pre>

<p>此插入脚本使用用户 ID 标记向已登录用户创建的所有 Windows Phone (MPNS) 应用程序注册发送推送通知（包括插入项的文本）。</p></li></ol>


##<a name="update-app"></a>更新应用程序以要求在注册之前登录

[WACOM.INCLUDE [mobile-services-ios-push-notifications-app-users-login](../includes/mobile-services-ios-push-notifications-app-users-login.md)]

##<a name="test"></a>测试应用

[WACOM.INCLUDE [mobile-services-ios-push-notifications-app-users-test-app](../includes/mobile-services-ios-push-notifications-app-users-test-app.md)]



<!-- Anchors. -->
[更新服务以要求对注册进行身份验证]: #register
[更新应用程序以要求在注册之前登录]: #update-app
[测试应用程序]: #test
[后续步骤]:#next-steps


<!-- URLs. -->
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-ios-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-ios-get-started-push/

[Azure 管理门户]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library

[23]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push1-ios.png
[24]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push2-ios.png
[25]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push3-ios.png
[26]: ./media/mobile-services-ios-get-started-push/mobile-quickstart-push4-ios.png
