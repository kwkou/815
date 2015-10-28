<properties 
   pageTitle="使用 C# 创建 Azure SQL 数据库" 
   description="本文介绍如何使用适用于 .NET 的 Azure SQL 数据库库通过 C# 创建 Azure SQL 数据库。" 
   services="sql-database" 
   documentationCenter="" 
   authors="stevestein" 
   manager="jeffreyg" 
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="09/01/2015"
   wacn.date="10/17/2015"/>

# 使用 C&#x23; 创建 SQL 数据库

**单一数据库**

> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-get-started-csharp)
- [PowerShell](/documentation/articles/sql-database-get-started-powershell)



本文介绍使用[适用于 .NET 的 Azure SQL 数据库](https://www.nuget.org/packages/Microsoft.Azure.Management.Sql)通过 C# 创建 Azure SQL 数据库的命令。

本文介绍如何创建单一数据库，若要创建弹性数据库，请参阅[创建弹性数据库池](/documentation/articles/sql-database-elastic-pool-portal)。

为简明起见，我们已分开列出各个代码段，并在本文底部的某个部分中提供了一个示例控制台应用程序，其中结合了所有命令。

适用于 .NET 的 Azure SQL 数据库库提供了基于 [Azure 资源管理器](/documentation/articles/resource-group-overview)的 API，用于包装[基于资源管理器的 SQL 数据库 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx)。此客户端库遵循基于资源管理器的客户端库的通用模式。资源管理器需要资源组，并要求使用 [Azure Active Directory](https://msdn.microsoft.com/zh-cn/library/azure/mt168838.aspx) (AAD) 进行身份验证。

<br>

> [AZURE.NOTE]适用于 .NET 的 Azure SQL 数据库库目前以预览版提供。

<br>

若要完成本文中的步骤，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“免费试用”，然后再回来完成本文的相关操作即可。
- Visual Studio。如需 Visual Studio 的免费副本，请参阅 [Visual Studio 下载](https://www.visualstudio.com/downloads/download-visual-studio-vs)页。


## 安装所需的库

使用[包管理器控制台](http://docs.nuget.org/Consume/Package-Manager-Console)安装以下包，即可获取所需的管理库：

    PM> Install-Package Microsoft.Azure.Management.Sql –Pre
    PM> Install-Package Microsoft.Azure.Management.Resources –Pre
    PM> Install-Package Microsoft.Azure.Common.Authentication –Pre


## 使用 Azure Active Directory 配置身份验证

首先必须通过设置所需的身份验证，使客户端应用程序能够访问 REST API。

[Azure 资源管理器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn948464.aspx) 使用 Azure Active Directory 进行身份验证。

若要基于当前的用户对客户端应用程序进行身份验证，你必须先将该应用程序注册到与创建了 Azure 资源的订阅关联的 AAD 域中。如果 Azure 订阅是以 Microsoft 帐户而不是工作或学校帐户创建的，则你已经有了默认的 AAD 域。可以在 [Azure 门户](https://manage.windowsazure.cn/)中完成应用程序的注册。

若要创建新应用程序并将其注册到正确的 Active Directory 中，请执行以下操作：

1. 滚动左侧的菜单，找到 **Active Directory** 服务并将它打开。

    ![AAD][1]

2. 选择用于对应用程序进行身份验证的目录，然后单击该目录的**名称**。

    ![目录][4]

3. 在目录页上，单击“应用程序”。

    ![应用程序][5]

4. 单击“添加”以创建新的应用程序。

    ![添加应用程序][6]

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

    ![权限][2]

2. 单击“保存”。



### 标识域名

代码中需要用到域名。轻松标识正确域名的一种方式是：

1. 转到 [Azure 预览门户](https://manage.windowsazure.cn)。
2. 将鼠标悬停在右上角的名称上，并记下弹出窗口中显示的域。

    ![标识域名][3]





**其他 AAD 资源**

在[这篇有用的博客文章](http://www.cloudidentity.com/blog/2013/09/12/active-directory-authentication-library-adal-v1-for-net-general-availability/)中，可以找到有关使用 Azure Active Directory 进行身份验证的其他信息。


### 检索当前用户的访问令牌 

客户端应用程序必须检索当前用户的应用程序访问令牌。当用户首次执行此代码时，系统会提示用户输入其用户凭据，生成的令牌将在本地缓存。后续的执行将从缓存中检索令牌，并且仅在令牌已过期时才提示用户登录。

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
        new Uri(clientAppUri) /* redirect URI */,
        PromptBehavior.Auto /* with Auto user will not be prompted if an unexpired token is cached */);

        return token;
    }



> [AZURE.NOTE]本文中的示例使用每个 API 请求的同步形式，并会一直阻塞，直到对基础服务的 REST 调用完成。有可用的异步方法。



## 创建资源组

使用资源管理器时，必须在资源组中创建所有资源。资源组是一个容器，包含应用程序的相关资源。使用 Azure SQL 数据库时，必须在现有资源组中创建数据库服务器，再在服务器上创建数据库。然后在应用程序可以连接到服务器或数据库之前，你还必须在服务器上创建防火墙规则，以便能够从客户端 IP 地址进行访问。


    // Create a resource management client 
    ResourceManagementClient resourceClient = new ResourceManagementClient(new TokenCloudCredentials("XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /*subscription id*/, token.AccessToken ));
    
    // Resource group parameters
    ResourceGroup resourceGroupParameters = new ResourceGroup()
    {
        Location = "South Central US"
    };
    
    //Create a resource group
    var resourceGroupResult = resourceClient.ResourceGroups.CreateOrUpdate("ResourceGroupName", resourceGroupParameters);



## 创建服务器 

SQL 数据库包含在服务器中。服务器名称在所有 Azure SQL Server 中必须全局唯一，因此，如果该服务器名称已被使用，你将会收到错误。还必须指出的是，该命令可能需要数分钟才能运行完毕。


    // Create a SQL Database management client
    SqlManagementClient sqlClient = new SqlManagementClient(new TokenCloudCredentials("XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /* Subscription id*/, token.AccessToken));

    // Create a server
    ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
    {
        Location = "South Central US",
        Properties = new ServerCreateOrUpdateProperties()
        {
            AdministratorLogin = "ServerAdmin",
            AdministratorLoginPassword = securelyStoredPassword,
            Version = "12.0"
        }
    };

    var serverResult = sqlClient.Servers.CreateOrUpdate("ResourceGroupName", "ServerName", serverParameters);




## 创建服务器防火墙规则，以允许对服务器进行访问

默认情况下，无法从任何位置连接到服务器。为了连接到服务器或者服务器上的任何数据库，必须定义[防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/ee621782.aspx)以允许从客户端 IP 地址进行访问。

以下示例将创建一个规则，用于实现从任何 IP 地址对服务器进行访问。建议你创建适当的 SQL 登录名和密码来保护数据库，并且不要依赖防火墙规则作为防范入侵的主要防御机制。


    // Create a firewall rule on the server to allow TDS connection 
    FirewallRuleCreateOrUpdateParameters firewallParameters = new FirewallRuleCreateOrUpdateParameters()
    {
        Properties = new FirewallRuleCreateOrUpdateProperties()
        {
            StartIpAddress = "0.0.0.0",
            EndIpAddress = "255.255.255.255"
        }
    };

    var firewallResult = sqlClient.FirewallRules.CreateOrUpdate("ResourceGroupName", "ServerName", "FirewallRuleName", firewallParameters);




若要允许其他 Azure 服务访问服务器，请添加一个防火墙规则并将 tartIpAddress 和 EndIpAddress 都设置为 0.0.0.0。请注意，这会允许来自*任何* Azure 订阅的 Azure 流量访问该服务器。


## 创建数据库

如果服务器上没有同名的数据库，则以下命令将创建新的基本数据库；如果具有同名的数据库，则会更新数据库。

        // Create a database

        // Retrieve the server on which the database will be created
        Server currentServer = sqlClient.Servers.Get("ResourceGroupName", "ServerName").Server;
 
        // Create a database: configure create or update parameters and properties explicitly
        DatabaseCreateOrUpdateParameters newDatabaseParameters = new DatabaseCreateOrUpdateParameters()
        {
            Location = currentServer.Location,
            Properties = new DatabaseCreateOrUpdateProperties()
            {
                Edition = "Standard",
                RequestedServiceObjectiveName = "S0",
            }
        };

        var dbResponse = sqlClient.Databases.CreateOrUpdate("ResourceGroupNname", "ServerName", "Database1", newDatabaseParameters);






## 示例控制台应用程序


    using Microsoft.Azure;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.Resources.Models;
    using Microsoft.Azure.Management.Sql;
    using Microsoft.Azure.Management.Sql.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using System;
    
    
    namespace CreateSqlDatabase
    {
     class Program
     {

        #region user variables
        
        // Azure subscription id
        private static string subscriptionId = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
     
        
        // application client ID from Azure Active Directory (AAD)
        private static string clientAppUri = "http://app1";
        private static string clientId = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";

        private static string resourcegroupName = "rg1";

        private static string dataCenterLocation = "Japan West";

        private static string databaseName = "newDatabaseName";
        private static string databaseEdition = "Standard";
        private static string databasePerformanceLevel = "S0";

        private static string serverName = "serverName";
        private static string serverAdmin = "admin";
        private static string serverAdminPassword = securelyStoredPassword;
        private static string serverVersion = "12.0";

        private static string firewallRuleName = "rule1";
        private static string firewallRuleStartIp = "0.0.0.0";
        private static string firewallRuleEndIp = "255.255.255.255";



        private static string domainName = "microsoft.partner.onmschina.cn";

        private static string serverLocation = "Japan West";

        #endregion


        static void Main(string[] args)
        {
            var token = GetAccessToken();

            // Who am I?
            Console.WriteLine("Identity: {0} {1}", token.UserInfo.GivenName, token.UserInfo.FamilyName);
            Console.WriteLine("Token expires on {0}", token.ExpiresOn);
            Console.WriteLine("");

            // Create a resource management client 
            ResourceManagementClient resourceClient = new ResourceManagementClient(new TokenCloudCredentials(subscriptionId, token.AccessToken));

            // Resource group parameters
            ResourceGroup resourceGroupParameters = new ResourceGroup()
            {
                Location = dataCenterLocation
            };

            //Create a resource group
            var resourceGroupResult = resourceClient.ResourceGroups.CreateOrUpdate(resourcegroupName, resourceGroupParameters);

            Console.WriteLine("Resource group {0} create or update completed with status code {1} ", resourceGroupResult.ResourceGroup.Name, resourceGroupResult.StatusCode);

            //create a SQL Database management client
            TokenCloudCredentials tokenCredentials = new TokenCloudCredentials(subscriptionId, token.AccessToken);

            SqlManagementClient sqlClient = new SqlManagementClient(tokenCredentials);

            // Create a server
            ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
            {
                Location = serverLocation,
                Properties = new ServerCreateOrUpdateProperties()
                {
                    AdministratorLogin = serverAdmin,
                    AdministratorLoginPassword = serverAdminPassword,
                    Version = serverVersion
                }
            };

            var serverResult = sqlClient.Servers.CreateOrUpdate(resourcegroupName, serverName, serverParameters);

            var serverGetResult = sqlClient.Servers.Get(resourcegroupName, serverName);


            Console.WriteLine("Server {0} create or update completed with status code {1}", serverResult.Server.Name, serverResult.StatusCode);

            // Create a firewall rule on the server to allow TDS connection 

            FirewallRuleCreateOrUpdateParameters firewallParameters = new FirewallRuleCreateOrUpdateParameters()
            {
                Properties = new FirewallRuleCreateOrUpdateProperties()
                {
                    StartIpAddress = firewallRuleStartIp,
                    EndIpAddress = firewallRuleEndIp
                }
            };

            var firewallResult = sqlClient.FirewallRules.CreateOrUpdate(resourcegroupName, serverName, firewallRuleName, firewallParameters);

            Console.WriteLine("Firewall rule {0} create or update completed with status code: {1}", firewallResult.FirewallRule.Name, firewallResult.StatusCode);

            // Create a database

            // Retrieve the server on which the database will be created
            Server currentServer = sqlClient.Servers.Get(resourcegroupName, serverName).Server;

            // Create a database
            DatabaseCreateOrUpdateParameters newDatabaseParameters = new DatabaseCreateOrUpdateParameters()
            {
                Location = currentServer.Location,
                Properties = new DatabaseCreateOrUpdateProperties()
                {
                    Edition = databaseEdition,
                    RequestedServiceObjectiveName = databasePerformanceLevel,
                }
            };

            var dbResponse = sqlClient.Databases.CreateOrUpdate(resourcegroupName, serverName, databaseName, newDatabaseParameters);

            Console.WriteLine("Database {0} created with status code: {1}. Service Objective: {2} ", dbResponse.Database.Name, dbResponse.StatusCode, dbResponse.Database.Properties.ServiceObjective);
           
            Console.WriteLine(System.Environment.NewLine + "Press any key to exit.");
            Console.ReadKey();
        }

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
            new Uri(clientAppUri) /* redirect URI */,
            PromptBehavior.Auto /* with Auto user will not be prompted if an unexpired token is cached */);

            return token;
        }
      }
    }


## 后续步骤

- [使用 C# 连接和查询 SQL 数据库](/documentation/articles/sql-database-connect-query)
- [使用 SQL Server Management Studio (SSMS) 进行连接](/documentation/articles/sql-database-connect-to-database)

## 其他资源

- [SQL 数据库](/documentation/services/sql-databases/)





<!--Image references-->
[1]: ./media/sql-database-get-started-csharp/aad.png
[2]: ./media/sql-database-get-started-csharp/permissions.png
[3]: ./media/sql-database-get-started-csharp/getdomain.png
[4]: ./media/sql-database-get-started-csharp/aad2.png
[5]: ./media/sql-database-get-started-csharp/aad-applications.png
[6]: ./media/sql-database-get-started-csharp/add.png
[7]: ./media/sql-database-get-started-csharp/add-application.png
[8]: ./media/sql-database-get-started-csharp/add-application2.png
[9]: ./media/sql-database-get-started-csharp/clientid.png

<!---HONumber=74-->