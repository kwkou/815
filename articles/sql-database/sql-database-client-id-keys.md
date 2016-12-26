<properties
   pageTitle="获取对应用程序进行身份验证所需的值以便通过代码访问 SQL 数据库 | Azure"
   description="创建服务主体以便通过代码访问 SQL 数据库。"
   services="sql-database"
   documentationCenter=""
   authors="stevestein"
   manager="jhubbard"
   editor=""
   tags=""/>  


<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="09/30/2016"
   wacn.date="12/26/2016"
   ms.author="sstein"/>  

# 获取对应用程序进行身份验证所需的值以便通过代码访问 SQL 数据库

若要通过代码创建并管理 SQL 数据库，必须在创建 Azure 资源的订阅中的 Azure Active Directory (AAD) 域内注册应用。

## 创建服务主体以便从应用程序访问资源

需要安装并运行最新的 [Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/mt619274.aspx)。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

以下 PowerShell 脚本创建 Active Directory (AD) 应用程序，以及对 C# 应用进行身份验证时所需的服务主体。该脚本将输出前面 C# 示例所需的值。有关详细信息，请参阅[使用 Azure PowerShell 创建用于访问资源的服务主体](/documentation/articles/resource-group-authenticate-service-principal/)。

   
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




## 另请参阅

- [使用 C# 创建 SQL 数据库](/documentation/articles/sql-database-get-started-csharp/)
- [通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->