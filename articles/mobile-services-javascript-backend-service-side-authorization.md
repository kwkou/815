<properties
	pageTitle="在装有 JavaScript 后端的移动服务中进行用户服务端授权 | 移动开发人员中心"
	description="了解如何在 Azure 移动服务的 JavaScript 后端对用户授权。"
	services="mobile-services"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags wacn.date="06/29/2015" 
	ms.service="mobile-services"
	ms.workload="mobile"
	ms.tgt_pltfrm=""
	ms.topic="article"
	ms.date="02/18/2015"
	ms.author="krisragh"/>

# 移动服务中的用户服务端授权

> [AZURE.SELECTOR-LIST(平台 | 后端)]
- [（任何 | .NET）](/documentation/articles/mobile-services-dotnet-backend-service-side-authorization)
- [（任何 | Javascript）](/documentation/articles/mobile-services-javascript-backend-service-side-authorization)

本主题说明如何使用服务器端脚本为用户授权。在本教程中，你将在 Azure 移动服务中注册脚本，根据用户 ID 筛选查询，然后只授予用户对其自己数据的访问权限。

本教程基于"移动服务快速入门"，并且是在[向现有移动服务应用程序添加身份验证]教程的基础上制作的。请先完成[向现有移动服务应用程序添加身份验证]教程。

## <a name="register-scripts"></a>注册脚本

1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击你的移动服务。单击"数据"选项卡，然后单击 **TodoItem** 表。

2. 单击"脚本"，然后选择"插入"操作。

   	![][2]

3. 将现有脚本替换为以下函数，然后单击"保存"。在插入之前，此脚本会在项中添加已经过身份验证的用户的用户 ID。

        function insert(item, user, request) {
          item.userId = user.userId;
          request.execute();
        }


    > [AZURE.NOTE] 若要正常完成此操作，[必须启用动态架构](https://msdn.microsoft.com/zh-CN/library/azure/jj193175.aspx)。默认情况下，已经为新的移动服务启用了此设置。

5. 同样，请将现有的 **Read** 操作替换为以下函数。此脚本将会筛选返回的 TodoItem 对象，使用户只会收到他们自己的插入项。

        function read(query, user, request) {
           query.where({ userId: user.userId });
           request.execute();
        }




## <a name="test-app"></a>测试应用

1. 你会发现，现在当你运行客户端应用程序时，尽管已存在完成前面的教程时在 _TodoItem_ 表中创建的项，但并没有返回任何项。发生此情况的原因是，以前插入项时并未使用用户 ID 列，而现在该列的值为 null。请在 _TodoItem_  表中检查新添加的项是否有关联的 userId 值。

2. 如果你有其他登录帐户，可以通过关闭、再删除、然后重新运行应用程序，来验证用户是否只能看到他们自己的数据。显示登录凭据对话框时，请输入一个不同的登录名，然后检查在前一登录名下输入的项是否未显示。

<!-- Anchors. -->
[注册服务器脚本]: #register-scripts
[后续步骤]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-ios-authorize-users-in-scripts/mobile-services-selection.png
[1]: ./media/mobile-services-ios-authorize-users-in-scripts/mobile-portal-data-tables.png
[2]: ./media/mobile-services-ios-authorize-users-in-scripts/mobile-insert-script-users.png
[3]: ./media/mobile-services-ios-authorize-users-in-scripts/mobile-quickstart-startup-ios.png

<!-- URLs. -->

[Windows 推送通知和 Live Connect]: https://appdev.microsoft.com/StorePortals/zh-CN/Home/Index
[移动服务服务器脚本参考]: /develop/mobile/how-to-guides/work-with-server-scripts/
[我的应用程序仪表板]: https://account.live.com/developers/applications/index
[移动服务入门]: /develop/mobile/tutorials/get-started/#create-new-service
[将移动服务添加到现有应用程序]: /develop/mobile/tutorials/get-started-with-data-ios
[向现有移动服务应用程序添加身份验证]: /develop/mobile/tutorials/get-started-with-users-ios
[向现有的应用程序添加推送通知]: /develop/mobile/tutorials/get-started-with-push-ios

[Azure 管理门户]: https://manage.windowsazure.cn/

<!--HONumber=50-->