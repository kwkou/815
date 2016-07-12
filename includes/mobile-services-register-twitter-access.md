

新的 Twitter v1.1 API 要求您的应用在访问资源之前进行身份验证。首先，您需要获取使用 OAuth 2.0 来请求访问权限所需的凭据。然后，您要将凭据安全存储在移动服务的应用设置中。

1. 如果您尚未注册您的应用，请完成主题<a href="/zh-cn/documentation/articles/mobile-services-how-to-register-twitter-authentication/" target="_blank">注册应用以便在移动服务中进行 Twitter 登录</a>中的步骤。 
  
  	Twitter 生成让您访问 Twitter v1.1 API 所需的凭据。您可从 Twitter 开发人员 Web 应用获取这些凭据。

2. 导航到 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268300" target="_blank">Twitter 开发人员</a> Web 应用，使用你的 Twitter 帐户凭据登录，然后选择你的 Twitter 应用。

3. 在应用的“密钥和访问令牌”选项卡中，记下以下值：

	+ **使用者密钥**
	+ **使用者机密**
	+ **访问令牌**
	+ **访问令牌机密**

4. 登录到 [Azure 管理门户]，单击“移动服务”，然后单击你的移动服务。

5. 单击“标识”选项卡，输入从 Twitter 获取的“使用者密钥”和“使用者机密”值，然后单击“保存”。

	![](./media/mobile-services-register-twitter-access/mobile-identity-tab-twitter-only.png)

2. 单击“配置”选项卡，向下滚动到“应用设置”，输入你从 Twitter Web 应用获取的下述每个项的“名称”和“值”对，然后单击“保存”。

	+ `TWITTER_ACCESS_TOKEN`
	+ `TWITTER_ACCESS_TOKEN_SECRET`

	![](./media/mobile-services-register-twitter-access/mobile-schedule-job-app-settings.png)

	这样可将 Twitter 访问令牌存储在应用设置中。与“标识”选项卡上的使用者凭据相同，访问凭据也加密存储在应用设置中，你可在服务器脚本中访问这些凭据，而无需在脚本文件中对其进行硬编码。有关详细信息，请参阅[应用设置]。

<!-- URLs. -->
[Mobile Services server script reference]: http://go.microsoft.com/fwlink/?LinkId=262293
[Azure 管理门户]: https://manage.windowsazure.cn/
[Register your apps for Twitter login with Mobile Services]: /documentation/articles/mobile-services-how-to-register-twitter-authentication/
[Twitter Developers]: http://go.microsoft.com/fwlink/p/?LinkId=268300
[应用设置]: http://msdn.microsoft.com/zh-cn/library/azure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7

<!---HONumber=74-->