在你的域名的记录已传播后，你应该能够使用浏览器验证你的自定义域名是否可用于访问 Azure 的 Web 应用。

> [AZURE.NOTE] CNAME 通过 DNS 系统向外传播可能需要一段时间。可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 之类的服务确认该 CNAME 是否可用。

如果尚未将你的 Web 应用添加为流量管理器终结点，则必须在名称解析工作前执行此操作，因为自定义域名将路由到流量管理器。然后，流量管理器将路由到的你 Web 应用。根据[添加或删除终结点](/documentation/articles/traffic-manager-endpoints/)中的信息在流量管理器配置文件中将你的 Web 应用添加为终结点。

> [AZURE.NOTE] 如果在添加终结点时你的 Web 应用未列出，请验证是否已将其配置为**标准** App Service 计划模式。必须将 Web 应用配置为**标准**模式，流量管理器才起作用。

1. 在你的浏览器中，打开 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 在“Web Apps”选项卡中，单击 Web 应用的名称，选择“设置”，然后选择“自定义域和 SSL”

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

3. 在“自定义域和 SSL”边栏选项卡中，单击“自带外部域”。

	![](./media/custom-dns-web-site/dncmntask-cname-7.png)

4. 使用“域名”文本框输入要与此 Web 应用相关联的流量管理器域名 (contoso.trafficmanager.cn)。

5. 单击“保存”以保存域名配置。

	在完成配置后，自定义域名将在 Web 应用的“域名”部分列出。

此时，你应该可以在浏览器中输入流量管理器域名 (contoso.trafficmanager.cn)，并查看是否成功地转到该 Web 应用。

<!---HONumber=Mooncake_0118_2016-->