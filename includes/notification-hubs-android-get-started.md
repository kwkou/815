

>[AZURE.NOTE]若要完成此过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转到 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>。


1. 导航到 <a href="http://cloud.google.com/console" target="_blank">Google Cloud Console</a> 网站，使用你的 Google 帐户凭据登录，然后单击“创建项目”。

   	![](./media/notification-hubs-android-get-started/mobile-services-google-new-project.png)

	>[AZURE.NOTE]如果你已拥有现成项目，在登录后你将定向到“项目”页。<strong></strong>若要从仪表板新建一个项目，请展开“API 项目”，单击“其他项目”下面的“创建...”，然后输入项目名称并单击“创建项目”。<strong></strong><strong></strong><strong></strong><strong></strong>

2. 输入项目名称，接受服务条款并单击“创建”。如果系统请求，请执行 SMS 验证，然后再次单击“创建”。

3. 记下“项目”部分中的项目编号。

	在教程的稍后部分中，您要将此值设置为客户端中的 PROJECT\_ID 变量。

4. 在左栏中，展开“API 和身份验证”，单击“API”，然后向下滚动并单击开关以启用 **Google Cloud Messaging for Android**，最后再接受服务条款。

	![](./media/notification-hubs-android-get-started/mobile-services-google-enable-GCM.png)

5. 单击“凭据”，然后单击“创建新密钥”

   	![](./media/notification-hubs-android-get-started/mobile-services-google-create-server-key.png)

6. 在“创建新密钥”中，单击“服务器密钥”。在下一个窗口中，单击“创建”。

   	![](./media/notification-hubs-android-get-started/mobile-services-google-create-server-key2.png)

7. 记下“API 密钥”值。

   	![](./media/notification-hubs-android-get-started/mobile-services-google-create-server-key3.png)

	接下来，你将使用此 API 密钥值，让 Azure 对 GCM 进行身份验证并代表你的应用程序发送推送通知。

<!---HONumber=71-->