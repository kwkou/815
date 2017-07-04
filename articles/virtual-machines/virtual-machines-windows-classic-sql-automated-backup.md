<properties
    pageTitle="SQL Server 虚拟机的自动备份（经典） | Azure"
    description="介绍在使用 Resource Manager 的 Azure 虚拟机中运行的 SQL Server 的自动备份功能。"
    services="virtual-machines-windows"
    documentationcenter="na"
    author="rothja"
    manager="jhubbard"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="3333e830-8a60-42f5-9f44-8e02e9868d7b"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="iaas-sql-server"
    ms.date="01/18/2017"
    wacn.date="03/20/2017"
    ms.author="jroth" />

# Azure 虚拟机中 SQL Server 的自动备份（经典）
> [AZURE.SELECTOR]
- [Resource Manager](/documentation/articles/virtual-machines-windows-sql-automated-backup/)
- [经典](/documentation/articles/virtual-machines-windows-classic-sql-automated-backup/)

自动备份将在运行 SQL Server 2014 Standard 或 Enterprise 的 Azure VM 上自动为所有现有数据库和新数据库配置[托管备份到 Azure](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)。这样，你便可以配置使用持久 Azure Blob 存储的定期数据库备份。自动备份依赖于 [SQL Server IaaS 代理扩展](/documentation/articles/virtual-machines-windows-classic-sql-server-agent-extension/)。

> [AZURE.IMPORTANT] 
Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。若要查看本文的 Resource Manager 版本，请参阅 [Azure 虚拟机中 SQL Server 的自动备份 (Resource Manager)](/documentation/articles/virtual-machines-windows-sql-automated-backup/)。

## 先决条件
若要使用自动备份，请考虑以下先决条件：

**操作系统**：

* Windows Server 2012
* Windows Server 2012 R2

**SQL Server 版本**：

* SQL Server 2014 Standard
* SQL Server 2014 Enterprise

> [AZURE.NOTE]
SQL Server 2016 尚不支持自动备份。
> 
> 

**数据库配置**：

* 目标数据库必须使用完整恢复模式。

**Azure PowerShell**：

* [安装最新的 Azure PowerShell 命令](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

**SQL Server IaaS 扩展**：

* [安装 SQL Server IaaS 扩展](/documentation/articles/virtual-machines-windows-classic-sql-server-agent-extension/)。

## 设置
下表描述了可为自动备份配置的选项。对于经典 VM，必须使用 PowerShell 配置以下设置。

| 设置 | 范围（默认值） | 说明 |
| --- | --- | --- |
| **自动备份** |启用/禁用（已禁用） |为运行 SQL Server 2014 Standard 或 Enterprise 的 Azure VM 启用或禁用自动备份。 |
| **保留期** |1-30 天（30 天） |保留备份的天数。 |
| **存储帐户** |Azure 存储帐户（为指定的 VM 创建的存储帐户） |用于在 Blob 存储中存储自动备份文件的 Azure 存储帐户。将在此位置创建容器，用于存储所有备份文件。备份文件命名约定包括日期、时间和计算机名称。 |
| **加密** |启用/禁用（已禁用） |启用或禁用加密。启用加密时，用于还原备份的证书将使用相同的命名约定存放在同一 automaticbackup 容器中的指定存储帐户内。如果密码发生更改，将使用该密码生成新证书，但旧证书在备份之前仍会还原。 |
| **密码** |密码文本（无） |加密密钥的密码。仅当启用了加密时才需要此设置。若要还原加密的备份，必须具有创建该备份时使用的正确密码和相关证书。 |
| **备份系统数据库** | 启用/禁用（已禁用） | 对 Master、Model 和 MSDB 进行完整备份 |
| **配置备份计划** | 手动/自动（自动） | 选择“自动”，根据日志增长情况自动进行完整备份和日志备份。选择“手动”，指定进行完整备份和日志备份的计划。 |

## 使用 PowerShell 进行配置
下面的 PowerShell 示例将为现有 SQL Server 2014 VM 配置自动备份。**New-AzureVMSqlServerAutoBackupConfig** 命令可将自动备份设置配置为在 $storageaccount 变量指定的 Azure 存储帐户中存储备份。这些备份将保留 10 天。**Set-AzureVMSqlServerExtension** 命令可使用这些设置更新指定的 Azure VM。

    $storageaccount = "<storageaccountname>"
    $storageaccountkey = (Get-AzureStorageKey -StorageAccountName $storageaccount).Primary
    $storagecontext = New-AzureStorageContext -StorageAccountName $storageaccount -StorageAccountKey $storageaccountkey
    $autobackupconfig = New-AzureVMSqlServerAutoBackupConfig -StorageContext $storagecontext -Enable -RetentionPeriod 10

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoBackupSettings $autobackupconfig | Update-AzureVM

可能需要花费几分钟来安装和配置 SQL Server IaaS 代理。

若要启用加密，请修改上述脚本，使其将 EnableEncryption 参数连同 CertificatePassword 参数的密码（安全字符串）一起传递。以下脚本将启用上一示例中的自动备份设置，并添加加密。

    $storageaccount = "<storageaccountname>"
    $storageaccountkey = (Get-AzureStorageKey -StorageAccountName $storageaccount).Primary
    $storagecontext = New-AzureStorageContext -StorageAccountName $storageaccount -StorageAccountKey $storageaccountkey
    $password = "P@ssw0rd"
    $encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force  
    $autobackupconfig = New-AzureVMSqlServerAutoBackupConfig -StorageContext $storagecontext -Enable -RetentionPeriod 10 -EnableEncryption -CertificatePassword $encryptionpassword

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoBackupSettings $autobackupconfig | Update-AzureVM

若要禁用自动备份，请对 **New-AzureVMSqlServerAutoBackupConfig** 运行不带 **-Enable** 参数的同一个脚本。与安装一样，可能需要花费几分钟时间来禁用自动备份。

> [AZURE.NOTE]
禁用和卸载 SQL Server IaaS 代理不会删除以前配置的托管备份设置。应该在禁用或卸载 SQL Server IaaS 代理之前禁用自动备份。
> 
> 

## 后续步骤
自动备份将在 Azure VM 上配置托管备份。因此，请务必[查看有关托管备份的文档](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)，以了解其行为和影响。

有关 Azure VM 上的其他 SQL Server 备份和还原指南，可查看以下主题：[备份和还原 Azure 虚拟机中的 SQL Server](/documentation/articles/virtual-machines-windows-sql-backup-recovery/)。

若要了解其他可用的自动化任务，请参阅 [SQL Server IaaS 代理扩展](/documentation/articles/virtual-machines-windows-classic-sql-server-agent-extension/)。

若要深入了解如何在 Azure VM 上运行 SQL Server，请参阅 [Azure 虚拟机上的 SQL Server 概述](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: wording update-->