1.  登录到 [Azure 管理门户][]，单击“移动服务” ，然后单击你的应用程序。

    ![][0]

2.  单击“数据”选项卡 ，然后单击“创建” 。

    ![][1]

    此时将显示“创建新表” 对话框。

3.  为所有权限保留默认的“具有应用程序密钥的任何人” 设置，在“表名”中键入 *Registrations *，然后单击勾选按钮。

    ![][2]

这样可创建 "Registrations" 表，它存储用于发送推送通知的通道 URI。

接下来，你将修改应用程序以启用推送通知。

  [Azure 管理门户]: https://manage.windowsazure.cn/
  [0]: ./media/mobile-services-create-new-push-table/mobile-services-selection.png
  [1]: ./media/mobile-services-create-new-push-table/mobile-create-table.png
  [2]: ./media/mobile-services-create-new-push-table/mobile-create-registrations-table.png
