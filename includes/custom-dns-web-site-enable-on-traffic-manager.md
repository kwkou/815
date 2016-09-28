在传播域名记录后，应可使用浏览器验证自定义域名能否用于访问 Azure App Service 中的 Web 应用。

> [AZURE.NOTE] CNAME 通过 DNS 系统向外传播可能需要一段时间。可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 等服务验证该 CNAME 是否可用。

如果尚未将 Web 应用添加为流量管理器终结点，必须在解析名称前执行此操作，因为自定义域名将路由到流量管理器。然后，流量管理器路由到 Web 应用。根据[添加或删除终结点](/documentation/articles/traffic-manager-endpoints/)中的信息，在流量管理器配置文件中将 Web 应用添加为终结点。

> [AZURE.NOTE] 如果在添加终结点时 Web 应用未列出，请验证是否已将其配置为**标准**应用服务计划模式。必须将 Web 应用设为**标准**模式才可使用流量管理器。

1. 在浏览器中，打开 [Azure 门户预览](https://portal.azure.cn)。

1. 在“Web 应用”选项卡中，单击 Web 应用的名称，选择“设置”，然后选择“自定义域和 SSL”。

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

1. 在“自定义域和 SSL”边栏选项卡中，单击“自带外部域”。

	![](./media/custom-dns-web-site/dncmntask-cname-7.png)

1. 使用“域名”文本框输入要与此 Web 应用相关联的流量管理器域名。

	![](./media/custom-dns-web-site/dncmntask-cname-8.png)

1. 单击“保存”以保存域名配置。

	完成配置后，自定义域名将在 Web 应用的“域名”部分列出。

此时，应可在浏览器中输入流量管理器域名，并查看它是否成功转至 Web 应用。

<!---HONumber=Mooncake_0919_2016-->