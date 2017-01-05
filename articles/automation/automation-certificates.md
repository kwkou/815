<properties
    pageTitle="Azure 自动化中的证书资产 | Azure"
    description="可以安全地将证书存储在 Azure 自动化中，以便可以通过 Runbook 访问这些证书，对 Azure 和第三方资源进行身份验证。本文介绍了有关证书的详细信息，以及如何在文本和图形创作中使用证书。"
    services="automation"
    documentationcenter=""
    author="mgoedtel"
    manager="stevenka"
    editor="tysonn" />  

<tags
    ms.assetid="ac9c22ae-501f-42b9-9543-ac841cf2cc36"
    ms.service="automation"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="12/19/2016"
    wacn.date="01/05/2017"
    ms.author="magoedte;bwren" />  


# Azure 自动化中的证书资产

可以安全地将证书存储在 Azure 自动化中，以便使用 **Get-AzureAutomationCertificate** 活动通过 Runbook 访问这些证书。这样，你便可以创建使用证书进行身份验证的 Runbook，或者将证书添加到 Azure 或第三方资源。

> [AZURE.NOTE] 
Azure 自动化中的安全资产包括凭据、证书、连接和加密的变量。这些资产已使用针对每个自动化帐户生成的唯一密钥加密并存储在 Azure 自动化中。此密钥由主证书加密，并存储在 Azure 自动化中。在存储安全资产之前，会先使用主证书来解密自动化帐户的密钥，然后使用该密钥来加密资产。
> 

## Windows PowerShell Cmdlet

下表中的 cmdlet 用于通过 Windows PowerShell 创建和管理自动化证书资产。可在自动化 Runbook 中使用的 [Azure PowerShell 模块](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)已随附了这些 cmdlet。

|Cmdlet|说明|
|:---|:---|
|[Get-AzureAutomationCertificate](https://msdn.microsoft.com/zh-cn/library/dn913765.aspx)|检索有关证书的信息。只能从 Get-AzureAutomationCertificate 活动中检索证书本身。|
|[New-AzureAutomationCertificate](https://msdn.microsoft.com/zh-cn/library/dn913764.aspx)|将新证书创建到 Azure 自动化中。|
|[Remove-AzureAutomationCertificate](https://msdn.microsoft.com/zh-cn/library/dn913773.aspx)|从 Azure自动化中删除证书。|
|[Set-AzureAutomationCertificate](https://msdn.microsoft.com/zh-cn/library/dn913763.aspx)|设置现有证书的属性，包括上载证书文件和设置 .pfx 的密码。|
|[Add-AzureCertificate](https://msdn.microsoft.com/zh-cn/library/azure/dn495214.aspx)|为指定的云服务上载服务证书。|


## 创建新证书

创建新证书时，需要将 .cer 或 .pfx 文件上载到 Azure 自动化。将证书标记为可导出后，你可以将其转出 Azure 自动化证书存储区。如果证书不可导出，则它只可用于在 Runbook 中签名。

### 使用 Azure 经典管理门户创建新证书
1. 在你的自动化帐户中，单击窗口顶部的“资产”。
2. 在窗口底部，单击“添加设置”。
3. 单击“添加凭据”。
4. 在“凭据类型”下拉列表中，选择“证书”。
5. 在“名称”框中键入证书的名称，然后单击右箭头。
6. 浏览到 .cer 或 .pfx 文件。如果选择了 .pfx 文件，请指定密码，以及它是否允许导出。
7. 单击复选标记上载该证书文件。

### 使用 Windows PowerShell 创建新证书

以下示例演示了如何创建新的自动化证书并将其标记为可导出。这将导入现有的 .pfx 文件。

    $certName = 'MyCertificate'
    $certPath = '.\MyCert.pfx'
    $certPwd = ConvertTo-SecureString -String 'P@$$w0rd' -AsPlainText -Force

    New-AzureAutomationCertificate -AutomationAccountName "MyAutomationAccount" -Name $certName -Path $certPath -Password $certPwd -Exportable

## 使用证书

必须通过 **Get-AutomationCertificate** 活动来使用证书。不能使用 [Get-AzureAutomationCertificate](https://msdn.microsoft.com/zh-cn/library/dn913765.aspx) cmdlet，因为它返回有关证书资产而不是证书本身的信息。

### 文本 Runbook 示例

以下示例代码演示了如何将证书添加到 Runbook 中的云服务。在此示例中，已从加密的自动化变量检索了密码。

    $serviceName = 'MyCloudService'
    $cert = Get-AutomationCertificate -Name 'MyCertificate'
    $certPwd = Get-AutomationVariable -Name 'MyCertPassword'
    Add-AzureCertificate -ServiceName $serviceName -CertToDeploy $cert

<!---HONumber=Mooncake_1226_2016-->