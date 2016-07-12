
<!--
../includes/sql-database-include-ip-address-22-v12portal.md

Latest Freshness check:  2016-03-21 , daleche.

As of circa 2015-09-04, the following topics might include this include:
/documentation/articles/sql-database-configure-firewall-settings/
/documentation/articles/sql-database-connect-query


## Server-level firewall rules

### Add a server-level firewall rule through the new Azure portal
-->


1. 登录到 [Azure 门户](https://manage.windowsazure.cn)（网址为 http://manage.windowsazure.cn/）。

2. 在左侧的横幅中，单击“浏览全部”。此时会显示“浏览”边栏选项卡。

3. 滚动并单击“SQL Server”。此时会显示“SQL Server”边栏选项卡。

4. 为方便起见，可单击以前的“浏览”边栏选项卡上的最小化控件。

5. 在筛选器文本框中，开始键入你的服务器的名称。此时会显示你的行。

6. 单击服务器所对应的行。此时会显示服务器的边栏选项卡。

7. 在服务器边栏选项卡上单击“设置”。此时会显示“设置”边栏选项卡。

8. 单击“防火墙”。此时会显示“防火墙设置”边栏选项卡。

9. 单击“添加客户端 IP”。在第一个文本框中键入新规则的名称。

10. 键入你想要启用的范围的下限和上限 IP 地址值。
	- 为方便起见，可以让下限值以 **.0** 结尾，让上限值以 **.255** 结尾。

11. 单击“保存”。



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
