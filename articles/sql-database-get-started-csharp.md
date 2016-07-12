<properties
	pageTitle="试用 SQL 数据库：使用 C# 创建 SQL 数据库 | Azure"
	description="尝试使用 SQL 数据库开发 SQL 和 C# 应用，并使用适用于 .NET 的 SQL 数据库库以 C# 创建 Azure SQL 数据库。"
	keywords="试用 sql, sql c#"   
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
   ms.service="sql-database"
   ms.date="03/24/2016"
   wacn.date="05/16/2016"/>

# 试用 SQL 数据库：使用 C&#x23; 通过适用于 .NET 的 SQL 数据库库创建 SQL 数据库

**单一数据库**

> [AZURE.SELECTOR]
- [Azure 管理门户](/documentation/articles/sql-database-get-started/)
- [C#](/documentation/articles/sql-database-get-started-csharp/)
- [PowerShell](/documentation/articles/sql-database-get-started-powershell/)

了解如何使用[适用于 .NET 的 Azure SQL 数据库](https://www.nuget.org/packages/Microsoft.Azure.Management.Sql)通过 C# 命令创建 Azure SQL 数据库。你可以使用 SQL 和 C# 创建单一数据库以试用 SQL 数据库。为简明起见，我们已分开列出各个代码段，并在本文底部的某个部分中提供了一个示例控制台应用程序，其中结合了所有命令。

适用于 .NET 的 Azure SQL 数据库库提供了基于 [Azure 资源管理器](/documentation/articles/resource-group-overview/)的 API，用于包装[基于资源管理器的 SQL 数据库 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx)。此客户端库遵循基于资源管理器的客户端库的通用模式。资源管理器需要资源组，并要求使用 [Azure Active Directory](https://msdn.microsoft.com/zh-cn/library/azure/mt168838.aspx) (AAD) 进行身份验证。

<br>

> [AZURE.NOTE] 适用于 .NET 的 Azure SQL 数据库库目前以预览版提供。

<br>

若要完成本文中的步骤，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- Visual Studio。如需 Visual Studio 的免费副本，请参阅 [Visual Studio 下载](https://www.visualstudio.com/downloads/download-visual-studio-vs)页。


## 安装所需的库

若要使用 C# 来设置 SQL 数据库，请通过在 Visual Studio 中使用[程序包管理器控制台](http://docs.nuget.org/Consume/Package-Manager-Console)（“工具”>“NuGet 程序包管理器”>“程序包管理器控制台”）安装以下程序包来获取所需的管理库：

    Install-Package Microsoft.Azure.Management.Sql –Pre
    Install-Package Microsoft.Azure.Management.Resources –Pre
    Install-Package Microsoft.Azure.Common.Authentication –Pre


## 使用 Azure Active Directory 配置身份验证

首先必须通过设置所需的身份验证，使客户端应用程序能够访问 SQL 数据库服务。

若要基于当前的用户对客户端应用程序进行身份验证，你必须先将该应用程序注册到与创建了 Azure 资源的订阅关联的 AAD 域中。如果 Azure 订阅是以 Microsoft 帐户而不是工作或学校帐户创建的，则你已经有了默认的 AAD 域。

若要创建新应用程序并将其注册到正确的 Active Directory 中，请执行以下操作：

1. 转到 [Azure 管理门户](https://manage.windowsazure.cn)。
1. 在左侧选择“Active Directory”服务，选择用来对应用程序进行身份验证的目录，然后单击该目录的名称。

    ![试用 SQL 数据库：设置 Azure Active Directory (AAD)。][1]

2. 在目录页上，单击“应用程序”。

    ![包含“应用程序”的目录页。][5]

4. 单击“添加”以创建新的应用程序。

5. 选择“添加我的组织正在开发的应用程序”。

5. 提供应用的“名称”，然后选择“本机客户端应用程序”。

    ![提供有关 SQL C# 应用程序的信息。][7]

6. 提供“重定向 URI”。它不需要是实际的终结点，只要是有效的 URI 即可。

    ![添加 SQL C# 应用程序的重定向 URL。][8]

7. 完成创建应用，单击“配置”，然后复制“客户端 ID”（稍后需要在代码中使用客户端 ID）。

    ![获取 SQL C# 应用程序的客户端 ID。][9]


1. 在页面底部单击“添加应用程序”。
1. 选择“Microsoft 应用”。
1. 选择“Azure 服务管理 API”，然后完成向导。
2. 选择 API 之后，需要通过选择“访问 Azure 服务管理(预览)”，授予访问此 API 所需的特定权限。

    ![设置权限。][2]

2. 单击“保存”。



**其他 AAD 资源**

在[这篇有用的博客文章](http://www.cloudidentity.com/blog/2013/09/12/active-directory-authentication-library-adal-v1-for-net-general-availability)中，可以找到有关使用 Azure Active Directory 进行身份验证的其他信息。


### 检索当前用户的访问令牌

客户端应用程序必须检索当前用户的应用程序访问令牌。当用户首次执行此代码时，系统会提示用户输入其用户凭据，生成的令牌将在本地缓存。后续的执行将从缓存中检索令牌，并且仅在令牌已过期时才提示用户登录。

以下代码将返回 Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationResult，下面的其他代码段中需要用到它。

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

使用资源管理器时，必须在资源组中创建所有资源。资源组是一个容器，包含应用程序的相关资源。使用 Azure SQL 数据库时，必须在现有资源组中创建数据库服务器。

        static void CreateResourceGroup()
        {
            creds = new Microsoft.Rest.TokenCredentials(token.AccessToken);

            // Create a resource management client
            ResourceManagementClient resourceClient = new ResourceManagementClient(creds);

            // Resource group parameters
            ResourceGroup resourceGroupParameters = new ResourceGroup()
            {
                Location = location,
            };

            //Create a resource group
            resourceClient.SubscriptionId = subscriptionId;
            var resourceGroupResult = resourceClient.ResourceGroups.CreateOrUpdate(resourceGroupName, resourceGroupParameters);
        }


## 创建服务器

SQL 数据库包含在服务器中。服务器名称在所有 Azure SQL Server 中必须全局唯一，因此，如果该服务器名称已被使用，你将会收到错误。还必须指出的是，该命令可能需要数分钟才能运行完毕。

    static void CreateServer()
    {
        // Create a SQL Database management client
        SqlManagementClient sqlClient = new SqlManagementClient(new TokenCloudCredentials(subscriptionId, token.AccessToken));

        // Create a server
        ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
        {
            Location = location,
            Properties = new ServerCreateOrUpdateProperties()
            {
                AdministratorLogin = administratorLogin,
                AdministratorLoginPassword = administratorPassword,
                Version = serverVersion
            }
        };
            var serverResult = sqlClient.Servers.CreateOrUpdate(resourceGroupName, serverName, serverParameters);
    }


### 创建一个外部服务器管理员


    // Create a server external admin
    ServerAdministratorCreateOrUpdateParameters aadAdminParameters = new ServerAdministratorCreateOrUpdateParameters()
    {
        Properties = new ServerAdministratorCreateOrUpdateProperties()
        {
            AdministratorType = "ActiveDirectory",
            Login = "<login>",
            Sid = new Guid("<Azure AD admin sid>"),
            TenantId = new Guid("<Azure AD tenant id>")
        }
    };

    var adminResult = sqlClient.ServerAdministrators.CreateOrUpdate(resourceGroupName, serverName, "ActiveDirectory", aadAdminParameters);
    var adminGetResult = sqlClient.ServerAdministrators.Get(resourceGroupName, serverName, "ActiveDirectory");




## 创建服务器防火墙规则，以允许对服务器进行访问

默认情况下，无法从任何位置连接到服务器。为了连接到服务器或者服务器上的任何数据库，必须定义[防火墙规则](/documentation/articles/sql-database-firewall-configure/)以允许从客户端 IP 地址进行访问。

以下示例将创建一个规则，用于实现从任何 IP 地址对服务器进行访问。建议你创建适当的 SQL 登录名和密码来保护数据库，并且不要依赖防火墙规则作为防范入侵的主要防御机制。


        static void CreateFirewallRule()
        {
            // Create a firewall rule on the server
            FirewallRuleCreateOrUpdateParameters firewallParameters = new FirewallRuleCreateOrUpdateParameters()
            {
                Properties = new FirewallRuleCreateOrUpdateProperties()
                {
                    StartIpAddress = "0.0.0.0",
                    EndIpAddress = "255.255.255.255"
                }
            };
            var firewallResult = sqlClient.FirewallRules.CreateOrUpdate(resourceGroupName, serverName, firewallRuleName, firewallParameters);
        }




若要允许其他 Azure 服务访问服务器，请添加一个防火墙规则并将 tartIpAddress 和 EndIpAddress 都设置为 0.0.0.0。请注意，这会允许来自任何 Azure 订阅的 Azure 流量访问该服务器。


## 使用 C&#x23; 创建 SQL 数据库

如果服务器上不存在同名的数据库，则以下 C# 命令将创建新的 SQL 数据库；如果具有同名的数据库，则会更新数据库。


        static void CreateDatabase()
        {
            // Create a database

            // Retrieve the server on which the database will be created
            Server currentServer = sqlClient.Servers.Get(resourceGroupName, serverName).Server;

            // Create a database: configure create or update parameters and properties explicitly
            DatabaseCreateOrUpdateParameters newDatabaseParameters = new DatabaseCreateOrUpdateParameters()
            {
                Location = currentServer.Location,
                Properties = new DatabaseCreateOrUpdateProperties()
                {
                    Edition = databaseEdition,
                    RequestedServiceObjectiveName = databasePerfLevel,
                }
            };
            var dbResponse = sqlClient.Databases.CreateOrUpdate(resourceGroupName, serverName, databaseName, newDatabaseParameters);
        }



## 示例 C&#x23; 控制台应用程序

以下示例将创建资源组、服务器、防火墙规则和 SQL 数据库。本文顶部的“使用 Azure Active Directory 配置身份验证”一节介绍了获取 clientId、redirectUri 和 domainName 变量值的具体位置。


    using Microsoft.Azure;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.Resources.Models;
    using Microsoft.Azure.Management.Sql;
    using Microsoft.Azure.Management.Sql.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;

    namespace SqlDbConsoleApp
    {
    class Program
    {
        // Get these values from the Azure portal
        static string subscriptionId = "<Azure subscription id>";
        static string clientId = "<Azure App clientId>";
        static string redirectUri = "<Azure App redirectURI>";
        static string domainName = "<domain>";

        // You create these values
        static string resourceGroupName = "<your resource group name>";
        static string location = "<Azure data center location>";

        static string serverName = "<your server name>";
        static string administratorLogin = "<your server admin>";

        // store your password securely!
        static string administratorPassword = "<your server admin password>";
        static string serverVersion = "12.0";

        static string firewallRuleName = "<your firewall rule name>";

        static string databaseName = "dbfromcsharparticle";
        static string databaseEdition = "Basic"; // Basic, Standard, or Premium
        static string databasePerfLevel = "";

        static AuthenticationResult token;
        static Microsoft.Rest.TokenCredentials creds;

        static SqlManagementClient sqlClient;


        static void Main(string[] args)
        {
            Console.WriteLine("Logging in...");

            token = GetAccessToken();

            Console.WriteLine("Logged in as: " + token.UserInfo.DisplayableId);

            Console.WriteLine("Creating resource group...");
            CreateResourceGroup();

            sqlClient = new SqlManagementClient(new TokenCloudCredentials(subscriptionId, token.AccessToken));

            Console.WriteLine("Creating server...");
            CreateServer();

            Console.WriteLine("Creating firewall rule...");
            CreateFirewallRule();

            Console.WriteLine("Creating database...");

            DatabaseCreateOrUpdateResponse dbResponse = CreateDatabase();
            Console.WriteLine("Status: " + dbResponse.Status.ToString()
                + " Code: " + dbResponse.StatusCode.ToString());

            Console.WriteLine("Press enter to exit...");
            Console.ReadLine();
        }

        static void CreateResourceGroup()
        {
            creds = new Microsoft.Rest.TokenCredentials(token.AccessToken);

            // Create a resource management client
            ResourceManagementClient resourceClient = new ResourceManagementClient(creds);

            // Resource group parameters
            ResourceGroup resourceGroupParameters = new ResourceGroup()
            {
                Location = location,
            };

            //Create a resource group
            resourceClient.SubscriptionId = subscriptionId;
            var resourceGroupResult = resourceClient.ResourceGroups.CreateOrUpdate(resourceGroupName, resourceGroupParameters);
        }

        static void CreateServer()
        {
            // Create a SQL Database management client
            //SqlManagementClient sqlClient = new SqlManagementClient(new TokenCloudCredentials(subscriptionId, token.AccessToken));

            // Create a server
            ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
            {
                Location = location,
                Properties = new ServerCreateOrUpdateProperties()
                {
                    AdministratorLogin = administratorLogin,
                    AdministratorLoginPassword = administratorPassword,
                    Version = serverVersion
                }
            };
            var serverResult = sqlClient.Servers.CreateOrUpdate(resourceGroupName, serverName, serverParameters);
        }

        static void CreateFirewallRule()
        {
            // Create a firewall rule on the server
            FirewallRuleCreateOrUpdateParameters firewallParameters = new FirewallRuleCreateOrUpdateParameters()
            {
                Properties = new FirewallRuleCreateOrUpdateProperties()
                {
                    // replace with your client IP address
                    StartIpAddress = "0.0.0.0",
                    EndIpAddress = "255.255.255.255"
                }
            };
            var firewallResult = sqlClient.FirewallRules.CreateOrUpdate(resourceGroupName, serverName, firewallRuleName, firewallParameters);
        }

        static DatabaseCreateOrUpdateResponse CreateDatabase()
        {
            // Create a database

            // Retrieve the server on which the database will be created
            Server currentServer = sqlClient.Servers.Get(resourceGroupName, serverName).Server;

            // Create a database: configure create or update parameters and properties explicitly
            DatabaseCreateOrUpdateParameters newDatabaseParameters = new DatabaseCreateOrUpdateParameters()
            {
                Location = currentServer.Location,
                Properties = new DatabaseCreateOrUpdateProperties()
                {
                    Edition = databaseEdition,
                    RequestedServiceObjectiveName = databasePerfLevel,
                }
            };
            var dbResponse = sqlClient.Databases.CreateOrUpdate(resourceGroupName, serverName, databaseName, newDatabaseParameters);
            return dbResponse;
        }

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
    }
    }


## 后续步骤
既然你已试用 SQL 数据库并使用 C# 设置了数据库，现在可以阅读以下文章：

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)

## 其他资源

- [SQL 数据库](/documentation/services/sql-databases)





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

<!---HONumber=Mooncake_0509_2016-->
