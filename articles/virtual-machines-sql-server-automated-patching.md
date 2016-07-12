<properties
	pageTitle="VM 中的 SQL Server 自动修补（经典） | Azure"
	description="介绍 Azure 中使用经典部署模式运行的 SQL Server 虚拟机的自动修补功能。"
	services="virtual-machines"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar"
	tags="azure-resource-manager" />
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/08/2016"
	wacn.date="05/24/2016" />

# Azure 虚拟机中 SQL Server 的自动修补（经典）

自动修补将为运行 SQL Server 2012 或 2014 的 Azure 虚拟机建立一个维护时段。只能在此维护时段内安装自动更新。对于 SQL Server，这可以确保在数据库的最佳可能时间发生系统更新和任何关联的重新启动。具体的操作取决于 SQL Server IaaS 代理。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

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

Azure 中 SQL Server VM 的一个相关功能是 [Azure 虚拟机中的 SQL Server 自动备份](/documentation/articles/virtual-machines-windows-classic-sql-automated-backup/)。

请查看其他[有关在 Azure 虚拟机中运行 SQL Server 的资源](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0104_2016-->