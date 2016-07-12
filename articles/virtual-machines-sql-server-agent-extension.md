<!-- Ibiza Portal: tested -->

<properties
	pageTitle="适用于 SQL Server VM 的 SQL Server 代理扩展（经典）| Azure"
	description="本主题介绍如何管理可以自动执行特定 SQL Server 管理任务的 SQL Server 代理扩展。这些任务包括自动备份、自动修补和 Azure 密钥保管库集成。本主题使用经典部署模式。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="rothja"
	manager="jhubbard"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/11/2016"
	wacn.date="06/27/2016"/>

# 适用于 SQL Server VM 的 SQL Server 代理扩展（经典）

Azure 虚拟机上运行的 SQL Server IaaS 代理扩展 (SQLIaaSAgent) 可以自动执行管理任务。本主题概述了该扩展支持的服务以及有关安装、状态及删除的说明。

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

##<a name="supported-services"></a> 支持的服务

SQL Server IaaS 代理扩展支持以下管理任务：

| 管理功能 | 说明 |
|---------------------|-------------------------------|
| **SQL 自动备份** | 对 VM 中的 SQL Server 默认实例自动执行所有数据库的备份计划。有关详细信息，请参阅 [Automated backup for SQL Server in Azure Virtual Machines (Classic)（Azure 虚拟机中 SQL Server 的自动备份（经典））](/documentation/articles/virtual-machines-windows-classic-sql-automated-backup/)。|
| **SQL 自动修补** | 配置维护时段，在此期间可能更新你的 VM，因此可以避免在工作负荷的高峰时间进行更新。有关详细信息，请参阅 [Automated Patching for SQL Server in Azure Virtual Machines (Classic)（Azure 虚拟机中 SQL Server 的自动修补（经典））](/documentation/articles/virtual-machines-windows-classic-sql-automated-patching/)。|
| **Azure 密钥保管库集成** | 可让你在 SQL Server VM 上自动安装和配置 Azure 密匙保管库。有关详细信息，请参阅 [Configure Azure Key Vault Integration for SQL Server on Azure VMs (Classic)（为 Azure VM 上的 SQL Server 配置 Azure 密钥保管库集成（经典））](/documentation/articles/virtual-machines-windows-classic-ps-sql-keyvault/)。|

## 先决条件

在 VM 上使用 SQL Server IaaS 代理扩展的要求：

- Azure VM 来宾代理（将新的 Azure VM 上自动安装 BGInfo 扩展）。
- Windows Server 2012、Windows Server 2012 R2 或更高版本。
- SQL Server 2012、SQL Server 2014 或更高版本。

使用 Powershell cmdlet 的要求：

- 最新的 Azure PowerShell [可在此获得](/documentation/articles/powershell-install-configure/)。

## 安装

当你预配某个 SQL Server 虚拟机库映像时，系统会自动安装 SQL Server IaaS 代理扩展。

如果你创建仅有 OS 的 Windows Server 虚拟机，可以使用 **Set-AzureVMSqlServerExtension** PowerShell cmdlet 手动安装扩展。使用命令来配置代理的服务之一，例如自动修补。VM 将安装代理（如果尚未安装）。

>[AZURE.NOTE] 有关使用 **Set-AzureVMSqlServerExtension** PowerShell 的说明，请参阅本文的[支持的服务](#supported-services)部分中列出的各个主题。

## 状态

验证是否已安装扩展的方法之一是在 Azure 门户预览中查看代理状态。在虚拟机边栏选项卡中选择“所有设置”，然后单击“扩展”。随后应会列出 **SQLIaaSAgent** 扩展。

![Azure 门户预览中的 SQL Server IaaS 代理扩展](./media/virtual-machines-windows-classic-sql-server-agent-extension/azure-sql-server-iaas-agent-portal.png)

也可以使用 **Get-AzureVMSqlServerExtension** Azure Powershell cmdlet。

	Get-AzureVM -ServiceName "service" -Name "vmname" | Get-AzureVMSqlServerExtension

## 删除   

在 Azure 门户预览中，可以通过单击虚拟机属性的“扩展”边栏选项卡中的省略号来卸载扩展。然后单击“删除”。

![在 Azure 门户预览中卸载 SQL Server IaaS 代理扩展](./media/virtual-machines-windows-classic-sql-server-agent-extension/azure-sql-server-iaas-agent-uninstall.png)

也可以使用 **Remove-AzureVMSqlServerExtension** Powershell cmdlet。

	Get-AzureVM -ServiceName "service" -Name "vmname" | Remove-AzureVMSqlServerExtension | Update-AzureVM

## 后续步骤

开始使用扩展支持的服务之一。有关详细信息，请参阅本文的[支持的服务](#supported-services)部分中提到的主题。

有关在 Azure 虚拟机中运行 SQL Server 的详细信息，请参阅 [Azure 虚拟机中的 SQL Server 概述](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0620_2016-->