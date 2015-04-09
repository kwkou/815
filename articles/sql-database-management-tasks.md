<properties umbracoNaviHide="0" pageTitle="如何管理 SQL Database" metaKeywords="Azure SQL database, SQL database, manage sql database, add logins, connect to sql database" description="了解如何管理 Azure SQL Database." linkid="devnav-manage-services-cloud-services" urlDisplayName="Cloud Services" headerExpose="" footerExpose="" disqusComments="1" title="How to Manage SQL Database" authors="" />


<h1><a id="swap"></a>如何管理 SQL Database</h1>

本文档演示如何在 Azure SQL Database 中执行简单管理任务。 

##目录##

* [如何：使用 Management Studio 连接到 Azure 中的 SQL Database](#connect)
* [如何：将登录名和用户添加到 Azure 中的 SQL Database](#addlogins)


<h2><a id="connect"></a>如何：使用 Management Studio 连接到 Azure 中的 SQL Database</h2>

Management Studio 是一种管理工具，让您能够管理单个工作区中的多个 SQL Server 实例和服务器。如果您已经有了本地 SQL Server 实例，则可同时打开与本地实例和 Azure 上的逻辑服务器的连接，以便并行执行任务。

Management Studio 包括在管理门户中当前不可用的功能，例如语法检查程序以及保存脚本和命名查询以供重复使用的功能。SQL Database 只是一个表格格式数据流 (TDS) 终结点。适用于 TDS 的任何工具（包括 Management Studio）都可以有效地执行 SQL Database 操作。您为本地服务器开发的脚本将在 SQL Database 逻辑服务器上运行。 

在下一步中，您将使用 Management Studio 连接到 Azure 上的逻辑服务器。执行此步骤需要您具有 SQL Server Management Studio 2008 R2 或 2012 版本。如果您需要下载或连接到 Management Studio 的帮助，请参阅此站点上的[使用 Management Studio 管理 SQL Database][]。

在能够连接之前，有时必须创建防火墙例外，以便允许本地系统的端口 1433 上的出站请求。默认情况下，安全的计算机通常不打开端口 1433。 

##配置本地服务器的防火墙

1. 在"高级安全 Windows 防火墙"中，创建新的出站规则。

2. 选择"端口"****、指定"TCP 1433"、指定"允许连接"****，并确保"公用"****配置文件处于选中状态。

3. 提供一个有意义的名称，例如 *WindowsAzureSQLDatabase (tcp-out) port 1433*。 


##连接到逻辑服务器

1. 在 Management Studio 的"连接到服务器"中，确保"数据库引擎"处于选中状态，然后按以下格式输入逻辑服务器名称：*servername*.database.chinacloudapi.cn

	您还可在管理门户中的服务器仪表板上的"管理 URL"中获取完全限定服务器名称。

2. 在"身份验证"中，选择"SQL Server 身份验证"****，然后输入您在配置逻辑服务器时创建的管理员登录名。

3. 单击"选项"****。 

4. 在"连接到数据库"中，指定"master"****。


##连接到本地服务器

1. 在 Management Studio 的"连接到服务器"中，确保"数据库引擎"处于选中状态，然后按以下格式输入本地实例的名称：*servername*\\*instancename*。如果服务器是本地的，而且是默认实例，请输入 *localhost*。

2. 在"身份验证"中，选择"Windows 身份验证"****，然后输入作为 sysadmin 角色成员的 Windows 帐户。


<h2><a id="addlogins"></a>如何：将登录名和用户添加到 Azure 中的 SQL Database</h2>

部署数据库后，您需要配置登录名和分配权限。在下一步中，您将运行两个脚本。

对于第一个脚本，您将连接到 master 数据库并运行创建登录名的脚本。登录名将用于支持读写操作，并且委托操作任务，例如能够在没有"SA"权限的情况下运行系统查询。

您创建的登录名必须是 SQL Server 身份验证登录名。如果您已经有了使用 Windows 用户标识或声明标识的现成脚本，则该脚本将不在 SQL Database 上运行。

第二个脚本用于分配数据库用户权限。对于此脚本，您将连接到已在 Azure 上加载的数据库。

##创建登录名

1. 在 Management Studio 中，连接到 Azure 上的逻辑服务器、展开 Databases 文件夹、右键单击"master"****，然后选择"新建查询"****。

2. 使用以下 Transact-SQL 语句来创建登录名。将该密码替换为有效密码。分别选择每个登录名，然后单击"执行"****。为剩余的登录名重复以上操作。

<div style="width:auto; height:auto; overflow:auto"><pre>
    -- run on master, execute each line separately
    -- use this login to manage other logins on this server
    CREATE LOGIN sqladmin WITH password='&lt;ProvidePassword&gt;'; 
    CREATE USER sqladmin FROM LOGIN sqladmin;
    EXEC sp_addrolemember 'loginmanager', 'sqladmin';

    -- use this login to create or copy a database
    CREATE LOGIN sqlops WITH password='&lt;ProvidePassword&gt;';
    CREATE USER sqlops FROM LOGIN sqlops;
    EXEC sp_addrolemember 'dbmanager', 'sqlops';
</pre></div>


##创建数据库用户

1. 展开 Databases 文件夹、右键单击"school"****，然后选择"新建查询"****。

2. 使用以下 Transact-SQL 语句来添加数据库用户。将该密码替换为有效密码。 

<div style="width:auto; height:auto; overflow:auto"><pre>
    -- run on a regular database, execute each line separately
    -- use this login for read operations
    CREATE LOGIN sqlreader WITH password='&lt;ProvidePassword&gt;';
    CREATE USER sqlreader FROM LOGIN sqlreader;
    EXEC sp_addrolemember 'db_datareader', 'sqlreader';

    -- use this login for write operations
    CREATE LOGIN sqlwriter WITH password='&lt;ProvidePassword&gt;';
    CREATE USER sqlwriter FROM LOGIN sqlwriter;
    EXEC sp_addrolemember 'db_datawriter', 'sqlwriter';

    -- grant DMV permissions on the school database to 'sqlops'
    GRANT VIEW DATABASE STATE to 'sqlops';
</pre></div>

##查看和测试登录名

1. 在新查询窗口中，连接到"master"****并执行以下语句： 

        SELECT * from sys.sql_logins;

2. 在 Management Studio 中，右键单击"school"****数据库并选择"新建查询"****。

3. 在"查询"菜单上，指向"连接"****，然后单击"更改连接"****。

4. 以 *sqlreader* 身份登录。

5. 复制并尝试运行以下语句。您应该收到一条错误信息，指出对象不存在。

        INSERT INTO dbo.StudentGrade (CourseID, StudentID, Grade)
        VALUES (1061, 30, 9);

6. 打开第二个查询窗口，并将连接上下文更改为 *sqlwriter*。相同的查询现在应该成功运行。

您现在已经创建和测试了几个登录名。有关详细信息，请参阅[在 SQL Database 中管理数据库和登录名][]和[使用动态管理视图监视 SQL Database][]。

[管理数据库和 SQL Database 中的登录名]: http://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx
[使用动态管理视图监视 SQL Database]: http://msdn.microsoft.com/zh-cn/library/azure/ff394114.aspx
[使用 Management Studio 管理 SQL Database]: http://www.windowsazure.cn/zh-cn/develop/net/common-tasks/sql-azure-management/





