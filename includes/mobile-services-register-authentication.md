为了能够对用户进行身份验证，你必须通过标识提供者注册你的应用程序。然后，你需要向移动服务注册标识提供者生成的客户端密钥。

1.  登录到 [Azure 管理门户][]，单击“移动服务”，然后单击你的移动服务 。

    ![][0]

2.  单击“仪表板”选项卡， 并记下“移动服务 URL”值。 

    ![][1]

    注册你的应用程序时，可能需要向标识提供者提供此值。

3.  从以下列表中选择支持的标识提供者，并按步骤向该标识提供者注册你的应用程序：

-   [Microsoft 帐户][]
-   [Facebook 登录][]
-   [Twitter 登录][]
-   [Google 登录][]
-   [Azure Active Directory][]

    请记住，要记下标识提供者生成的客户端标识和密钥值。

    <div class="dev-callout"><b>安全说明</b>

    <p>标识提供者生成的密钥是一个重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。</p>
	</div>

1.  回到管理门户中，单击“标识”选项卡， 输入从标识提供者获取的应用程序标识符和共享密钥值，然后单击“保存”。 

    ![][2]

    你的移动服务和应用程序现已配置为使用你选择的身份验证提供者。

  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-register-authentication/mobile-services-selection.png
  [1]: ./media/mobile-services-register-authentication/mobile-service-uri.png
  [Microsoft 帐户]: /zh-cn/documentation/articles/mobile-services-how-to-register-microsoft-authentication/
  [Facebook 登录]: /zh-cn/documentation/articles/mobile-services-how-to-register-facebook-authentication/
  [Twitter 登录]: /zh-cn/documentation/articles/mobile-services-how-to-register-twitter-authentication/
  [Google 登录]: /zh-cn/documentation/articles/mobile-services-how-to-register-google-authentication/
  [Azure Active Directory]: /zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/
  [2]: ./media/mobile-services-register-authentication/mobile-identity-tab.png
