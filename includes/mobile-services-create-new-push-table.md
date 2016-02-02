
1. 登录到 [Azure 经典门户](https://manage.windowsazure.cn/)，单击“移动服务”，然后单击你的应用。

	![](./media/mobile-services-create-new-push-table/mobile-services-selection.png)

2. 单击“数据”选项卡，然后单击“创建”。

	![](./media/mobile-services-create-new-push-table/mobile-create-table.png)

	此时将显示“创建新表”对话框。

3. 为所有权限保留默认的“具有应用程序密钥的任何人”设置，在“表名”中键入 _Registrations_，然后单击勾选按钮。

	![](./media/mobile-services-create-new-push-table/mobile-create-registrations-table.png)

  这样可创建 **Registrations** 表，它存储用于发送推送通知的通道 URI。

接下来，您将修改应用以启用推送通知。

<!-- URLs -->
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0118_2016-->