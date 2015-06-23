<properties 
	pageTitle="使用 PowerShell 管理 SQL Azure 资源" 
	description="使用命令行管理 SQL Azure" 
	services="sql-database" 
	documentationCenter="" 
	authors="TigerMint" 
	manager="" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/13/2015"
	wacn.date="05/25/2015"/>

# 使用 PowerShell 管理 SQL Azure 资源


在本主题中，你将使用 PowerShell 脚本创建 Azure SQL Database 逻辑服务器、数据库和防火墙规则。

## 步骤 1：安装 Azure SDK

如果你尚未这样做，请按[如何安装和配置 Azure PowerShell](powershell-install-configure) 中的说明在本地计算机上安装 Azure PowerShell。然后，打开 Azure PowerShell 命令提示符。


## 步骤 2：配置 PowerShell 脚本
此 PowerShell 脚本将创建服务器、数据库和服务器防火墙规则。


1. 将以下 PowerShell cmdlet 块复制到文本编辑器中。
```powershell
		Add-AzureAccount #Only needed if you have not been authenicated yet. For Azure Automation, you will need to set up a Service Principal.
		Switch-AzureMode -Name AzureServiceManagement
		$AzureServer = New-AzureSqlDatabaseServer -AdministratorLogin "<Service Admin Login>" -AdministratorLoginPassword "<ServerLoginPassword>" -Location "<Location>" -Version "12.0" -verbose
		New-AzureSqlDatabase -ServerName $AzureServer.ServerName -DatabaseName  "<Database1>" -Edition "Standard" -verbose
		New-AzureSqlDatabaseServerFirewallRule -ServerName $AzureServer.ServerName -RuleName "<FirewallRuleName>" -StartIpAddress "<IP4StartRange>" -EndIpAddress "<IP4EndRange>" -verbose
```
2. 将 < > 内的所有内容替换为所需值。如需有效 Azure SQL Server 位置的列表，可以在 Azure Powershell 命令提示符下运行以下 cmdlet

		Switch-AzureMode -Name AzureResourceManager
		$AzureSQLLocations = Get-AzureLocation | Where-Object Name -Like "*SQL/Servers"
		$AzureSQLLocations.Locations

## 步骤 3：运行 PowerShell 脚本

检查你合并输入的 Azure PowerShell cmdlet。

复制步骤 2 中配置的 PowerShell cmdlet，并将其粘贴到 Azure PowerShell 命令提示符下。这将以 PowerShell 命令序列的形式发出这些 cmdlet，并创建 Azure SQL 服务器、数据库和服务器防火墙规则。  

如果以后你要再次创建这些 Azure SQL 资源，你可以： 

- 将这些命令保存为 PowerShell 脚本文件 (*.ps1)
- 在 Azure 管理门户的"自动化"部分中将这些命令保存为 Azure Automation Runbook 

## 示例

此 PowerShell 脚本将在中国北部创建资源 
```powershell
		Add-AzureAccount #Needed if you have not been authenicated yet. For Azure Automation, you will need to set up a Service Principal.
		Switch-AzureMode -Name AzureServiceManagement
		$AzureServer = New-AzureSqlDatabaseServer -AdministratorLogin "admin" -AdministratorLoginPassword "P@ssword" -Location "China North" -Version "12.0" -verbose
		New-AzureSqlDatabase -ServerName $AzureServer.ServerName -DatabaseName  "Database1" -Edition "Standard" -verbose
		New-AzureSqlDatabaseServerFirewallRule -ServerName $AzureServer.ServerName -RuleName "MyFirewallRule" -StartIpAddress "192.168.1.1" -EndIpAddress "192.168.1.1" -verbose
```
## 资源

有关 Azure SQL PowerShell Cmdlet 的详细信息，请[单击此处](https://msdn.microsoft.com/zh-cn/library/dn546726.aspx)

<!--HONumber=55-->