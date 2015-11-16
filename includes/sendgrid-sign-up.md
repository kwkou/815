Azure 客户每月可解锁 25,000 封免费电子邮件。通过每月的这 25,000 封免费电子邮件，将可使用高级报告和分析以及[所有 API][]（Web、SMTP、事件、分析等）。有关 SendGrid 提供的其他服务的信息，请参阅[ SendGrid 功能][]页。

### 注册 SendGrid 帐户

1. 登录到 [Azure 管理门户][]。

2. 在该管理门户的下方窗格中，单击“新建”。

	![command-bar-new][command-bar-new]

3. 单击“应用商店”。

	![sendgrid-store][sendgrid-store]

4. 在“选择应用程序和服务”对话框中，选择“SendGrid”，然后单击右箭头。

5. 在“个性化应用程序和服务”对话框中，选择要注册的 **SendGrid** 计划。

6. 输入一个名称以标识 Azure 设置中的 **SendGrid** 服务，或使用 **SendGridEmailDelivery.Simplified.SMTPWebAPI** 的默认值。名称的长度必须介于 1 到 100 个字符之间，并只能包含字母字符、短划线、句点和下划线。名称在订阅的 Azure 应用商店项目的列表中必须是唯一的。

	![store-screen-2][store-screen-2]

7. 选择区域值，例如美国西部。

8. 单击右箭头。

9. 在“查看购买”选项卡上，检查计划和定价信息，并查看法律条款。如果同意这些条款，请单击复选标记。单击复选标记后，你的 SendGrid 帐户将开始 [SendGrid 设置过程]。

	![store-screen-3][store-screen-3]

10. 确认你的购买后，你将重定向到外接程序仪表板，并且将显示消息“购买 SendGrid”。

	![sendgrid-purchasing-message][sendgrid-purchasing-message]

	随后将立即设置 SendGrid 帐户，并将显示消息“成功购买了外接程序 SendGrid”。此时已创建你的帐户和凭据。此时你已准备好发送电子邮件。

	若要修改订阅计划或查看 SendGrid 联系人设置，请单击 SendGrid 服务的名称以打开 SendGrid 应用商店仪表板。

	![sendgrid-add-on-dashboard][sendgrid-add-on-dashboard]

	若要使用 SendGrid 发送电子邮件，必须提供帐户凭据（用户名和密码）。

### 查找 SendGrid 凭据 ###

1. 单击“连接信息”。

	![sendgrid-connection-info-button][sendgrid-connection-info-button]

2. 在“连接信息”对话框中，复制“密码”和“用户名”，供本教程后面使用。

	![sendgrid-connection-info][sendgrid-connection-info]

	若要设置电子邮件传达能力设置，请单击“管理”按钮。此操作将打开 Sendgrid.com Web 界面，从中可登录并打开 SendGrid 控制面板。

	![sendgrid-control-panel][sendgrid-control-panel]

	有关 SendGrid 入门的详细信息，请参阅[ SendGrid 入门][]。

<!--images-->

[command-bar-new]: ./media/sendgrid-sign-up/sendgrid_BAR_NEW.PNG
[sendgrid-store]: ./media/sendgrid-sign-up/sendgrid_offerings_store.png
[store-screen-2]: ./media/sendgrid-sign-up/sendgrid_store_scrn2.png
[store-screen-3]: ./media/sendgrid-sign-up/sendgrid_store_scrn3.png
[sendgrid-purchasing-message]: ./media/sendgrid-sign-up/sendgrid_purchasing_message.png
[sendgrid-add-on-dashboard]: ./media/sendgrid-sign-up/sendgrid_add-on_dashboard.png
[sendgrid-connection-info]: ./media/sendgrid-sign-up/sendgrid_connection_info.png
[sendgrid-connection-info-button]: ./media/sendgrid-sign-up/sendgrid_connection_info_button.png
[sendgrid-control-panel]: ./media/sendgrid-sign-up/sendgrid_control_panel.png

<!--Links-->

[ SendGrid 功能]: http://sendgrid.com/features
[Azure 管理门户]: https://manage.windowsazure.cn
[ SendGrid 入门]: http://sendgrid.com/docs
[SendGrid 设置过程]: https://support.sendgrid.com/hc/en-us/articles/200181628-Why-is-my-account-being-provisioned-
[所有 API]: https://sendgrid.com/docs/API_Reference/index.html

<!---HONumber=74-->