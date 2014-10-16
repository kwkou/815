<div class="dev-callout"><b>说明</b>

<p>若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>。</p>
</div>

1.  导航到 [Google Cloud Console][] 网站，使用你的 Google 帐户凭据登录，然后单击“创建项目” 。

    ![][0]

	<div class="dev-callout"><b>说明</b>

    <p>如果你已拥有现成项目，则在登录后你将定向到“项目” 页。若要从仪表板新建一个项目，请展开“API 项目” ，单击“其他项目”下面的“创建...” ，然后输入项目名称并单击“创建项目” 。</p>
	</div>

2.  输入项目名称，接受服务条款并单击“创建” 。执行请求的 SMS 验证，再次单击“创建” 。

3.  记下“项目”部分中的项目编号 。

    在教程的稍后部分中，你要将此值设置为客户端中的 PROJECT\_ID 变量。

4.  在左栏中，单击“API 和身份验证” ，然后向下滚动并单击开关以启用 "Google Cloud Messaging for Android"，再接受服务条款。

    ![][1]

5.  单击“凭据”  ，然后单击“创建新密钥” 

    ![][2]

6.  在“创建新密钥”中， 单击“服务器密钥” 。在下一个窗口中，单击“创建” 。

    ![][3]

7.  记下“API 密钥” 值。

    ![][4]

    接下来，你将使用此 API 密钥值，让移动服务向 GCM 进行身份验证并代表你的应用程序发送推送通知。

  [accounts.google.com]: http://go.microsoft.com/fwlink/p/?LinkId=268302
  [Google Cloud Console]: http://cloud.google.com/console
  [0]: ./media/notification-hubs-android-get-started/mobile-services-google-new-project.png
  [1]: ./media/notification-hubs-android-get-started/mobile-services-google-enable-GCM.png
  [2]: ./media/notification-hubs-android-get-started/mobile-services-google-create-server-key.png
  [3]: ./media/notification-hubs-android-get-started/mobile-services-google-create-server-key2.png
  [4]: ./media/notification-hubs-android-get-started/mobile-services-google-create-server-key3.png
