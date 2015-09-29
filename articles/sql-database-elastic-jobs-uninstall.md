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
	ms.date="08/09/2015" 
	wacn.date="09/15/2015"/>

#卸载弹性数据库作业组件
可以使用门户或 PowerShell 卸载**弹性数据库作业**组件。

##使用 Azure 门户卸载弹性数据库作业组件

1. 打开 [Azure 门户](https://manage.windowsazure.cn)。
2. 导航到包含**弹性数据库作业**组件的订阅，即装有弹性数据库作业组件的订阅。
3. 单击“浏览”，然后单击“资源组”。
4. 选择名为“\_\_ElasticDatabaseJob”的资源组。
5. 删除该资源组。

##使用 PowerShell 卸载弹性数据库作业组件

1.	启动 Microsoft Azure PowerShell 命令窗口，并导航到 Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x 文件夹下的 tools 子目录：键入 cd tools

		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*>cd tools

2.	执行.\\UninstallElasticDatabaseJobs.ps1 PowerShell 脚本。

		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>Unblock-File .\UninstallElasticDatabaseJobs.ps1
		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>.\UninstallElasticDatabaseJobs.ps1

假设组件的安装使用了默认值，则你可以简单地执行以下脚本：

		$ResourceGroupName = "__ElasticDatabaseJob"
		Switch-AzureMode AzureResourceManager
		
		$resourceGroup = Get-AzureResourceGroup -Name $ResourceGroupName
		if(!$resourceGroup)
		{
		    Write-Host "The Azure Resource Group: $ResourceGroupName has already been deleted.  Elastic database job components are uninstalled."
		    return
		}
		
		Write-Host "Removing the Azure Resource Group: $ResourceGroupName.  This may take a few minutes.”
		Remove-AzureResourceGroup -Name $ResourceGroupName -Force
		Write-Host "Completed removing the Azure Resource Group: $ResourceGroupName.  Elastic database job compoennts are now uninstalled."

## 后续步骤

若要重新安装弹性数据库作业，请参阅[安装弹性数据库作业服务](/documentation/articles/sql-database-elastic-jobs-service-installation)

有关弹性数据库作业的概述，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。

<!--Image references-->
[1]: ./media/sql-database-elastic-job-uninstall/
 

<!---HONumber=69-->