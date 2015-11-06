<properties
   pageTitle="通过 Azure 资源管理器对服务主体进行身份验证"
   description="介绍如何通过基于角色的访问控制向服务主体授予访问权限，并对服务主体进行身份验证。演示如何使用 PowerShell 和 Azure CLI 执行这些任务。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="08/14/2015"
   wacn.date="10/3/2015"/>

# 通过 Azure 资源管理器对服务主体进行身份验证

本主题说明如何允许服务主体（例如自动化的过程、应用程序或服务）访问订阅中的其他资源。借助 Azure 资源管理器，可以使用基于角色的访问控制向服务主体授予允许的操作，并对该服务主体进行身份验证。本主题说明如何使用 PowerShell 和 Azure CLI 将角色分配给服务主体，并对服务主体进行身份验证。

它演示如何使用用户名和密码或证书进行身份验证。

您可以使用适用于 Mac、Linux 和 Windows 的 Azure PowerShell 或 Azure CLI。如果你未安装 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。如果您未安装 Azure CLI，请参阅[安装和配置 Azure CLI](/documentation/articlesxplat-cli-install)。

## 概念
1. Azure Active Directory (AAD) - 云的标识与访问管理服务。有关详细信息，请参阅[什么是 Azure Active Directory](/documentation/articles/active-directory-whatis)
2. 服务主体 - 目录中需要访问其他资源的应用程序实例。
3. AD 应用程序 - 向 AAD 标识某个应用程序的目录记录。有关详细信息，请参阅 [Azure AD 中的身份验证基本知识](https://msdn.microsoft.com/zh-cn/library/azure/874839d9-6de6-43aa-9a5c-613b0c93247e#BKMK_Auth)。

## 使用密码对服务主体进行身份验证 - PowerShell

在此部分中，您将执行以下用于创建 Azure Active Directory 应用程序的服务主体、将角色分配给服务主体，通过提供应用程序标识符和密码作为服务主体进行身份验证的步骤。

1. 运行 **New-AzureADApplication** 命令创建新的 AAD 应用程序。提供应用程序的显示名称、描述应用程序的页面的 URI（该链接未验证）、标识应用程序的 URI，以及应用程序标识的密码。

        PS C:\> $azureAdApplication = New-AzureADApplication -DisplayName "<Your Application Display Name>" -HomePage "<https://YourApplicationHomePage>" -IdentifierUris "<https://YouApplicationUri>" -Password "<Your_Password>"

     检查新的应用程序对象。需要使用 **ApplicationId** 属性创建服务主体、角色分配以及获取 JWT 令牌。

        PS C:\> $azureAdApplication

        Type                    : Application
        ApplicationId           : a41acfda-d588-47c9-8166-d659a335a865
        ApplicationObjectId     : a26aaa48-bd52-4a7f-b29f-1bebf74c91e3
        AvailableToOtherTenants : False
        AppPermissions          : {{
                            "claimValue": "user_impersonation",
                            "description": "Allow the application to access <Your Application Display Name> on behalf of the signed-in user.",
                            "directAccessGrantTypes": [],
                            "displayName": "Access <Your Application Display Name>",
                            "impersonationAccessGrantTypes": [
                              {
                                "impersonated": "User",
                                "impersonator": "Application"
                              }
                            ],
                            "isDisabled": false,
                            "origin": "Application",
                            "permissionId": "b866ef28-9abb-4698-8c8f-eb4328533831",
                            "resourceScopeType": "Personal",
                            "userConsentDescription": "Allow the application to access <Your Application Display Name> on your behalf.",
                            "userConsentDisplayName": "Access <Your Application Display Name>",
                            "lang": null
                          }}


2. 创建应用程序的服务主体。

        PS C:\> New-AzureADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

     现在你已在目录中创建服务主体，但未将任何权限或范围分配给服务。需要显式向服务主体授予权限，才能在某个范围执行操作。

3. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。对于 **ServicePrincipalName** 参数，请提供你在创建应用程序时使用的 **ApplicationId** 或 **IdentifierUris**。有关基于角色的访问控制的详细信息，请参阅<!--[-->管理和审核对资源的访问权限<!--](/documentation/articles/resource-group-rbac)-->

        PS C:\> New-AzureRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $azureAdApplication.ApplicationId

4. 检索在其中创建了角色分配的订阅。稍后将使用此订阅来获取服务主体角色分配所在租户的 **TenantId**。

        PS C:\> $subscription = Get-AzureSubscription | where { $_.IsCurrent }

     如果角色分配不是在当前选择的订阅中创建的，你可以指定 **SubscriptoinId** 或 **SubscriptionName** 参数来检索其他订阅。

5. 运行 **Get-Credential** 命令，以创建包含你的凭据的新 **PSCredential** 对象。

        PS C:\> $creds = Get-Credential

     系统会提示你输入凭据。

     ![][1]

     对于用户名，请使用你在创建应用程序时所用的 **ApplicationId** 或 **IdentifierUris**。对于密码，请使用你在创建帐户时指定的密码。

6. 使用输入的凭据作为 **Add-AzureAccount** Cmdlet 的输入，以将服务主体登录：

        PS C:\> Add-AzureAccount -Environment AzureChinaCloud -Credential $creds -ServicePrincipal -Tenant $subscription.TenantId

     现在，你应该已经作为所创建 AAD 应用程序的服务主体进行身份验证。


## 使用证书对服务主体进行身份验证 - PowerShell

在此部分中，您将执行以下用于创建 Azure Active Directory 应用程序的服务主体、将角色分配给服务主体，通过提供证书作为服务主体进行身份验证的步骤。本主题假定您已获得所颁发的证书。

它介绍了两种使用证书的方法 - 密钥凭据和密钥值。您可以使用任意一种方法。

首先，当创建应用程序时，必须在 PowerShell 中设置以后将用到的某些值。

1. 对于这两种方法，从您的证书创建一个 X509Certificate 对象并检索密钥值。使用指向您的证书的路径和该证书的密码。

        $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate("C:\certificates\examplecert.pfx", "yourpassword")
        $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

2. 如果您使用密钥凭据，请创建密钥凭据对象并将其值设置为上一步中的 `$keyValue`。

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

        $azureAdApplication = New-AzureADApplication -DisplayName "<Your Application Display Name>" -HomePage "<https://YourApplicationHomePage>" -IdentifierUris "<https://YouApplicationUri>" -KeyValue $keyValue -KeyType AsymmetricX509Cert       
        
    或者，使用第二个示例分配密钥凭据。

         $azureAdApplication = New-AzureADApplication -DisplayName "<Your Application Display Name>" -HomePage "<https://YourApplicationHomePage>" -IdentifierUris "<https://YouApplicationUri>" -KeyCredentials $keyCredential

    检查新的应用程序对象。需要使用 **ApplicationId** 属性创建服务主体、角色分配以及获取 JWT 令牌。

        PS C:\> $azureAdApplication

        Type                    : Application
        ApplicationId           : 76fa8d97-f07e-4b9a-b871-a57a7acd777a
        ApplicationObjectId     : c36b7b57-a949-4401-b381-18a5210aff10
        AvailableToOtherTenants : False
        AppPermissions          : {{
                            "claimValue": "user_impersonation",
                            "description": "Allow the application to access <Your Application Display Name> on behalf of the signed-in
                          user.",
                            "directAccessGrantTypes": [],
                            "displayName": "Access <Your Application Display Name>",
                            "impersonationAccessGrantTypes": [
                              {
                                "impersonated": "User",
                                "impersonator": "Application"
                              }
                            ],
                            "isDisabled": false,
                            "origin": "Application",
                            "permissionId": "9f13c6c6-35ba-43d6-b8b3-6a87aa641388",
                            "resourceScopeType": "Personal",
                            "userConsentDescription": "Allow the application to access <Your Application Display Name> on your behalf.",
                            "userConsentDisplayName": "Access <Your Application Display Name>",
                            "lang": null
                          }}

4. 创建应用程序的服务主体。

        PS C:\> New-AzureADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

    现在你已在目录中创建服务主体，但未将任何权限或范围分配给服务。需要显式向服务主体授予权限，才能在某个范围执行操作。

5. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。对于 **ServicePrincipalName** 参数，请提供你在创建应用程序时使用的 **ApplicationId** 或 **IdentifierUris**。有关基于角色的访问控制的详细信息，请参阅<!--[-->管理和审核对资源的访问权限<!--](/documentation/articles/resource-group-rbac)-->

        PS C:\> New-AzureRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $azureAdApplication.ApplicationId

6. 若要从应用程序进行身份验证，请包含以下代码。检索客户端之后，您可以访问订阅中的资源。

        string clientId = "<Client ID for your AAD app>"; 
        var subscriptionId = "<Your Azure SubscriptionId>"; 
        string tenant = "<AAD tenant name>.onmicrosoft.com"; 

        var authContext = new AuthenticationContext(string.Format("https://login.windows.net/{0}", tenant)); 

        X509Certificate2 cert = null; 
        X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser); 
        string certName = "examplecert"; 
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
        

## 使用密码对服务主体进行身份验证 - Azure CLI

首先，创建一个服务主体。为此，我们必须在目录中创建一个应用程序。本部分将指导你在目录中创建新的应用程序。

1. 运行 **azure ad app create** 命令创建新的 AAD 应用程序。提供应用程序的显示名称、描述应用程序的页面的 URI（该链接未验证）、标识应用程序的 URI，以及应用程序标识的密码。

        azure ad app create --name "<Your Application Display Name>" --home-page "<https://YourApplicationHomePage>" --identifier-uris "<https://YouApplicationUri>" --password <Your_Password>
        
    将返回该 Azure AD 应用程序。需要使用 ApplicationId 属性创建服务主体、角色分配以及获取 JWT 令牌。

        info:    Executing command ad app create
        + Creating application exampleapp                                                
        data:    Application Id:          b57dd71d-036c-4840-865e-23b71d8098ec
        data:    Application Object Id:   d5c519e2-6149-447e-b323-88d2c4ea27de
        data:    Application Permissions:  
        data:                             claimValue:  user_impersonation
        data:                             description:  Allow the application to access exampleapp on behalf of the signed-in user.
        ...
        info:    ad app create command OK

2. 创建应用程序的服务主体。提供上一步返回的应用程序 ID。

        azure ad sp create b57dd71d-036c-4840-865e-23b71d8098ec
        
    随后将返回新的服务主体。授权时需要使用对象 ID。
    
        info:    Executing command ad sp create
        + Creating service principal for application b57dd71d-036c-4840-865e-23b71d8098ec
        data:    Object Id:               47193a0a-63e4-46bd-9bee-6a9f6f9c03cb
        data:    Display Name:            exampleapp
        ...
        info:    ad sp create command OK

    现在你已在目录中创建服务主体，但未将任何权限或范围分配给服务。需要显式向服务主体授予权限，才能在某个范围执行操作。

3. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。对于 **ServicePrincipalName** 参数，请提供你在创建应用程序时使用的 **ApplicationId** 或 **IdentifierUris**。有关基于角色的访问控制的详细信息，请参阅<!--[-->管理和审核对资源的访问权限<!--](/documentation/articles/resource-group-rbac)-->

        azure role assignment create --objectId 47193a0a-63e4-46bd-9bee-6a9f6f9c03cb -o Reader -c /subscriptions/{subscriptionId}/

4. 若要确定服务主体角色分配所在租户的 **TenantId**，请列出帐户，然后查找输出中的 **TenantId** 属性。

        azure account list

5. 使用作为你的标识的服务主体登录。对于用户名，请使用你在创建应用程序时所用的 **ApplicationId**。对于密码，请使用你在创建帐户时指定的密码。

        azure login -u "<ApplicationId>" -p "<password>" --service-principal --tenant "<TenantId>"

    现在，你应该已经作为所创建 AAD 应用程序的服务主体进行身份验证。

## 后续步骤
  
- 有关基于角色的访问控制的概述，请参阅<!--[-->管理和审核对资源的访问权限<!--](/documentation/articles/resource-group-rbac)-->  
- 若要了解有关使用服务主体的门户的信息，请参阅[使用 Azure 门户创建新的 Azure 服务主体](/documentation/articles/resource-group-create-service-principal-portal)  
- 有关在 Azure 资源管理器中实现安全性的指南，请参阅 [Azure 资源管理器的安全注意事项](/documentation/articles/best-practices-resource-manager-security)


<!-- Images. -->
[1]: ./media/resource-group-authenticate-service-principal/arm-get-credential.png

<!---HONumber=71-->