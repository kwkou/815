<properties linkid="dev-net-how-to-sql-azure" urlDisplayName="SQL Database" pageTitle="How to use SQL Database (.NET) - Azure feature guide" metaKeywords="Get started SQL Azure, Getting started SQL Azure, SQL Azure database connection, SQL Azure ADO.NET, SQL Azure ODBC, SQL Azure EntityClient" description="Get started with SQL Database. Learn how to create a SQL Database instance and connect to it using ADO.NET, ODBC, and EntityClient Provider." metaCanonical="" services="sql-database" documentationCenter=".NET" title="How to use Azure SQL Database in .NET applications" authors="" solutions="" manager="" editor="" />

# 如何在 .NET 应用程序中使用 Azure SQL Database

本指南说明如何在 Azure SQL Database 上创建逻辑服务器和数据库实例，以及如何使用下列 .NET Framework 数据提供程序技术连接到数据库：ADO.NET、ODBC 和 EntityClient Provider。

## <a name="Whatis"></a>什么是 SQL Database

SQL Database 为 Azure 提供关系数据库管理系统并且基于 SQL Server 技术。使用 SQL Database 实例，你可以轻松地配置关系数据库解决方案并将其部署到云中，并且利用能够为企业级可用性、可缩放性和安全性提供内置数据保护和自愈优势的分布式数据中心。

## 目录

-   [登录 Azure][登录 Azure]
-   [创建和配置 SQL Database][创建和配置 SQL Database]
-   [连接到 SQL Database][连接到 SQL Database]
-   [使用 ADO.NET 进行连接][使用 ADO.NET 进行连接]
-   [使用 ODBC 进行连接][使用 ODBC 进行连接]
-   [使用 EntityClient Provider 进行连接][使用 EntityClient Provider 进行连接]
-   [后续步骤][后续步骤]

## <a name="PreReq1"></a>登录 Azure

SQL Database 在 Azure 上提供关系数据存储、访问和管理服务。若要使用它，你将需要一个 Azure 订阅。

1.  打开 Web 浏览器并浏览到 [][]<http://www.windowsazure.cn></a>。若要开始使用免费帐户，请单击右上角的“免费试用”并执行相应步骤。

2.  现已创建你的帐户。一切准备就绪，即可开始。

## <a name="PreReq2"></a><span class="short-header">创建和配置 SQL Database</span>

接下来，你需要创建和配置数据库及服务器。在 Azure 管理门户中，你可以先使用已修订的工作流创建数据库，然后配置服务器。

### 创建数据库实例和逻辑服务器

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  单击页面底部的“+新建”。

3.  单击“数据服务”。

4.  单击“SQL Database”。

5.  单击“自定义创建”。

6.  在“名称”中，输入数据库名称。

7.  选择版本、最大大小和排序规则。考虑到本指南的用途，你可以使用默认值。

    SQL Database 提供两个数据库版本。Web 版数据库的大小最大可达 5 GB。企业版数据库的大小最大可达 50 GB。最大大小是在首次创建数据库时指定的，稍后可以使用“更改数据库”对其进行更改。最大大小可限制数据库的大小。

    在 Azure 上创建的每个 SQL Database 实际上都具有三个副本。这样做是为了确保实现高可用性。故障转移是透明的并且是该服务的一部分。[服务级别协议][服务级别协议]为 SQL Database 提供 99.9% 的运行时间。

8.  在“服务器”中，选择“新建 SQL Database 服务器”。

9.  单击箭头，转到下一页。

10. 在“服务器设置”中，输入 SQL Server 身份验证登录名。

    SQL Database 使用 SQL 身份验证进行加密连接。将使用你提供的名称创建一个分配给 sysadmin 固定服务器角色的新 SQL Server 身份验证登录名。该登录名不能是电子邮件地址、Windows 用户帐户或 Windows Live ID。SQL Database 不支持声明，也不支持 Windows 身份验证。

11. 提供由大小写值以及数字或符号共同组成的 8 个以上字符的强密码。

12. 选择区域。区域将确定服务器的地理位置。区域不能随意切换，因此要选择一个对此服务器有效的区域。选择一个最靠近你的位置。将 Azure 应用程序和数据库放置在同一区域可以降低出口带宽成本以及减少数据延迟情况。

13. 确保“允许 Azure 服务访问服务器”选项处于选中状态，以便你能够使用 SQL Database 的管理门户、存储服务以及 Azure 上的其他服务连接到此数据库。

14. 完成后，请单击页面底部的复选标记。

请注意，你没有指定服务器名称。SQL Database 会自动生成服务器名称以确保没有重复的 DNS 条目。服务器名称是一个由 10 个字符组成的字母数字字符串。你不能更改 SQL Database 服务器的名称。

创建数据库后，单击它以打开其仪表板。仪表板提供可在应用程序代码中复制和使用的连接字符串。仪表板还显示了你在从 Management Studio 或其他管理工具中连接到数据库时所需要指定的管理 URL。

![图像][图像]

在下一步中，你将配置防火墙以便允许你网络上运行的应用程序通过建立连接来访问相关数据。

### 配置逻辑服务器的防火墙

1.  单击“SQL Database”，单击页面顶部的“服务器”，然后单击你刚才创建的服务器。

    ![图像 2][图像 2]

2.  单击**“配置”**。

3.  复制当前客户端 IP 地址。如果你从某网络进行连接，则为你的路由器或代理服务器侦听的 IP 地址。SQL Database 会检测当前连接所使用的 IP 地址，以便你可以创建一个接受来自该设备的连接请求的防火墙规则。

4.  将 IP 地址粘贴到“起始 IP 地址”和“结束 IP 地址”中，以确定允许访问服务器的地址范围。日后，如果你遇到指示该范围太窄的连接错误，则可以编辑此规则来扩大范围。

    如果客户端计算机使用动态分配的 IP 地址，你必须指定较宽范围的地址，使其足以包含分配给你网络中计算机的 IP 地址。从较窄的范围开始，然后仅在你需要时对其进行扩展。

5.  为防火墙规则输入名称，例如，你的计算机或公司的名称。

6.  单击该规则旁边的复选标记进行保存。

    ![图像 3][图像 3]

7.  单击页面底部的“保存”以完成该步骤。如果没有看到“保存”，请刷新浏览器页面。

现在，你拥有数据库实例、逻辑服务器、允许来自你的 IP 地址的入站连接的防火墙规则以及管理员登录名。你现在可以通过编程方式连接到数据库。

## <a name="Connect-DB"></a><span class="short-header">连接到 SQL Database</span>

本节介绍如何使用不同的 .NET Framework 数据提供程序连接到 SQL Database 实例。

如果你选择使用 Visual Studio 并且你的配置没有将 Azure Web 应用程序包含为前端，则无需在开发计算机上安装其他工具或 SDK。
你现在即可开始开发应用程序。

你可以使用 Visual Studio 中的所有相同设计器工具处理 SQL Database，就像可以使用这些工具处理 SQL Server 一样。服务器资源管理器允许你查看（但不能编辑）数据库对象。Visual Studio 实体数据模型设计器功能完备，你可以用它来创建针对 SQL Database 的用于 Entity Framework 的模型。

### <a name="using-sql-server"></a>使用用于 SQL Server 的 .NET Framework 数据提供程序

**System.Data.SqlClient** 命名空间是用于 SQL Server 的 .NET Framework 数据提供程序。

标准连接字符串类似于以下内容：

    Server=tcp:.database.chinacloudapi.cn;
    Database=;
    User ID=@;
    Password=;
    Trusted_Connection=False;
    Encrypt=True;

你可以使用 **SQLConnectionStringBuilder** 类生成连接字符串，如以下代码示例所示：

    SqlConnectionStringBuilder csBuilder;
    csBuilder = new SqlConnectionStringBuilder();
    csBuilder.DataSource = xxxxxxxxxx.database.chinacloudapi.cn;
    csBuilder.InitialCatalog = testDB;
    csBuilder.Encrypt = true;
    csBuilder.TrustServerCertificate = false;
    csBuilder.UserID = MyAdmin;
    csBuilder.Password = pass@word1;

如果事先知道连接字符串的元素，则可以将其存储在配置文件中并在运行时进行检索，从而构造一个连接字符串。下面是配置文件中的一个连接字符串示例：

    <connectionStrings>
      <add name="ConnectionString" 
           connectionString ="Server=tcp:xxxxxxxxxx.database.chinacloudapi.cn;Database=testDB;User ID=MyAdmin@xxxxxxxxxx;Password=pass@word1;Trusted_Connection=False;Encrypt=True;" />
    </connectionStrings>

若要检索配置文件中的连接字符串，你可以使用 **ConfigurationManager** 类：

    SqlConnectionStringBuilder csBuilder;
    csBuilder = new SqlConnectionStringBuilder(ConfigurationManager.ConnectionStrings["ConnectionString"].ConnectionString);
    After you have built your connection string, you can use the SQLConnection class to connect the SQL Database server:
    SqlConnection conn = new SqlConnection(csBuilder.ToString());
    conn.Open();

### <a name="using-ODBC"></a>使用用于 ODBC 的 .NET Framework 数据提供程序

**System.Data.Odbc** 命名空间是用于 ODBC 的 .NET Framework 数据提供程序。下面是一个 ODBC 连接字符串示例：

    Driver={SQL Server Native Client 10.0};
    Server=tcp:.database.chinacloudapi.cn;
    Database=;
    Uid=@;
    Pwd=;
    Encrypt=yes;

**OdbcConnection** 类表示与数据源的开放连接。下面是有关如何打开连接的代码示例：

    string cs = "Driver={SQL Server Native Client 10.0};" +
                "Server=tcp:xxxxxxxxxx.database.chinacloudapi.cn;" +
                "Database=testDB;"+
                "Uid=MyAdmin@xxxxxxxxxx;" +
                "Pwd=pass@word1;"+
                "Encrypt=yes;";

    OdbcConnection conn = new OdbcConnection(cs);
    conn.Open();

如果要在运行时构建连接字符串，你可以使用 **OdbcConnectionStringBuilder** 类。

### <a name="using-entity"></a>使用 EntityClient Provider

**System.Data.EntityClient** 命名空间是用于 Entity Framework 的 .NET Framework 数据提供程序。

使用 Entity Framework，开发人员可以通过针对概念应用程序模型进行编程（而不是直接针对关系存储架构进行编程）来创建数据访问应用程序。通过提供基础数据提供程序和关系数据库的 **EntityConnection**，可在特定于存储的 ADO.NET 数据提供程序的基础上构建 Entity Framework。

若要构造一个 **EntityConnection** 对象，你必须引用一组包含所需模型和映射的元数据，还要引用特定于存储的数据提供程序名称和连接字符串。**EntityConnection** 建立之后，即可通过概念模型生成的类访问实体。

下面是一个连接字符串示例：

    metadata=res://*/SchoolModel.csdl|res://*/SchoolModel.ssdl|res://*/SchoolModel.msl;provider=System.Data.SqlClient;provider connection string="Data Source=xxxxxxxxxx.database.chinacloudapi.cn;Initial Catalog=School;Persist Security Info=True;User ID=MyAdmin;Password=***********"

有关详细信息，请参阅[针对 Entity Framework 的 EntityClient Provider][针对 Entity Framework 的 EntityClient Provider]。

## <a name="next-steps"></a> 后续步骤

现在，你已经了解连接到 SQL Database 的相关基础知识，查看下面的资源可了解有关 SQL Database 的更多信息。

-   [开发：操作方法主题 (SQL Database)][开发：操作方法主题 (SQL Database)]
-   [SQL Database][SQL Database]

  [登录 Azure]: #PreReq1
  [创建和配置 SQL Database]: #PreReq2
  [连接到 SQL Database]: #connect-db
  [使用 ADO.NET 进行连接]: #using-sql-server
  [使用 ODBC 进行连接]: #using-ODBC
  [使用 EntityClient Provider 进行连接]: #using-entity
  [后续步骤]: #next-steps
  []: http://www.windowsazure.cn
  [Azure 管理门户]: http://manage.windowsazure.cn
  [服务级别协议]: {localLink:1132} "SLA"
  [图像]: ./media/sql-database-dotnet-how-to-use/SQLDbDashboard.PNG
  [图像 2]: ./media/sql-database-dotnet-how-to-use/SQLDBFirewall.PNG
  [图像 3]: ./media/sql-database-dotnet-how-to-use/SQLDBIPRange.PNG
  [针对 Entity Framework 的 EntityClient Provider]: http://msdn.microsoft.com/zh-cn/library/bb738561.aspx
  [开发：操作方法主题 (SQL Database)]: http://msdn.microsoft.com/zh-cn/library/azure/ee621787.aspx
  [SQL Database]: http://msdn.microsoft.com/zh-cn/library/azure/ee336279.aspx
