1.  如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用程序的[“提交应用程序”页][]，用 Microsoft 帐户登录，然后单击“应用程序名称” 。

    ![][0]

2.  在“应用程序名称” 中为应用程序键入一个名称，单击“保留应用程序名称” ，然后单击“保存”。 

    ![][1]

    此操作为应用程序创建一个新的 Windows 应用商店注册。

3.  在 Visual Studio 2012 Express for Windows 8 中，打开你在完成教程[移动服务入门][]时创建的项目。

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

9.  单击“正在对你的服务进行身份验证” ，然后记下“客户端密钥”和“程序包安全标识符(SID)”的值。 

    ![][6]

    <div class="dev-callout"><b>安全说明</b>

    <p>客户端密钥和程序包 SID 是重要的安全凭据。请勿与任何人分享这些密钥或将密钥随应用程序分发。</p>
	</div>

10. 登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][7]

11. 单击“推送”选项卡，再 单击“启用增强的推送” ，然后单击“是” 以接受配置更改。

    ![][8]

    这样可以更新你的移动服务的配置，以使用通知中心提供的增强的推送功能。对于你的付费移动服务，有些通知中心使用是免费的。有关详细信息，请参阅[移动服务定价详细信息][]。

    <div class="dev-callout"><b>重要说明</b>

    <p>此操作可重置你的推送凭据，并更改脚本中的推送方法行为。这些更改无法恢复。不要使用此方法向生产移动服务添加通知中心。有关如何在生产移动服务中启用增强的推送通知的指导，请参阅<a href="http://go.microsoft.com/fwlink/p/?LinkId=391951">本指南</a>。</p>
	</div>

12. 输入在步骤 4 中从 WNS 获取的“客户端密钥” 和 “程序包 SID”值，然后单击“保存”。 

    ![][9]

    > [WACOM.NOTE] 如果你在门户中的“推送”选项卡上 为增强的推送通知设置了 WNS 凭据，则这些凭据与通知中心共享，用于配置应用程序的通知中心。

  [“提交应用程序”页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
  [0]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-submit-win8-app.png
  [1]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-win8-app-name.png
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/
  [2]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-store-association.png
  [3]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-select-app-name.png
  [4]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-win8-edit-app.png
  [5]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-win8-edit2-app.png
  [6]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-win8-app-push-auth.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [7]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-services-selection.png
  [8]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-enable-enhanced-push.png
  [移动服务定价详细信息]: http://go.microsoft.com/fwlink/p/?LinkID=311786
  [本指南]: http://go.microsoft.com/fwlink/p/?LinkId=391951
  [9]: ./media/mobile-services-javascript-backend-register-windows-store-app/mobile-push-tab.png
