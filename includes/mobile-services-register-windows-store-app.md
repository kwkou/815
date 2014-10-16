1.  如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用程序的[“提交应用程序”页][]，用 Microsoft 帐户登录，然后单击“应用程序名称” 。

    ![][0]

2.  在“应用程序名称” 中为应用程序键入一个名称，单击“保留应用程序名称” ，然后单击“保存”。 

    ![][1]

    此操作为应用程序创建一个新的 Windows 应用商店注册。

3.  在 Visual Studio 中，打开你在完成[移动服务入门][]教程后创建的项目。

4.  在解决方案资源管理器中，右键单击项目，单击“应用商店” ，然后单击“将应用程序与应用商店关联...” 。

    ![][2]

    此时将显示“将应用程序与 Windows 应用商店关联” 向导。

5.  在该向导中，单击“登录” ，然后用你的 Microsoft 帐户登录。

6.  选择在第 2 步中注册的应用程序，单击“下一步” ，然后单击“关联” 。

    ![][3]

    这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

7.  回到新应用程序的“Windows 开发人员中心”页，单击“服务” 。

    ![][4]

8.  在“服务”页中，单击“Azure 移动服务”下的“Live 服务站点”。 

    ![][5]

9.  在“应用程序设置”中 ，记下“客户端 ID”、“客户端密钥”和“程序包安全标识符(SID)”的值。 

    ![][6]

    > [WACOM.NOTE] 客户端密钥和程序包 SID 是重要的安全凭据。请勿与任何人分享这些密钥或将密钥随应用程序分发。

10. （可选）单击“API 设置” ，启用“增强的重定向安全” ，提供“重定向 URL”中的 `https://<mobile_service>.azure-mobile.net/login/microsoftaccount` 的值， ，然后单击“保存”。 

    ![][7]

    这样可为你的应用程序启用 Microsoft 帐户身份验证。

11. 登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][8]

12. 单击“推送”选项卡， 输入在步骤 4 中从 WNS 获取的“客户端密钥”和“程序包 SID”值 ，然后单击“保存”。 

    ![][9]

你现在可以使用 Microsoft 帐户，通过向移动服务提供“标识”选项卡中的客户端 ID 和客户端密钥值，在你的应用程序中进行身份验证。 

  [“提交应用程序”页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
  [0]: ./media/mobile-services-register-windows-store-app/mobile-services-submit-win8-app.png
  [1]: ./media/mobile-services-register-windows-store-app/mobile-services-win8-app-name.png
  [移动服务入门]: /en-us/develop/mobile/tutorials/get-started/#create-new-service
  [2]: ./media/mobile-services-register-windows-store-app/mobile-services-store-association.png
  [3]: ./media/mobile-services-register-windows-store-app/mobile-services-select-app-name.png
  [4]: ./media/mobile-services-register-windows-store-app/mobile-services-win8-edit-app.png
  [5]: ./media/mobile-services-register-windows-store-app/mobile-services-win8-edit2-app.png
  [6]: ./media/mobile-services-register-windows-store-app/mobile-services-win8-app-push-auth.png
  [7]: ./media/mobile-services-register-windows-store-app/mobile-services-win8-app-push-auth-2.png
  [Azure 管理门户]: https://manage.windowsazure.com/
  [8]: ./media/mobile-services-register-windows-store-app/mobile-services-selection.png
  [9]: ./media/mobile-services-register-windows-store-app/mobile-push-tab.png
