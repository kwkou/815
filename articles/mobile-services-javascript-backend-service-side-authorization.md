<properties
	pageTitle="在 JavaScript 后端移动服务中对用户进行服务端授权 | Microsoft Azure"
	description="了解如何在 Azure 移动服务的 JavaScript 后端对用户授权。"
	services="mobile-services"
	documentationCenter=""
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="03/09/2016"
	wacn.date="04/11/2016"/>

#  移动服务中的用户服务端授权

> [AZURE.SELECTOR]
- [.NET 后端](/documentation/articles/mobile-services-dotnet-backend-service-side-authorization/)
- [JavaScript 后端](/documentation/articles/mobile-services-javascript-backend-service-side-authorization/) 

本主题说明如何使用服务器端脚本为用户授权。在本教程中，你将在 Azure 移动服务中注册脚本，根据用户 ID 筛选查询，然后只授予用户对其自己数据的访问权限。根据用户 ID 筛选用户的查询结果是最基本的授权形式。根据具体的方案，你可能还需要创建“用户”或“角色”表，以跟踪更详细的用户授权信息，例如，给定的用户有权访问哪些终结点。

本教程基于“移动服务快速入门”，并且是在 [向现有移动服务应用添加身份验证] 教程的基础上制作的。请先完成 [向现有移动服务应用添加身份验证] 教程。

##  <a name="register-scripts"></a>注册脚本

1. 登录到 [Azure 管理门户]，单击“移动服务”，然后单击你的移动服务。单击“数据”选项卡，然后单击 **TodoItem** 表。

2. 单击“脚本”，选择“Insert”操作，将现有脚本替换为以下函数，然后单击“保存”。

        function insert(item, user, request) {
          item.userId = user.userId;
          request.execute();
        }

	在插入之前，此脚本会在项中添加已经过身份验证的用户的用户 ID。

    >[AZURE.NOTE] 请确保已启用“[动态架构](https://msdn.microsoft.com/zh-cn/library/azure/jj193175.aspx)”。否则，不会自动添加 userId 列。默认情况下，已经为新的移动服务启用了此设置。

3. 同样，请将现有的 **Read** 操作替换为以下函数。此脚本将会筛选返回的 TodoItem 对象，使用户只会收到他们自己的插入项。

        function read(query, user, request) {
           query.where({ userId: user.userId });
           request.execute();
        }

##  <a name="test-app"></a>测试应用程序

1. 你会发现，现在当你运行客户端应用程序时，尽管已存在完成前面的教程时在 TodoItem 表中创建的项，但并没有返回任何项。发生此情况的原因是，以前插入项时并未使用用户 ID 列，而现在该列的值为 null。请在 TodoItem 表中检查新添加的项是否有关联的 userId 值。

2. 如果你有其他登录帐户，可以通过关闭、再删除、然后重新运行应用程序，来验证用户是否只能看到他们自己的数据。显示登录凭据对话框时，请输入一个不同的登录名，然后检查在前一登录名下输入的项是否未显示。


<!-- Anchors. -->

[Register server scripts]: #register-scripts
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->


[Windows Push Notifications & Live Connect]: http://go.microsoft.com/fwlink/p/?LinkID=257677
[Mobile Services server script reference]: /documentation/articles/mobile-services-how-to-use-server-scripts/
[My Apps dashboard]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[向现有移动服务应用添加身份验证]: /documentation/articles/mobile-services-ios-get-started-users/

[Azure 管理门户]: https://manage.windowsazure.cn/
 

<!---HONumber=Mooncake_0118_2016-->