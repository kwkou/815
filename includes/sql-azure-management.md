
# 使用 SQL Server Management Studio 管理 Azure SQL 数据库 

你可以使用 SQL Server Management Studio (SSMS) 来管理 Azure SQL 数据库逻辑服务器与数据库。本主题将指导你使用 SSMS 完成常见任务。在开始之前，你应该已在 Azure SQL 数据库中创建了逻辑服务器和数据库。若要开始，请先阅读[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)，然后返回此处。

建议你每当使用 Azure SQL 数据库时，都使用最新版本的 SSMS。请访问[下载 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 以获取最新版本的 SSMS。


## 连接到 SQL 数据库逻辑服务器

连接到 SQL Database 需要您知道在 Azure 上的服务器名称。您可能需要登录到门户来获取此信息。

1.  登录到 [Azure 经典管理门户](http://manage.windowsazure.cn)。

2.  在左窗格中，单击“SQL 数据库”。

3.  在“SQL 数据库”主页上，单击页面顶部的“服务器”以列出与你的订阅关联的所有服务器。查找要连接到的服务器的名称，然后将它复制到剪贴板上。

	接下来，将您的 SQL 数据库 防火墙配置为允许从您的本地计算机连接。通过将您的本地计算机 IP 地址添加到防火墙例外列表中来执行此操作。

1.  在“SQL 数据库”主页上，单击“服务器”，然后单击要连接到的服务器。

2.  单击页面顶部的“配置”。

3.  复制“当前客户端 IP 地址”中的 IP 地址。

4.  在“配置”页中，“允许的 IP 地址”包括三个框，你可以在其中指定规则名称和作为开始和结束值的 IP 地址范围。对于规则名称，您可以输入您的计算机的名称。对于开始和结束范围，将您的计算机的 IP 地址粘贴到两个框中，然后单击显示的复选框。

	规则名称必须是唯一的。如果这是您的开发计算机，则您可以将 IP 地址输入到 IP 范围开始框和 IP 范围结束框中。否则，您可能需要输入一组范围更广泛的 IP 地址来容纳来自您组织中的其他计算机的连接。
 
5. 单击页面底部的“保存”。

    **注意：**在防火墙设置的更改生效之前，可能最多有五分钟的延迟。

	您现在已准备好使用 Management Studio 连接到 SQL 数据库。

1.  在任务栏上，单击“开始”、指向“所有程序”、指向“Microsoft SQL Server 2014”，然后单击“SQL Server Management Studio”。

2.  在**“连接到服务器”**中，指定完全限定的服务器名称作为 *serverName*.database.chinacloudapi.cn。在 Azure 上，服务器名称是由字母数字字符组成的自动生成的字符串。

3.  选择“SQL Server 身份验证”。

4.  在“登录”框中，输入你在创建服务器时在门户中指定的 SQL Server 管理员登录名。

5.  在“密码”框中，输入你在创建服务器时在门户中指定的密码。

8.  单击“连接”以建立连接。

包含最新更新的 SQL Server 2014 SSMS 为创建和修改 Azure SQL 数据库等任务提供扩展的支持。此外，你也可以使用 Transact-SQL 语句来完成这些任务。下面的步骤提供了这些语句的示例。有关将 Transact-SQL 与 SQL 数据库结合使用的详细信息（包括有关受支持命令的详细信息），请参阅 [Transact-SQL 参考（SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/bb510741.aspx)。

<h2><a id="Step3" name="Step3"> </a>步骤 3：创建并管理数据库</h2>

连接到 **master** 数据库时，你可以在服务器上创建新数据库以及修改或删除现有数据库。下面的步骤介绍如何通过 Management Studio 完成几个常见数据库管理任务。若要执行这些任务，请确保已使用在设置服务器时创建的服务器级别主体登录名连接到 **master** 数据库。

若要在 Management Studio 中打开查询窗口，请打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

-   使用 **CREATE DATABASE** 语句可创建新数据库。有关详细信息，请参阅 [CREATE DATABASE（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)。以下语句将创建名为 **myTestDB** 的新数据库，并指定它是默认大小上限为 250 GB 的“标准 S0 版本”数据库。

        CREATE DATABASE myTestDB
        (EDITION='Standard',
         SERVICE_OBJECTIVE='S0');

单击“执行”运行查询。

-   例如，如果要更改数据库的名称和版本，请使用 **ALTER DATABASE** 语句修改现有数据库。有关详细信息，请参阅 [ALTER DATABASE（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/ms174269.aspx)。以下语句将修改你在前一步骤中创建的数据库，以将版本更改为“标准 S1”。

        ALTER DATABASE myTestDB
        MODIFY
        (SERVICE_OBJECTIVE='S1');

-   使用 **DROP DATABASE** 语句可删除现有数据库。有关详细信息，请参阅 [DROP DATABASE（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/ms178613.aspx)。以下语句会删除 **myTestDB** 数据库，但现在请不要删除此数据库，因为你将在下一步骤中用它来创建登录名。

        DROP DATABASE myTestBase;

-   master 数据库具有你可用于查看有关所有数据库的详细信息的 **sys.databases** 视图。若要查看所有现有数据库，请执行以下语句：

        SELECT * FROM sys.databases;

-   在 SQL 数据库中，不支持将 **USE** 语句用于在数据库之间切换。您需要改为建立直接到目标数据库的连接。

>[AZURE.NOTE]创建或修改数据库的许多 Transact-SQL 语句必须在其自己的批处理中运行，无法与其他 Transact-SQL 语句分组在一起。有关详细信息，请参阅上面列出的链接中提供的特定于语句的信息。

<h2><a id="Step4" name="Step4"> </a>步骤4：创建并管理登录名</h2>

**master** 数据库跟踪登录名以及哪些登录名有权创建数据库或其他登录名。通过使用你在设置服务器时创建的服务器级别主体登录名连接到 **master** 数据库来管理登录名。你可以使用 **CREATE LOGIN**、**ALTER LOGIN** 或 **DROP LOGIN** 语句对将管理整个服务器上的登录的 master 数据库执行查询。有关详细信息，请参阅[在 SQL 数据库中管理数据库和登录名](http://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx)。


-   使用 **CREATE LOGIN** 语句可创建新的服务器级别登录名。有关详细信息，请参阅 [CREATE LOGIN（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/ms189751.aspx)。以下语句将创建一个名为 **login1** 的新登录名。将 **password1** 替换为你选择的密码。

        CREATE LOGIN login1 WITH password='password1';

-   使用 **CREATE USER** 语句授予数据库级别权限。必须在 **master** 数据库中创建所有登录名，但要使登录名可连接到不同数据库，必须对该数据库使用 **CREATE USER** 语句授予此登录名数据库级别权限。有关详细信息，请参阅 [CREATE USER（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/ms173463.aspx)。

-   若要为 login1 提供对名为 **myTestDB** 的数据库的权限，请完成以下步骤：

 1.  若要刷新“对象资源管理器”以查看你刚创建的 **myTestDB** 数据库，请在“对象资源管理器”中右键单击服务器名称，然后单击“刷新”。  

     如果你关闭了连接，可以通过选择“文件”菜单上的“连接对象资源管理器”来重新连接。

 2. 右键单击“myTestDB”数据库并选择“新建查询”。

    3.  对 myTestDB 数据库执行以下语句来创建与服务器级别登录名 **login1** 对应的名为 **login1User** 的数据库用户。

            CREATE USER login1User FROM LOGIN login1;

-   使用 **sp\_addrolemember** 存储过程为用户帐户提供对数据库的适当级别的权限。有关详细信息，请参阅 [sp\_addrolemember (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187750.aspx)。下面的语句通过将 **login1User** 添加到 **db\_datareader** 角色，为 **login1User** 提供对数据库的只读权限。

        exec sp_addrolemember 'db_datareader', 'login1User';    

-   例如，如果要更改用于登录的密码，请使用 **ALTER LOGIN** 语句修改现有登录名。有关详细信息，请参阅 [ALTER LOGIN（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/ms189828.aspx)。应对 **master** 数据库运行 **ALTER LOGIN** 语句。切换回连接到该数据库的查询窗口。

    以下语句将修改 **login1** 登录名来重置密码。将 **newPassword** 替换为你选择的密码，并将 **oldPassword** 替换为登录名的当前密码。

        ALTER LOGIN login1
        WITH PASSWORD = 'newPassword'
        OLD_PASSWORD = 'oldPassword';

-   使用 **DROP LOGIN** 语句可删除现有登录名。删除处于服务器级别的登录名还会删除任何关联的数据库用户帐户。有关详细信息，请参阅 [DROP DATABASE（SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/ms178613.aspx)。应对 **master** 数据库运行 **DROP LOGIN** 语句。以下语句将删除 **login1** 登录名。

        DROP LOGIN login1;

-   master 数据库具有可用于查看登录名的 **sys.sql\_logins** 视图。若要查看所有现有登录名，请执行以下语句：

        SELECT * FROM sys.sql_logins;

<h2><a id="Step5" name="Step5"> </a>步骤 5：使用动态管理视图监视 SQL 数据库</h2>

SQL Database 支持多个您可用于监视单个数据库的动态管理视图。下面是您可通过这些视图检索的监视器数据类型的一些示例。有关完整的详细信息和更多用法示例，请参阅[使用动态管理视图监视 SQL 数据库](https://msdn.microsoft.com/zh-cn/library/azure/ff394114.aspx)。

-   查询动态管理视图需要 **VIEW DATABASE STATE** 权限。若要向特定数据库用户授予 **VIEW DATABASE STATE** 权限，请连接到要使用服务器级别主体登录名管理的数据库并对该数据库执行以下语句：

        GRANT VIEW DATABASE STATE TO login1User;

-   使用 **sys.dm\_db\_partition\_stats** 视图计算数据库大小。**sys.dm\_db\_partition\_stats** 视图返回数据库中每个分区的页和行计数信息，你可以使用这些信息来计算数据库大小。下面的查询返回您的数据库的大小（以 MB 为单位）：

        SELECT SUM(reserved_page_count)*8.0/1024
        FROM sys.dm_db_partition_stats;   

-   使用 **sys.dm\_exec\_connections** 和 **sys.dm\_exec\_sessions** 视图可检索与数据库关联的当前用户连接和内部任务的相关信息。下面的查询返回有关当前连接的信息：

        SELECT
            e.connection_id,
            s.session_id,
            s.login_name,
            s.last_request_end_time,
            s.cpu_time
        FROM
            sys.dm_exec_sessions s
            INNER JOIN sys.dm_exec_connections e
              ON s.session_id = e.session_id;

-   使用 **sys.dm\_exec\_query\_stats** 视图可检索缓存的查询计划的总体性能统计信息。下面的查询返回有关按平均 CPU 时间排名的前五个查询的信息。

        SELECT TOP 5 query_stats.query_hash AS "Query Hash",
            SUM(query_stats.total_worker_time), SUM(query_stats.execution_count) AS "Avg CPU Time",
            MIN(query_stats.statement_text) AS "Statement Text"
        FROM
            (SELECT QS.*,
            SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
            ((CASE statement_end_offset
                WHEN -1 THEN DATALENGTH(ST.text)
                ELSE QS.statement_end_offset END
                    - QS.statement_start_offset)/2) + 1) AS statement_text
             FROM sys.dm_exec_query_stats AS QS
             CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
        GROUP BY query_stats.query_hash
        ORDER BY 2 DESC;

<!---HONumber=69-->