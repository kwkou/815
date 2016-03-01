<properties
    pageTitle="使用 Transact-SQL 为 Azure SQL 数据库配置异地复制 | Microsoft Azure"
    description="使用 Transact-SQL 为 Azure SQL 数据库配置异地复制"
    services="sql-database"
    documentationCenter=""
    authors="carlrabeler"
    manager="jeffreyg"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="11/10/2015"
    wacn.date="12/22/2015"/>

# 使用 Transact-SQL 为 Azure SQL 数据库配置异地复制



> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-geo-replication-powershell)
- [Transact-SQL](/documentation/articles/sql-database-geo-replication-transact-sql)


本文说明如何使用 Transact-SQL 为 Azure SQL 数据库配置异地复制。


异地复制可让你在不同的数据中心位置（区域）最多创建 4 个副本（辅助）数据库。在数据中心发生服务中断或无法连接到主数据库时，可以使用辅助数据库。

异地复制仅适用于标准和高级数据库。

标准数据库可以有一个不可读取的辅助副本，并且必须使用建议的区域。高级数据库最多可以有四个位于任何可用区域并且可读取的辅助副本。


若要配置异地复制，需要提供以下各项：

- Azure 订阅 - 如果你没有 Azure 订阅，只需单击本页顶部的“免费试用”，然后返回以完成本文。
- 逻辑 Azure SQL 数据库服务器 <MyLocalServer> 和 SQL 数据库 <MyDB> - 你要复制到不同地理区域的主数据库。
- 一个或多个逻辑 Azure SQL 数据库服务器 <MySecondaryServer(n)> - 将在其中创建异地复制辅助数据库的伙伴服务器的逻辑服务器。
- 主数据库上 DBManager 的登录信息，拥有你要异地复制的本地数据库的 db\_ownership，以及你要配置异地复制的伙伴服务器上的 DBManager。
- 最新版本的 SQL Server Management Studio - 若要获取最新版本的 SQL Server Management Studio (SSMS)，请转到[下载 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。有关使用 SQL Server Management Studio 管理 Azure SQL 数据库逻辑服务器和数据库的详细信息，请参阅[使用 SQL Server Management Studio 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms)



## 连接到 SQL 数据库逻辑服务器

连接到 SQL 数据库需要你知道在 Azure 上的服务器名称，并事先为你要从中使用 Management Studio 进行连接的客户端 IP 地址创建防火墙规则。你可能需要登录到门户来获取此信息并完成此任务。

1.  登录到 [Azure 管理门户](http://manage.windowsazure.cn)。

2.  在左窗格中，单击“SQL 数据库”。

3.  在“SQL 数据库”主页上，单击页面顶部的“服务器”以列出与你的订阅关联的所有服务器。查找要连接到的服务器的名称，然后将它复制到剪贴板上。

	接下来，将您的 SQL 数据库防火墙配置为允许从您的本地计算机连接。通过将您的本地计算机 IP 地址添加到防火墙例外列表中来执行此操作。

1.  在“SQL 数据库”主页上，单击“服务器”，然后单击要连接到的服务器。

2.  单击页面顶部的“配置”。

3.  复制“当前客户端 IP 地址”中的 IP 地址。

4.  在“配置”页中，“允许的 IP 地址”包括三个框，你可以在其中指定规则名称和作为开始和结束值的 IP 地址范围。对于规则名称，您可以输入您的计算机的名称。对于开始和结束范围，将您的计算机的 IP 地址粘贴到两个框中，然后单击显示的复选框。

	规则名称必须是唯一的。如果这是您的开发计算机，则您可以将 IP 地址输入到 IP 范围开始框和 IP 范围结束框中。否则，您可能需要输入一组范围更广泛的 IP 地址来容纳来自您组织中的其他计算机的连接。

5. 单击页面底部的“保存”。

    **注意：**在防火墙设置的更改生效之前，可能最多有五分钟的延迟。

	您现在已准备好使用 Management Studio 连接到 SQL 数据库。

1.  在任务栏上，单击“开始”、指向“所有程序”、指向“Microsoft SQL Server 2014”，然后单击“SQL Server Management Studio”。

2.  在“连接到服务器”中，指定完全限定的服务器名称作为 <MyLocalServer>.database.chinacloudapi.cn。在 Azure 上，服务器名称是由字母数字字符组成的自动生成的字符串。

3.  选择“SQL Server 身份验证”。

4.  在“登录”框中，输入你在创建服务器时在门户中指定的 SQL Server 管理员登录名。

5.  在“密码”框中，输入你在创建服务器时在门户中指定的密码。

8.  单击“连接”以建立连接。



## 添加辅助数据库

可以使用 **ALTER DATABASE** 语句在伙伴服务器上创建异地复制的辅助数据库。在包含要复制的数据库服务器的 master 数据库上执行此语句。异地复制数据库（“主数据库”）具备与要复制的数据库相同的名称，并且默认与主数据库具有相同的服务级别。辅助数据库可以是可读或不可读，并且可以是单一数据库或弹性数据库。有关详细信息，请参阅 [ALTER DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。创建辅助数据库并设定种子之后，数据将开始以异步方式从主数据库复制。以下步骤说明如何使用 Management Studio 配置异地复制。提供使用单一数据库或弹性数据库创建不可读和可读辅助副本的步骤。

> [AZURE.NOTE]如果指定的伙伴服务器上存在辅助数据库（例如，由于当前存在或以前存在异地复制关系），命令将会失败。


### 添加不可读的辅助数据库（单一数据库）

可以使用以下步骤将不可读的辅助数据库创建为单一数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有<MySecondaryServer1>中的不可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer1> WITH (ALLOW_CONNECTIONS = NO);

4. 单击“执行”运行查询。


### 添加可读的辅助数据库（单一数据库）
可以使用以下步骤将可读辅助数据库创建为单一数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有辅助服务器上可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer2> WITH (ALLOW_CONNECTIONS = ALL);

4. 单击“执行”运行查询。



### 添加不可读的辅助数据库（弹性数据库）
可以使用以下步骤将不可读的辅助数据库创建为弹性数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有弹性池中辅助服务器上的不可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer3> WITH (ALLOW_CONNECTIONS = NO)
           , ELASTIC_POOL (name = MyElasticPool1);

4. 单击“执行”运行查询。



### 添加可读的辅助数据库（弹性数据库）
可以使用以下步骤将可读的辅助数据库创建为弹性数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有弹性池中辅助服务器上的可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer4> WITH (ALLOW_CONNECTIONS = NO)
           , ELASTIC_POOL (name = MyElasticPool2);

4. 单击“执行”运行查询。



## 删除辅助数据库

可以使用 **ALTER DATABASE** 语句永久终止辅助数据库与其主数据库之间的复制关系。此语句将在主数据库所在的 master 数据库上运行。终止关系后，辅助数据库将成为常规读写数据库。如果与辅助数据库的连接断开，命令将会成功，但辅助数据库必须等到连接恢复后才变为可读写。有关详细信息，请参阅 [ALTER DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。

使用以下步骤从异地复制合作关系中删除异地复制的辅助数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句来删除异地复制的辅助数据库。

        ALTER DATABASE <MyDB>
           REMOVE SECONDARY ON SERVER <MySecondaryServer1>;

4. 单击“执行”运行查询。


## 启动计划的故障转移，将辅助数据库升级为新的主数据库

可以使用 **ALTER DATABASE** 语句来升级辅助数据库，以计划好的方式使它成为新的主数据库，并将现有的主数据库降级为辅助数据库。此语句将在要升级的异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器中的 master 数据库上运行。此功能适用于计划的故障转移（例如灾难恢复演练期间），它要求主数据库可用。

该命令将执行以下工作流：

1. 暂时将复制切换为同步模式，导致所有未完成的事务被排送到辅助数据库并阻止所有新事务；

2. 切换异地复制合作关系中两个数据库的角色。

此顺序可确保不发生任何数据丢失。切换角色时，有一小段时间无法使用这两个数据库（大约为 0 到 25 秒）。在正常情况下，完成整个操作所需的时间应该少于一分钟。有关详细信息，请参阅 [ALTER DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。


> [AZURE.NOTE]发出该命令时，如果主数据库不可用，则命令将会失败并出现一条错误消息，指出没有可用的主服务器。在少数情况下，操作无法完成并且可能会出现停滞。在此情况下，用户可以执行强制故障转移命令并接受数据丢失。

使用以下步骤来启动计划的故障转移。

1. 在 Management Studio 中，连接到异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使异地复制的数据库成为具有<ElasticPool2>中<MySecondaryServer4>上可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB> FAILOVER;

4. 单击“执行”运行查询。



## 启动从主数据库到辅助数据库的非计划故障转移

可以使用 **ALTER DATABASE** 语句来升级辅助数据库，以非计划的方式使它成为新的主数据库，当主数据库不可用时，强制将现有主数据库降级为辅助数据库。此语句将在要升级的异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器中的 master 数据库上运行。

此功能适用于还原数据库的可用性至关重要并且可接受部分数据丢失的灾难恢复场合。调用强制故障转移时，指定的辅助数据库将立即成为主数据库，并开始接受写入事务。一旦原始主数据库能够与此新主数据库重新连接，将在原始主数据库上创建增量备份，旧的主数据库将变成新主数据库的辅助数据库；然后，它只是新主数据库的同步副本。

但是，由于辅助数据库上不支持时间点还原，发生强制故障转移之前，如果用户想要恢复已提交到旧主数据库但尚未复制到新主数据库的数据，则应该咨询支持人员来恢复这些丢失的数据。

> [AZURE.NOTE]如果在主数据库和辅助数据库联机时发出此命令，旧的主数据库将变为新的辅助数据库，但不会尝试数据同步。因此可能会发生部分数据丢失。


如果主数据库包含多个辅助数据库，该命令只会在执行命令的辅助服务器上成功。但是，其他辅助数据库不知道发生了强制故障转移。用户必须使用“删除辅助数据库”API 手动修复此配置，然后在这些附加的辅助数据库上重新配置异地复制。

使用以下步骤强制从异地复制合作关系中删除异地复制的辅助数据库。

1. 在 Management Studio 中，连接到异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使<MyLocalDB>成为具有<ElasticPool2>中<MySecondaryServer4>上的不可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>   FORCE_FAILOVER_ALLOW_DATA_LOSS;

4. 单击“执行”运行查询。

## 监视异地复制配置和运行状况

监视任务包括监视异地复制配置和监视数据复制运行状况。可以使用 master 数据库中的 **sys.dm\_geo\_replication\_links** 动态管理视图返回 Azure SQL 数据库逻辑服务器上每个数据库的所有现有复制链接的相关信息。此视图针对每个主数据库和辅助数据库之间的复制链接包含一行。可以使用 **sys.dm\_replication\_status** 动态管理视图针对当前参与复制链接的每个 Azure SQL 数据库返回一行。这包括主数据库和辅助数据库。如果给定主数据库有多个连续复制链接，此表将对每种关系包含一行。将在所有数据库（包括逻辑 master）中创建该视图。但是，在逻辑 master 中查询此视图会返回空集。可以使用 **sys.dm\_operation\_status** 动态管理视图来显示所有数据库操作的状态，包括复制链接的状态。有关详细信息，请参阅 [sys.geo\_replication\_links（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt575501.aspx)、[sys.dm\_geo\_replication\_link\_status（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt575504.aspx)和 [sys.dm\_operation\_status（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn270022.aspx)。

使用以下步骤监视异地复制合作关系。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下语句显示具有异地复制链接的所有数据库。

        SELECT database_id,start_date, partner_server, partner_database,  replication_state, is_target_role, is_non_redable_secondary FROM sys.geo_replication_links;

4. 单击“执行”运行查询。
5. 打开“数据库”文件夹、展开“系统数据库”、右键单击“MyDB”，然后单击“新建查询”。
6. 使用以下语句显示复制延迟和辅助数据库 MyDB 上次复制的开始时间。

        SELECT link_guid, partner_server, last_replication, replication_lag_sec FROM sys.dm_ replication_status

7. 单击“执行”运行查询。
8. 使用以下语句显示与 MyDB 数据库关联的最近异地复制操作。

        SELECT * FROM sys.dm_ operation_status where major_resource_is = 'MyDB'
        ORDER BY start_time DESC

9. 单击“执行”运行查询。


## 后续步骤

- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills)


## 其他资源

- [新异地复制功能的亮点](https://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication)
- [将云应用程序设计为使用异地复制实现业务连续性](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/documentation/services/sql-databases/)

<!---HONumber=Mooncake_1207_2015-->