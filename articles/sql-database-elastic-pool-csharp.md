<properties
    pageTitle="C# 数据库开发：弹性数据库池 | Azure"
    description="使用 C# 数据库开发技术创建 Azure SQL 数据库弹性数据库池，以便可以在多个数据库之间共享资源。"
    services="sql-database"
    keywords="c# 数据库,sql 开发"
    documentationCenter=""
    authors="stevestein"
    manager="jhubbard"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="05/03/2016"
    wacn.date="06/14/2016"/>

# C&#x23; 数据库开发：为 SQL 数据库创建和配置弹性数据库池

> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-elastic-pool-csharp/)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-powershell/)


本文说明如何为来自使用 C# 数据库开发技术的应用程序 SQL 数据库创建[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)。

> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤[使用 PowerShell 升级到 V12 并创建池](/documentation/articles/sql-database-upgrade-server-powershell/)。

这些示例使用了[适用于 .NET 的 Azure SQL 数据库库](https://www.nuget.org/packages/Microsoft.Azure.Management.Sql)。
为简明起见，我们已分开列出各个代码段，并在本文底部的某个部分中提供了一个示例控制台应用程序，其中结合了所有命令。


> [AZURE.NOTE] 适用于 .NET 的 Azure SQL 数据库库目前以预览版提供。



如果你没有 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。如需 Visual Studio 的免费副本，请参阅 [Visual Studio 下载](https://www.visualstudio.com/downloads/download-visual-studio-vs)页。

## 安装所需的库

使用[程序包管理器控制台](http://docs.nuget.org/Consume/Package-Manager-Console)安装以下包，即可获取用于在 SQL 上进行开发所需的管理库：

    Install-Package Microsoft.Azure.Management.Sql –Pre
    Install-Package Microsoft.Azure.Management.Resources –Pre
    Install-Package Microsoft.Azure.Common.Authentication –Pre


## 使用 Azure Active Directory 配置身份验证

在开始使用 C# 进行 SQL 开发之前，必须在 Azure 管理门户中完成一些任务。首先请通过设置所需的身份验证，使应用程序能够访问 REST API。

[Azure 资源管理器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn948464.aspx) 使用 Azure Active Directory 进行身份验证，而不是早期 Azure 服务管理 REST API 使用的证书。

若要基于当前的用户对客户端应用程序进行身份验证，你必须先将该应用程序注册到与创建了 Azure 资源的订阅关联的 AAD 域中。如果 Azure 订阅是以 Microsoft 帐户而不是工作或学校帐户创建的，则你已经有了默认的 AAD 域。可以在[管理门户](https://manage.windowsazure.cn)中完成应用程序的注册。

若要创建新应用程序并将其注册到正确的 Active Directory 中，请执行以下操作：

1. 滚动左侧的菜单，找到 **Active Directory** 服务并将它打开。

    ![C# SQL 数据库开发：Active Directory 设置][1]

2. 选择用于对应用程序进行身份验证的目录，然后单击该目录的**名称**。

    ![选择目录。][4]

3. 在目录页上，单击“应用程序”。

    ![单击“应用程序”。][5]

4. 单击“添加”以创建新的应用程序。

    ![单击“添加”按钮：创建 C# 应用程序。][6]

5. 选择“添加我的组织正在开发的应用程序”。

5. 提供应用的“名称”，然后选择“本机客户端应用程序”。

    ![添加应用程序][7]

6. 提供“重定向 URI”。它不需要是实际的终结点，只要是有效的 URI 即可。

    ![添加应用程序][8]

7. 完成创建应用，单击“配置”，然后复制“客户端 ID”（后面需要在代码中使用客户端 ID）。

    ![获取客户端 ID][9]


1. 在页面底部单击“添加应用程序”。
1. 选择“Microsoft 应用”。
1. 选择“Azure 服务管理 API”，然后完成向导。
2. 选择 API 之后，需要通过选择“访问 Azure 服务管理(预览)”，授予访问此 API 所需的特定权限。

    ![设置权限][2]

2. 单击“保存”。


**其他 AAD 资源**

在[这篇有用的博客文章](http://www.cloudidentity.com/blog/2013/09/12/active-directory-authentication-library-adal-v1-for-net-general-availability)中，可以找到有关使用 Azure Active Directory 进行身份验证的其他信息。


### 检索当前用户的访问令牌

客户端应用程序必须检索当前用户的应用程序访问令牌。当用户首次执行代码时，系统会提示用户输入其用户凭据，生成的令牌将在本地缓存。后续的执行将从缓存中检索令牌，并且仅在令牌已过期时才提示用户登录。


    private static AuthenticationResult GetAccessToken()
    {
        AuthenticationContext authContext = new AuthenticationContext
            ("https://login.chinacloudapi.cn/" + domainName /* Tenant ID or AAD domain */);

        AuthenticationResult token = authContext.AcquireToken
            ("https://management.azure.com/"/* the Azure Resource Management endpoint */,
                clientId,
        new Uri(redirectUri) /* redirect URI */,
        PromptBehavior.Auto /* with Auto user will not be prompted if an unexpired token is cached */);

        return token;
    }



> [AZURE.NOTE] 本文中的示例使用每个 API 请求的同步形式，并会一直阻塞，直到对基础服务的 REST 调用完成。有可用的异步方法。



## 创建资源组

使用资源管理器时，必须在资源组中创建所有资源。资源组是一个容器，包含应用程序的相关资源。若要创建弹性数据库池，现有资源组中需有一个 Azure SQL 数据库服务器。运行以下 C# 代码创建资源组：


    // Create a resource management client
    ResourceManagementClient resourceClient = new ResourceManagementClient(new TokenCloudCredentials("XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /*subscription id*/, token.AccessToken ));

    // Resource group parameters
    ResourceGroup resourceGroupParameters = new ResourceGroup()
    {
        Location = "China North"
    };

    //Create a resource group
    var resourceGroupResult = resourceClient.ResourceGroups.CreateOrUpdate("resourcegroup-name", resourceGroupParameters);



## 创建服务器

弹性数据库池将包含在 Azure SQL 数据库服务器中，因此下一步就是创建服务器。服务器名称在所有 Azure SQL Server 中必须全局唯一，因此，如果该服务器名称已被使用，你将会收到错误。还必须指出的是，该命令可能需要数分钟才能运行完毕。为了使应用程序能够连接到服务器，你还必须在服务器上创建防火墙规则，以允许从客户端 IP 地址进行访问。


    // Create a SQL Database management client
    SqlManagementClient sqlClient = new SqlManagementClient(new TokenCloudCredentials("XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /* Subscription id*/, token.AccessToken));

    // Create a server
    ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
    {
        Location = "China North",
        Properties = new ServerCreateOrUpdateProperties()
        {
            AdministratorLogin = "ServerAdmin",
            AdministratorLoginPassword = "P@ssword1",
            Version = "12.0"
        }
    };

    var serverResult = sqlClient.Servers.CreateOrUpdate("resourcegroup-name", "server-name", serverParameters);




## 创建服务器防火墙规则，以允许对服务器进行访问

默认情况下，服务器没有防火墙规则，因此无法从任何位置连接到服务器。为了连接到服务器或者服务器上的任何数据库，必须定义[防火墙规则](/documentation/articles/sql-database-firewall-configure/)以允许从客户端 IP 地址进行访问。

以下示例将创建一个服务器防火墙规则，用于实现从任何 IP 地址对服务器进行访问。建议你创建适当的 SQL 登录名和密码来保护数据库，并且不要依赖防火墙规则作为防范入侵的主要防御机制。有关详细信息，请参阅[管理 Azure SQL 数据库的数据库和登录名](/documentation/articles/sql-database-manage-logins/)。


    // Create a firewall rule on the server to allow TDS connection
    FirewallRuleCreateOrUpdateParameters firewallParameters = new FirewallRuleCreateOrUpdateParameters()
    {
        Properties = new FirewallRuleCreateOrUpdateProperties()
        {
            StartIpAddress = "0.0.0.0",
            EndIpAddress = "255.255.255.255"
        }
    };

    var firewallResult = sqlClient.FirewallRules.CreateOrUpdate("resourcegroup-name", "server-name", "FirewallRule1", firewallParameters);




若要允许其他 Azure 服务访问服务器，请添加一个防火墙规则并将 tartIpAddress 和 EndIpAddress 都设置为 0.0.0.0。请注意，这会允许来自任何 Azure 订阅的 Azure 流量访问该服务器。


## 创建数据库

以下示例创建一个新的基本数据库；如果服务器上已有同名的数据库，将会更新现有的数据库。

        // Create a database

        // Retrieve the server on which the database will be created
        Server currentServer = sqlClient.Servers.Get("resourcegroup-name", "server-name").Server;

        // Create a database: configure create or update parameters and properties explicitly
        DatabaseCreateOrUpdateParameters newDatabaseParameters = new DatabaseCreateOrUpdateParameters()
        {
            Location = currentServer.Location,
            Properties = new DatabaseCreateOrUpdateProperties()
            {
                Edition = "Basic",
                RequestedServiceObjectiveName = "Basic",
                MaxSizeBytes = 2147483648,
                Collation = "SQL_Latin1_General_CP1_CI_AS"
            }
        };

        var dbResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database1", newDatabaseParameters);



## 创建弹性数据库池

以下示例将创建一个新的弹性数据库池：



    // Create elastic pool: configure create or update parameters and properties explicitly
    ElasticPoolCreateOrUpdateParameters newPoolParameters = new ElasticPoolCreateOrUpdateParameters()
    {
        Location = "China North",
        Properties = new ElasticPoolCreateOrUpdateProperties()
        {
            Edition = "Standard",
            Dtu = 100,  // alternatively set StorageMB, if both are specified they must agree based on the eDTU:storage ratio of the edition
            DatabaseDtuMin = 0,
            DatabaseDtuMax = 100
         }
    };

    // Create the pool
    var newPoolResponse = sqlClient.ElasticPools.CreateOrUpdate("resourcegroup-name", "server-name", "ElasticPool1", newPoolParameters);


## 更新弹性数据库池

以下示例将更新现有弹性数据库池的性能特征：

    // Retrieve existing pool properties
    var currentPool = sqlClient.ElasticPools.Get("resourcegroup-name", "server-name", "ElasticPool1").ElasticPool;

    // Configure create or update parameters with existing property values, override those to be changed.
    ElasticPoolCreateOrUpdateParameters updatePoolParameters = new ElasticPoolCreateOrUpdateParameters()
    {
        Location = currentPool.Location,
        Properties = new ElasticPoolCreateOrUpdateProperties()
        {
            Edition = currentPool.Properties.Edition,
            DatabaseDtuMax = 50, /* Setting DatabaseDtuMax to 50 limits the eDTUs that any one database can consume */
            DatabaseDtuMin = 10, /* Setting DatabaseDtuMin above 0 limits the number of databases that can be stored in the pool */
            Dtu = (int)currentPool.Properties.Dtu,
            StorageMB = currentPool.Properties.StorageMB,  /* For a Standard pool there is 1 GB of storage per eDTU. */
        }
    };

    newPoolResponse = sqlClient.ElasticPools.CreateOrUpdate("resourcegroup-name", "server-name", "ElasticPool1", newPoolParameters);



## 将现有数据库移入弹性数据库池

创建池后，还可以使用 Transact-SQL 将现有数据库移入和移出池。有关详细信息，请参阅[使用 Transact-SQL 监视和管理弹性数据库池](/documentation/articles/sql-database-elastic-pool-manage-tsql/)。*

以下示例将现有的 Azure SQL 数据库移到池中：


    // Update database service objective to add the database to a pool

    // Retrieve current database properties
    currentDatabase = sqlClient.Databases.Get("resourcegroup-name", "server-name", "Database1").Database;

    // Configure create or update parameters with existing property values, override those to be changed.
    DatabaseCreateOrUpdateParameters updatePooledDbParameters = new DatabaseCreateOrUpdateParameters()
    {
        Location = currentDatabase.Location,
        Properties = new DatabaseCreateOrUpdateProperties()
        {
            Edition = "Standard",
            RequestedServiceObjectiveName = "ElasticPool",
            ElasticPoolName = "ElasticPool1",
            MaxSizeBytes = currentDatabase.Properties.MaxSizeBytes,
            Collation = currentDatabase.Properties.Collation,
        }
    };

    // Update the database
    var dbUpdateResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database1", updatePooledDbParameters);




## 在弹性数据库池中创建新数据库

创建一个池后，你还可以使用 Transact-SQL 在池中创建新的弹性数据库。有关详细信息，请参阅[使用 Transact-SQL 监视和管理弹性数据库池](/documentation/articles/sql-database-elastic-pool-manage-tsql/)。

以下示例将直接在池中创建一个新的数据库：


    // Create a new database in the pool

    // Create a database: configure create or update parameters and properties explicitly
    DatabaseCreateOrUpdateParameters newPooledDatabaseParameters = new DatabaseCreateOrUpdateParameters()
    {
        Location = currentServer.Location,
        Properties = new DatabaseCreateOrUpdateProperties()
        {
            Edition = "Standard",
            RequestedServiceObjectiveName = "ElasticPool",
            ElasticPoolName = "ElasticPool1",
            MaxSizeBytes = 268435456000, // 250 GB,
            Collation = "SQL_Latin1_General_CP1_CI_AS"
        }
    };

    var poolDbResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database2", newPooledDatabaseParameters);



## 列出弹性数据库池中的所有数据库

以下示例将列出池中的所有数据库：

    //List databases in the elastic pool
    DatabaseListResponse dbListInPool = sqlClient.ElasticPools.ListDatabases("resourcegroup-name", "server-name", "ElasticPool1");
    Console.WriteLine("Databases in Elastic Pool {0}", "server-name.ElasticPool1");
    foreach (Database db in dbListInPool)
    {
        Console.WriteLine("  Database {0}", db.Name);
    }




## 示例控制台应用程序


    using Microsoft.Azure;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.Resources.Models;
    using Microsoft.Azure.Management.Sql;
    using Microsoft.Azure.Management.Sql.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using System;
    using System.Security;

    namespace AzureSqlDatabaseRestApiExamples
    {
    class Program
    {
        /// <summary>
        /// Prompts for user credentials when first run or if the cached credentials have expired.
        /// </summary>
        /// <returns>The access token from AAD.</returns>
        private static AuthenticationResult GetAccessToken()
        {
            AuthenticationContext authContext = new AuthenticationContext
                ("https://login.chinacloudapi.cn/" + domainName /* Tenant ID or AAD domain */);

            AuthenticationResult token = authContext.AcquireToken
                ("https://management.azure.com/"/* the Azure Resource Management endpoint */,
                    clientId,
            new Uri(redirectUri) /* redirect URI */,
            PromptBehavior.Auto /* with Auto user will not be prompted if an unexpired token is cached */);

            return token;
        }

        private static AuthenticationResult GetAccessTokenUsingUserCredentials(UserCredential userCredential)
        {
            AuthenticationContext authContext = new AuthenticationContext
                ("https://login.chinacloudapi.cn/" /* AAD URI */
                + "YOU.partner.onmschina.cn" /* Tenant ID or AAD domain */);

            AuthenticationResult token = authContext.AcquireToken(
                "https://management.azure.com/"/* the Azure Resource Management endpoint */,
                "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /* application client ID from AAD*/,
                userCredential);

            return token;
        }
        private static SecureString convertToSecureString(string secret)
        {
            var secureStr = new SecureString();
            if (secret.Length > 0)
            {
                foreach (var c in secret.ToCharArray()) secureStr.AppendChar(c);
            }
            return secureStr;
        }

        static void Main(string[] args)
        {
            var token = GetAccessToken();

            // Who am I?
            Console.WriteLine("Identity is {0} {1}", token.UserInfo.GivenName, token.UserInfo.FamilyName);
            Console.WriteLine("Token expires on {0}", token.ExpiresOn);
            Console.WriteLine("");

            // Create a resource management client
            ResourceManagementClient resourceClient = new ResourceManagementClient(new TokenCloudCredentials("XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /*subscription id*/, token.AccessToken));

            // Resource group parameters
            ResourceGroup resourceGroupParameters = new ResourceGroup()
            {
                Location = "China North"
            };

            //Create a resource group
            var resourceGroupResult = resourceClient.ResourceGroups.CreateOrUpdate("resourcegroup-name", resourceGroupParameters);

            Console.WriteLine("Resource group {0} create or update completed with status code {1} ", resourceGroupResult.ResourceGroup.Name, resourceGroupResult.StatusCode);

            //create a SQL Database management client
            TokenCloudCredentials tokenCredentials = new TokenCloudCredentials("XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /* Subscription id*/, token.AccessToken);

            SqlManagementClient sqlClient = new SqlManagementClient(tokenCredentials);

            // Create a server
            ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
            {
                Location = "China North",
                Properties = new ServerCreateOrUpdateProperties()
                {
                    AdministratorLogin = "ServerAdmin",
                    AdministratorLoginPassword = "P@ssword1",
                    Version = "12.0"
                }
            };

            var serverResult = sqlClient.Servers.CreateOrUpdate("resourcegroup-name", "server-name", serverParameters);

            var serverGetResult = sqlClient.Servers.Get("resourcegroup-name", "server-name");


            Console.WriteLine("Server {0} create or update completed with status code {1}", serverResult.Server.Name, serverResult.StatusCode);

            // Create a firewall rule on the server to allow TDS connection

            FirewallRuleCreateOrUpdateParameters firewallParameters = new FirewallRuleCreateOrUpdateParameters()
            {
                Properties = new FirewallRuleCreateOrUpdateProperties()
                {
                    StartIpAddress = "0.0.0.0",
                    EndIpAddress = "255.255.255.255"
                }
            };

            var firewallResult = sqlClient.FirewallRules.CreateOrUpdate("resourcegroup-name", "server-name", "FirewallRule1", firewallParameters);

            Console.WriteLine("Firewall rule {0} create or update completed with status code {1}", firewallResult.FirewallRule.Name, firewallResult.StatusCode);


            var dbResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database1", newDatabaseParameters);

            Console.WriteLine("Database {0} create or update completed with status code {1}. Service Objective {2} ", dbResponse.Database.Name, dbResponse.StatusCode, dbResponse.Database.Properties.ServiceObjective);


           // Create an elastic database pool
            var newPoolResponse = sqlClient.ElasticPools.CreateOrUpdate("resourcegroup-name", "server-name", "ElasticPool1", newPoolParameters);

            Console.WriteLine("Elastic pool {0} create or update completed with status code {1}.", newPoolResponse.ElasticPool.Name, newPoolResponse.StatusCode);

            // Update pool: retrieve existing pool properties
            var currentPool = sqlClient.ElasticPools.Get("resourcegroup-name", "server-name", "ElasticPool1").ElasticPool;

            // Update pool: configure create or update parameters with existing property values, override those to be changed.
            ElasticPoolCreateOrUpdateParameters updatePoolParameters = new ElasticPoolCreateOrUpdateParameters()
            {
                Location = currentPool.Location,
                Properties = new ElasticPoolCreateOrUpdateProperties()
                {
                    Edition = currentPool.Properties.Edition,
                    DatabaseDtuMax = 50, /* Setting DatabaseDtuMax to 50 limits the eDTUs that any one database can consume */
                    DatabaseDtuMin = 10, /* Setting DatabaseDtuMin above 0 limits the number of databases that can be stored in the pool */
                    Dtu = (int)currentPool.Properties.Dtu,
                    StorageMB = currentPool.Properties.StorageMB,  /* For a Standard pool there is 1 GB of storage per eDTU. */
                }
            };
            newPoolResponse = sqlClient.ElasticPools.CreateOrUpdate("resourcegroup-name", "server-name", "ElasticPool1", newPoolParameters);

            Console.WriteLine("Elastic pool {0} create or update completed with status code {1}.", newPoolResponse.ElasticPool.Name, newPoolResponse.StatusCode);

            // Update a databases service objective to add the database to a pool

            // Update database: retrieve current database properties
            currentDatabase = sqlClient.Databases.Get("resourcegroup-name", "server-name", "Database1").Database;

            // Update database: configure create or update parameters with existing property values, override those to be changed.
            DatabaseCreateOrUpdateParameters updatePooledDbParameters = new DatabaseCreateOrUpdateParameters()
            {
                Location = currentDatabase.Location,
                Properties = new DatabaseCreateOrUpdateProperties()
                {
                    Edition = "Standard",
                    RequestedServiceObjectiveName = "ElasticPool",
                    ElasticPoolName = "ElasticPool1",
                    MaxSizeBytes = currentDatabase.Properties.MaxSizeBytes,
                    Collation = currentDatabase.Properties.Collation,
                }
            };


            // Create a new database in the pool

            // Create a database: configure create or update parameters and properties explicitly
            DatabaseCreateOrUpdateParameters newPooledDatabaseParameters = new DatabaseCreateOrUpdateParameters()
            {
                Location = currentServer.Location,
                Properties = new DatabaseCreateOrUpdateProperties()
                {
                    Edition = "Standard",
                    RequestedServiceObjectiveName = "ElasticPool",
                    ElasticPoolName = "ElasticPool1",
                    MaxSizeBytes = 268435456000, // 250 GB,
                    Collation = "SQL_Latin1_General_CP1_CI_AS"
                }
            };

            var poolDbResponse = sqlClient.Databases.CreateOrUpdate("resourcegroup-name", "server-name", "Database2", newPooledDatabaseParameters);

            Console.WriteLine("Database {0} create or update completed with status code {1}. Service Objective: {2}({3}) ", poolDbResponse.Database.Name, poolDbResponse.StatusCode, poolDbResponse.Database.Properties.ServiceObjective, poolDbResponse.Database.Properties.ElasticPoolName);
      }
    }







## 其他资源


[SQL 数据库](/documentation/services/sql-databases)

[Azure 资源管理 API](https://msdn.microsoft.com/zh-cn/library/azure/dn948464.aspx)

<!--Image references-->
[1]: ./media/sql-database-elastic-pool-csharp/aad.png
[2]: ./media/sql-database-elastic-pool-csharp/permissions.png
[3]: ./media/sql-database-elastic-pool-csharp/getdomain.png
[4]: ./media/sql-database-elastic-pool-csharp/aad2.png
[5]: ./media/sql-database-elastic-pool-csharp/aad-applications.png
[6]: ./media/sql-database-elastic-pool-csharp/add.png
[7]: ./media/sql-database-elastic-pool-csharp/add-application.png
[8]: ./media/sql-database-elastic-pool-csharp/add-application2.png
[9]: ./media/sql-database-elastic-pool-csharp/clientid.png

<!---HONumber=Mooncake_0530_2016-->
