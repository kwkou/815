<properties
    pageTitle="在 Azure 自动化中通过证书管理 Resource Manager 资源"
    description="在 Azure 自动化中通过证书管理 Resource Manager 资源"
    service=""
    resource="automation"
    authors="Steve Shi"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Automation, PowerShell, Certification, ARM"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="automation-aog"
    ms.date=""
    wacn.date="03/31/2017" />

# 在 Azure 自动化中通过证书管理 Resource Manager 资源

借助 Azure 自动化，用户可以自动完成通常要在云环境和企业环境中执行的手动、长时间进行、易出错且重复性高的任务。它可以节省时间，可以提高常规管理任务的可靠性，甚至可以将这些任务安排成按特定的时间间隔自动执行。

运行 Azure 自动化任务需要首先在自动化任务中使用 PowerShell 命令登录 Azure，通常通过调用存放在自动化账户里的 Azure 账号用户名密码，使用 `Add-AzureAccount`，`Login-AzureRmAccount` 等 PowerShell 命令登录后，才能正常运行后续的命令。

使用证书验证来替代通常的用户名密码验证，可以避免由于以下原因导致的自动化任务不可用：

1. 更改了 Azure 账号密码，但忘了更新自动化账户的 Runbook 所使用的账号密码。
2. 启用了[多重身份验证](/documentation/services/multi-factor-authentication/)的 Azure 账号，在自动化任务中调用用户名密码无法完成登录验证。

## 前提条件

- 有效的 Azure 订阅
- 已经安装了 [Azure PowerShell 3.1.0](https://github.com/Azure/azure-powershell/releases/tag/v3.1.0-November2016) 模块的 Windows 电脑

## 操作步骤

### 查看并更新 Azure 自动化账户的 PowerShell 模块版本

由于 Azure 自动化账户默认使用的 PowerShell 版本过旧，需要更新相应的模块版本。请参阅[如何更新 Azure 自动化服务的 PowerShell 模块](/documentation/articles/aog-automation-powershell-module-update/)，更新 Azure 自动化账户资产里的 `AzureRm.Profile`、`AzureRM.Storage`、`AzureRM.Compute` 模块至 2.2.0 或以上，Azure 模块至 3.1.0 或以上，否则 PowerShell 脚本会报错。

本地电脑上的 AzureRM 子模块默认存放在 `C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ResourceManager\AzureResourceManager`， Azure 模块默认存放在 `C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement`。如果压缩 zip 文件大小超过了 40MB，导致上传至自动化账户时报错，请使用 7zip 压缩工具，在压缩时选择极限压缩，保证压缩文件不要超过 40MB。

### 准备验证使用的证书

用户可以使用以下 PowerShell 命令在本地电脑中生成自签名证书，请自行更改证书 CN 名 `$CertCNName` 变量和 `$CentPlainPassword` 密码变量，此段命令会在本地电脑中生成一张 CN 名为 AzureRmAutologin 的自签名证书，并将 .pfx 格式的证书文件以 `$CentPlainPassword` 密码变量打包，输出至当前用户的桌面上。

    $CertCNName = "AzureRmAutologin"
    $CertPlainPassword = "Password"
    #密码存放位置为当前用户的桌面
    $CertPath = Join-Path $env:USERPROFILE ("\Desktop\" + $CertDNSName + ".pfx")
    #生成证书
    $Cert = New-SelfSignedCertificate -Subject ("CN="+$CertCNName) -CertStoreLocation cert:\LocalMachine\My -KeyExportPolicy Exportable -Provider "Microsoft Enhanced RSA and AES Cryptographic Provider" -KeySpec KeyExchange
    $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force
    #输出.pfx 格式证书
    Export-PfxCertificate -Cert ("Cert:\localmachine\my\" + $Cert.Thumbprint) -FilePath $CertPath -Password $CertPassword -Force | Write-Verbose
    $KeyValue = [System.Convert]::ToBase64String($Cert.GetRawCertData())

### 通过 PowerShell 登录 Azure 经典模式和 Resource Manager 模式

使用以下 PowerShell 命令登录 Azure，从 `Get-AzureRmSubscription` 命令的输出中确认需要配置的订阅号，并使用订阅号替换标黄的部分。关于经典模式和 Resource Manager 模式的介绍，请参考[了解部署模型和资源状态](/documentation/articles/resource-manager-deployment-model/)。

    #输入管理用户名和密码
    $cred = Get-Credential
    #登录 Azure 经典模式
    Add-AzureAccount -Environment AzureChinaCloud -Credential $cred
    #登录 Azure Resource Manager 模式
    Add-AzureRmAccount -EnvironmentName AzureChinaCloud -Credential $cred
    #获取订阅信息
    Get-AzureRmSubscription | select SubscriptionName, SubscriptionId
    #请输入选定的订阅号
    $SubscriptionId = "xxxxxxxxx"
    #在 Azure 经典模式中选定订阅
    Select-AzureSubscription -SubscriptionId $SubscriptionId
    #在 Azure Resource Manager 模式中选定订阅
    Select-AzureRmSubscription -SubscriptionId $SubscriptionId

配置登录所使用的 Azure AD 应用，和应用所对应的服务主体。<br>
您可以将以下代码中引号部分修改为您想要设置的值，这些值不会影响我们正在进行的配置。<br>
如需限制应用的服务主体对订阅的管理权限，请参考[使用角色分配来管理对 Azure 订阅资源的访问权限](/documentation/articles/role-based-access-control-configure/)。

    #从证书新建 Azure AD 应用
    $Application = New-AzureRmADApplication -DisplayName "AzureRmAutologin" -HomePage "http://www.contoso.com" -IdentifierUris "http://www.contoso.com/example" -CertValue $KeyValue -EndDate $Cert.NotAfter -StartDate $Cert.NotBefore
    #从 Azure AD 应用生成相应的服务主体
    New-AzureRmADServicePrincipal -ApplicationId $Application.ApplicationId
    #等待服务主体生成
    Start-Sleep 15
    #为应用的服务主体配置订阅的管理员权限
    New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId

### 配置 Azure 自动化所使用的连接资产和证书资产：

    #请输入自动化账号名
    $AutomationAccountName = "Automation Account Name"
    #请输入连接资产名
    $ConnectionAssetName = "AutoConnection"
    #配置连接资产信息
    $ConnectionFieldValues = @{"ApplicationId" = $Application.ApplicationId; "TenantId" = $Subscription.TenantId; "CertificateThumbprint" = $Cert.Thumbprint; "SubscriptionId" = $SubscriptionId}
    #在自动化账号中生成连接资产
    $ServicePrincipalConnection = New-AzureAutomationConnection -Name $ConnectionAssetName -AutomationAccountName $AutomationAccountName -ConnectionTypeName AzureServicePrincipal -ConnectionFieldValues $ConnectionFieldValues
    #请输入证书资产名
    $CertificateAssetName = "ARMAutoCert"
    #将证书资产导入自动化账号
    $CertificateAsset = New-AzureAutomationCertificate -Name $CertificateAssetName -AutomationAccountName $AutomationAccountName -Password $CertPassword -Path $CertPath -Exportable:$true
    #尝试在本地 PowerShell 中使用证书和服务主体进行登录
    Login-AzureRmAccount -ServicePrincipal -EnvironmentName azurechinacloud -ApplicationId $Application.ApplicationId -CertificateThumbprint $cert.Thumbprint -TenantId $Subscription.TenantId

[AZURE.NOTE] 由于 `Login-AzureRmAccount -ServicePrincipal` 时，PowerShell 会通过命令中提供的证书指纹，在本地电脑的证书库中搜索对应的证书，如果无法找到则会提示从证书指纹中找不到证书。所以在配置自动化账号时，要将证书导入自动化账号，自动化作业运行时才会从自动化账号的资产库中查到此证书。

在自动化 Runbook 中使用连接资产登录 Azure Resource Manager 模式。<br>
将上文生成的连接资产名填在 `$connectionName` 变量中，将需要选定的订阅号填入下方 `xxxxxxxxx` 处，保存 Runbook，即可进行登录测试。

    workflow cheshARMautologin
    {
    $connectionName = "AutoConnection"
    #从自动化账号中获取连接资产
        $ServicePrincipalConnection = Get-AutomationConnection -Name $connectionName
        "Signing in to Azure..."
        Login-AzureRmAccount -ServicePrincipal `
            -EnvironmentName AzureChinaCloud `
            -TenantId $ServicePrincipalConnection.TenantId `
            -ApplicationId $ServicePrincipalConnection.ApplicationId `
            -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

        #设定命令集使用的订阅号，也可使用 select-azurermsubscription 命令
        "Setting context to a specific subscription"
        Set-AzureRmContext -SubscriptionId xxxxxxxxx
        "Get Azure Resource Groups for test"
        Get-AzureRmResourceGroup
    }