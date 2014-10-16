首先，你使用 Visual Studio 2013 中的“添加推送通知”向导，向 Windows 应用商店注册你的应用程序，然后配置你的移动服务以启用推送通知，并将代码添加到你的应用程序以注册设备通道。

1.  如果你尚未执行此操作，请执行[在 Visual Studio 2013 中导入 publishsettings 文件][]中的步骤，将你的 publisher.settings 文件导入 Visual Studio。

    如果你已经使用 Visual Studio 来创建或管理你的 Azure 订阅中的移动服务，则无需执行此操作。

2.  在 Visual Studio 2013 中，打开解决方案资源管理器，右键单击项目，单击“添加”，然后单击“推送通知...” 。

    ![mobile-add-push-notifications-vs2013][]

    这样可以启动“添加推送通知”向导。

3.  单击“下一步” ，登录你的 Windows 应用商店帐户，然后在“保留新名称”中提供一个名称 ，再单击“保留” 。

    ![mobile-add-push-notifications-vs2013-2][]

    这样可以创建新的应用程序注册。

4.  在“应用程序名称”列表中单击新注册， 然后单击“下一步” 。

    ![mobile-add-push-notifications-vs2013-3][]

5.  在“选择服务” 对话框中，单击你在完成[移动服务入门][]或[数据处理入门][]教程时创建的移动服务的名称，然后单击“下一步”和“完成”。 

    移动服务将会更新，以注册你的应用程序包 SID 和客户端密钥，还将创建新的 "channels" 表。移动服务现在配置为与 Windows 推送通知服务 (WNS) 配合使用，因此能够向你的应用程序发送通知。

	<div class="dev-callout"><b>说明</b>

    <p>当你的应用程序尚未配置为连接到移动服务时，向导也会完成在"数据处理入门"中演示的相同配置任务。</p>
	</div>

  [在 Visual Studio 2013 中导入 publishsettings 文件]: /zh-cn/documentation/articles/mobile-services-windows-how-to-import-publishsettings/
  [mobile-add-push-notifications-vs2013]: ../includes/media/mobile-services-create-new-push-vs2013/mobile-add-push-notifications-vs2013.png
  [mobile-add-push-notifications-vs2013-2]: ../includes/media/mobile-services-create-new-push-vs2013/mobile-add-push-notifications-vs2013-2.png
  [mobile-add-push-notifications-vs2013-3]: ../includes/media/mobile-services-create-new-push-vs2013/mobile-add-push-notifications-vs2013-3.png
  [移动服务入门]: /en-us/develop/mobile/tutorials/get-started/
  [数据处理入门]: /en-us/develop/mobile/tutorials/get-started-with-data-dotnet/
