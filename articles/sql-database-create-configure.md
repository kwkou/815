<properties linkid="manage-services-how-to-configure-a-sqldb" urlDisplayName="How to configure" pageTitle="如何配置 SQL Database - Azure" metaKeywords="Azure creating SQL Server, Azure configuring SQL Server" description="了解如何在 Azure 中使用 SQL Server 创建和配置逻辑服务器。" metaCanonical="" services="sql-database" documentationCenter="" title="How to Create and Configure SQL Database" authors="" solutions="" manager="" editor="" />



<h1><a id="configLogical"></a>如何创建和配置 Azure SQL Database</h1>

在此主题中，你将要使用 Azure 管理门户的"快速创建"****选项创建并配置一个新的 Azure SQL 数据库。此过程演示如何使用现有服务器创建 SQL 数据库，并演示如何根据需要创建新的服务器。

> [WACOM.NOTE] 使用"快速创建"创建 SQL Database 会设置一个标准 (S0) 数据库****。若要在除标准 (S0) 以外的服务层和性能级别创建 SQL Database，请使用"自定义创建"****。有关使用"自定义创建"****创建 Azure SQL Database 的详细信息，请参阅 [Microsoft Azure SQL Database 入门](/zh-cn/documentation/articles/sql-database-get-started/)。

##目录##
* [如何：创建 Azure SQL Database](#createDatabase)
* [如何：配置逻辑服务器的防火墙](#configFWLogical)


<a id="createDatabase"></a>

##如何：创建 Azure SQL Database

1. 登录到[管理门户](http://manage.windowsazure.cn)。

2. 在页面底部，单击"新建"****。

	![Click SQL Databases][1]

3. 依次单击"数据服务"、"SQL DATABASE"和"快速创建"************。

	![Click New, Data Services, and Quick Create][2]
	 
5. 在"快速创建"中，输入新数据库的名称，选择一个订阅，然后从"服务器"列表中选择服务器（跳到下一步以创建新服务器********）。

	![Create a new SQL Database in an existing server][7]

	或者，你可以通过选择"新建 SQL 数据库服务器"来创建一个新的服务器****。
    ![Create a new SQL Database and a new server][8]

	1. 选择区域。区域将确定服务器的地理位置。区域不能随意切换，因此要选择一个对此服务器有效的区域。选择一个最靠近你的位置。将 Azure 应用程序和数据库放置在同一区域可以降低出口带宽成本以及减少数据延迟情况。
	2. 输入一个没有空格的单词作为登录名。 

		SQL Database 使用 SQL 身份验证进行加密连接。将使用你提供的名称创建一个分配给 sysadmin 固定服务器角色的新 SQL Server 身份验证登录名。 

		登录名不能电子邮件地址、Windows 用户帐户或 Windows Live ID。SQL Database 既不支持声明，也不支持 Windows 身份验证。 
	3.提供由大小写值以及数字或符号共同组成的 8 个以上字符的强密码。

	


9. 完成后，请单击页面底部的"创建 SQL DATABASE"复选标记****。

###自动生成服务器名称

请注意，如果你创建了新服务器，则你并未指定服务器名称。SQL Database 会自动生成服务器名称以确保没有重复的 DNS 条目。服务器名称是一个由 10 个字符组成的字母数字字符串。你不能更改 SQL Database 服务器的名称。

在下一步中，你将配置防火墙以便允许你网络上运行的应用程序通过建立连接来访问相关数据。

<a id="configFWLogical"></a>

##如何：配置逻辑服务器的防火墙

1. 在[管理门户](http://manage.windowsazure.cn)中，单击"SQL 数据库"****，然后单击"服务器"****

	![Click Servers][4]
2. 在列表中单击你刚刚创建的服务器。

2. 单击"配置"****。

	![Click Configure][5]

3. 在"允许的 IP 地址"****部分中，单击"添加到允许的 IP 地址"****。它是你的路由器或代理服务器当前侦听的 IP 地址。SQL Database 会检测当前连接所使用的 IP 地址，并创建一个接受来自该设备的连接请求的防火墙规则。 
	![Click Add to the allowed IP addresses][6]

	生成规则的默认名称。根据需要编辑该名称。 
	

4. 当你从其他计算机连接到数据库时，必须创建一个新规则以允许其 IP 地址进行连接。使用"开始"和"结束"框创建 IP 地址的范围。

	如果客户端计算机使用动态分配的 IP 地址，你可以指定较宽范围的地址，使其足以包含分配给你网络中计算机的 IP 地址。从较窄的范围开始，然后仅在你需要时对其进行扩展。

7. 单击页面底部的"保存"以完成该步骤****。 

现在，你拥有 SQL Database、逻辑服务器、允许来自你的 IP 地址的入站连接的防火墙规则以及管理员登录名。 

**注意：**刚刚创建的逻辑服务器是虚拟的，将动态托管在数据中心的物理服务器上。删除该服务器是不可恢复的操作。请务必备份随后上载到该服务器的任何数据库。


<!--Image references-->
[1]: ./media/sql-database-create-configure/click-new.png
[2]: ./media/sql-database-create-configure/new-data-services-sql-storage-quick-create.png
[3]: ./media/sql-database-create-configure/server-settings.png
[4]: ./media/sql-database-create-configure/click-servers.png
[5]: ./media/sql-database-create-configure/click-configure.png
[6]: ./media/sql-database-create-configure/allowed-ip-addresses.png
[7]: ./media/sql-database-create-configure/quick-create-existing-server.png
[8]: ./media/sql-database-create-configure/quick-create-new-server.png



