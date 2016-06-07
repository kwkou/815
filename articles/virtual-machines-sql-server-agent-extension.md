<properties
	pageTitle="SQL Server IaaS 代理扩展（经典）| Azure"
	description="本主题介绍可使 Azure 上运行 SQL Server 的 VM 使用自动化功能的 SQL Server 代理扩展。它使用经典部署模式。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="rothja"
	manager="jhubbard"
   editor=""    
   tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/08/2016"
	wacn.date="06/07/2016"/>

# SQL Server IaaS 代理扩展（经典）

通过此扩展能够使 Azure 虚拟机中的 SQL Server 利用本文章中列出的某些服务，这些服务仅在安装此扩展的情况下使用。为 Azure 门户中的 SQL Server 库映像自动安装此扩展。它可以在 Azure 中的任何 SQL Server VM 上进行安装，该 VM 已安装 Azure VM 来宾代理。

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。本文介绍使用经典部署模型。Microsoft 建议大多数新部署使用资源管理器模型。


## 先决条件
使用 Powershell cmdlet 的要求：

- 最新的 Azure PowerShell [可在此获得](/documentation/articles/powershell-install-configure)

在 VM 上使用扩展的要求：

- Azure VM 来宾代理
- Windows Server 2012、Windows Server 2012 R2 或更高版本
- SQL Server 2012、SQL Server 2014 或更高版本

## 服务适用于扩展

- **SQL 自动备份**：此服务对 VM 中的 SQL Server 默认实例自动执行所有数据库的备份计划。有关详细信息，请参阅 [Azure 虚拟机（经典）中 SQL Server 的自动备份](/documentation/articles/virtual-machines-windows-classic-sql-automated-backup)。
- **SQL 自动修补**：使用此服务能够配置维护时段，在此期间可能更新你的 VM，因此可以避免在工作负荷的高峰时间进行更新。有关详细信息，请参阅 [Azure 虚拟机（经典）中的 SQL Server 的自动修补](/documentation/articles/virtual-machines-windows-classic-sql-automated-patching)。
- **Azure 密钥保管库集成**：此服务让你能够在 SQL Server VM 上自动安装和配置 Azure 密匙保管库。有关详细信息，请参阅[为 Azure VM（经典）上的 SQL Server 配置 Azure 密匙保管库集成](/documentation/articles/virtual-machines-windows-classic-ps-sql-keyvault)。

## 使用 Powershell 添加扩展
如果使用 [Azure 门户](/documentation/articles/virtual-machines-windows-portal-sql-server-provision)设置 SQL Server VM，将会自动安装该扩展。针对使用 Azure 经典门户配置的 SQL Server VM 或生成你自己的 SQL 许可证的 VM，可以使用 **Set-AzureVMSqlServerExtension** Azure PowerShell cmdlet 添加此扩展。

### 语法

Set-AzureVMSqlServerExtension [[-ReferenceName] [String]] [-VM] IPersistentVM [[-Version] [String]] [[-AutoPatchingSettings] [AutoPatchingSettings]] [-AutoBackupSettings[AutoBackupSettings]] [-Profile [AzureProfile]] [CommonParameters]

> [AZURE.NOTE] 建议省略 -Version 参数。省略之后，默认值就是最新版本的扩展。

### 示例
下面的示例将使用 $abs 中定义的配置来配置自动备份设置（未在此处显示）。serviceName 是托管虚拟机的云服务名称。有关完整示例，请参阅 [Azure 虚拟机（经典）中 SQL Server 的自动备份](/documentation/articles/virtual-machines-windows-classic-sql-automated-backup)。

	Get-AzureVM -ServiceName "serviceName" -Name "vmName" | Set-AzureVMSqlServerExtension -AutoBackupSettings $abs | Update-AzureVM**

## 检查扩展的状态
如果想要检查此扩展的状态和与之关联的服务，可以使用任一门户。在现有 VM 的详细信息中，找到“设置”下的“扩展”。

你还可以使用 **Get-AzureVMSqlServerExtension** Azure Powershell cmdlet。

### 语法

Get-AzureVMSqlServerExtension [[-VM] [IPersistentVM]] [-Profile [AzureProfile]] [CommonParameters]

### 示例
	Get-AzureVM -ServiceName "service" -Name "vmname" | Get-AzureVMSqlServerExtension

## 使用 Powershell 删除扩展   
如果想要从 VM 中删除此扩展，则可以使用 **Remove-AzureVMSqlServerExtension** Azure Powershell cmdlet。

### 语法

Remove-AzureVMSqlServerExtension [-Profile [AzureProfile]] -VM IPersistentVM [CommonParameters]

<!---HONumber=Mooncake_0509_2016-->