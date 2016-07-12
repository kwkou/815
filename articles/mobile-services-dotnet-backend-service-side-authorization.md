<properties
	pageTitle="在 .NET 后端移动服务中对用户进行服务端授权 | Microsoft Azure"
	description="了解如何在 .NET 后端移动服务中限制已授权用户的访问权限"
	services="mobile-services"
	documentationCenter="windows"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="03/09/2016"
	wacn.date="04/18/2016"/>

# 移动服务中的用户服务端授权
> [AZURE.SELECTOR]
- [.NET 后端](/documentation/articles/mobile-services-dotnet-backend-service-side-authorization/)
- [Javascript 后端](/documentation/articles/mobile-services-javascript-backend-service-side-authorization/)


本主题说明如何使用服务器端逻辑为用户授权。在本教程中，你将要修改表控制器，根据用户 ID 筛选查询，然后只授予用户对其自己数据的访问权限。根据用户 ID 筛选用户的查询结果是最基本的授权形式。根据具体的方案，你可能还需要创建“用户”或“角色”表，以跟踪更详细的用户授权信息，例如，给定的用户有权访问哪些终结点。

本教程基于“移动服务快速入门”，并且是在[向现有移动服务应用程序添加身份验证]教程的基础上制作的。请先完成[向现有移动服务应用程序添加身份验证]教程。

## <a name="register-scripts"></a>修改数据访问方法

1. 在 Visual Studio 中打开你的移动项目，展开 DataObjects 文件夹，然后打开 **TodoItem.cs**。**TodoItem** 类定义数据对象，你需要添加用于筛选的 **UserId** 属性。将下面的新 UserId 属性添加到 **TodoItem** 类：

		public string UserId { get; set; }

	>[AZURE.NOTE] 若要进行此数据模型更改并维护数据库中的现有数据，必须使用 [Code First 迁移](/documentation/articles/mobile-services-dotnet-backend-how-to-use-code-first-migrations/)。

2. 在 Visual Studio 中，展开“控制器”文件夹，打开 **TodoItemController.cs**，然后添加以下 using 语句：

		using Microsoft.WindowsAzure.Mobile.Service.Security;

3. 找到 **PostTodoItem** 方法，并在该方法的开头添加以下代码。

		// Get the logged in user
		var currentUser = User as ServiceUser;
	
		// Set the user ID on the item
		item.UserId = currentUser.Id;
	
	此代码会将已经过身份验证的用户的用户 ID 添加到项中，然后将此 ID 插入 TodoItem 表。

4. 找到 **GetAllTodoItems** 方法，并将现有的 **return** 语句替换为以下代码行：

				// Get the logged in user
				var currentUser = User as ServiceUser;

				return Query().Where(todo => todo.UserId == currentUser.Id);
	此查询可筛选返回的 TodoItem 对象，使得每个用户只收到他们插入的项。

5. 将移动服务项目重新发布到 Azure。


## <a name="test-app"></a>测试应用程序

1. 你会发现，现在当你运行客户端应用程序时，尽管已存在完成前面的教程时创建的项，但并没有返回任何项。发生此情况的原因是，以前插入项时并未使用用户 ID 列，而现在该列的值为 null。

2. 如果你有其他登录帐户，可以通过关闭、再删除、然后重新运行应用程序，来验证用户是否只能看到他们自己的数据。显示登录凭据对话框时，请输入一个不同的登录名，然后检查在前一登录名下输入的项是否未显示。



<!-- Anchors. -->
[Register server scripts]: #register-scripts
[Next Steps]: #next-steps

<!-- Images. -->

[3]: ./media/mobile-services-dotnet-backend-ios-authorize-users-in-scripts/mobile-quickstart-startup-ios.png

<!-- URLs. -->
[向现有移动服务应用程序添加身份验证]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started-users/

<!---HONumber=Mooncake_0118_2016-->