
<!--
../includes/sql-database-include-ip-address-22-v12portal.md

Latest Freshness check:  2016-03-21 , daleche.

As of circa 2015-09-04, the following topics might include this include:
/documentation/articles/sql-database-configure-firewall-settings/
/documentation/articles/sql-database-connect-query/


## Server-level firewall rules

### Add a server-level firewall rule through the new Azure portal
-->


1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)（网址为 http://manage.windowsazure.cn/）。

2. 滚动并单击“SQL 数据库”。

3. 单击顶部显示的“服务器”，

4. 单击“配置”。 

5. 在“允许的 IP 地址”部分添加你想要启用的范围的下限和上限 IP 地址值。
	- 为方便起见，可以让下限值以 **.0** 结尾，让上限值以 **.255** 结尾。



<!-- Image references. -->

[b21-FindServerInPortal]: ./media/sql-database-include-ip-address-22-v12portal/firewall-ip-b21-v12portal-findsvr.png

[b31-SettingsFirewallNavig]: ./media/sql-database-include-ip-address-22-v12portal/firewall-ip-b31-v12portal-settingsfirewall.png

[b41-AddRange]: ./media/sql-database-include-ip-address-22-v12portal/firewall-ip-b41-v12portal-addrange.png



<!--
These includes/ files are a sequenced set, but you can pick and choose:

../includes/sql-database-include-ip-address-22-v12portal.md
? ../includes/sql-database-include-ip-address-*.md
-->

<!---HONumber=Mooncake_0503_2016-->
