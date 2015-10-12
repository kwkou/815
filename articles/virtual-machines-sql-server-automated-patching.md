<properties 
   pageTitle="Azure 虚拟机中 SQL Server 的自动修补"
   description="介绍 Azure 中运行的 SQL Server 虚拟机的自动修补功能。"
   services="virtual-machines"
   documentationCenter="na"
   authors="rothja"
   manager="jeffreyg"
   editor="monicar" />
<tags 
   ms.service="virtual-machines"
   ms.date="08/05/2015"
   wacn.date="09/18/2015" />

# Azure 虚拟机中 SQL Server 的自动修补

自动修补将为运行 SQL Server 2012 或 2014 的 Azure 虚拟机建立一个维护时段。只能在此维护时段内安装自动更新。对于 SQL Server，这可以确保在数据库的最佳可能时间发生系统更新和任何关联的重新启动。具体的操作取决于 SQL Server IaaS 代理。

>[AZURE.NOTE]自动修补依赖 SQL Server IaaS 代理。要安装和配置该代理，必须在目标虚拟机上运行 Azure VM 代理。较新的虚拟机库映像已默认启用此选项，但现有 VM 可能缺少 Azure VM 代理。如果使用你自己的 VM 映像，也需要安装 SQL Server IaaS 代理。有关详细信息，请参阅 [VM 代理和扩展](http://azure.microsoft.com/blog/2014/04/15/vm-agent-and-extensions-part-2/)。

## 在门户中配置自动修补

在创建新的 SQL Server 虚拟机时，可以使用 [Azure 预览门户](http://go.microsoft.com/fwlink/?LinkID=525040&clcid=0x409)配置自动修补。以下屏幕截图显示了“可选配置”|“SQL 自动修补”下的这些选项。

![Azure 门户中的 SQL 自动修补](./media/virtual-machines-sql-server-automated-patching/IC778484.jpg)

对于现有的 SQL Server 2012 或 2014 虚拟机，请在虚拟机属性的“配置”部分选择“自动修补”设置。在“自动修补”窗口中，可以启用功能、设置维护计划和起始小时，以及选择维护时段持续时间。如以下屏幕快照中所示。

![Azure 门户中的自动修补配置](./media/virtual-machines-sql-server-automated-patching/IC792132.jpg)

>[AZURE.NOTE]当你首次启用自动修补时，Azure 将在后台配置 SQL Server IaaS 代理。在此期间，门户将不会显示自动修补已配置。请等待几分钟，以便安装和配置代理。之后，该门户将反映新的设置。

## 使用 PowerShell 配置自动修补

你也可以使用 PowerShell 来配置自动修补。

以下示例使用 PowerShell 在现有的 SQL Server VM 上配置自动修补。**New-AzureVMSqlServerAutoPatchingConfig** 命令将为自动更新配置新的维护时段。

    $aps = New-AzureVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"
    
    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -AutoPatchingSettings $aps | Update-AzureVM

下表根据此示例描述了对目标 Azure VM 产生的实际效果：

|参数|效果|
|---|---|
|**DayOfWeek**|每个星期四安装修补程序。|
|**MaintenanceWindowStartingHour**|在上午 11:00 开始更新。|
|**MaintenanceWindowsDuration**|必须在 120 分钟内安装修补程序。根据开始时间，修补必须在下午 1:00 之前完成。|
|**PatchCategory**|此参数的唯一可能设置为“Important”。|

可能需要花费几分钟来安装和配置 SQL Server IaaS 代理。

若要禁用自动修补，请对 New-AzureVMSqlServerAutoPatchingConfig 运行不带 -Enable 参数的同一个脚本。与安装一样，可能需要花费几分钟时间来禁用自动修补。

## 禁用和卸载 SQL Server IaaS 代理

如果你要禁用自动备份和修补的 SQL Server IaaS 代理，请使用以下命令：

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension -Disable | Update-AzureVM

若要卸载 SQL Server IaaS 代理，请使用以下语法：

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Set-AzureVMSqlServerExtension –Uninstall | Update-AzureVM

也可以使用 **Remove-AzureVMSqlServerExtension** 命令卸载该扩展：

    Get-AzureVM -ServiceName <vmservicename> -Name <vmname> | Remove-AzureVMSqlServerExtension | Update-AzureVM

## 兼容性

以下产品与自动修补的 SQL Server IaaS 代理功能兼容：

- Windows Server 2012

- Windows Server 2012 R2

- SQL Server 2012

- SQL Server 2014

## 后续步骤

Azure 中 SQL Server VM 的一个相关功能是 [Azure 虚拟机中的 SQL Server 自动备份](/documentation/articles/virtual-machines-sql-server-automated-backup)。

请查看其他[有关在 Azure 虚拟机中运行 SQL Server 的资源](/documentation/articles/virtual-machines-sql-server-infrastructure-services)。

<!---HONumber=70-->