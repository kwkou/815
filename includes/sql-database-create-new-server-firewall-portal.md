
<!--
includes/sql-database-create-new-server-firewall-portal.md

Latest Freshness check:  2016-08-01 , rickbyh.

As of circa 2016-04-11, the following topics might include this include:
articles/sql-database/sql-database-get-started-tutorial.md
articles/sql-database/sql-database-configure-firewall-settings

-->
## 创建新的 Aure SQL 服务器级防火墙

在 Azure 门户预览中使用以下步骤来创建服务器级别防火墙规则，以允许从单个 IP 地址（客户端计算机）或整个 IP 地址范围连接到 SQL 数据库逻辑服务器。

1. 如果当前未连接，请连接到 [Azure 门户预览](http://portal.azure.cn)。
2. 在“默认”边栏选项卡中，单击“SQL Server”。

  	![新的服务器防火墙](./media/sql-database-create-new-server-firewall-portal/sql-database-create-new-server-firewall-portal-1.png)  


3. 在“SQL Server”边栏选项卡中，单击要在其上创建防火墙规则的服务器。

 	![新的服务器防火墙](./media/sql-database-create-new-server-firewall-portal/sql-database-create-new-server-firewall-portal-2.png)  

           
4. 检查服务器的属性。

 	![新的服务器防火墙](./media/sql-database-create-new-server-firewall-portal/sql-database-create-new-server-firewall-portal-3.png)  

      
5. 在“设置”边栏选项卡中，单击“防火墙”。

 	![新的服务器防火墙](./media/sql-database-create-new-server-firewall-portal/sql-database-create-new-server-firewall-portal-4.png)  

    

 	> [AZURE.NOTE] 还可从“数据库”边栏选项卡的工具栏中访问服务器级“防火墙设置”边栏选项卡。

6. 单击“添加客户端 IP”，让 Azure 为客户端 IP 地址创建规则。

      ![新的服务器防火墙](./media/sql-database-create-new-server-firewall-portal/sql-database-create-new-server-firewall-portal-5.png)  


7. （可选）若要允许访问某个 IP 地址范围，请单击添加的 IP 地址以编辑防火墙地址。

      ![新的服务器防火墙](./media/sql-database-create-new-server-firewall-portal/sql-database-create-new-server-firewall-portal-6.png)  

    
8. 单击“保存”以创建服务器级防火墙规则。

     ![新的服务器防火墙](./media/sql-database-create-new-server-firewall-portal/sql-database-create-new-server-firewall-portal-7.png)  


	>[AZURE.IMPORTANT] 客户端 IP 地址可能会不定时地更改，因此可能需要创建新的防火墙规则才能访问服务器。可通过使用 [Bing](http://www.bing.com/search?q=my%20ip%20address) 来查看 IP 地址。然后添加一个 IP 地址或 IP 地址范围。有关详细信息，请参阅[管理防火墙设置](/documentation/articles/sql-database-configure-firewall-settings/#manage-existing-server-level-firewall-rules-through-the-azure-portal)。

<!---HONumber=Mooncake_1010_2016-->