<properties
	pageTitle="在 Visual Studio 2013 中导入发布设置文件 | Windows Azure"
	description="了解如何在 Visual Studio 2013 中为 Azure 移动服务应用程序导入订阅发布设置文件。"
	documentationCenter=""
	services="mobile-services"
	manager="dwrede"
	editor=""
	authors="ggailey777"/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/18/2015" 
	wacn.date="10/22/2015"/>

#  在 Visual Studio 2013 中导入订阅发布设置文件

在您能够创建移动服务之前，必须将来自 Azure 的订阅发布设置文件导入 Visual Studio。这使 Visual Studio 能够代表您连接到 Azure。

>[AZURE.NOTE]从 Visual Studio 2013 Update 2 开始，您不再需要使用发布设置文件。Visual Studio 能够通过使用您提供的凭据直接连接到 Azure。

1. 在 Visual Studio 2013 中，打开“解决方案资源管理器”、右键单击项目、单击“添加”，然后单击“连接的服务...”。 

	![添加连接的服务](./media/mobile-services-windows-how-to-import-publishsettings/mobile-add-connected-service.png)

2. 在“服务管理器”对话框中，单击“创建服务...”，然后从“创建移动服务”对话框中的“订阅”中选择“&lt;导入...&gt;”。

	![从 VS 2013 创建新移动服务](./media/mobile-services-windows-how-to-import-publishsettings/mobile-create-service-from-vs2013.png)

3. 在“导入 Azure 订阅”中，单击“下载订阅文件”，登录到你的 Azure 帐户（如果需要），并在浏览器请求保存文件时单击“保存”。

	![在 VS 中下载订阅文件](./media/mobile-services-windows-how-to-import-publishsettings/mobile-import-azure-subscription.png)

	> [AZURE.NOTE]登录窗口显示在浏览器中，可能位于 Visual Studio 窗口后面。请记住，应记下您保存下载的 .publishsettings 文件的位置。如果您的项目已连接到 Azure 订阅，可以跳过此步骤。

4. 单击“浏览”，导航到 .publishsettings 文件的保存位置，选择该文件，然后依次单击“打开”和“导入”。

	![在 VS 中导入订阅](./media/mobile-services-windows-how-to-import-publishsettings/mobile-import-azure-subscription-2.png)

	Visual Studio 导入连接至您的 Azure 订阅所需的数据。您的订阅已经具有一个或多个现有移动服务时，会显示服务名称。

	> [AZURE.NOTE]导入发布设置之后，应考虑删除所下载的 .publishsettings 文件，因为该文件包含可供他人访问您的帐户的信息。如果您打算保留该文件，以备在其他连接的应用项目中使用，请保护该文件的安全。

<!-- Anchors. -->

<!-- Images. -->
[1]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-live-connect-add-app.png
[2]: ./media/mobile-services-how-to-register-microsoft-authentication/mobile-live-connect-app-api-settings.png
<!-- URLs. -->
[Single sign-on for Windows Store apps by using Live Connect]: /documentation/articles/mobile-services-how-to-register-windows-live-connect-single-sign-on
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Get started with Mobile Services]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started
[Get started with authentication]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-users
[Get started with push notifications]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push
[Authorize users with scripts]: /documentation/articles/mobile-services-windows-store-dotnet-authorize-users-in-scripts
[JavaScript and HTML]: /documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-js
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=74-->