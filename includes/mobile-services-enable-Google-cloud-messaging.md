


1. 导航到 <a href="http://cloud.google.com/console" target="_blank">Google Cloud Console</a> 网站、使用您的 Google 帐户凭据登录，然后单击"创建项目"。

   	![](./media/notification-hubs-android-get-started/mobile-services-google-new-project.png)   

	>[WACOM.NOTE]如果您已拥有现成项目，则在登录后您将定向到 <strong></strong>"项目"页。若要从仪表板新建一个项目，请展开"API 项目"<strong></strong>、单击"其他项目"<strong></strong>下面的"创建..."<strong></strong>，然后输入项目名称并单击"创建项目"<strong></strong>。

2. 输入项目名称，接受服务条款并单击"创建"。执行请求的 SMS 验证，再次单击"创建"。

3. 记下"项目"部分中的项目编号。 

	在教程的稍后部分中，您要将此值设置为客户端中的 PROJECT_ID 变量。

4. 在左栏中，单击"API 和身份验证"，然后向下滚动并单击开关以启用"Google Cloud Messaging for Android"并接受服务条款。 

	![](./media/notification-hubs-android-get-started/mobile-services-google-enable-GCM.png)

5. 单击"凭据"，然后单击"创建新密钥" 

   	![](./media/notification-hubs-android-get-started/mobile-services-google-create-server-key.png)

6. 在"创建新密钥"中，单击"服务器密钥"。在下一个窗口中，单击"创建"。

   	![](./media/notification-hubs-android-get-started/mobile-services-google-create-server-key2.png)

7. 记下"API 密钥"值。

   	![](./media/notification-hubs-android-get-started/mobile-services-google-create-server-key3.png) 

	接下来，您将使用此 API 密钥值，让移动服务对 GCM 进行身份验证并代表您的应用发送推送通知。

<!--HONumber=41-->
