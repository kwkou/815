针对本地移动服务测试客户端应用程序之后，此教程的最后一个阶段是将移动服务发布到 Azure，并针对实时服务运行应用程序。

1.  在“解决方案资源管理器”中，右键单击移动服务项目并单击“发布” 

    ![][0]

    这样可以显示“发布 Web”对话框。

2.  单击“导入” ，单击“浏览”， 导航到你此前保存发布配置文件的位置，选择发布配置文件，单击“确定”。 

    ![][1]

    这样可以加载 Visual Studio 需要的信息，以便将你的移动服务发布到 Azure。

	<div class="dev-callout"><b>安全说明</b>

    <p>导入发布配置文件之后，应考虑删除所下载的文件，因为该文件包含可供他人用来访问你的服务的信息。</p>
	</div>

3.  单击“验证连接” 以验证是否正确配置了发布，然后单击“发布” 。

    ![][2]

    发布成功之后，你将在 Azure 中再次看到确认页，它可以确认移动服务正常运行。

  [0]: ./media/mobile-services-dotnet-backend-publish-service/mobile-quickstart-publish.png
  [1]: ./media/mobile-services-dotnet-backend-publish-service/mobile-quickstart-publish-import-profile.png
  [2]: ./media/mobile-services-dotnet-backend-publish-service/mobile-quickstart-publish-2.png
