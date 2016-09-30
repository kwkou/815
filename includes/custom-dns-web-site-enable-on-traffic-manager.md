在你的域名的记录已传播后，你应该能够使用浏览器验证你的自定义域名是否可用于访问 Azure 的 Web 应用。

> [AZURE.NOTE] CNAME 通过 DNS 系统向外传播可能需要一段时间。可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 之类的服务确认该 CNAME 是否可用。

如果尚未将你的 Web 应用添加为流量管理器终结点，则必须在名称解析工作前执行此操作，因为自定义域名将路由到流量管理器。然后，流量管理器将路由到的你 Web 应用。根据[添加或删除终结点](/documentation/articles/traffic-manager-endpoints/)中的信息在流量管理器配置文件中将你的 Web 应用添加为终结点。

> [AZURE.NOTE] 如果在添加终结点时你的 Web 应用未列出，请验证是否已将其配置为**标准** App Service 计划模式。必须将 Web 应用配置为**标准**模式，流量管理器才起作用。

<!---HONumber=Mooncake_0118_2016-->