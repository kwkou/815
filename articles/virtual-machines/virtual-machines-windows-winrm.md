<properties
    pageTitle="为 Azure VM 设置 WinRM 访问权限 | Azure"
    description="设置 WinRM 访问，与 Resource Manager 部署模型中创建的 Azure 虚拟机一起使用。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="singhkays"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="9718e85b-d360-4621-90b8-0b0b84a21208"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="06/16/2016"
    wacn.date="05/15/2017"
    ms.author="kasing"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="2587d2d258f58453cda2565e1a450faf4820f718"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="setting-up-winrm-access-for-virtual-machines-in-azure-resource-manager"></a>为 Azure Resource Manager 中的虚拟机设置 WinRM 访问权限
## <a name="winrm-in-azure-service-management-vs-azure-resource-manager"></a>Azure 服务管理中的 WinRM 与 Azure Resource Manager

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

* 有关 Azure Resource Manager 的概述，请参阅此[文章](/documentation/articles/resource-group-overview/)
* 有关 Azure 服务管理和 Azure Resource Manager 之间的差异，请参阅此[文章](/documentation/articles/resource-manager-deployment-model/)

在两个堆栈之间设置 WinRM 配置的主要差异是将证书安装到 VM 的方式。 在 Azure Resource Manager 堆栈中，证书被建模为由密钥保管库资源提供程序管理的资源。 因此，在 VM 中使用自己的证书之前，用户需要提供这些证书并将其上载到密钥保管库。

为 VM 设置 WinRM 连接需执行以下步骤

1. 创建密钥保管库
2. 创建自签名证书
3. 将自签名证书上载到密钥保管库
4. 获取密钥保管库中自签名证书的 URL
5. 创建 VM 时引用你的自签名证书 URL

## <a name="step-1-create-a-key-vault"></a>步骤 1：创建密钥保管库
可使用以下命令来创建密钥保管库

    New-AzureRmKeyVault -VaultName "<vault-name>" -ResourceGroupName "<rg-name>" -Location "<vault-location>" -EnabledForDeployment -EnabledForTemplateDeployment

## <a name="step-2-create-a-self-signed-certificate"></a>步骤 2：创建自签名证书
可使用此 PowerShell 脚本创建自签名证书

    $certificateName = "somename"

    $thumbprint = (New-SelfSignedCertificate -DnsName $certificateName -CertStoreLocation Cert:\CurrentUser\My -KeySpec KeyExchange).Thumbprint

    $cert = (Get-ChildItem -Path cert:\CurrentUser\My\$thumbprint)

    $password = Read-Host -Prompt "Please enter the certificate password." -AsSecureString

    Export-PfxCertificate -Cert $cert -FilePath ".\$certificateName.pfx" -Password $password

## <a name="step-3-upload-your-self-signed-certificate-to-the-key-vault"></a>步骤 3：将自签名证书上载到密钥保管库
将证书上载到在步骤 1 中创建的密钥保管库之前，需将其转换为 Microsoft.Compute 资源提供程序可识别的格式。 以下 PowerShell 脚本将允许你执行该操作

    $fileName = "<Path to the .pfx file>"
    $fileContentBytes = Get-Content $fileName -Encoding Byte
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
    Set-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>" -SecretValue $secret

## <a name="step-4-get-the-url-for-your-self-signed-certificate-in-the-key-vault"></a>步骤 4：获取密钥保管库中自签名证书的 URL
预配 VM 时，Microsoft.Compute 资源提供程序需要指向密钥保管库中密钥的 URL。 这将使 Microsoft.Compute 资源提供程序能够下载密钥，并在 VM 上创建等效证书。

> [AZURE.NOTE]
> 密钥 URL 还需要包含版本。 示例 URL 类似于以下链接：https://contosovault.vault.azure.cn:443/secrets/contososecret/01h9db0df2cd4300a20ence585a6s7ve
> 
> 

#### <a name="templates"></a>模板
可使用以下代码获取模板中 URL 的链接

    "certificateUrl": "[reference(resourceId(resourceGroup().name, 'Microsoft.KeyVault/vaults/secrets', '<vault-name>', '<secret-name>'), '2015-06-01').secretUriWithVersion]"

#### <a name="powershell"></a>PowerShell
可使用以下 PowerShell 命令获取此 URL

    $secretURL = (Get-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id

## <a name="step-5-reference-your-self-signed-certificates-url-while-creating-a-vm"></a>步骤 5：创建 VM 时引用你的自签名证书 URL
#### <a name="azure-resource-manager-templates"></a>Azure 资源管理器模板
通过模板创建 VM 时，在密钥部分和 winRM 部分中引用该证书，如下所示：

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

针对上述内容的示例模板可在此处 [201-vm-winrm-keyvault-windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows) 找到

此模板的源代码可在 [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows) 上找到

>[AZURE.NOTE]
> 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。 例如，替换某些终结点（将“blob.core.chinacloudapi.cn”替换为“blob.core.chinacloudapi.cn”，将“chinacloudapp.cn”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

#### <a name="powershell"></a>PowerShell
    $vm = New-AzureRmVMConfig -VMName "<VM name>" -VMSize "<VM Size>"
    $credential = Get-Credential
    $secretURL = (Get-AzureKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id
    $vm = Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName "<Computer Name>" -Credential $credential -WinRMHttp -WinRMHttps -WinRMCertificateUrl $secretURL
    $sourceVaultId = (Get-AzureRmKeyVault -ResourceGroupName "<Resource Group name>" -VaultName "<Vault Name>").ResourceId
    $CertificateStore = "My"
    $vm = Add-AzureRmVMSecret -VM $vm -SourceVaultId $sourceVaultId -CertificateStore $CertificateStore -CertificateUrl $secretURL

## <a name="step-6-connecting-to-the-vm"></a>步骤 6：连接到 VM
需要先确保你的计算机针对 WinRM 远程管理进行了配置，然后才能连接到 VM。 以管理员身份启动 PowerShell 并执行以下命令以确保已完成设置。

    Enable-PSRemoting -Force

> [AZURE.NOTE]
> 如果以上命令无效，可能需要确保 WinRM 服务正在运行。 可使用 `Get-Service WinRM`
> 
> 

设置完成后，即可使用以下命令连接到 VM

    Enter-PSSession -ConnectionUri https://<public-ip-dns-of-the-vm>:5986 -Credential $cred -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck) -Authentication Negotiate