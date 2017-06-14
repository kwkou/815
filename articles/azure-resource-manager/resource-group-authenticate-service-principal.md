<properties
    pageTitle="使用 PowerShell 创建 Azure 应用标识 | Azure"
    description="介绍如何使用 Azure PowerShell 创建 Azure Active Directory 应用程序和服务主体，并通过基于角色的访问控制向其授予资源访问权限。 它演示如何使用密码或证书对应用程序进行身份验证。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="d2caf121-9fbe-4f00-bf9d-8f3d1f00a6ff"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="multiple"
    ms.workload="na"
    ms.date="04/03/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="0230288335356c585213ee65358456e414c0d472"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="use-azure-powershell-to-create-a-service-principal-to-access-resources"></a>使用 Azure PowerShell 创建服务主体来访问资源
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-authenticate-service-principal/)
- [Azure CLI](/documentation/articles/resource-group-authenticate-service-principal-cli/)
- [门户](/documentation/articles/resource-group-create-service-principal-portal/)

当某个应用或脚本需要访问资源时，可以为应用设置一个标识，然后使用其自身的凭据进行身份验证。 此标识称为服务主体。 使用此方法可实现以下目的：

* 可以将权限分配给应用标识，这些权限不同于你自己的权限。通常情况下，这些权限仅限于应用需执行的操作。
* 执行无人参与的脚本时，使用证书进行身份验证。

本主题介绍如何通过 [Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azure/overview) 为应用程序进行一切所需设置，使之能够使用自己的凭据和标识运行。

## <a name="required-permissions"></a>所需的权限
若要完成本主题，必须在 Azure Active Directory 和 Azure 订阅中均具有足够的权限。 具体而言，必须能够在 Azure Active Directory 中创建应用并向角色分配服务主体。 

检查帐户是否有足够权限的最简方法是使用门户。 请参阅[检查所需的权限](/documentation/articles/resource-group-create-service-principal-portal/#required-permissions)。

现在转到[密码](#create-service-principal-with-password)或[证书](#create-service-principal-with-certificate)身份验证部分。

## <a name="create-service-principal-with-password"></a>使用密码创建服务主体
以下脚本为应用程序创建标识，然后将该标识分配到指定范围的“参与者”角色：

    Param (

     # Use to set scope to resource group. If no value is provided, scope is set to subscription.
     [Parameter(Mandatory=$false)]
     [String] $ResourceGroup,

     # Use to set subscription. If no value is provided, default subscription is used. 
     [Parameter(Mandatory=$false)]
     [String] $SubscriptionId,

     [Parameter(Mandatory=$true)]
     [String] $ApplicationDisplayName,

     [Parameter(Mandatory=$true)]
     [String] $Password
    )

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    Import-Module AzureRM.Resources

    if ($SubscriptionId -eq "") 
    {
        $SubscriptionId = (Get-AzureRmContext).Subscription.SubscriptionId
    }
    else
    {
        Set-AzureRmContext -SubscriptionId $SubscriptionId
    }

    if ($ResourceGroup -eq "")
    {
        $Scope = "/subscriptions/" + $SubscriptionId
    }
    else
    {
        $Scope = (Get-AzureRmResourceGroup -Name $ResourceGroup -ErrorAction Stop).ResourceId
    }

    # Create Azure Active Directory application with password
    $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $ApplicationDisplayName) -IdentifierUris ("http://" + $ApplicationDisplayName) -Password $Password

    # Create Service Principal for the AD app
    $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
    Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id 

    $NewRole = $null
    $Retries = 0;
    While ($NewRole -eq $null -and $Retries -le 6)
    {
        # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
        Sleep 15
        New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId -Scope $Scope | Write-Verbose -ErrorAction SilentlyContinue
        $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
        $Retries++;
    }

有关该脚本的几个注意事项：

* 若要向标识授予对默认订阅的访问权限，不需要提供 ResourceGroup 或 SubscriptionId 参数。
* 仅当你要将角色分配范围限制为某个资源组时，才需要指定 ResourceGroup 参数。
* 对于单租户应用程序，不会验证主页和标识符 URI。
* 在此示例中，你要将服务主体添加到“参与者”角色。 对于其他角色，请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)。
* 该脚本休眠 15 秒，让新的服务主体有时间传遍 Azure Active Directory。 如果脚本等待时长不足，将显示错误，称“PrincipalNotFound: 主体 {id} 不存在于目录中”。
* 如果需要向服务主体授予对其他订阅或资源组的访问权限，请使用不同的范围再次运行 `New-AzureRMRoleAssignment` cmdlet。

### <a name="provide-credentials-through-powershell"></a>通过 PowerShell 提供凭据
现在，需要以应用程序方式登录以执行相应操作。 对于用户名，请使用针对应用程序创建的 `ApplicationId`。 对于密码，请使用你在创建帐户时指定的密码。 

    $creds = Get-Credential
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud -Credential $creds -ServicePrincipal -TenantId {tenant-id}

租户 ID 不是敏感信息，可将它直接嵌入脚本中。 如果需要检索租户 ID，请使用：

    (Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId

## <a name="create-service-principal-with-self-signed-certificate"></a>使用自签名证书创建服务主体
若要使用 Windows 10 上的 Azure PowerShell 2.0 或 Windows Server 2016 Technical Preview 生成自签名证书和服务主体，请使用以下脚本：

    Param (

     # Use to set scope to resource group. If no value is provided, scope is set to subscription.
     [Parameter(Mandatory=$false)]
     [String] $ResourceGroup,

     # Use to set subscription. If no value is provided, default subscription is used. 
     [Parameter(Mandatory=$false)]
     [String] $SubscriptionId,

     [Parameter(Mandatory=$true)]
     [String] $ApplicationDisplayName
    )

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    Import-Module AzureRM.Resources

    if ($SubscriptionId -eq "") 
    {
        $SubscriptionId = (Get-AzureRmContext).Subscription.SubscriptionId
    }
    else
    {
        Set-AzureRmContext -SubscriptionId $SubscriptionId
    }

    if ($ResourceGroup -eq "")
    {
        $Scope = "/subscriptions/" + $SubscriptionId
    }
    else
    {
        $Scope = (Get-AzureRmResourceGroup -Name $ResourceGroup -ErrorAction Stop).ResourceId
    }

    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=exampleappScriptCert" -KeySpec KeyExchange
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

    # Use Key credentials
    $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $ApplicationDisplayName) -IdentifierUris ("http://" + $ApplicationDisplayName) -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore

    $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
    Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id 

    $NewRole = $null
    $Retries = 0;
    While ($NewRole -eq $null -and $Retries -le 6)
    {
        # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
        Sleep 15
        New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId -Scope $Scope | Write-Verbose -ErrorAction SilentlyContinue
        $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
        $Retries++;
    }

有关该脚本的几个注意事项：

* 若要向标识授予对默认订阅的访问权限，不需要提供 ResourceGroup 或 SubscriptionId 参数。
* 仅当你要将角色分配范围限制为某个资源组时，才需要指定 ResourceGroup 参数。
* 对于单租户应用程序，不会验证主页和标识符 URI。
* 在此示例中，你要将服务主体添加到“参与者”角色。 对于其他角色，请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)。
* 该脚本休眠 15 秒，让新的服务主体有时间传遍 Azure Active Directory。 如果脚本等待时长不足，将显示错误，称“PrincipalNotFound: 主体 {id} 不存在于目录中”。
* 如果需要向服务主体授予对其他订阅或资源组的访问权限，请使用不同的范围再次运行 `New-AzureRMRoleAssignment` cmdlet。

如果未使用 **Windows 10 或 Windows Server 2016 Technical Preview**，需要从 Microsoft 脚本中心下载 [自签名证书生成器](https://gallery.technet.microsoft.com/scriptcenter/Self-signed-certificate-5920a7c6/) 。 解压其内容，并导入所需的 cmdlet。

    # Only run if you could not use New-SelfSignedCertificate
    Import-Module -Name c:\ExtractedModule\New-SelfSignedCertificateEx.ps1

  
在脚本中替换以下两行代码以生成证书。

    New-SelfSignedCertificateEx  -StoreLocation CurrentUser -StoreName My -Subject "CN=exampleapp" -KeySpec "Exchange" -FriendlyName "exampleapp"
    $cert = Get-ChildItem -path Cert:\CurrentUser\my | where {$PSitem.Subject -eq 'CN=exampleapp' }

### <a name="provide-certificate-through-automated-powershell-script"></a>通过自动执行的 PowerShell 脚本提供证书
以服务主体方式登录时，需提供 AD 应用所在目录的租户 ID。 租户是 Azure Active Directory 的实例。 如果只有一个订阅，可以使用：

    Param (

        [Parameter(Mandatory=$true)]
        [String] $CertSubject,

        [Parameter(Mandatory=$true)]
        [String] $ApplicationId,

        [Parameter(Mandatory=$true)]
        [String] $TenantId
    )

    $Thumbprint = (Get-ChildItem cert:\CurrentUser\My\ | Where-Object {$_.Subject -match $CertSubject }).Thumbprint
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud -ServicePrincipal -CertificateThumbprint $Thumbprint -ApplicationId $ApplicationId -TenantId $TenantId

应用程序 ID 和租户 ID 不是敏感信息，可将它们直接嵌入脚本中。 如果需要检索租户 ID，请使用：

    (Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId

如果需要检索应用程序 ID，请使用：

    (Get-AzureRmADApplication -DisplayNameStartWith {display-name}).ApplicationId

## <a name="create-service-principal-with-certificate-from-certificate-authority"></a>使用证书颁发机构提供的证书创建服务主体
若要使用证书颁发机构颁发的证书创建服务主体，请使用以下脚本：

    Param (
        [Parameter(Mandatory=$true)]
        [String] $ApplicationDisplayName,

        [Parameter(Mandatory=$true)]
        [String] $SubscriptionId,

        [Parameter(Mandatory=$true)]
        [String] $CertPath,

        [Parameter(Mandatory=$true)]
        [String] $CertPlainPassword
    )

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    Import-Module AzureRM.Resources
    Set-AzureRmContext -SubscriptionId $SubscriptionId

    $KeyId = (New-Guid).Guid
    $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force

    $PFXCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($CertPath, $CertPassword)
    $KeyValue = [System.Convert]::ToBase64String($PFXCert.GetRawCertData())

    $KeyCredential = New-Object  Microsoft.Azure.Commands.Resources.Models.ActiveDirectory.PSADKeyCredential
    $KeyCredential.StartDate = $PFXCert.NotBefore
    $KeyCredential.EndDate= $PFXCert.NotAfter
    $KeyCredential.KeyId = $KeyId
    $KeyCredential.CertValue = $KeyValue

    # Use Key credentials
    $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $ApplicationDisplayName) -IdentifierUris ("http://" + $KeyId) -KeyCredentials $keyCredential

    $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
    Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id 

    $NewRole = $null
    $Retries = 0;
    While ($NewRole -eq $null -and $Retries -le 6)
    {
        # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
        Sleep 15
        New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId | Write-Verbose -ErrorAction SilentlyContinue
        $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
        $Retries++;
    }

    $NewRole

有关该脚本的几个注意事项：

* 访问范围限定于订阅。
* 对于单租户应用程序，不会验证主页和标识符 URI。
* 在此示例中，你要将服务主体添加到“参与者”角色。 对于其他角色，请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)。
* 该脚本休眠 15 秒，让新的服务主体有时间传遍 Azure Active Directory。 如果脚本等待时长不足，将显示错误，称“PrincipalNotFound: 主体 {id} 不存在于目录中”。
* 如果需要向服务主体授予对其他订阅或资源组的访问权限，请使用不同的范围再次运行 `New-AzureRMRoleAssignment` cmdlet。

### <a name="provide-certificate-through-automated-powershell-script"></a>通过自动执行的 PowerShell 脚本提供证书
以服务主体方式登录时，需提供 AD 应用所在目录的租户 ID。 租户是 Azure Active Directory 的实例。

    Param (
 
     [Parameter(Mandatory=$true)]
     [String] $CertPath,

     [Parameter(Mandatory=$true)]
     [String] $CertPlainPassword,
 
     [Parameter(Mandatory=$true)]
     [String] $ApplicationId,

     [Parameter(Mandatory=$true)]
     [String] $TenantId
    )

    $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force
    $PFXCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($CertPath, $CertPassword)
    $Thumbprint = $PFXCert.Thumbprint

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud -ServicePrincipal -CertificateThumbprint $Thumbprint -ApplicationId $ApplicationId -TenantId $TenantId

应用程序 ID 和租户 ID 不是敏感信息，可将它们直接嵌入脚本中。 如果需要检索租户 ID，请使用：

    (Get-AzureRmSubscription -SubscriptionName "Contoso Default").TenantId

如果需要检索应用程序 ID，请使用：

    (Get-AzureRmADApplication -DisplayNameStartWith {display-name}).ApplicationId

## <a name="change-credentials"></a>更改凭据

若要更改 AD 应用的凭据（为了保障安全或由于凭据过期的缘故），请使用 [Remove-AzureRmADAppCredential](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.3.0/remove-azurermadappcredential) 和 [New-AzureRmADAppCredential](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/new-azurermadappcredential) cmdlet。

若要删除应用程序的所有凭据，请使用：

    Remove-AzureRmADAppCredential -ApplicationId 8bc80782-a916-47c8-a47e-4d76ed755275 -All

若要添加密码，请使用：

    New-AzureRmADAppCredential -ApplicationId 8bc80782-a916-47c8-a47e-4d76ed755275 -Password p@ssword!

若要添加证书值，请按本主题所示创建自签名证书。 然后，使用：

    New-AzureRmADAppCredential -ApplicationId 8bc80782-a916-47c8-a47e-4d76ed755275 -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore

## <a name="save-access-token-to-simplify-log-in"></a>保存访问令牌来简化登录
若要避免每次登录时都需要提供服务主体凭据，可保存访问令牌。

若要在以后的会话中使用当前访问令牌，请保存该配置文件。

    Save-AzureRmProfile -Path c:\Users\exampleuser\profile\exampleSP.json

打开该配置文件，并检查其内容。 请注意，它包含访问令牌。 无需再次手动登录，只需加载配置文件。

    Select-AzureRmProfile -Path c:\Users\exampleuser\profile\exampleSP.json

> [AZURE.NOTE]
> 访问令牌会过期，因此使用保存的配置文件仅适合在令牌有效期间使用。
>  

也可从要登录的 PowerShell 调用 REST 操作。 可以从身份验证响应中检索访问令牌，将其用于其他操作。 若要通过示例来了解如何通过调用 REST 操作来检索访问令牌，请参阅[生成访问令牌](/documentation/articles/resource-manager-rest-api/#generating-an-access-token)。

## <a name="debug"></a>调试

创建服务主体时，你可能会遇到以下错误：

* “Authentication_Unauthorized”或“在上下文中找不到订阅”。 - 如果帐户不具有在 Azure Active Directory 上注册应用[所需的权限](#required-permissions)，会收到此错误。 通常情况下，如果在 Azure Active Directory 中只有管理员用户才能注册应用，而你的帐户不是管理员，则会出现此错误。 可要求管理员为你分配管理员角色，或者允许用户注册应用。

* 你的帐户“无权对作用域 '/subscriptions/{guid}' 执行 'Microsoft.Authorization/roleAssignments/write' 操作。”- 当帐户没有足够权限，无法为标识分配角色时，将会出现此错误。 可要求订阅管理员将你添加到“用户访问管理员”角色。

## <a name="sample-applications"></a>示例应用程序
以下示例应用程序演示如何以服务主体身份登录。

**.NET**

* [在 .NET 中使用模板部署启用 SSH 的 VM](https://github.com/Azure-Samples/resource-manager-dotnet-template-deployment/)
* [使用 .NET 管理 Azure 资源和资源组](https://github.com/Azure-Samples/resource-manager-dotnet-resources-and-groups/)

**Java**

* [资源入门 - 在 Java 中使用 Azure Resource Manager 模板部署资源](https://github.com/Azure-Samples/resources-java-deploy-using-arm-template/)
* [资源入门 - 在 Java 中管理资源组](https://github.com/Azure-Samples/resources-java-manage-resource-group/)

**Python**

* [在 Python 中使用模板部署启用 SSH 的 VM](https://github.com/Azure-Samples/resource-manager-python-template-deployment/)
* [使用 Python 管理 Azure 资源和资源组](https://github.com/Azure-Samples/resource-manager-python-resources-and-groups/)

**Node.js**

* [在 Node.js 中使用模板部署启用 SSH 的 VM](https://github.com/Azure-Samples/resource-manager-node-template-deployment/)
* [使用 Node.js 管理 Azure 资源和资源组](https://github.com/Azure-Samples/resource-manager-node-resources-and-groups/)

**Ruby**

* [在 Ruby 中使用模板部署启用 SSH 的 VM](https://github.com/Azure-Samples/resource-manager-ruby-template-deployment/)
* [使用 Ruby 管理 Azure 资源和资源组](https://github.com/Azure-Samples/resource-manager-ruby-resources-and-groups/)

## <a name="next-steps"></a>后续步骤
* 有关将应用程序集成到 Azure 以管理资源的详细步骤，请参阅 [使用 Azure Resource Manager API 进行授权的开发人员指南](/documentation/articles/resource-manager-api-authentication/)。
* 有关应用程序和服务主体的详细说明，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)。 
* 有关 Active Directory 身份验证的详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)。


<!-- Update_Description: update meta properties; wording update; update link reference -->
