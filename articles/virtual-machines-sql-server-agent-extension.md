<properties 
	pageTitle="Azure 虚拟机中的 SQL Server" 
	description="介绍 SQL Server 代理扩展，利用该扩展能够使Azure 上的云中运行 SQL Server 的虚拟机使用自动化功能，并介绍如何安装代理（如果尚未自动安装）。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="jeffgoll" 
	manager="jeffreyg"
	editor="08/29/2015"/>

<tags 
	ms.service="virtual-machines"
	ms.date="06/17/2015"
	wacn.date=""/>

# SQL Server IaaS 代理扩展

通过此扩展能够使 Azure 虚拟机中的 SQL Server 利用本文章中列出的某些服务，这些服务仅在安装此扩展的情况下使用。为 Azure 预览门户中的 SQL Server 库映像会自动安装此扩展。它可以在 Azure 中的任何 SQL Server VM 上进行安装，该 VM 已安装 Azure VM 来宾代理。
 
## 先决条件
使用 Powershell cmdlet 的要求：

- [此处提供](http://www.windowsazure.cn/downloads/)最新的 Azure 命令行 SDK

在 VM 上使用扩展的要求：

- Azure VM 来宾代理
- Windows Server 2012、Windows Server 2012 R2 或更高版本
- SQL Server 2012、SQL Server 2014 或更高版本
 
## 服务适用于扩展

- **SQL 自动备份**：此服务对 VM 中的 SQL Server 默认实例自动执行所有数据库的备份计划。若要查看有关此服务的详细信息，请参阅 [Azure 虚拟机中 SQL Server 的自动备份](https://msdn.microsoft.com/zh-cn/library/azure/dn906091.aspx)。
- **SQL 自动修补**：使用此服务能够配置维护时段，在此期间可能更新你的 VM，因此可以避免在工作负荷的高峰时间进行更新。若要查看有关此服务的详细信息，请参阅 [Azure 虚拟机中 SQL Server 的自动修补](https://msdn.microsoft.com/zh-cn/library/azure/dn961166.aspx)。

## 使用 Powershell 添加扩展
如果使用 [Azure 预览门户](https://manage.windowsazure.cn)设置 SQL Server VM，将会自动安装该扩展。针对使用 [Azure 管理门户](https://manage.windowsazure.cn)配置的 SQL Server VM 或生成你自己 SQL 许可证的 VM，可以使用以下 Azure PowerShell cmdlet 将此扩展添加到现有 VM。

**Set-AzureVMSqlServerExtension**

### 语法

Set-AzureVMSqlServerExtension [-VM] <IPersistentVM> [[-Version] <string>] [-AutoBackupSettings <AutoBackupSettings>] [-AutoPatchingSetttings <AutoPatchingSetttings>] [-Confirm] [-WhatIf] [<CommonParameters>]

> [AZURE.NOTE]建议省略 –版本参数。省略之后，默认值就是最新版本的扩展。

### 示例
	Get-AzureVM –ServiceName serviceName –Name vmName | Set-AzureVMSqlServerExtension –AutoBackupSettings $abs | Update-AzureVM**

## 检查扩展的状态
如果想要检查此扩展的状态和与之关联的服务，可以使用任一门户。在现有 VM 的详细信息中，找到“设置”下的“扩展”。

此外可以使用以下 Azure Powershell cmdlet。

**Get-AzureVMSqlServerExtension**

### 语法

Get-AzureVMSqlServerExtension [[-VM] <IPersistentVM>] [[-Version] <string>] [<CommonParameters>]

> [AZURE.NOTE]可以省略 –版本参数。省略之后，默认值就是最新版本的扩展。

### 示例
	Get-AzureVM –ServiceName "service" –Name "vmname" | Get-AzureVMSqlServerExtension

## 使用 Powershell 删除扩展   
如果想要从 VM 中删除此扩展，则可以使用以下 Azure Powershell cmdlet。

**Remove-AzureVMSqlServerExtension**

### 语法
Remove-AzureVMSqlServerExtension -VM <IPersistentVM> [<CommonParameters>]

<!---HONumber=67-->