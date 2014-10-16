新的 Twitter v1.1 API 要求你的应用程序在访问资源之前进行身份验证。首先，你需要获取使用 OAuth 2.0 来请求访问权限所需的凭据。然后，你要将凭据安全存储在移动服务的应用程序设置中。

1.  如果你尚未注册你的应用程序，请完成主题[注册应用程序以便在移动服务中进行 Twitter 登录][]中的步骤。

    Twitter 生成让你访问 Twitter v1.1 API 所需的凭据。你可从 Twitter 开发人员网站获取这些凭据。

2.  导航到 [Twitter 开发人员][]网站，使用你的 Twitter 帐户凭据登录，导航到“我的应用程序” ，并选择你的 Twitter 应用程序。

    ![][0]

3.  在应用程序的“详细信息”选项卡中 ，记下以下值：

    -   "使用者密钥"
    -   "使用者机密"
    -   "访问令牌"
    -   "访问令牌机密"

    ![][1]

4.  登录到 [Windows Azure 管理门户][]，单击“移动服务” ，然后单击你的移动服务。

5.  单击“标识”选项卡， 输入从 Twitter 获取的“使用者密钥”和“使用者机密”值， 然后单击“保存” 。

    ![][2]

6.  单击“配置”选项卡， 向下滚动到“应用程序设置”， 输入你从 Twitter 网站获取的下述每个项的“名称” 和“值”对， 然后单击“保存” 。

    -   `TWITTER_ACCESS_TOKEN`
    -   `TWITTER_ACCESS_TOKEN_SECRET`

    ![][3]

    这样可将 Twitter 访问令牌存储在应用程序设置中。与“标识”选项卡上的使用者凭据相同， 访问凭据也加密存储在应用程序设置中，你可在服务器脚本中访问这些凭据，而无需在脚本文件中对其进行硬编码。有关详细信息，请参阅[应用程序设置][]。

  [注册应用程序以便在移动服务中进行 Twitter 登录]: /zh-cn/documentation/articles/mobile-services-how-to-register-twitter-authentication/
  [Twitter 开发人员]: http://go.microsoft.com/fwlink/p/?LinkId=268300
  [0]: ./media/mobile-services-register-twitter-access/mobile-twitter-my-apps.png
  [1]: ./media/mobile-services-register-twitter-access/mobile-twitter-app-secrets.png
  [Windows Azure 管理门户]: https://manage.windowsazure.com/
  [2]: ./media/mobile-services-register-twitter-access/mobile-identity-tab-twitter-only.png
  [3]: ./media/mobile-services-register-twitter-access/mobile-schedule-job-app-settings.png
  [应用程序设置]: http://msdn.microsoft.com/en-us/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7
