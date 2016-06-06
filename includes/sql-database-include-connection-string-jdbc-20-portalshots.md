<!--
../includes/sql-database-include-connection-string-20-portalshots.md

Latest Freshness check:  2015-09-02 , GeneMi.

## Connection string
-->


### 从 Azure 门户获取连接字符串


使用 [Azure 管理门户](http://manage.windowsazure.cn)获取客户端程序与 Azure SQL 数据库进行交互所需的连接字符串：


1. 单击“所有项目”>“SQL 数据库”。

2. 在“SQL 数据库”边栏选项卡左上角附近的筛选器文本框中输入你的数据库的名称。

3. 单击数据库所对应的行。

4. 单击“查看 ADO .Net、ODBC、PHP 和 JDBC 的 SQL 数据库连接字符串”。

6. 如果你想要使用 JDBC 连接库，可复制标为“JDBC”的字符串。

7. 将连接字符串信息粘贴到客户端程序代码中。你需要将 {your\_password\_here} 替换为实际密码。

有关详细信息，请参阅：<br/>[连接字符串和配置文件](https://msdn.microsoft.com/zh-cn/library/ms378428.aspx)。

<!-- Image references. -->

[1-select-sql]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-select-sql.png


[2-select-database]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-select-database.PNG

[3-get-connection-string]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-jdbc.PNG


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

../includes/sql-database-include-connection-string-20-portalshots.md
../includes/sql-database-include-connection-string-30-compare.md
../includes/sql-database-include-connection-string-40-config.md
-->

<!---HONumber=Mooncake_0104_2016-->
