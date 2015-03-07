在您的域名的记录已传播后，您应该能够使用浏览器验证您的自定义域名是否可用于访问您的网站。

> [WACOM.NOTE] CNAME 通过 DNS 系统向外传播可能需要一段时间。可使用 <a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> 之类的服务确认该 CNAME 是否可用。

如果尚未将您的网站添加为 Traffic Manager 终结点，则必须在名称解析工作前执行此操作，因为自定义域名将路由到 Traffic Manager。然后，Traffic Manager 将路由到的您网站。使用[添加或删除终结点](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh744839.aspx)中的信息在 Traffic Manager 配置文件中将你的网站添加为终结点。

> [WACOM.NOTE] 如果在添加终结点时您的网站未列出，请验证是否已将其配置为标准模式。必须将网站配置为标准模式，Traffic Manager 才起作用。<!--HONumber=41-->
