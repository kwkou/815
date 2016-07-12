<properties
   pageTitle="通过 Azure 资源管理器对服务主体进行身份验证 | Azure"
   description="介绍如何通过基于角色的访问控制向服务主体授予访问权限，并对服务主体进行身份验证。演示如何使用 PowerShell 和 Azure CLI 执行这些任务。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="03/10/2016"
   wacn.date="04/11/2016"/>

# 通过 Azure 资源管理器对服务主体进行身份验证

本主题说明如何允许服务主体（例如自动化的过程、应用程序或服务）访问订阅中的其他资源。借助 Azure 资源管理器，可以使用基于角色的访问控制向服务主体授予允许的操作，并对该服务主体进行身份验证。

本主题演示如何使用适用于 Mac、Linux 和 Windows 的 Azure PowerShell 或 Azure CLI 来创建应用程序和服务主体，为服务主体分配角色，以及以服务主体身份进行验证。如果你未安装 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。
如果你未安装 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。有关使用门户执行这些步骤的信息，请参阅[使用门户创建 Active Directory 应用程序和服务主体](/documentation/articles/resource-group-create-service-principal-portal/)

## 概念
1. Azure Active Directory - 云的标识与访问管理服务。有关详细信息，请参阅[什么是 Azure Active Directory](/documentation/articles/active-directory-whatis/)
2. 服务主体 - 目录中需要访问其他资源的应用程序实例。
3. Active Directory 应用程序 - 向 AAD 标识某个应用程序的目录记录。

有关应用程序和服务主体的详细说明，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)。
有关 Active Directory 身份验证的详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)。

本主题说明用于创建 Active Directory 应用程序以及对该应用程序进行验证的 4 个途径。这些途径会因你想要使用密码还是证书进行身份验证，以及想要使用 PowerShell 还是 Azure CLI 而有所不同。这些途径是：

- [使用密码进行身份验证 - PowerShell](#authenticate-with-password---powershell)
- [使用证书进行身份验证 - PowerShell](#authenticate-with-certificate---powershell)
- [使用密码进行身份验证 - Azure CLI](#authenticate-with-password---azure-cli)
- [使用证书进行身份验证 - Azure CLI](#authenticate-with-certificate---azure-cli)

## 使用密码进行身份验证 - PowerShell

在此部分中，您将执行以下用于创建 Azure Active Directory 应用程序的服务主体、将角色分配给服务主体，通过提供应用程序标识符和密码作为服务主体进行身份验证的步骤。

1. 登录到你的帐户。

        PS C:\> Login-AzureRmAccount -EnvironmentName AzureChinaCloud

1. 运行 **New-AzureRmADApplication** 命令创建新的 Active Directory 应用程序。提供应用程序的显示名称、描述应用程序的页面的 URI（该链接未验证）、标识应用程序的 URI，以及应用程序标识的密码。

        PS C:\> $azureAdApplication = New-AzureRmADApplication -DisplayName "exampleapp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.contoso.org/example" -Password "<Your_Password>"

     检查新的应用程序对象。需要使用 **ApplicationId** 属性来创建服务主体、进行角色分配以及获取访问令牌。

        PS C:\> $azureAdApplication

        DisplayName             : exampleapp
        Type                    : Application
        ApplicationId           : 8bc80782-a916-47c8-a47e-4d76ed755275
        ApplicationObjectId     : c95e67a3-403c-40ac-9377-115fa48f8f39
        AvailableToOtherTenants : False
        AppPermissions          : {}
        IdentifierUris          : {https://www.contoso.org/example}
        ReplyUrls               : {}

2. 通过传入 Active Directory 应用程序的应用程序 ID 创建应用程序的服务主体。

        PS C:\> New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

     现在你已在目录中创建服务主体，但未将任何权限或范围分配给服务。
     需要显式向服务主体授予权限，才能在某个范围执行操作。

3. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。对于 **ServicePrincipalName** 参数，请提供你在创建应用程序时使用的 **ApplicationId** 或 **IdentifierUris**。<!--有关基于角色的访问控制的详细信息，请参阅[管理和审核对资源的访问权限](/documentation/articles/resource-group-rbac/)-->

        PS C:\> New-AzureRmRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $azureAdApplication.ApplicationId

你已创建 Active Directory 应用程序并为该应用程序创建服务主体。你已向角色分配服务主体。现在，你需要以服务主体身份登录，以便以服务主体身份执行操作。在本主题中说明了三个选项：

- [通过 PowerShell 手动提供凭据](#manually-provide-credentials-through-powershell)
- [通过自动执行的 PowerShell 脚本提供凭据](#provide-credentials-through-automated-powershell-script)
- [通过应用程序中的代码提供凭据](#provide-credentials-through-code-in-an-application)

<a id="manually-provide-credentials-through-powershell" /></a>
### 通过 PowerShell 手动提供凭据

在执行按需脚本或命令时，可以手动为服务主体提供凭据。

1. 运行 **Get-Credential** 命令，以创建包含你的凭据的新 **PSCredential** 对象。

        PS C:\> $creds = Get-Credential

2. 系统会提示你输入凭据。对于用户名，请使用你在创建应用程序时所用的 **ApplicationId** 或 **IdentifierUris**。对于密码，请使用你在创建帐户时指定的密码。

     ![输入凭据][1]

3. 检索在其中创建了角色分配的订阅。将使用此订阅来获取服务主体角色分配所在租户的 **TenantId**。

        PS C:\> $subscription = Get-AzureRmSubscription

     如果角色分配不是在当前选择的订阅中创建的，你可以指定 **SubscriptoinId** 或 **SubscriptionName** 参数来检索其他订阅。

4. 通过使用 **Login-AzureRmAccount** cmdlet 以服务主体身份登录，但提供凭据对象，并指定该帐户是服务主体。

        PS C:\> Login-AzureRmAccount -EnvironmentName AzureChinaCloud -Credential $creds -ServicePrincipal -Tenant $subscription.TenantId
        Environment           : AzureCloud
        Account               : {guid}
        Tenant                : {guid}
        Subscription          : {guid}
        CurrentStorageAccount :

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

<a id="provide-credentials-through-automated-powershell-script" /></a>
### 通过自动执行的 PowerShell 脚本提供凭据

本部分说明如何以服务主体身份登录，而无需手动输入凭据。你无需手动提供密码，因为将从密钥保管库中检索它。

> [AZURE.NOTE] 直接在 PowerShell 脚本中包含密码不安全，因为密码将以文本的形式公开。请改用密钥保管库等服务来存储密码，并在执行脚本时检索它。

这些步骤假定你已设置密码存储的密钥保管库和机密。若要通过模板部署密钥保管库和机密，请参阅[密钥保管库模板格式]()。若要了解有关密钥保管库的信息，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)。

1. 从密钥保管库中检索密码（在下面的示例中，使用名称 **appPassword** 存储为机密）。

        PS C:\> $secret = Get-AzureKeyVaultSecret -VaultName examplevault -Name appPassword
        
2. 获取 Active Directory 应用程序。在登录时，需要应用程序 ID。

        PS C:\> $azureAdApplication = Get-AzureRmADApplication -IdentifierUri "https://www.contoso.org/example"

3. 通过提供应用程序 ID 和密码作为凭据创建一个新的 **PSCredential** 对象。

        PS C:\> $creds = New-Object System.Management.Automation.PSCredential ($azureAdApplication.ApplicationId, $secret.SecretValue)
    
4. 检索在其中创建了角色分配的订阅。稍后将使用此订阅来获取服务主体角色分配所在租户的 **TenantId**。

        PS C:\> $subscription = Get-AzureRmSubscription

     如果角色分配不是在当前选择的订阅中创建的，你可以指定 **SubscriptoinId** 或 **SubscriptionName** 参数来检索其他订阅。

5. 通过使用 **Login-AzureRmAccount** cmdlet 以服务主体身份登录，但提供凭据对象，并指定该帐户是服务主体。
    
        PS C:\> Login-AzureRmAccount -EnvironmentName AzureChinaCloud -Credential $creds -ServicePrincipal -TenantId $subscription.TenantId

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

<a id="provide-credentials-through-code-in-an-application" /></a>
### 通过应用程序中的代码提供凭据

若要从 .NET 应用程序进行身份验证，请包含以下代码。你将需要 Active Directory 应用程序的应用程序 ID、密码和订阅的租户 ID。
检索访问令牌之后，你便可以使用订阅中的资源了。

    public static string GetAccessToken()
    {
          var authenticationContext = new AuthenticationContext("https://login.chinacloudapi.cn/{tenantId or tenant name}");  
      var credential = new ClientCredential(clientId: "{application id}", clientSecret: "{application password}");
          var result = authenticationContext.AcquireToken(resource: "https://management.core.chinacloudapi.cn/", clientCredential:credential);

      if (result == null) {
        throw new InvalidOperationException("Failed to obtain the JWT token");
      }

      string token = result.AccessToken;

      return token;
    }


## 使用证书进行身份验证 - PowerShell

在此部分中，您将执行以下用于创建 Azure Active Directory 应用程序的服务主体、将角色分配给服务主体，通过提供证书作为服务主体进行身份验证的步骤。本主题假定您已获得所颁发的证书。

它介绍了两种使用证书的方法 - 密钥凭据和密钥值。您可以使用任意一种方法。

首先，当创建应用程序时，必须在 PowerShell 中设置以后将用到的某些值。

1. 登录到你的帐户。

        PS C:\> Login-AzureRmAccount -EnvironmentName AzureChinaCloud

1. 使用这两种方法时，都将从你的证书创建一个 X509Certificate2 对象并检索密钥值。使用指向您的证书的路径和该证书的密码。

        $cert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @("C:\certificates\examplecert.pfx", "yourpassword")
        $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

2. 如果你使用密钥凭据，请创建密钥凭据对象并将其值设置为上一步中的 `$keyValue`。下面的示例包括调用 `Add-Type` 以从程序集中添加类型。该示例中所示的路径应类似于你的路径，但可能稍有不同。

        Add-Type -Path 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ResourceManager\AzureResourceManager\AzureRM.Resources\Microsoft.Azure.Commands.Resources.dll'
        $currentDate = Get-Date
        $endDate = $currentDate.AddYears(1)
        $keyId = [guid]::NewGuid()
        $keyCredential = New-Object  Microsoft.Azure.Commands.Resources.Models.ActiveDirectory.PSADKeyCredential
        $keyCredential.StartDate = $currentDate
        $keyCredential.EndDate= $endDate
        $keyCredential.KeyId = $keyId
        $keyCredential.Type = "AsymmetricX509Cert"
        $keyCredential.Usage = "Verify"
        $keyCredential.Value = $keyValue

3. 在目录中创建一个应用程序。第一个命令演示如何使用密钥值。

        $azureAdApplication = New-AzureRmADApplication -DisplayName "exampleapp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.contoso.org/example" -KeyValue $keyValue -KeyType AsymmetricX509Cert       
        
    或者，使用第二个示例分配密钥凭据。

         $azureAdApplication = New-AzureRmADApplication -DisplayName "example" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.contoso.org/example" -KeyCredentials $keyCredential

    检查新的应用程序对象。需要使用 **ApplicationId** 属性来创建服务主体、进行角色分配以及获取访问令牌。

        PS C:\> $azureAdApplication

        DisplayName             : exampleapp
        Type                    : Application
        ApplicationId           : 1975a4fd-1528-4086-a992-787dbd23c46b
        ApplicationObjectId     : 9665e5f3-84b7-4344-92e2-8cc20ad16a8b
        AvailableToOtherTenants : False
        AppPermissions          : {}
        IdentifierUris          : {https://www.contoso.org/example}
        ReplyUrls               : {}       

4. 通过传入 Active Directory 应用程序的应用程序 ID 创建应用程序的服务主体。

        PS C:\> New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

    现在你已在目录中创建服务主体，但未将任何权限或范围分配给服务。
    需要显式向服务主体授予权限，才能在某个范围执行操作。

5. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。对于 **ServicePrincipalName** 参数，请提供你在创建应用程序时使用的 **ApplicationId** 或 **IdentifierUris**。有关基于角色的访问控制的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)。

        PS C:\> New-AzureRmRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $azureAdApplication.ApplicationId

你已创建 Active Directory 应用程序并为该应用程序创建服务主体。你已向角色分配服务主体。现在，你需要以服务主体身份登录，以便以服务主体身份执行操作。在本主题中说明了两个选项：

- [通过自动执行的 PowerShell 脚本提供证书](#provide-certificate-through-automated-powershell-script)
- [通过应用程序中的代码提供证书](#provide-certificate-through-code-in-an-application)

<a id="provide-certificate-through-automated-powershell-script" /></a>
### 通过自动执行的 PowerShell 脚本提供证书

1. 获取 Active Directory 应用程序。在登录时，需要应用程序 ID

        PS C:\> $azureAdApplication = Get-AzureRmADApplication -IdentifierUri "https://www.contoso.org/example"
        
2. 检索在其中创建了角色分配的订阅。稍后将使用此订阅来获取服务主体角色分配所在租户的 **TenantId**。

        PS C:\> $subscription = Get-AzureRmSubscription

     如果角色分配不是在当前选择的订阅中创建的，你可以指定 **SubscriptoinId** 或 **SubscriptionName** 参数来检索其他订阅。

3. 获取将用于身份验证的证书。

        PS C:\> $cert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @("C:\certificates\examplecert.pfx", "yourpassword")

4. 若要使用 PowerShell 进行身份验证，请提供证书指纹、应用程序 ID 和租户 ID。

        PS C:\> Login-AzureRmAccount -EnvironmentName AzureChinaCloud -CertificateThumbprint $cert.Thumbprint -ApplicationId $azureAdApplication.ApplicationId -ServicePrincipal -TenantId $subscription.TenantId

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

<a id="provide-certificate-through-code-in-an-application" /></a>
### 通过应用程序中的代码提供证书

若要从 .NET 应用程序进行身份验证，请包含以下代码。检索客户端之后，您可以访问订阅中的资源。

    var clientId = "<Application ID for your AAD app>"; 
    var subscriptionId = "<Your Azure SubscriptionId>"; 
    var tenant = "<Tenant id or tenant name>"; 
    var certName = "examplecert"; 

        var authContext = new AuthenticationContext(string.Format("https://login.chinacloudapi.cn/{0}", tenant)); 

    X509Certificate2 cert = null; 
    X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser); 
    
    try 
    { 
      store.Open(OpenFlags.ReadOnly); 
      var certCollection = store.Certificates; 
      var certs = certCollection.Find(X509FindType.FindBySubjectName, certName, false); 
      cert = certs[0]; 
    } 
    finally 
    { 
      store.Close(); 
    }        

    var certCred = new ClientAssertionCertificate(clientId, cert); 
        var token = authContext.AcquireToken("https://management.core.chinacloudapi.cn/", certCred); 
    var creds = new TokenCloudCredentials(subscriptionId, token.AccessToken); 
    var client = new ResourceManagementClient(creds); 
        

## 使用密码进行身份验证 - Azure CLI

首先，创建一个服务主体。为此，我们必须在目录中创建一个应用程序。本部分将指导你在目录中创建新的应用程序。

1. 切换到 Azure 资源管理器模式并登录到你的帐户。

        azure config mode arm
        azure login

2. 运行 **azure ad app create** 命令创建新的 Active Directory 应用程序。提供应用程序的显示名称、描述应用程序的页面的 URI（该链接未验证）、标识应用程序的 URI，以及应用程序标识的密码。

        azure ad app create --name "exampleapp" --home-page "https://www.contoso.org" --identifier-uris "https://www.contoso.org/example" --password <Your_Password>
        
    返回 Active Directory 应用程序。需要使用 AppId 属性来创建服务主体、进行角色分配以及获取访问令牌。

        data:    AppId:          4fd39843-c338-417d-b549-a545f584a745
        data:    ObjectId:       4f8ee977-216a-45c1-9fa3-d023089b2962
        data:    DisplayName:    exampleapp
        ...
        info:    ad app create command OK

3. 创建应用程序的服务主体。提供上一步返回的应用程序 ID。

        azure ad sp create 4fd39843-c338-417d-b549-a545f584a745
        
    随后将返回新的服务主体。授权时需要使用对象 ID。
    
        info:    Executing command ad sp create
        - Creating service principal for application 4fd39843-c338-417d-b549-a545f584a74+
        data:    Object Id:        7dbc8265-51ed-4038-8e13-31948c7f4ce7
        data:    Display Name:     exampleapp
        data:    Service Principal Names:
        data:                      4fd39843-c338-417d-b549-a545f584a745
        data:                      https://www.contoso.org/example
        info:    ad sp create command OK

    现在你已在目录中创建服务主体，但未将任何权限或范围分配给服务。
    需要显式向服务主体授予权限，才能在某个范围执行操作。

4. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。有关基于角色的访问控制的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)。

        azure role assignment create --objectId 7dbc8265-51ed-4038-8e13-31948c7f4ce7 -o Reader -c /subscriptions/{subscriptionId}/

你已创建 Active Directory 应用程序并为该应用程序创建服务主体。你已向角色分配服务主体。现在，你需要以服务主体身份登录，以便以服务主体身份执行操作。在本主题中说明了三个选项：

- [通过 Azure CLI 手动提供凭据](#manually-provide-credentials-through-azure-cli)
- [通过自动执行的 Azure CLI 脚本提供凭据](#provide-credentials-through-automated-azure-cli-script)
- [通过应用程序中的代码提供凭据](#provide-credentials-through-code-in-an-application-cli)

<a id="manually-provide-credentials-through-Azure-cli" /></a>
### 通过 Azure CLI 手动提供凭据

如果要手动以服务主体身份登录，可以使用 **azure login** 命令。你必须提供租户 ID、应用程序 ID 和密码。
直接在脚本中包含密码不安全，因为密码将存储在文件中。有关执行自动执行的脚本时的更好选择，请参阅下一部分。

1. 确定包含服务主体的订阅的 **TenantId**。如果你要检索当前进行验证的订阅的租户 ID，则不需要提供订阅 ID 作为参数。**-r** 开关检索不带引号的值。

        tenantId=$(azure account show -s <subscriptionId> --json | jq -r '.[0].tenantId')

2. 对于用户名，请使用你在创建服务主体时所用的 **AppId**。如果需要检索应用程序 ID，请使用以下命令。在 **search** 参数中提供 Active Directory 应用程序的名称。

        appId=$(azure ad app show --search exampleapp --json | jq -r '.[0].appId')

3. 以服务主体身份登录。

        azure login -u "$appId" --service-principal --tenant "$tenantId"

出现提示时，请提供在创建帐户时指定的密码。

    info:    Executing command login
    Password: ********

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

<a id="provide-credentials-through-automated-azure-cli-script" /></a>
### 通过自动执行的 Azure CLI 脚本提供凭据

本部分说明如何以服务主体身份登录，而无需手动输入凭据。

> [AZURE.NOTE] 在 Azure CLI 脚本中包含密码不安全，因为密码将以文本的形式公开。请改用密钥保管库等服务来存储密码，并在执行脚本时检索它。

这些步骤假定你已设置密码存储的密钥保管库和机密。若要通过模板部署密钥保管库和机密，请参阅[密钥保管库模板格式]()。若要了解有关密钥保管库的信息，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)。

1. 从密钥保管库中检索密码（在下面的示例中，使用名称 **appPassword** 存储为机密）。包括 **-r** 开关可删除从 json 输出返回的起始和结束双引号。

        secret=$(azure keyvault secret show --vault-name examplevault --secret-name appPassword --json | jq -r '.value')
    
2. 确定包含服务主体的订阅的 **TenantId**。如果你要检索当前进行验证的订阅的租户 ID，则不需要提供订阅 ID 作为参数。

        tenantId=$(azure account show -s <subscriptionId> --json | jq -r '.[0].tenantId')

3. 对于用户名，请使用你在创建服务主体时所用的 **AppId**。如果需要检索应用程序 ID，请使用以下命令。在 **search** 参数中提供 Active Directory 应用程序的名称。

        appId=$(azure ad app show --search exampleapp --json | jq -r '.[0].appId')

4. 通过提供应用程序 ID、密钥保管库中的密码和租户 ID 以服务主体身份登录。

        azure login -u "$appId" -p "$secret" --service-principal --tenant "$tenantId"
    
现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

<a id="provide-credentials-through-code-in-an-application-cli" /></a>
### 通过应用程序中的代码提供凭据

若要从 .NET 应用程序进行身份验证，请包含以下代码。你将需要 Active Directory 应用程序的应用程序 ID、密码和订阅的租户 ID。
检索访问令牌之后，你便可以使用订阅中的资源了。

    public static string GetAccessToken()
    {
      var authenticationContext = new AuthenticationContext("https://login.windows.net/{tenantId or tenant name}");  
      var credential = new ClientCredential(clientId: "{application id}", clientSecret: "{application password}");
      var result = authenticationContext.AcquireToken(resource: "https://management.core.windows.net/", clientCredential:credential);

      if (result == null) {
        throw new InvalidOperationException("Failed to obtain the JWT token");
      }

      string token = result.AccessToken;

      return token;
    }

## 使用证书进行身份验证 - Azure CLI

在本部分，你将执行以下步骤，为使用证书进行身份验证的 Azure Active Directory 应用程序创建服务主体。
本主题假设你获得所颁发的证书并已安装 [OpenSSL](http://www.openssl.org/)。

1. 使用以下命令创建 **.pem** 文件：

        openssl pkcs12 -in C:\certificates\examplecert.pfx -out C:\certificates\examplecert.pem -nodes

2. 打开 **.pem** 文件并复制证书数据。查找 **-----BEGIN CERTIFICATE-----** 和 **-----END CERTIFICATE-----** 之间的长字符序列。

3. 通过运行 **azure ad app create** 命令创建新的 Active Directory 应用程序，并提供你在上一步中复制为密钥值的证书数据。

        azure ad app create -n "exampleapp" --home-page "https://www.contoso.org" -i "https://www.contoso.org/example" --key-value <certificate data>

    返回 Active Directory 应用程序。需要使用 AppId 属性来创建服务主体、进行角色分配以及获取访问令牌。

        data:    AppId:          4fd39843-c338-417d-b549-a545f584a745
        data:    ObjectId:       4f8ee977-216a-45c1-9fa3-d023089b2962
        data:    DisplayName:    exampleapp
        ...
        info:    ad app create command OK

4. 创建应用程序的服务主体。提供上一步返回的应用程序 ID。

        azure ad sp create 4fd39843-c338-417d-b549-a545f584a745
        
    随后将返回新的服务主体。授权时需要使用对象 ID。
    
        info:    Executing command ad sp create
        - Creating service principal for application 4fd39843-c338-417d-b549-a545f584a74+
        data:    Object Id:        7dbc8265-51ed-4038-8e13-31948c7f4ce7
        data:    Display Name:     exampleapp
        data:    Service Principal Names:
        data:                      4fd39843-c338-417d-b549-a545f584a745
        data:                      https://www.contoso.org/example
        info:    ad sp create command OK
        
5. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。有关基于角色的访问控制的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)。

        azure role assignment create --objectId 7dbc8265-51ed-4038-8e13-31948c7f4ce7 -o Reader -c /subscriptions/{subscriptionId}/

你已创建 Active Directory 应用程序并为该应用程序创建服务主体。你已向角色分配服务主体。现在，你需要以服务主体身份登录，以便以服务主体身份执行操作。在本主题中说明了两个选项：

- [通过自动执行的 Azure CLI 脚本提供证书](#provide-certificate-through-automated-azure-cli-script)
- [通过应用程序中的代码提供证书](#provide-certificate-through-code-in-an-application-cli)

<a id="provide-certificate-through-automated-azure-cli-script" /></a>
### 通过自动执行的 Azure CLI 脚本提供证书

1. 你需要检索证书指纹，并删除不需要的字符。

        cert=$(openssl x509 -in "C:\certificates\examplecert.pem" -fingerprint -noout | sed 's/SHA1 Fingerprint=//g'  | sed 's/://g')
    
     它返回的指纹值类似于：

        30996D9CE48A0B6E0CD49DBB9A48059BF9355851

2. 确定包含服务主体的订阅的 **TenantId**。如果你要检索当前进行验证的订阅的租户 ID，则不需要提供订阅 ID 作为参数。**-r** 开关检索不带引号的值。

        tenantId=$(azure account show -s <subscriptionId> --json | jq -r '.[0].tenantId')

3. 对于用户名，请使用你在创建服务主体时所用的 **AppId**。如果需要检索应用程序 ID，请使用以下命令。在 **search** 参数中提供 Active Directory 应用程序的名称。

        appId=$(azure ad app show --search exampleapp --json | jq -r '.[0].appId')

4. 若要使用 Azure CLI 进行身份验证，请提供证书指纹、证书文件、应用程序 ID 和租户 ID。

        azure login --service-principal --tenant "$tenantId" -u "$appId" --certificate-file C:\certificates\examplecert.pem --thumbprint "$cert"

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

<a id="provide-certificate-through-code-in-an-application-cli" /></a>
### 通过应用程序中的代码提供证书

若要从 .NET 应用程序进行身份验证，请包含以下代码。检索客户端之后，您可以访问订阅中的资源。

    var clientId = "<Application ID for your AAD app>"; 
    var subscriptionId = "<Your Azure SubscriptionId>"; 
    var tenant = "<Tenant id or tenant name>"; 
    var certName = "examplecert"; 

    var authContext = new AuthenticationContext(string.Format("https://login.windows.net/{0}", tenant)); 

    X509Certificate2 cert = null; 
    X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser); 
    
    try 
    { 
      store.Open(OpenFlags.ReadOnly); 
      var certCollection = store.Certificates; 
      var certs = certCollection.Find(X509FindType.FindBySubjectName, certName, false); 
      cert = certs[0]; 
    } 
    finally 
    { 
      store.Close(); 
    }        

    var certCred = new ClientAssertionCertificate(clientId, cert); 
    var token = authContext.AcquireToken("https://management.core.windows.net/", certCred); 
    var creds = new TokenCloudCredentials(subscriptionId, token.AccessToken); 
    var client = new ResourceManagementClient(creds); 
       
若要获取有关使用证书和 Azure CLI 的详细信息，请参阅[从 Linux 命令行对 Azure 服务主体进行基于证书的身份验证](http://blogs.msdn.com/b/arsen/archive/2015/09/18/certificate-based-auth-with-azure-service-principals-from-linux-command-line.aspx)

## 后续步骤
  
- 若要了解有关使用服务主体的门户的信息，请参阅[使用 Azure 门户创建新的 Azure 服务主体](/documentation/articles/resource-group-create-service-principal-portal/)  
- 有关在 Azure 资源管理器中实现安全性的指南，请参阅 [Azure 资源管理器的安全注意事项](/documentation/articles/best-practices-resource-manager-security/)


<!-- Images. -->
[1]: ./media/resource-group-authenticate-service-principal/arm-get-credential.png

<!---HONumber=Mooncake_0405_2016-->