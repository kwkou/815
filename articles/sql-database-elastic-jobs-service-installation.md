<properties 
	pageTitle="安装弹性数据库作业 | Azure" 
	description="演练如何安装弹性作业功能。" 
	services="sql-database" 
	documentationCenter="" 
	manager="jhubbard" 
	authors="ddove" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="05/02/2016" 
	wacn.date="06/14/2016"/>

# 安装弹性数据库作业概述

可以通过 PowerShell 或 Azure 管理门户安装[**弹性数据库作业**](/documentation/articles/sql-database-elastic-jobs-overview/)。只有安装了 PowerShell 包，才能获取使用 PowerShell API 创建和管理作业的权限。此外，PowerShell API 目前提供的功能明显多于门户。

如果你从现有的**弹性数据库池**通过门户安装了**弹性数据库作业**，最新的 Powershell 预览包含用于升级现有安装的脚本。强烈建议将安装升级到最新的**弹性数据库作业**组件，以便利用通过 PowerShell API 公开的新功能。

## 先决条件
* Azure 订阅。若要获取试用版，请参阅[试用](/pricing/1rmb-trial)。
* Azure PowerShell。使用 [Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376)安装最新版本。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。
* [NuGet 命令行实用程序](https://nuget.org/nuget.exe)用于安装弹性数据库作业包。有关详细信息，请参阅 http://docs.nuget.org/docs/start-here/installing-nuget。

## 下载并导入弹性数据库作业 PowerShell 包
1. 启动 Azure PowerShell 命令窗口，并导航到 NuGet 命令行实用程序 (nuget.exe) 所下载到的目录。

2. 使用以下命令，将**弹性数据库作业**包下载并导入到当前目录：

		PS C:\>.\nuget install Microsoft.Azure.SqlDatabase.Jobs -prerelease

    **弹性数据库作业**文件放在本地目录中名为 **Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x**（其中的 *x.x.xxxx.x* 表示版本号）的文件夹内。PowerShell cmdlet（包括所需的客户端.dll）位于 **tools\\ElasticDatabaseJobs** 子目录中，用于安装、升级和卸载的 PowerShell 脚本也位于 **tools** 子目录中。

3. 键入 cd tools，导航到 Microsoft.Azure.SqlDatabase.Jobs.x.x.xxx.x 文件夹下的 tools 子目录，例如：

		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*>cd tools

4.	执行 .\\InstallElasticDatabaseJobsCmdlets.ps1 脚本，将 ElasticDatabaseJobs 目录复制到 $home\\Documents\\WindowsPowerShell\\Modules 中。这也会自动导入要使用的模块，例如：

		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>Unblock-File .\InstallElasticDatabaseJobsCmdlets.ps1
		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>.\InstallElasticDatabaseJobsCmdlets.ps1

## 使用 PowerShell 安装弹性数据库作业组件
1.	启动 Azure PowerShell 命令窗口，并导航到 Microsoft.Azure.SqlDatabase.Jobs.x.x.xxx.x 文件夹下的 \\tools 子目录：键入 cd \\tools

		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*>cd tools

2.	执行.\\InstallElasticDatabaseJobs.ps1 PowerShell 脚本，并提供其所请求的变量的值。此脚本将根据[弹性数据库作业组件和定价](/documentation/articles/sql-database-elastic-jobs-overview/#components-and-pricing)中所述创建组件，并将 Azure 云服务配置为适当使用依赖组件。

		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>Unblock-File .\InstallElasticDatabaseJobs.ps1
		PS C:*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>.\InstallElasticDatabaseJobs.ps1

当你运行此命令时，会打开一个窗口，要求你提供**用户名**和**密码**。这不是你的 Azure 凭据，请输入用户名和密码，将其作为你要为新服务器创建的管理员凭据。

你可以根据所需的设置，修改此示例调用中提供的参数。下面提供了有关每个参数行为的详细信息：

<table style="width:100%">
  <tr>
    <th>参数</th>
    <th>说明</th>
  </tr>

<tr>
	<td>ResourceGroupName</td>
	<td>提供为了包含新建 Azure 组件而创建的 Azure 资源组名称。此参数默认为“__ElasticDatabaseJob”。不建议更改此值。</td>
	</tr>

</tr>

	<tr>
	<td>ResourceGroupLocation</td>
	<td>提供用于保存新建 Azure 组件的 Azure 位置。此参数默认为“中国北部”位置。</td>
</tr>

<tr>
	<td>ServiceWorkerCount</td>
	<td>提供要安装的服务辅助角色数目。此参数默认为 1。可以使用更多的辅助角色来扩大服务，并提供高可用性。建议针对需要服务高可用性的部署使用“2”。</td>
	</tr>

</tr>
	<tr>
	<td>ServiceVmSize</td>
	<td>提供在云服务中使用的 VM 大小。此参数默认为 A0。接受参数值 A0/A1/A2/A3，这会导致辅助角色分别使用 ExtraSmall/Small/Medium/Large 大小。有关辅助角色大小的详细信息，请参阅 [弹性数据库作业组件和定价](/documentation/articles/sql-database-elastic-jobs-overview/#components-and-pricing)。</td>
</tr>

</tr>
	<tr>
	<td>SqlServerDatabaseSlo</td>
	<td>提供标准版的服务级别目标。此参数默认为 S0。接受参数值 S0/S1/S2/S3，这会导致 Azure SQL 数据库使用各自的 SLO。有关 SQL 数据库 SLO 的详细信息，请参阅 [弹性数据库作业组件和定价](/documentation/articles/sql-database-elastic-jobs-overview/#components-and-pricing)。</td>
</tr>

</tr>
	<tr>
	<td>SqlServerAdministratorUserName</td>
	<td>提供新建 Azure SQL 数据库服务器的管理员用户名。如果你未指定，系统将打开 PowerShell 凭据窗口提示你输入凭据。</td>
</tr>

</tr>
	<tr>
	<td>SqlServerAdministratorPassword</td>
	<td>提供新建 Azure SQL 数据库服务器的管理员密码。如果你未提供，系统将打开 PowerShell 凭据窗口提示你输入凭据。</td>
</tr>
</table>

对于要针对大量数据库同时运行大量作业的系统，建议指定如下参数：-ServiceWorkerCount 2 -ServiceVmSize A2 -SqlServerDatabaseSlo S2。

    PS C:*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>Unblock-File .\InstallElasticDatabaseJobs.ps1
    PS C:*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>.\InstallElasticDatabaseJobs.ps1 -ServiceWorkerCount 2 -ServiceVmSize A2 -SqlServerDatabaseSlo S2

## 使用 PowerShell 更新现有的弹性数据库作业组件安装

可以在现有的安装中更新**弹性数据库作业**，以实现缩放和高可用性。此程序允许升级将来的服务代码，而无需删除并重新创建控制数据库。也可以在同一版本中使用此过程来修改服务 VM 的大小或服务器的辅助角色计数。

若要更新安装的 VM 大小，请运行以下脚本，并将参数更新为你选择的值。

    PS C:*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>Unblock-File .\UpdateElasticDatabaseJobs.ps1
    PS C:*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>.\UpdateElasticDatabaseJobs.ps1 -ServiceVmSize A1 -ServiceWorkerCount 2

<table style="width:100%">
  <tr>
  <th>参数</th>
  <th>说明</th>
</tr>

  <tr>
	<td>ResourceGroupName</td>
	<td>标识最初安装弹性数据库作业组件时使用的 Azure 资源组名称。此参数默认为“__ElasticDatabaseJob”。不建议更改此值，因为你应该不必指定此参数。</td>
	</tr>
</tr>

</tr>

  <tr>
	<td>ServiceWorkerCount</td>
	<td>提供要安装的服务辅助角色数目。此参数默认为 1。可以使用更多的辅助角色来扩大服务，并提供高可用性。建议针对需要服务高可用性的部署使用“2”。</td>
</tr>

</tr>

	<tr>
	<td>ServiceVmSize</td>
	<td>提供在云服务中使用的 VM 大小。此参数默认为 A0。接受参数值 A0/A1/A2/A3，这会导致辅助角色分别使用 ExtraSmall/Small/Medium/Large 大小。有关辅助角色大小的详细信息，请参阅 [弹性数据库作业组件和定价](/documentation/articles/sql-database-elastic-jobs-overview/#components-and-pricing)。</td>
</tr>

</table>

## 后续步骤

确保已在组中的每个数据库上创建对脚本执行具有适当权限的凭据。有关详细信息，请参阅[保护你的 SQL 数据库](/documentation/articles/sql-database-security/)。


<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-service-installation/screen-1.png
[2]: ./media/sql-database-elastic-jobs-service-installation/credentials.png
[3]: ./media/sql-database-elastic-jobs-service-installation/start-board.png
[4]: ./media/sql-database-elastic-jobs-service-installation/not-done.png
 

<!---HONumber=Mooncake_0530_2016-->
