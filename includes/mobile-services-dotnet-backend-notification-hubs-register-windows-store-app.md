

1. 如果尚未注册应用，请在开发人员中心内导航到 Windows 应用商店应用的["提交应用"页]，用 Microsoft 帐户登录，然后单击"应用名称"。

   	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-services-submit-win8-app.png)

2. 在"应用名称"中为应用键入一个名称，单击"保留应用名称"，然后单击"保存"。

   	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-services-win8-app-name.png)

   	此操作为应用创建一个新的 Windows 应用商店注册。

3. 在 Visual Studio 中，打开您在完成**移动服务入门**教程后创建的 Windows Store 应用项目。

4. 在"解决方案资源管理器"中，右键单击项目，单击"应用商店"，然后单击"将应用与应用商店关联..."。 

  	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-services-store-association.png)

   	此时将显示"将应用与 Windows 应用商店关联"向导。

5. 在该向导中，单击"登录"，然后用您的 Microsoft 帐户登录。

6. 选择在第 2 步中注册的应用、单击"下一步"，然后单击"关联"。

   	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-services-select-app-name.png)

   	这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。    

7. （可选）还重复步骤 4-6 以注册通用 Windows 应用的 Windows Phone Store 项目。

8. 回到新应用的"Windows 开发人员中心"页，单击"服务"。 

   	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-services-win8-edit-app.png) 

9. 在"服务"页中，单击"Azure 移动服务"下的"Live 服务站点"。

	![](./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-win8-edit2-app.png)

10. 单击**验证服务**，然后记下**客户端密钥**和**软件包安全标识 (SID)** 的值。 

   	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-services-win8-app-push-auth.png)

    <div class="dev-callout"><b>安全说明</b>
	<p>客户端密钥和程序包 SID 是重要的安全凭据。请勿与任何人分享这些密钥或将密钥随应用分发。</p>
    </div> 

11. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的应用。

   	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-services-selection.png)

12. 单击"推送"选项卡，输入在步骤 4 中从 WNS 获取的"客户端密钥"和"软件包 SID"值，然后单击"保存"。	

   	![](./media/mobile-services-dotnet-backend-notification-hubs-register-windows-store-app/mobile-push-tab.png)

	>[WACOM.NOTE]在门户的**推送**选项卡中为增强型推送通知设置 WNS 凭证后，这些凭证将与通知中心共享，以配置您的应用的通知中心。

<!-- URLs. -->
["提交应用"页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[Azure 管理门户]: https://manage.windowsazure.cn/
