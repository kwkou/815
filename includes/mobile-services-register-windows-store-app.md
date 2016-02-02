
1. 如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用的“[提交应用程序]”页，用 Microsoft 帐户登录，然后单击“应用程序名称”。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-submit-win8-app.png)

2. 选择“通过保留唯一名称来创建新应用程序”，单击“继续”，在“应用程序名称”中输入应用程序的名称，单击“保留应用程序名称”，然后单击“保存”。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-app-name.png)

   	此操作为应用创建一个新的 Windows 应用商店注册。

3. 在 Visual Studio 中，打开你在完成[移动服务入门]教程后创建的项目。

4. 在“解决方案资源管理器”中，右键单击 Windows 应用商店应用程序项目，单击“应用商店”，然后单击“将应用与应用商店关联...”。

  	![](./media/mobile-services-register-windows-store-app/mobile-services-store-association.png)

   	此时将显示“将应用与 Windows 应用商店关联”向导。

5. 在向导中，单击“登录”，以你的 Microsoft 帐户登录，选择你在步骤 2 中注册的应用程序，单击“下一步”，然后单击“关联”。

   	这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

6. （可选）对于 Windows 通用应用程序，请对 Windows Phone 应用商店项目重复执行步骤 4 和 5。

7. 回到新应用的“Windows 开发人员中心”页，单击“服务”。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-edit-app.png)

8. 在“服务”页中，单击“Azure 移动服务”下的“Live 服务站点”。

	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-edit2-app.png)

9. 单击“API 设置”，选择启用“移动或桌面客户端应用程序”，提供移动服务 URL 设为“目标域”，在“重定向 URL”中提供 `https://<mobile_service>.azure-mobile.net/login/microsoftaccount/` 的值，然后单击“保存”。

	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-app-push-auth-2.png)

10. 在“应用设置”中，记下“客户端 ID”、“客户端密钥”和“程序包安全标识符 (SID)”的值。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-app-push-auth.png)

    >[AZURE.NOTE]客户端密钥和程序包 SID 是重要的安全凭据。请勿与任何人分享这些密钥或将密钥随应用分发。

10. 登录到 [Azure 经典门户](https://manage.windowsazure.cn/)，单击“移动服务”，然后单击你的应用。

11. 单击“标识”选项卡，输入在步骤 4 中从 WNS 获取的“客户端密钥”和“程序包 SID”值，然后单击“保存”。

   	![](./media/mobile-services-register-windows-store-app/mobile-push-tab.png)

12. 单击“标识”选项卡。请注意，已在上一步中设置“客户端密钥”值和“程序包 SID”值。输入你先前记下的“客户端 ID”，然后单击“保存”。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-identity-tab.png)
 
现在，你可以使用 Microsoft 帐户在应用中进行身份验证。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/#create-new-service
[提交应用程序]: http://go.microsoft.com/fwlink/p/?LinkID=266582

<!---HONumber=Mooncake_0118_2016-->