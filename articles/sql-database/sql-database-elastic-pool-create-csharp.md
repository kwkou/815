<properties
    pageTitle="使用 C# 创建弹性数据库池 | Azure"
    description="使用 C# 数据库开发技术在 Azure SQL 数据库中创建可缩放的弹性数据库池，以便可以在多个数据库之间共享资源。"
    services="sql-database"
    documentationCenter=""
    authors="stevestein"
    manager="jhubbard"
    editor=""/>  


<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="csharp"
    ms.workload="data-management"
    ms.date="10/04/2016"
    wacn.date="10/31/2016"
    ms.author="sstein"/>  


# 使用 C&#x23; 创建弹性数据库池

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-elastic-pool-create-powershell/)
- [C#](/documentation/articles/sql-database-elastic-pool-create-csharp/)


本文说明如何使用 C# 通过[用于 .NET 的 Azure SQL 数据库库](https://www.nuget.org/packages/Microsoft.Azure.Management.Sql)创建 Azure SQL 数据库弹性数据库池。若要创建独立 SQL 数据库，请参阅 [Use C# to create a SQL database with the SQL Database Library for .NET](/documentation/articles/sql-database-get-started-csharp/)（使用 C# 通过用于.NET 的 Azure SQL 数据库库创建 SQL 数据库）。

适用于 .NET 的 Azure SQL 数据库库提供了基于 [Azure 资源管理器](/documentation/articles/resource-group-overview/)的 API，用于包装[基于资源管理器的 SQL 数据库 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx)。

>[AZURE.NOTE] SQL 数据库的许多新功能仅在使用 [Azure Resource Manager 部署模型](/documentation/articles/resource-group-overview/)时才可用，因此，始终应该使用最新版本的**用于 .NET 的 Azure SQL 数据库管理库（[文档](https://msdn.microsoft.com/zh-cn/library/azure/mt349017.aspx) | [NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Management.Sql)）**。以前的[基于经典部署模型的库](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Sql)只是为了向后兼容而受到支持，因此，建议使用较新的基于 Resource Manager 的库。

若要完成本文中的步骤，需要做好以下准备：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页右上角的“申请试用”，然后再回来完成本文的相关操作即可。
- Visual Studio。如需 Visual Studio 的免费副本，请参阅 [Visual Studio 下载](https://www.visualstudio.com/downloads/download-visual-studio-vs)页。


## 创建控制台应用并安装所需的库

1. 启动 Visual Studio。
2. 单击“文件”>“新建”>“项目”。
3. 创建 C# **控制台应用程序**并将其命名为 *SqlElasticPoolConsoleApp*


若要使用 C# 创建 SQL 数据库，请加载所需的管理库（使用[包管理器控制台](http://docs.nuget.org/Consume/Package-Manager-Console)）：

1. 单击“工具”->“NuGet 包管理器”->“包管理器控制台”。
2. 键入 `Install-Package Microsoft.Azure.Management.Sql –Pre` 安装 [Microsoft Azure SQL 管理库](https://www.nuget.org/packages/Microsoft.Azure.Management.Sql)。
3. 键入 `Install-Package Microsoft.Azure.Management.ResourceManager –Pre` 安装 [Microsoft Azure Resource Manager 库](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)。
4. 键入 `Install-Package Microsoft.Azure.Common.Authentication –Pre` 安装 [Microsoft Azure 通用身份验证库](https://www.nuget.org/packages/Microsoft.Azure.Common.Authentication)。



> [AZURE.NOTE] 本文中的示例使用每个 API 请求的同步形式，并会一直阻塞，直到对基础服务的 REST 调用完成。有可用的异步方法。


## 创建 SQL 弹性数据库池 - C# 示例

以下示例创建资源组、服务器、防火墙规则、弹性池，然后在池中创建 SQL 数据库。请参阅[创建用于访问资源的服务主体](#create-a-service-principal-to-access-resources)获取 `_subscriptionId, _tenantId, _applicationId, and _applicationSecret` 变量。

将 **Program.cs** 的内容替换为以下内容，使用应用值更新 `{variables}`（请不要包含 `{}`）。



	using Microsoft.Azure;
	using Microsoft.Azure.Management.ResourceManager;
	using Microsoft.Azure.Management.ResourceManager.Models;
	using Microsoft.Azure.Management.Sql;
	using Microsoft.Azure.Management.Sql.Models;
	using Microsoft.IdentityModel.Clients.ActiveDirectory;
	using System;

	namespace SqlElasticPoolConsoleApp
	{
	    class Program
	        {

	        // For details about these four (4) values, see
	        // https://wwww.azure.cn/documentation/articles/resource-group-authenticate-service-principal/
	        static string _subscriptionId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}";
	        static string _tenantId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}";
	        static string _applicationId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}";
	        static string _applicationSecret = "{your-password}";

	        // Create management clients for the Azure resources your app needs to work with.
	        // This app works with Resource Groups, and Azure SQL Database.
	        static ResourceManagementClient _resourceMgmtClient;
	        static SqlManagementClient _sqlMgmtClient;

	        // Authentication token
	        static AuthenticationResult _token;

	        // Azure resource variables
	        static string _resourceGroupName = "{resource-group-name}";
	        static string _resourceGrouplocation = "{Azure-region}";

	        static string _serverlocation = _resourceGrouplocation;
	        static string _serverName = "{server-name}";
	        static string _serverAdmin = "{server-admin-login}";
	        static string _serverAdminPassword = "{server-admin-password}";

	        static string _firewallRuleName = "{firewall-rule-name}";
	        static string _startIpAddress = "{0.0.0.0}";
	        static string _endIpAddress = "{255.255.255.255}";

	        static string _poolName = "{pool-name}";
	        static string _poolEdition = "{Standard}";
	        static int _poolDtus = {100};
	        static int _databaseMinDtus = {10};
	        static int _databaseMaxDtus = {100};

	        static string _databaseName = "{elasticdbfromcsarticle}";



	        static void Main(string[] args)
	        {
	            // Authenticate:
	            _token = GetToken(_tenantId, _applicationId, _applicationSecret);
	            Console.WriteLine("Token acquired. Expires on:" + _token.ExpiresOn);

	            // Instantiate management clients:
	            _resourceMgmtClient = new ResourceManagementClient(new Microsoft.Rest.TokenCredentials(_token.AccessToken));
	            _sqlMgmtClient = new SqlManagementClient(new TokenCloudCredentials(_subscriptionId, _token.AccessToken));


	            Console.WriteLine("Resource group...");
	            ResourceGroup rg = CreateOrUpdateResourceGroup(_resourceMgmtClient, _subscriptionId, _resourceGroupName, _resourceGrouplocation);
	            Console.WriteLine("Resource group: " + rg.Id);


	            Console.WriteLine("Server...");
	            ServerGetResponse sgr = CreateOrUpdateServer(_sqlMgmtClient, _resourceGroupName, _serverlocation, _serverName, _serverAdmin, _serverAdminPassword);
	            Console.WriteLine("Server: " + sgr.Server.Id);

	            Console.WriteLine("Server firewall...");
	            FirewallRuleGetResponse fwr = CreateOrUpdateFirewallRule(_sqlMgmtClient, _resourceGroupName, _serverName, _firewallRuleName, _startIpAddress, _endIpAddress);
	            Console.WriteLine("Server firewall: " + fwr.FirewallRule.Id);

	            Console.WriteLine("Elastic pool...");
	            ElasticPoolCreateOrUpdateResponse epr = CreateOrUpdateElasticDatabasePool(_sqlMgmtClient, _resourceGroupName, _serverName, _poolName, _poolEdition, _poolDtus, _databaseMinDtus, _databaseMaxDtus);
	            Console.WriteLine("Elastic pool: " + epr.ElasticPool.Id);

	            Console.WriteLine("Database...");
	            DatabaseCreateOrUpdateResponse dbr = CreateOrUpdateDatabase(_sqlMgmtClient, _resourceGroupName, _serverName, _databaseName, _poolName);
	            Console.WriteLine("Database: " + dbr.Database.Id);


	            Console.WriteLine("Press any key to continue...");
	            Console.ReadKey();
	        }

	        static ResourceGroup CreateOrUpdateResourceGroup(ResourceManagementClient resourceMgmtClient, string subscriptionId, string resourceGroupName, string resourceGroupLocation)
	        {
	            ResourceGroup resourceGroupParameters = new ResourceGroup()
	            {
	                Location = resourceGroupLocation,
	            };
	            resourceMgmtClient.SubscriptionId = subscriptionId;
	            ResourceGroup resourceGroupResult = resourceMgmtClient.ResourceGroups.CreateOrUpdate(resourceGroupName, resourceGroupParameters);
	            return resourceGroupResult;
	        }

	        static ServerGetResponse CreateOrUpdateServer(SqlManagementClient sqlMgmtClient, string resourceGroupName, string serverLocation, string serverName, string serverAdmin, string serverAdminPassword)
	        {
	            ServerCreateOrUpdateParameters serverParameters = new ServerCreateOrUpdateParameters()
	            {
	                Location = serverLocation,
	                Properties = new ServerCreateOrUpdateProperties()
	                {
	                    AdministratorLogin = serverAdmin,
	                    AdministratorLoginPassword = serverAdminPassword,
	                    Version = "12.0"
	                }
	            };
	            ServerGetResponse serverResult = sqlMgmtClient.Servers.CreateOrUpdate(resourceGroupName, serverName, serverParameters);
	            return serverResult;
	        }


	        static FirewallRuleGetResponse CreateOrUpdateFirewallRule(SqlManagementClient sqlMgmtClient, string resourceGroupName, string serverName, string firewallRuleName, string startIpAddress, string endIpAddress)
	        {
	            FirewallRuleCreateOrUpdateParameters firewallParameters = new FirewallRuleCreateOrUpdateParameters()
	            {
	                Properties = new FirewallRuleCreateOrUpdateProperties()
	                {
	                    StartIpAddress = startIpAddress,
	                    EndIpAddress = endIpAddress
	                }
	            };
	            FirewallRuleGetResponse firewallResult = sqlMgmtClient.FirewallRules.CreateOrUpdate(resourceGroupName, serverName, firewallRuleName, firewallParameters);
	            return firewallResult;
	        }



	        static ElasticPoolCreateOrUpdateResponse CreateOrUpdateElasticDatabasePool(SqlManagementClient sqlMgmtClient, string resourceGroupName, string serverName, string poolName, string poolEdition, int poolDtus, int databaseMinDtus, int databaseMaxDtus)
	        {
	            // Retrieve the server that will host this elastic pool
	            Server currentServer = sqlMgmtClient.Servers.Get(resourceGroupName, serverName).Server;

	            // Create elastic pool: configure create or update parameters and properties explicitly
	            ElasticPoolCreateOrUpdateParameters newPoolParameters = new ElasticPoolCreateOrUpdateParameters()
	            {
	                Location = currentServer.Location,
	                Properties = new ElasticPoolCreateOrUpdateProperties()
	                {
	                    Edition = poolEdition,
	                    Dtu = poolDtus,
	                    DatabaseDtuMin = databaseMinDtus,
	                    DatabaseDtuMax = databaseMaxDtus
	                }
	            };

	            // Create the pool
	            var newPoolResponse = sqlMgmtClient.ElasticPools.CreateOrUpdate(resourceGroupName, serverName, poolName, newPoolParameters);
	            return newPoolResponse;
	        }




	        static DatabaseCreateOrUpdateResponse CreateOrUpdateDatabase(SqlManagementClient sqlMgmtClient, string resourceGroupName, string serverName, string databaseName, string poolName)
	        {
	            // Retrieve the server that will host this database
	            Server currentServer = sqlMgmtClient.Servers.Get(resourceGroupName, serverName).Server;

	            // Create a database: configure create or update parameters and properties explicitly
	            DatabaseCreateOrUpdateParameters newDatabaseParameters = new DatabaseCreateOrUpdateParameters()
	            {
	                Location = currentServer.Location,
	                Properties = new DatabaseCreateOrUpdateProperties()
	                {
	                    CreateMode = DatabaseCreateMode.Default,
	                    ElasticPoolName = poolName
	                }
	            };
	            DatabaseCreateOrUpdateResponse dbResponse = sqlMgmtClient.Databases.CreateOrUpdate(resourceGroupName, serverName, databaseName, newDatabaseParameters);
	            return dbResponse;
	        }



	        private static AuthenticationResult GetToken(string tenantId, string applicationId, string applicationSecret)
	        {
	            AuthenticationContext authContext = new AuthenticationContext("https://login.chinacloudapi.cn/" + tenantId);
	            _token = authContext.AcquireToken("https://management.core.chinacloudapi.cn/", new ClientCredential(applicationId, applicationSecret));
	            return _token;
	        }
	    }
	}






##<a id="create-a-service-principal-to-access-resources"></a>创建用于访问资源的服务主体

以下 PowerShell 脚本创建 Active Directory (AD) 应用程序，以及对 C# 应用进行身份验证时所需的服务主体。该脚本将输出前面 C# 示例所需的值。有关详细信息，请参阅 [Use Azure PowerShell to create a service principal to access resources](/documentation/articles/resource-group-authenticate-service-principal/)（使用 Azure PowerShell 创建用于访问资源的服务主体）。

   
    # Sign in to Azure.
    Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    
    # If you have multiple subscriptions, uncomment and set to the subscription you want to work with.
    #$subscriptionId = "{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}"
    #Set-AzureRmContext -SubscriptionId $subscriptionId
    
    # Provide these values for your new AAD app.
    # $appName is the display name for your app, must be unique in your directory.
    # $uri does not need to be a real uri.
    # $secret is a password you create.
    
    $appName = "{app-name}"
    $uri = "http://{app-name}"
    $secret = "{app-password}"
    
    # Create a AAD app
    $azureAdApplication = New-AzureRmADApplication -DisplayName $appName -HomePage $Uri -IdentifierUris $Uri -Password $secret
    
    # Create a Service Principal for the app
    $svcprincipal = New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId
    
    # To avoid a PrincipalNotFound error, I pause here for 15 seconds.
    Start-Sleep -s 15
    
    # If you still get a PrincipalNotFound error, then rerun the following until successful. 
    $roleassignment = New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $azureAdApplication.ApplicationId.Guid
    
    
    # Output the values we need for our C# application to successfully authenticate
    
    Write-Output "Copy these values into the C# sample app"
    
    Write-Output "_subscriptionId:" (Get-AzureRmContext).Subscription.SubscriptionId
    Write-Output "_tenantId:" (Get-AzureRmContext).Tenant.TenantId
    Write-Output "_applicationId:" $azureAdApplication.ApplicationId.Guid
    Write-Output "_applicationSecret:" $secret


  

## 后续步骤

- [管理你的池](/documentation/articles/sql-database-elastic-pool-manage-csharp/)
- [创建弹性作业](/documentation/articles/sql-database-elastic-jobs-overview/)：弹性作业可以根据池中数据库的数目来运行 T-SQL 脚本。
- [使用 Azure SQL 数据库进行扩展](/documentation/articles/sql-database-elastic-scale-introduction/)：使用弹性数据库工具进行扩展。

## 其他资源

- [SQL 数据库](/documentation/services/sql-databases/)
- [Azure 资源管理 API](https://msdn.microsoft.com/zh-cn/library/azure/dn948464.aspx)

<!---HONumber=Mooncake_1024_2016-->