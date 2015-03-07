

针对本地移动服务测试客户端应用之后，此教程的最后一个阶段是将移动服务发布到 Azure，并针对实时服务运行该应用。

1. 在"解决方案资源管理器"中，右键单击移动服务项目、单击"发布"，然后在"发布 Web"对话框中单击"Azure 移动服务"。

	![](./media/mobile-services-dotnet-backend-publish-service/mobile-quickstart-publish.png)
	
2. 使用您的 Azure 帐户凭据登录、从"现有移动服务"中选择您的服务，然后单击"确定"。

	![](./media/mobile-services-dotnet-backend-publish-service/mobile-quickstart-publish-select-service.png)

	Visual Studio 从 Azure 直接下载您的发布设置。

	>[WACOM.NOTE]Visual Studio 将存储您的 Azure 凭据，直到您显式注销。

3. 单击"验证连接"以验证是否正确配置了发布，然后单击"发布"。

	![](./media/mobile-services-dotnet-backend-publish-service/mobile-quickstart-publish-2.png)

	发布成功之后，您将在 Azure 中再次看到确认页，它可以确认移动服务正常运行。

<!--HONumber=41-->
