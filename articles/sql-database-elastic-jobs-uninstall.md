<properties
	pageTitle="如何卸载弹性数据库作业工具"
	description="如何卸载弹性数据库作业工具"
	services="sql-database"
	documentationCenter=""
	manager="jeffreyg"
	authors="sidneyh"
	editor=""/>

<tags
	ms.service="sql-database"  
	ms.date="04/20/2015"
	wacn.date="06/23/2015"/>

# 如何卸载弹性数据库作业组件

如果尝试安装弹性数据库作业服务时发生失败，请删除服务的资源组。

## 卸载服务组件

1. 打开 [Azure 门户](https://manage.windowsazure.cn)。
2. 导航到包含弹性作业的订阅。
3. 单**浏览**，然后单击**资源组**。
4. 选择名为“__ElasticDatabaseJob”的资源组。
5. 删除该资源组。

或者使用以下 PowerShell 脚本：

1. 启动 [Microsoft Azure PowerShell 窗口](powershell-install-configure)。
2. 确保使用 PowerShell SDK 0.8.10 或更高版本。
3. 运行以下脚本：

		$ResourceGroupName = "__ElasticDatabaseJob"
		Switch-AzureMode AzureResourceManager

		$resourceGroup = Get-AzureResourceGroup -Name $ResourceGroupName
		if(!$resourceGroup)
		{
		    Write-Host "The Azure Resource Group: $ResourceGroupName has already been deleted.  Elastic database job is uninstalled."
		    return
		}

		Write-Host "Removing the Azure Resource Group: $ResourceGroupName.  This may take a few minutes.”
		Remove-AzureResourceGroup -Name $ResourceGroupName -Force
		Write-Host "Completed removing the Azure Resource Group: $ResourceGroupName.  Elastic database job is now uninstalled."

## 后续步骤

若要重新安装弹性数据库作业，请参阅[安装弹性数据库作业服务](sql-database-elastic-jobs-service-installation)

有关弹性作业服务的概述，请参阅[弹性作业概述](sql-database-elastic-jobs-overview)。

<!--Image references-->
[1]: ./media/sql-database-elastic-job-uninstall/

<!---HONumber=61-->