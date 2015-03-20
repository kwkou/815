将应用注册到 APNS 并配置项目后，接下来必须配置移动服务以便与 APNS 集成。

1. 在 Keychain Access 中，在**密钥**或**我的证书**中右键单击快速入门应用的新证书，单击**导出**，将您的文件命名为 QuickstartPusher，选择 **.p12** 格式，然后单击**保存**。

   	![](./media/mobile-services-apns-configure-push/mobile-services-ios-push-step18.png)

  记下文件名和导出的证书的位置。

>[WACOM.NOTE] 本教程将创建一个 QuickstartPusher.p12 文件。你的文件名和位置可能不同。

2. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的应用。

   	![](./media/mobile-services-apns-configure-push/mobile-services-selection.png)

3. 单击**推送**选项卡，然后单击**上载**。

   	![](./media/mobile-services-apns-configure-push/mobile-push-tab-ios.png)

	此时将显示"上载证书"对话框。

4. 单击**文件**，选择导出的证书 QuickstartPusher.p12 文件，输入**密码**，请确保选择了正确**模式** （Dev/Sandbox 或 Prod/Production），单击勾选图标，然后单击**保存**。

   	![](./media/mobile-services-apns-configure-push/mobile-push-tab-ios-upload.png)

    > [WACOM.NOTE] 本教程使用开发证书。

现在，你的移动服务已配置为使用 APNS。

<!-- URLs. -->
[Azure 管理门户]: https://manage.windowsazure.cn/
