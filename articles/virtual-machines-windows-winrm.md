<properties
	pageTitle="为 Azure Resource Manager 中的虚拟机设置 WinRM 访问权限 | Azure"
	description="如何设置要用于 Azure Resource Manager 虚拟机的 WinRM 访问权限"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="singhkay"
	manager="drewm"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="06/16/2016"
	wacn.date="07/11/2016"/>

# 为 Azure Resource Manager 中的虚拟机设置 WinRM 访问权限

## Azure Service Management 中的 WinRM 与 Azure Resource Manager

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型

* 有关 Azure Resource Manager 的概述，请参阅此[文章](/documentation/articles/resource-group-overview/)
* 有关 Azure Service Management 和 Azure Resource Manager 之间的差异，请参阅此[文章](/documentation/articles/resource-manager-deployment-model/)

在两个堆栈之间设置 WinRM 配置的主要差异是将证书安装到 VM 的方式。在 Azure Resource Manager 堆栈中，证书被建模为由密钥保管库资源提供程序管理的资源。因此，在 VM 中使用自己的证书之前，用户需要提供这些证书并将其上传到密钥保管库。

为 VM 设置 WinRM 连接需执行以下步骤

1. 创建密钥保管库
2. 创建自签名证书
3. 获将自签名证书上传到密钥保管库
4. 获取密钥保管库中自签名证书的 URL
5. 创建 VM 时引用你的自签名的证书 URL

## 步骤 1：创建密钥保管库

可使用以下命令来创建密钥保管库

	New-AzureRmKeyVault -VaultName "<vault-name>" -ResourceGroupName "<rg-name>" -Location "<vault-location>" -EnabledForDeployment -EnabledForTemplateDeployment

## 步骤 2：创建自签名证书
可使用此 PowerShell 脚本创建自签名证书

	$certificateName = "somename"
	
	$thumbprint = (New-SelfSignedCertificate -DnsName "$certificateName" -CertStoreLocation Cert:\CurrentUser\My -KeySpec KeyExchange).Thumbprint
	
	$cert = (Get-ChildItem -Path cert:\CurrentUser\My\$thumbprint)
	
	$password = Read-Host -Prompt "Please enter the certificate password." -AsSecureString
	
	Export-PfxCertificate -Cert $cert -FilePath ".\$certificateName.pfx" -Password $password

## 步骤 3：获将自签名证书上传到密钥保管库

将证书上传到在步骤 1 中创建的密钥保管库之前，需将其转换为 Microsoft.Compute 资源提供程序可识别的格式。以下 PowerShell 脚本将允许你执行该操作

	$fileName = "<Path to the .pfx file>"
	$fileContentBytes = get-content $fileName -Encoding Byte
	$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
	
	$jsonObject = @"
	{
	  "data": "$filecontentencoded",
	  "dataType" :"pfx",
	  "password": "<password>"
	}
	"@
	
	$jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
	$jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)
	
	$secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText -Force
	Set-AzureRmKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>" -SecretValue $secret

## 步骤 4：获取密钥保管库中自签名证书的 URL

预配 VM 时，Microsoft.Compute 资源提供程序需要指向密钥保管库中密钥的 URL。这将使 Microsoft.Compute 资源提供程序能够下载密钥，并在 VM 上创建等效证书。

注意：密钥 URL 需要同时包含版本。示例 URL 类似如下
https://contosovault.vault.chinacloudapi.cn:443/secrets/contososecret/01h9db0df2cd4300a20ence585a6s7ve


#### 模板

可使用以下代码获取模板中 URL 的链接

    "certificateUrl": "[reference(resourceId(resourceGroup().name, 'Microsoft.KeyVault/vaults/secrets', '<vault-name>', '<secret-name>'), '2015-06-01').secretUriWithVersion]"

#### PowerShell

可使用以下 PowerShell 命令获取此 URL

	$secretURL = (Get-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id

## 步骤 5：创建 VM 时引用你的自签名的证书 URL

#### Azure Resource Manager 模板

通过模板创建 VM 时，将在密钥部分和 winRM 部分中引用该证书，如下所示

	"osProfile": {
          ...
          "secrets": [
            {
              "sourceVault": {
                "id": "<resource id of the Key Vault containing the secret>"
              },
              "vaultCertificates": [
                {
                  "certificateUrl": "<URL for the certificate you got in Step 4>",
                  "certificateStore": "<Name of the certificate store on the VM>"
                }
              ]
            }
          ],
          "windowsConfiguration": {
            ...
            "winRM": {
              "listeners": [
                {
                  "protocol": "http"
                },
                {
                  "protocol": "https",
                  "certificateUrl": "<URL for the certificate you got in Step 4>"
                }
              ]
            },
            ...
          }
        },

可在此处找到以上内容的示例模板 - https://azure.microsoft.com/documentation/templates/201-vm-winrm-keyvault-windows
可在 Github 中找到此模板的源代码 - https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

#### PowerShell

	$vm = New-AzureRmVMConfig -VMName "<VM name>" -VMSize "<VM Size>"
	$credential = Get-Credential
	$secretURL = (Get-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id
	$vm = Set-AzureRmVMOperatingSystem -VM $vm  -Windows -ComputerName "<Computer Name>" -Credential $credential -WinRMHttp -WinRMHttps -WinRMCertificateUrl $secretURL
	$sourceVaultId = (Get-AzureRmKeyVault -ResourceGroupName "<Resource Group name>" -VaultName "<Vault Name>").ResourceId
	$CertificateStore = "My"
	$vm = Add-AzureRmVMSecret -VM $vm -SourceVaultId $sourceVaultId -CertificateStore $CertificateStore -CertificateUrl $secretURL

## 步骤 6：连接到 VM
需要先确保你的计算机针对 WinRM 远程管理进行了配置，然后才能连接到 VM。以管理员身份启动 PowerShell 并执行以下命令以确保已设置完毕。

    Enable-PSRemoting -Force

>[AZURE.NOTE] 如果以上命令无效，可能需要确保 WinRM 服务正在运行。可使用 `Get-Service WinRM` 完成此操作

安装完成后，即可使用以下命令连接到 VM

    Enter-PSSession -ConnectionUri https://<public-ip-dns-of-the-vm>:5986 -Credential $cred -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck) -Authentication Negotiate

<!---HONumber=Mooncake_0704_2016-->