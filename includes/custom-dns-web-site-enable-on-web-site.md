在域名的记录已传播后，必须将其与您的 Web 应用关联。通过以下步骤使用 Web 浏览器启用域名。

> [WACOM.NOTE] 在之前的步骤中创建的 CNAME 记录通过 DNS 系统向外传播可能需要一段时间。在 CNAME 传播之前，无法将域名添加到您的 Azure Web 应用。如果您使用的是 A 记录，则在之前的步骤中创建的 **awverify** CNAME 传播之前，您无法将 A 记录域名添加到您的 Azure Web 应用。
> <p>可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 之类的服务确认该 CNAME 是否可用。

1. 在您的浏览器中，打开 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 在" Web 应用"选项卡中，单击您的 Web 应用的名称、选择"仪表板"，然后从页面底部选择"管理域"。

6. 使用"域名"文本框输入要与此 Web 应用相关联的域名。 

	![](./media/custom-dns-web-site/dncmntask-cname-7.png)

6. 单击右下角的复选标记以保存域名配置。

	在完成配置后，自定义域名将在您 Web 应用"配置"页的"域名"部分列出。

此时，您应该可以在浏览器中输入自定义域名，并查看它是否成功地将您转至 Azure Web 应用。 <!--HONumber=41-->
