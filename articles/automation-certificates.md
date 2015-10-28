<properties 
   pageTitle="Azure 自动化中的证书资产"
   description="可以安全地将证书存储在 Azure 自动化中，以便可以通过 Runbook 访问这些证书，对 Azure 和第三方资源进行身份验证。本文介绍了有关证书的详细信息，以及如何在文本和图形创作中使用证书。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags 
   ms.service="automation"
   ms.date="08/18/2015"
   wacn.date="09/15/2015"/>

# Azure 自动化中的证书资产

可以安全地将证书存储在 Azure 自动化中，以便使用 **Get-AutomationCertificate** 活动通过 Runbook 访问这些证书。这样，你便可以创建使用证书进行身份验证的 Runbook，或者将证书添加到 Azure，或添加到 Runbook 可能会创建或配置的或第三方资源。

>[AZURE.NOTE]Azure 自动化中的安全资产包括凭据、证书、连接和加密的变量。这些资产已使用针对每个自动化帐户生成的唯一密钥加密并存储在 Azure 自动化中。此密钥由主证书加密，并存储在 Azure 自动化中。在存储安全资产之前，会先使用主证书来解密自动化帐户的密钥，然后使用该密钥来加密资产。

## Windows PowerShell Cmdlet

下表中的 cmdlet 用于通过 Windows PowerShell 创建和管理自动化证书资产。可在自动化 Runbook 中使用的 [Azure PowerShell 模块](/documentation/articles/powershell-install-configure)已随附了这些 cmdlet。

|Cmdlet|说明|
|:---|:---|
|[Get-AzureAutomationCertificate](http://msdn.microsoft.com/zh-cn/library/dn913765.aspx)|检索有关证书的信息。只能从 Get-AutomationCertificate 活动中检索证书本身。|
|[New- AzureAutomationCertificate](http://msdn.microsoft.com/zh-cn/library/dn913764.aspx)|将新证书导入 Azure 自动化。|
|[Remove- AzureAutomationCertificate](http://msdn.microsoft.com/zh-cn/library/dn913773.aspx)|从 Azure自动化中删除证书。|
|[Set- AzureAutomationCertificate](http://msdn.microsoft.com/zh-cn/library/dn913763.aspx)|设置现有证书的属性，包括上载证书文件和设置 .pfx 的密码。|

## Runbook 活动

下表中的活动用于在 Runbook 中访问证书。

|活动|说明|
|:---|:---|
|Get-AutomationCertificate|在 Runbook 中获取要使用的证书。|

>[AZURE.NOTE]应避免在 GetAutomationCertificate 的 –Name 参数中使用变量，因为这可能会使设计时发现 Runbook 与证书资产之间的依赖关系变得复杂化。

## 创建新证书

创建新证书时，需要将 cer 或 pfx 文件上载到 Azure 自动化。将证书标记为可导出后，你可以将其转出 Azure 自动化证书存储区。如果证书不可导出，则它只可用于在 Runbook 中签名。

### 使用 Azure 门户创建新证书

1. 在你的自动化帐户中，单击窗口顶部的“资产”。
1. 在窗口底部，单击“添加设置”。
1. 单击“添加凭据”。
2. 在“凭据类型”下拉列表中，选择“证书”。
3. 在“名称”框中键入证书的名称，然后单击右箭头。
4. 浏览到 .cer 或 .pfx 文件。如果选择了 .pfx 文件，请指定密码，以及它是否允许导出。
1. 单击复选标记上载该证书文件。


### 使用 Azure 预览门户创建新证书

1. 在你的自动化帐户中，单击“资产”部分打开“资产”边栏选项卡。
1. 单击“证书”部分打开“证书”边栏选项卡。
1. 单击边栏选项卡顶部的“添加证书”。
2. 在“名称”框中键入证书的名称。
2. 单击“上载证书文件”下面的“选择文件”，并浏览到 .cer 或.pfx 文件。如果选择了 .pfx 文件，请指定密码，以及它是否允许导出。
1. 单击“创建”以保存新的证书资产。


### 使用 Windows PowerShell 创建新证书

以下示例命令演示了如何创建新的自动化证书并将其标记为可导出。这将导入现有的 pfx 文件。

	$certName = 'MyCertificate'
	$certPath = '.\MyCert.pfx'
	$certPwd = ConvertTo-SecureString -String 'P@$$w0rd' -AsPlainText -Force
	
	New-AzureAutomationCertificate -AutomationAccountName "MyAutomationAccount" -Name $certName -Path $certPath –Password $certPwd -Exportable

## 使用 Runbook 中的证书

必须通过 **Get-AutomationCertificate** 活动来使用 Runbook 中的证书。不能使用 [Get-AzureAutomationCertificate](http://msdn.microsoft.com/zh-cn/library/dn913765.aspx) cmdlet，因为它返回有关证书资产而不是证书本身的信息。

### 文本 Runbook 示例

以下示例代码演示了如何将证书添加到 Runbook 中的云服务。在此示例中，已从加密的自动化变量检索了密码。

	$serviceName = 'MyCloudService'
	$cert = Get-AutomationCertificate -Name 'MyCertificate'
	$certPwd = Get-AutomationVariable –Name 'MyCertPassword'
	Add-AzureCertificate -ServiceName $serviceName -CertToDeploy $cert
<!--
### 图形 Runbook 示例

通过在图形编辑器的“库”窗格中右键单击证书并选择“添加到 Canvas”，将 **Get-AutomationCerticiate** 添加到图形 Runbook。

![](./media/automation-certificates/certificate-add-canvas.png)

下图显示了在图形 Runbook 中使用证书的示例。这与上面演示的从文本 Runbook 向云服务添加证书的示例相同。

此示例将对使用连接对象向服务进行身份验证的 **SendTwilioSMS 活动**使用 **UseConnectionObject** 参数集。必须在此处使用[管道链接](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow)，因为序列链接将返回一个集合，其中包含不应在 Connection 参数中使用的单个对象。

![](./media/automation-certificates/add-certificate.png)


## 另请参阅

- [图形创作中的链接](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow) -->

<!---HONumber=69-->