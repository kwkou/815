
<!--
includes/sql-database-include-connection-string-20-portalshots.md

Latest Freshness check:  2015-09-02 , GeneMi.

## Connection string
-->


### 从 Azure 门户获取连接字符串


使用 [Azure 预览门户](https://manage.windowsazure.cn/)获取客户端程序与 Azure SQL 数据库进行交互所需的连接字符串：


1. 单击“浏览”>“SQL 数据库”。

2. 在“SQL 数据库”边栏选项卡左上角附近的筛选器文本框中输入你的数据库的名称。

3. 单击数据库所对应的行。

4. 在数据库的边栏选项卡显示以后，为了观看方便，你可以单击标准的最小化控件，以便折叠用于浏览和数据库筛选的边栏选项卡。
 
	![用于隔离数据库的筛选器][10-FilterDatabase]

5. 在数据库边栏选项卡上，单击“显示数据库连接字符串”。

6. 如果你想要使用 ADO.NET 连接库，可复制标为“ADO”的字符串。
 
	![复制数据库的 ADO 连接字符串][20-CopyAdoConnectionString]
 
7. 通过这种或那种格式，将连接字符串信息粘贴到客户端程序代码中。



有关详细信息，请参阅<br/>[连接字符串和配置文件](http://msdn.microsoft.com/zh-cn/library/ms254494.aspx)。



<!-- Image references. -->

[10-FilterDatabase]: ./media/sql-database-include-connection-string-20-portalshots/connqry-connstr-a.png

[20-CopyAdoConnectionString]: ./media/sql-database-include-connection-string-20-portalshots/connqry-connstr-b.png


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->

<!---HONumber=74-->