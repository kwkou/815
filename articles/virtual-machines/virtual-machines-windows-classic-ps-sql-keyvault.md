<properties
    pageTitle="在 Azure VM 上配置 SQL Serve 的 Azure 密钥保管库集成（经典）"
    description="了解如何自动配置用于 Azure 密钥保管库的 SQL Server 加密。本主题说明了如何将 Azure 密钥保管库集成和经典部署模型中创建的 SQL Server 虚拟机结合使用。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="rothja"
    manager="jhubbard"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="ab8d41a7-1971-4032-ab71-eb435c455dc1"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="09/26/2016"
    wacn.date="02/24/2017"
    ms.author="jroth" />  


# 在 Azure VM 上配置 SQL Serve 的 Azure 密钥保管库集成（经典）
> [AZURE.SELECTOR]
- [资源管理器](/documentation/articles/virtual-machines-windows-ps-sql-keyvault/)
- [经典](/documentation/articles/virtual-machines-windows-classic-ps-sql-keyvault/)

## 概述
SQL Server 加密功能多种多样，包括[透明数据加密 \(TDE\)](https://msdn.microsoft.com/zh-cn/library/bb934049.aspx)、[列级加密 \(CLE\)](https://msdn.microsoft.com/zh-cn/library/ms173744.aspx) 和[备份加密](https://msdn.microsoft.com/zh-cn/library/dn449489.aspx)。这些加密形式要求你管理和存储用于加密的加密密钥。Azure 密钥保管库 \(AKV\) 服务专用于在一个高度可用的安全位置改进这些密钥的安全性和管理。[SQL Server 连接器](http://www.microsoft.com/download/details.aspx?id=45344)使 SQL Server 能够使用 Azure 密钥保管库中的这些密钥。

> [AZURE.IMPORTANT] 
Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。

如果在本地计算机上运行 SQL Server，请[按照此处步骤通过本地 SQL Server 计算机访问 Azure 密钥保管库](https://msdn.microsoft.com/zh-cn/library/dn198405.aspx)。但对于 Azure VM 中的 SQL Server，你可以通过使用 *Azure 密钥保管库集成*功能节省时间。通过使用几个 Azure PowerShell cmdlet 来启用此功能，可以自动为 SQL VM 进行必要的配置以便访问密钥保管库。

启用此功能后，它会自动安装 SQL Server 连接器、配置 EKM 提供程序以访问 Azure 密钥保管库，并创建凭据以使你能够访问保管库。在前面提到的本地文档列出的步骤中，你可以看到此功能自动完成步骤 2 和步骤 3。你仍需手动执行的唯一操作是创建密钥保管库和密钥。之后，将自动进行 SQL VM 的整个设置。在此功能完成设置后，你可以执行 T-SQL 语句，以按照通常的方式加密你的数据库或备份。

[AZURE.INCLUDE [AKV 集成准备](../../includes/virtual-machines-sql-server-akv-prepare.md)]

## 配置 AKV 集成
使用 PowerShell 来配置 Azure 密钥保管库集成。以下各节概述了所需的参数，然后提供了一个示例 PowerShell 脚本。

### 安装 SQL Server IaaS 扩展
首先，请[安装 SQL Server IaaS 扩展](/documentation/articles/virtual-machines-windows-classic-sql-server-agent-extension/)。

### 了解输入参数
下表列出在下一节中运行 PowerShell 脚本所需的参数。

| 参数 | 说明 | 示例 |
| --- | --- | --- |
| **$akvURL** |**密钥保管库 URL** |“https://contosokeyvault.vault.chinacloudapi.cn/” |
| **$spName** |**服务主体名称** |“fde2b411-33d5-4e11-af04eb07b669ccf2” |
| **$spSecret** |**服务主体密码** |“9VTJSQwzlFepD8XODnzy8n2V01Jd8dAjwm/azF1XDKM =” |
| **$credName** |**凭据名称**：AKV 集成在 SQL Server 内创建一个凭据，使 VM 具有对密钥保管库的访问权限。为此凭据选择一个名称。 |“mycred1” |
| **$vmName** |**虚拟机名称**：以前创建的 SQL VM 的名称。 |“myvmname” |
| **$serviceName** |**服务名称**：与 SQL VM 关联的云服务名称。 |“mycloudservicename” |

### 使用 PowerShell 启用 AKV 集成
**New-AzureVMSqlServerKeyVaultCredentialConfig** cmdlet 为 Azure 密钥保管库集成功能创建配置对象。**Set-AzureVMSqlServerExtension** 通过 **KeyVaultCredentialSettings** 参数配置此集成。以下步骤显示如何使用这些命令。

1. 在 Azure PowerShell 中，首先使用特定的值配置输入参数，如本主题前面各节中所述。下面的脚本就是一个示例。
   
        $akvURL = "https://contosokeyvault.vault.chinacloudapi.cn/"
        $spName = "fde2b411-33d5-4e11-af04eb07b669ccf2"
        $spSecret = "9VTJSQwzlFepD8XODnzy8n2V01Jd8dAjwm/azF1XDKM="
        $credName = "mycred1"
        $vmName = "myvmname"
        $serviceName = "mycloudservicename"
2. 然后使用以下脚本来配置和启用 AKV 集成。
   
       $secureakv =  $spSecret | ConvertTo-SecureString -AsPlainText -Force
       $akvs = New-AzureVMSqlServerKeyVaultCredentialConfig -Enable -CredentialName $credname -AzureKeyVaultUrl $akvURL -ServicePrincipalName $spName -ServicePrincipalSecret $secureakv
       Get-AzureVM -ServiceName $serviceName -Name $vmName | Set-AzureVMSqlServerExtension -KeyVaultCredentialSettings $akvs | Update-AzureVM

SQL IaaS 代理扩展将使用此新配置来更新 SQL VM。

[AZURE.INCLUDE [AKV 集成后续步骤](../../includes/virtual-machines-sql-server-akv-next-steps.md)]

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->