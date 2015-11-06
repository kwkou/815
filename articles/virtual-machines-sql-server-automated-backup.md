<properties 
   pageTitle="Azure 虚拟机中 SQL Server 的自动备份"
   description="介绍 Azure 虚拟机中运行的 SQL Server 的自动备份功能。"
   services="virtual-machines"
   documentationCenter="na"
   authors="rothja"
   manager="jeffreyg"
   editor="monicar" />
<tags 
   ms.service="virtual-machines"
   ms.date="08/05/2015"
   wacn.date="09/18/2015" />

# Azure 虚拟机中 SQL Server 的自动备份

自动备份将在运行 SQL Server 2014 Standard 或 Enterprise 的 Azure VM 上，自动为所有现有数据库和新数据库配置[向 Windows Azure 的托管备份](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)。这样，你便可以配置使用持久 Azure Blob 存储的定期数据库备份。

>[AZURE.NOTE]自动备份依赖 SQL Server IaaS 代理。要安装和配置该代理，必须在目标虚拟机上运行 Azure VM 代理。较新的虚拟机库映像已默认启用此选项，但现有 VM 可能缺少 Azure VM 代理。如果使用你自己的 VM 映像，也需要安装 SQL Server IaaS 代理。有关详细信息，请参阅 [VM 代理和扩展](http://azure.microsoft.com/blog/2014/04/15/vm-agent-and-extensions-part-2/)。

## 自动备份设置

下表描述了可为自动备份配置的选项。实际配置步骤根据你使用的是 Azure 门户还是 Azure Windows PowerShell 命令而有所不同。

|设置|范围（默认值）|说明|
|---|---|---|
|**自动备份**|启用/禁用（已禁用）|为运行 SQL Server 2014 Standard 或 Enterprise 的 Azure VM 启用或禁用自动备份。|
|**保留期**|1-30 天（30 天）|保留备份的天数。|
|**存储帐户**|Azure 存储帐户（为指定的 VM 创建的存储帐户）|用于在 Blob 存储中存储自动备份文件的 Azure 存储帐户。在此位置创建容器，用于存储所有备份文件。备份文件命名约定包括日期、时间和计算机名称。|
|**加密**|启用/禁用（已禁用）|启用或禁用加密。启用加密时，用于还原备份的证书使用相同的命名约定存放在同一 automaticbackup 容器中的指定存储帐户内。如果密码发生更改，将使用该密码生成新证书，但旧证书在备份之前仍会还原。|
|**密码**|密码文本（无）|加密密钥的密码。仅当启用了加密时才需要此设置。若要还原加密的备份，必须具有创建该备份时使用的正确密码和相关证书。|

## 在门户中配置自动备份

在创建新的 SQL Server 2014 虚拟机时，可以使用 [Azure 预览门户](http://go.microsoft.com/fwlink/?LinkID=525040&clcid=0x409)配置自动备份。以下屏幕截图显示了“可选配置”|“SQL 自动备份”下的这些选项。

![Azure 门户中的 SQL 自动备份配置](./media/virtual-machines-sql-server-automated-backup/IC778483.jpg)

对于现有的 SQL Server 2014 虚拟机，在虚拟机属性的“配置”部分选择“自动备份”设置。在“自动备份”窗口中，可以启用功能、设置保持期、选择存储帐户，以及设置加密，如以下屏幕快照中所示。

![Azure 门户中的自动备份配置](./media/virtual-machines-sql-server-automated-backup/IC792133.jpg)

>[AZURE.NOTE]当你首次启用自动备份时，Azure 将在后台配置 SQL Server IaaS 代理。在此期间，门户将不会显示自动备份已配置。请等待几分钟，以便安装和配置代理。之后，该门户将反映新的设置。

## 使用 PowerShell 配置自动备份

在下面的 PowerShell 示例中，为现有 SQL Server 2014 VM 配置了自动备份。**New-AzureVMSqlServerAutoBackupConfig** 命令将自动备份设置配置为在 $storageaccount 变量指定的 Azure 存储帐户中存储备份。这些备份将保留 10 天。**Set-AzureVMSqlServerExtension** 命令使用这些设置更新指定的 Azure VM。

    $storageaccount = "<storageaccountname>"
    $storageaccountkey = (Get-AzureStorageKey -StorageAccountName $storageaccount).Primary
    $storagecontext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageaccount -StorageAccountKey $storageaccountkey
    $autobackupconfig = New-AzureVMSqlServerAutoBackupConfig -StorageContext $storagecontext -Enable -RetentionPeriod 10
    
    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoBackupSettings $autobackupconfig | Update-AzureVM

可能需要花费几分钟来安装和配置 SQL Server IaaS 代理。

若要启用加密，请修改上述脚本，使其将 EnableEncryption 参数连同 CertificatePassword 参数的密码（安全字符串）一起传递。以下脚本启用上一示例中的自动备份设置，并添加加密。

    $storageaccount = "<storageaccountname>"
    $storageaccountkey = (Get-AzureStorageKey -StorageAccountName $storageaccount).Primary
    $storagecontext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageaccount -StorageAccountKey $storageaccountkey
    $password = "P@ssw0rd"
    $encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force  
    $autobackupconfig = New-AzureVMSqlServerAutoBackupConfig -StorageContext $storagecontext -Enable -RetentionPeriod 10 -EnableEncryption -CertificatePassword $encryptionpassword
    
    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoBackupSettings $autobackupconfig | Update-AzureVM 

若要禁用自动备份，请对 **New-AzureVMSqlServerAutoBackupConfig** 运行不带 **-Enable** 参数的同一个脚本。与安装一样，可能需要花费几分钟时间来禁用自动备份。

## 禁用和卸载 SQL Server IaaS 代理

如果你要禁用自动备份和修补的 SQL Server IaaS 代理，请使用以下命令：

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -Disable | Update-AzureVM

若要卸载 SQL Server IaaS 代理，请使用以下语法：

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension –Uninstall | Update-AzureVM

也可以使用 **Remove-AzureVMSqlServerExtension** 命令卸载该扩展：

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Remove-AzureVMSqlServerExtension | Update-AzureVM

>[AZURE.NOTE]禁用和卸载 SQL Server IaaS 代理不会删除以前配置的托管备份设置。应该在禁用或卸载 SQL Server IaaS 代理之前禁用自动备份。

## 兼容性

以下产品与自动备份的 SQL Server IaaS 代理功能兼容：

- Windows Server 2012

- Windows Server 2012 R2

- SQL Server 2014 Standard

- SQL Server 2014 Enterprise

## 后续步骤

自动备份将在 Azure VM 上配置托管备份。因此，请务必[查看有关托管备份的文档](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)，以了解其行为和影响。

可以在以下主题中找到针对 Azure VM 上的 SQL Server 的其他备份和还原指导：[Azure 虚拟机中 SQL Server 的备份和还原](/documentation/articles/virtual-machines-sql-server-backup-and-restore)。

Azure 中 SQL Server VM 的一个相关功能是 [Azure 虚拟机中的 SQL Server 自动修补](/documentation/articles/virtual-machines-sql-server-automated-patching)。

请查看其他[有关在 Azure 虚拟机中运行 SQL Server 的资源](/documentation/articles/virtual-machines-sql-server-infrastructure-services)。

<!---HONumber=70-->