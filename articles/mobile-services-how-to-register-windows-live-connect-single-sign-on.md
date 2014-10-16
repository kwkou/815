<properties pageTitle="Register for single sign-on - Azure Mobile Services" metaKeywords="" description="Learn how to register for single sign-on authentication in your Azure Mobile Services application." metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your Windows Store apps to use Windows Live Connect single sign-on" authors="" solutions="" manager="" editor="" />

# 注册 Windows 应用商店应用程序以使用 Windows Live Connect 单一登录

本主题说明如何将应用程序注册到 Windows 应用商店，以便将单一登录所用的 Live Connect 用作 Azure 移动服务的标识提供者。若要使用推送通知，也需要完成此步骤。

<div class="dev-callout"><b>说明</b>

<p>你无需将应用程序注册到 Windows 应用商店，便可以在发布应用程序之前使用 Microsoft 帐户进行身份验证。如果你的 Windows 应用商店应用程序不需要单一登录或推送通知，则只需将应用程序注册到 Live Connect 即可使用 Microsoft 帐户登录。有关详细信息，请参阅<a href="/zh-cn/develop/mobile/how-to-guides/register-for-microsoft-authentication">Register your Windows Store apps to use a Microsoft Account login</a>。</p>
</div>

1.  如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用程序的[“提交应用程序”页][]，用 Microsoft 帐户登录，然后单击“应用程序名称” 。

    ![][]

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

7.  在 Live Connect 开发人员中心内导航到[我的应用程序][]页，然后在“我的应用程序”列表中单击你的应用程序 。

    ![][4]

8.  依次单击“编辑设置”和“API 设置”，并记下“客户端密钥”的值 。

    ![][5]

    "安全说明"

    客户端密钥是一个非常重要的安全凭据。请勿与任何人分享客户端密钥或将密钥随应用程序分发。

9.  在“重定向域”中，输入你在执行步骤 8 时获取的移动服务 URL，然后单击“保存” 。

现在，你可以使用 Live Connect 将身份验证集成到应用程序中。移动服务提供以下两种方法，让你使用 Live Connect 对用户进行身份验证：

-   针对 Windows 应用商店应用程序的单一登录。使用此方法时，用户只需使用 Live Connect 在你的应用程序中为身份验证授权一次，然后，Windows 将会根据用户首选项管理凭据。有关详细信息，请参阅[使用 Live Connect 实现对 Windows 应用商店应用程序的单一登录][]。

-   基本身份验证。此方法支持各种身份验证提供程序，不过要求用户在每次启动你的应用程序时登录。有关详细信息，请参阅[身份验证入门][]。

  [注册 Windows 应用商店应用程序以使用 Microsoft 帐户登录]: /zh-cn/develop/mobile/how-to-guides/register-for-microsoft-authentication
  [“提交应用程序”页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
  [0]: ./media/mobile-services-how-to-register-windows-live-connect-single-sign-on/mobile-services-submit-win8-app.png
  [1]: ./media/mobile-services-how-to-register-windows-live-connect-single-sign-on/mobile-services-win8-app-name.png
  [移动服务入门]: /zh-cn/develop/mobile/tutorials/get-started
  [2]: ./media/mobile-services-how-to-register-windows-live-connect-single-sign-on/mobile-services-store-association.png
  [3]: ./media/mobile-services-how-to-register-windows-live-connect-single-sign-on/mobile-services-select-app-name.png
  [我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
  [4]: ./media/mobile-services-how-to-register-windows-live-connect-single-sign-on/mobile-live-connect-apps-list.png
  [5]: ./media/mobile-services-how-to-register-windows-live-connect-single-sign-on/mobile-live-connect-app-api-settings.png
  [使用 Live Connect 实现对 Windows 应用商店应用程序的单一登录]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet
  [身份验证入门]: /zh-cn/develop/mobile/tutorials/get-started-with-users-dotnet
