

为了能够对用户进行身份验证，您必须通过标识提供程序注册您的应用。然后，您需要向移动服务注册标识提供程序生成的客户端密钥。

1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的移动服务。

   	![](./media/mobile-services-register-authentication/mobile-services-selection.png)

2. 单击"仪表板"选项卡，并记下"移动服务 URL"值。

   	![](./media/mobile-services-register-authentication/mobile-service-uri.png)

    注册您的应用时，可能需要向标识提供程序提供此值。

3. 从以下列表中选择支持的标识提供程序，并按步骤向该标识提供程序注册您的应用：

 - <a href="/zh-cn/documentation/articles/mobile-services-how-to-register-microsoft-authentication/" target="_blank">Microsoft 帐户</a>
 - <a href="/zh-cn/documentation/articles/mobile-services-how-to-register-facebook-authentication/" target="_blank">Facebook 登录</a>
 - <a href="/zh-cn/documentation/articles/mobile-services-how-to-register-twitter-authentication/" target="_blank">Twitter 登录</a>
 - <a href="/zh-cn/documentation/articles/mobile-services-how-to-register-google-authentication/" target="_blank">Google 登录</a>
 - <a href="/zh-cn/documentation/articles/mobile-services-how-to-register-active-directory-authentication/" target="_blank">Azure Active Directory</a>


    请记住，要记下标识提供程序生成的客户端标识和密钥值。

    <div class="dev-callout"><b>安全说明</b>
	<p>标识提供程序生成的密钥是一个重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用分发。</p>
    </div>

4. 回到管理门户中，单击"标识"选项卡，输入从标识提供程序获取的应用标识符和共享密钥值，然后单击"保存"。

   	![](./media/mobile-services-register-authentication/mobile-identity-tab.png)

	您的移动服务和应用现已配置为使用您选择的身份验证提供程序。

<!-- URLs. -->
[Azure 管理门户]: https://manage.windowsazure.cn/
