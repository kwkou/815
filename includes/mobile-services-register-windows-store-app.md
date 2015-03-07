
1. 如果尚未注册应用，请在开发人员中心内导航到 Windows 应用商店应用的["提交应用"页]，用 Microsoft 帐户登录，然后单击"应用名称"。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-submit-win8-app.png)

2. 在"应用名称"中为应用键入一个名称，单击"保留应用名称"，然后单击"保存"。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-app-name.png)

   	此操作为应用创建一个新的 Windows 应用商店注册。

3. 在 Visual Studio 中，打开您在完成[移动服务入门]教程后创建的项目。

4. 在"解决方案资源管理器"中，右键单击项目，单击"应用商店"，然后单击"将应用与应用商店关联..."。 

  	![](./media/mobile-services-register-windows-store-app/mobile-services-store-association.png)

   	此时将显示"将应用与 Windows 应用商店关联"向导。

5. 在该向导中，单击"登录"，然后用您的 Microsoft 帐户登录。

6. 选择在第 2 步中注册的应用、单击"下一步"，然后单击"关联"。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-select-app-name.png)

   	这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。    

7. 回到新应用的"Windows 开发人员中心"页，单击"服务"。 

   	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-edit-app.png) 

8. 在"服务"页中，单击"Azure 移动服务"下的"Live 服务站点"。

	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-edit2-app.png) 

9. 在"应用设置"中，记下"客户端 ID"、"客户端密钥"和"程序包安全标识符 (SID)"的值。 

   	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-app-push-auth.png)

    >[WACOM.NOTE]客户端密钥和程序包 SID 是重要的安全凭据。请勿与任何人分享这些密钥或将密钥随应用分发。

10. （可选）单击"API 设置"、启用"增强型重定向安全"、提供"重定向 URL"中 `https://<mobile_service>.azure-mobile.net/login/microsoftaccount` 的值，然后单击"保存"。

	![](./media/mobile-services-register-windows-store-app/mobile-services-win8-app-push-auth-2.png)

	这样可为您的应用启用 Microsoft 帐户身份验证。

11. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的应用。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-selection.png)

12. 单击"推送"选项卡、输入在步骤 4 中从 WNS 获取的"客户端密钥"和"程序包 SID"值，然后单击"保存"。

   	![](./media/mobile-services-register-windows-store-app/mobile-push-tab.png)

13. 单击"标识"选项卡。请注意，已在上一步中设置"客户端密钥"值和"程序包 SID"值。输入您先前记下的"客户端 ID"，然后单击"保存"。

   	![](./media/mobile-services-register-windows-store-app/mobile-services-identity-tab.png)
 
现在，您可以使用 Microsoft 帐户在应用中进行身份验证。  

<!-- Anchors. -->

<!-- Images. -->
 

<!-- URLs. -->
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/#create-new-service
["提交应用"页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[Azure 管理门户]: https://manage.windowsazure.cn/
<!--HONumber=41-->
