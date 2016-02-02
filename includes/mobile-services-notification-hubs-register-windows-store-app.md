

1. 如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用的“[提交应用程序]”页，用 Microsoft 帐户登录，然后单击“应用程序名称”。

   	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-services-submit-win8-app.png)

2. 在“应用名称”中为应用键入一个名称，单击“保留应用名称”，然后单击“保存”。

   	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-services-win8-app-name.png)

   	此操作为应用创建一个新的 Windows 应用商店注册。

3. 在 Visual Studio 中，打开你在完成**移动服务入门**教程后创建的项目。

4. 在解决方案资源管理器中，右键单击项目，单击“应用商店”，然后单击“将应用与应用商店关联...”。

  	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-services-store-association.png)

   	此时将显示“将应用与 Windows 应用商店关联”向导。

5. 在该向导中，单击“登录”，然后用你的 Microsoft 帐户登录。

6. 单击在第 2 步中注册的应用程序，单击“下一步”，然后单击“关联”。

   	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-services-select-app-name.png)

   	这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。    

7. （可选）还重复步骤 4-6 以注册通用 Windows 应用的 Windows Phone Store 项目。

8. 回到新应用的“Windows 开发人员中心”页，单击“服务”。

   	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-services-win8-edit-app.png) 

9. 在“服务”页中，单击“Azure 移动服务”下的“Live 服务站点”。

	![](./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-win8-edit2-app.png)

10. 单击“验证服务”，然后记下“客户端密钥”和“软件包安全标识 (SID)”的值。

   	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-services-win8-app-push-auth.png)

    > [AZURE.NOTE]客户端密钥和程序包 SID 是重要的安全凭据。请勿与任何人分享这些密钥或将密钥随应用分发。

11. 登录到 [Azure 经典门户](https://manage.windowsazure.cn/)，单击“移动服务”，然后单击你的应用。

   	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-services-selection.png)

12. 单击“推送”选项卡，输入从 WNS 获取的“客户端密钥”和“软件包 SID”值，然后单击“保存”。

	>[AZURE.NOTE]使用较旧的移动服务完成本教程后，你可能会在“推送”选项卡底部看到一个链接，其表示“启用增强推送”。立即单击此处以升级移动服务来与通知中心集成。无法还原此更改。有关如何在生产移动服务中启用增强的推送通知的详细信息，请参阅<a href="http://go.microsoft.com/fwlink/p/?LinkId=391951">此指南</a>。

   	![](./media/mobile-services-notification-hubs-register-windows-store-app/mobile-push-tab.png)

	>[AZURE.NOTE]在门户的“推送”选项卡中为增强型推送通知设置 WNS 凭证后，这些凭证将与通知中心共享，以配置你的应用的通知中心。

<!-- URLs. -->
[Get started with Mobile Services]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/
[提交应用程序]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->