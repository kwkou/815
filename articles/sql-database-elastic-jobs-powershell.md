<properties 
	pageTitle="使用 PowerShell 创建和管理弹性数据库作业" 
	description="使用 PowerShell 管理 Azure SQL 数据库池" 
	services="sql-database" documentationCenter=""  
	manager="jeffreyg" 
	authors="ddove"/>

<tags 
	ms.service="sql-database" 
	ms.date="09/17/2015" 
	wacn.date="10/17/2015"/>

# 使用 PowerShell 创建和管理 SQL 数据库弹性数据库作业（预览版）

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-elastic-jobs-create-and-manage)
- [PowerShell](/documentation/articles/sql-database-elastic-jobs-powershell)

## 概述

**弹性数据库作业**功能（预览版）可让你可靠地执行 Transact-SQL (T-SQL) 脚本或跨数据库组应用 DACPAC，该组包含自定义的数据库集合、[弹性数据库池（预览版）](/documentation/articles/sql-database-elastic-pool)中的所有数据库，或分片集合（使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library)创建）。在预览中，**弹性数据库作业**是当前的客户托管 Azure 云服务，可以让你执行即席任务和计划的管理任务，称为“作业”。使用这个功能，可以轻松可靠地大规模管理大量 Azure SQL 数据库，方法是运行 Transact-SQL 脚本来执行管理操作，例如架构更改、凭据管理、引用数据更新、性能数据收集或租户（客户）遥测数据收集。有关弹性数据库作业的详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。

使用适用于**弹性数据库作业**的 PowerShell API，可以灵活定义要针对哪组数据库执行哪些脚本。目前，通过 Azure 门户的**弹性数据库作业**功能具有缩减的功能集，并且限制为针对弹性数据库池执行。

**弹性数据库作业**（预览版）使用多个 Azure 组件来定义要执行的作业、定义何时执行作业、执行作业、跟踪作业成功或失败，以及选择性地为返回结果查询指定结果目标。因为此预览版中包含的 Powershell API 在通过门户初始预览之后包含其他功能，建议安装最新的**弹性数据库作业**组件。如果已安装，只要升级**弹性数据库作业**组件即可。有关从 [Nuget](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.Jobs) 安装的详细信息，请参阅[安装弹性数据库作业组件](/documentation/articles/sql-database-elastic-jobs-service-installation)。

本文将说明如何创建所有必要项目来创建和管理**弹性数据库作业**，Azure 订阅除外。如果你需要 Azure 订阅，只需单击本页顶部的“免费试用”，然后再回来完成本文的相关操作即可。本主题对[弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started)中的示例做了延伸。完成本主题后，你将学会如何创建和管理作业，以针对**分片集**定义的分片数据库组和自定义数据库集合执行管理操作。

## 先决条件
* Azure 订阅。如需免费试用，请参阅[免费试用一个月](/pricing/1rmb-trial/)。
* 必须先下载、导入**弹性数据库作业** PowerShell 包，然后安装弹性数据库作业组件。遵循这些步骤：[安装弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-service-installation)
* Azure PowerShell 版本为 0.8.16 或以上。通过 [Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376)安装最新版本 (0.9.5)。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。

### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID (**-SubscriptionId**) 或订阅名称 (**-SubscriptionName**)。如果你有多个订阅，你可以运行 **Get-Subscription** cmdlet，然后从结果集中复制所需的订阅信息。获取订阅信息后，请执行以下 cmdlet 将此订阅设置为默认值，也就是创建和管理作业的目标：

	Select-AzureSubscription -SubscriptionId {SubscriptionID}

建议使用 [PowerShell ISE](https://technet.microsoft.com/zh-cn/library/dd315244.aspx) 针对弹性数据库作业开发和执行 PowerShell 脚本。

## 弹性数据库作业对象

下表列出了**弹性数据库作业**的所有对象类型及其描述和相关 PowerShell API。

<table style="width:100%">
  <tr>
    <th>对象类型</th>
    <th>说明</th>
    <th>相关的 PowerShell API</th>
  </tr>
  <tr>
    <td>凭据</td>
    <td>连接到数据库以执行脚本或应用 DACPAC 时使用的用户名和密码。<p>密码在发送并存储在弹性数据库作业数据库之前会经过加密。密码由弹性数据库作业服务通过安装脚本创建和上载的凭据解密。</td>
	<td><p>Get-AzureSqlJobCredential</p>
	<p>New-AzureSqlJobCredential</p><p>Set-AzureSqlJobCredential</p></td></td>
  </tr>

  <tr>
    <td>脚本</td>
    <td>用于跨数据库执行的 Transact-SQL 脚本。脚本应该编写为幂等，因为服务将在失败时重试执行脚本。
	</td>
	<td>
	<p>Get-AzureSqlJobContent</p>
	<p>Get-AzureSqlJobContentDefinition</p>
	<p>New-AzureSqlJobContent</p>
	<p>Set-AzureSqlJobContentDefinition</p>
	</td>
  </tr>

  <tr>
    <td>DACPAC</td>
    <td>跨数据库应用的<a href="https://msdn.microsoft.com/zh-cn/library/ee210546.aspx">数据层应用程序</a>包。

	</td>
	<td>
	<p>Get-AzureSqlJobContent</p>
	<p>New-AzureSqlJobContent</p>
	<p>Set-AzureSqlJobContentDefinition</p>
	</td>
  </tr>
  <tr>
    <td>数据库目标</td>
    <td>指向 Azure SQL 数据库的数据库和服务器名称。

	</td>
	<td>
	<p>Get-AzureSqlJobTarget</p>
	<p>New-AzureSqlJobTarget</p>
	</td>
  </tr>
  <tr>
    <td>分片映射目标</td>
    <td>数据库目标和凭据的组合，用于确定弹性数据库分片映射内存储的信息。
	</td>
	<td>
	<p>Get-AzureSqlJobTarget</p>
	<p>New-AzureSqlJobTarget</p>
	<p>Set-AzureSqlJobTarget</p>
	</td>
  </tr>
<tr>
    <td>自定义集合目标</td>
    <td>共同用于执行的已定义数据库组。</td>
	<td>
	<p>Get-AzureSqlJobTarget</p>
	<p>New-AzureSqlJobTarget</p>
	</td>
  </tr>
<tr>
    <td>自定义集合子目标</td>
    <td>从自定义集合引用的数据库目标。</td>
	<td>
	<p>Add-AzureSqlJobChildTarget</p>
	<p>Remove-AzureSqlJobChildTarget</p>
	</td>
  </tr>

<tr>
    <td>作业</td>
    <td>
	<p>作业参数的定义，可用于触发执行或完成计划。</p>
	</td>
	<td>
	<p>Get-AzureSqlJob</p>
	<p>New-AzureSqlJob</p>
	<p>Set-AzureSqlJob</p>
	</td>
  </tr>

<tr>
    <td>作业执行</td>
    <td>
	<p>必要的任务容器，以便使用数据库连接的凭据执行脚本或将 DACPAC 应用到目标，将会根据执行策略处理失败。</p>
	</td>
	<td>
	<p>Get-AzureSqlJobExecution</p>
	<p>Start-AzureSqlJobExecution</p>
	<p>Stop-AzureSqlJobExecution</p>
	<p>Wait-AzureSqlJobExecution</p>
  </tr>

<tr>
    <td>作业任务执行</td>
    <td>
	<p>用于完成作业的单个工作单位。</p>
	<p>如果某个作业任务无法成功执行，将记录生成的异常消息，创建新的匹配作业任务并根据指定的执行策略执行。</p></p>
	</td>
	<td>
	<p>Get-AzureSqlJobExecution</p>
	<p>Start-AzureSqlJobExecution</p>
	<p>Stop-AzureSqlJobExecution</p>
	<p>Wait-AzureSqlJobExecution</p>
  </tr>

<tr>
    <td>作业执行策略</td>
    <td>
	<p>控制作业执行超时、重试限制和重试间隔。</p>
	<p>弹性数据库作业包括默认作业执行策略，该策略在本质上会导致使用重试间隔的指数回退无限期重试失败的作业任务。</p>
	</td>
	<td>
	<p>Get-AzureSqlJobExecutionPolicy</p>
	<p>New-AzureSqlJobExecutionPolicy</p>
	<p>Set-AzureSqlJobExecutionPolicy</p>
	</td>
  </tr>

<tr>
    <td>计划</td>
    <td>
	<p>基于时间的执行指定在重复间隔发生或单次数发生。</p>
	</td>
	<td>
	<p>Get-AzureSqlJobSchedule</p>
	<p>New-AzureSqlJobSchedule</p>
	<p>Set-AzureSqlJobSchedule</p>
	</td>
  </tr>

<tr>
    <td>作业触发器</td>
    <td>
	<p>作业与计划之间的映射，用于根据计划触发作业执行。</p>
	</td>
	<td>
	<p>New-AzureSqlJobTrigger</p>
	<p>Remove-AzureSqlJobTrigger</p>
	</td>
  </tr>
</table>

## 支持的弹性数据库作业组类型
**弹性数据库作业**可以跨数据库组执行 Transact-SQL (T-SQL) 脚本或应用 DACPAC。当提交作业以跨数据库组执行时，弹性数据库作业将“扩展”作业到子作业，其中每个作业针对组中的单一数据库执行请求的执行。
 
以下是当前支持的组类型列表：

* [分片映射](/documentation/articles/sql-database-elastic-scale-shard-map-management)：当提交作业以分片映射为目标时，作业先查询分片映射来确定其当前的分片集，然后展开作业到子作业，匹配分片映射内的每个分片。
* 自定义集合：指定以指向自定义的数据库集合。当提交作业以自定义集合为目标时，作业将展开作业到子作业，匹配自定义集合内当前定义的每个数据库。

## 设置弹性数据库作业连接
加载 PowerShell 模块之后，连接必须设为弹性数据库作业*控制数据库*，才能使用作业 API。此 cmdlet 的调用将在安装弹性数据库作业时，触发凭据窗口弹出以请求提供用户名/密码。本主题中提供的所有示例都假设已经执行第一个步骤。

建立与弹性数据库作业的连接：

	Use-AzureSqlJobConnection -CurrentAzureSubscription 

## 弹性数据库作业中的已加密凭据
数据库凭据可以插入弹性数据库作业*控制数据库*并加密其密码。必须存储凭据，使作业可以在稍后执行，包括使用作业计划。
 
加密是通过创建为安装脚本一部分的证书来进行的。安装脚本创建证书并将其上载到 Azure 云服务，以解密已存储的加密密码。Azure 云服务稍后在弹性数据库作业*控制数据库*内存储公钥，让 PowerShell API 或 Azure 门户界面加密提供的密码，而不需要在本地安装证书。
 
尽管凭据密码已加密并且通过弹性数据库作业对象的只读访问权而受到保护，但是具有弹性数据库作业对象读写访问权的恶意用户可以提取密码。凭据设计为跨作业执行重复使用。在建立连接时，凭据将传递到目标数据库。当前目标数据库上没有任何限制可用于每个凭据，因此恶意用户可能添加在其掌控之下的数据库目标，并且后续启动作业以此数据库为目标，获得凭据密码的信息。

弹性数据库作业的安全最佳实践如下：

* 将 API 的使用限制为受信任的个人。
* 凭据应该具有执行作业任务所需的最低权限。有关详细信息，请参阅这篇[授权和权限](https://msdn.microsoft.com/zh-cn/library/bb669084.aspx) SQL Server MSDN 文章。

### 跨数据库创建作业执行的加密凭据
为了创建新的加密凭据，Get-Credential cmdlet 将提示输入可以传递给 **New-AzureSqlJobCredential** cmdlet 的用户名和密码。

	$credentialName = "{Credential Name}"
	$databaseCredential = Get-Credential
	$credential = New-AzureSqlJobCredential -Credential $databaseCredential -CredentialName $credentialName
	Write-Output $credential

### 更新凭据
若要更新现有凭据以支持密码更改的情况，请使用 **Set-AzureSqlJobCredential** cmdlet 并设置 **CredentialName** 参数。

	$credentialName = "{Credential Name}"
	Set-AzureSqlJobCredential -CredentialName $credentialName -Credential $credential 

## 定义弹性数据库分片映射目标
使用分片映射作为数据库目标，针对分片集（使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library)创建）的所有数据库执行作业。本示例依赖于使用弹性数据库客户端库创建分片应用程序。下载并运行[弹性数据库工具示例入门](/documentation/articles/sql-database-elastic-scale-get-started)。

### 使用示例应用程序创建分片映射管理器

在此处，你将创建分片映射管理器以及多个分片，然后将数据插入分片。如果你已设置分片，则可以跳过下面的步骤，直接转到下一部分。

1. 生成并运行**弹性数据库工具入门**示例应用程序。一直执行到[下载和运行示例应用](/documentation/articles/sql-database-elastic-scale-get-started/#Getting-started-with-elastic-database-tools)部分中的步骤 7。在步骤 7 结束时，你将看到以下命令提示符：

	![命令提示符][1]

2.  在命令窗口中键入“1”，然后按 **Enter**。这会创建分片映射管理器，并将两个分片添加到服务器。然后键入“3”并按 **Enter**；重复该操作四次。这会在你的分片中插入示例数据行。
  
3.  [Azure 预览门户](https://manage.windowsazure.cn)应会在 v12 服务器中显示三个新的数据库：

	![Visual Studio 确认][2]

现在，使用 **New-AzureSqlJobTarget** cmdlet 创建分片映射目标。必须将分片映射管理器数据库设置为数据库目标，然后将特定分片映射指定为目标。

	$shardMapCredentialName = "{Credential Name}"
	$shardMapDatabaseName = "{ShardMapDatabaseName}" #example: ElasticScaleStarterKit_ShardMapManagerDb
	$shardMapDatabaseServerName = "{ShardMapServerName}"
	$shardMapName = "{MyShardMap}" #example: CustomerIDShardMap
	$shardMapDatabaseTarget = New-AzureSqlJobTarget -DatabaseName $shardMapDatabaseName -ServerName $shardMapDatabaseServerName
	$shardMapTarget = New-AzureSqlJobTarget -ShardMapManagerCredentialName $shardMapCredentialName -ShardMapManagerDatabaseName $shardMapDatabaseName -ShardMapManagerServerName $shardMapDatabaseServerName -ShardMapName $shardMapName
	Write-Output $shardMapTarget

## 创建 T-SQL 脚本用于跨数据库执行

在针对执行创建 T-SQL 脚本时，强烈建议将它们生成为对于失败幂等且具备弹性。每当执行发生失败时，不论失败的分类，弹性数据库作业将重试执行脚本。

使用 **New-AzureSqlJobContent** cmdlet 创建并保存执行脚本，然后设置 **-ContentName** 和 **-CommandText** 参数。

	$scriptName = "Create a TestTable"

	$scriptCommandText = "
	IF NOT EXISTS (SELECT name FROM sys.tables WHERE name = 'TestTable')
	BEGIN
		CREATE TABLE TestTable(
			TestTableId INT PRIMARY KEY IDENTITY,
			InsertionTime DATETIME2
		);
	END
	GO
	INSERT INTO TestTable(InsertionTime) VALUES (sysutcdatetime());
	GO"

	$script = New-AzureSqlJobContent -ContentName $scriptName -CommandText $scriptCommandText
	Write-Output $script

### 从文件创建新脚本
如果 T-SQL 脚本是在文件中定义的，请使用以下脚本来导入脚本：

	$scriptName = "My Script Imported from a File"
	$scriptPath = "{Path to SQL File}"
	$scriptCommandText = Get-Content -Path $scriptPath
	$script = New-AzureSqlJobContent -ContentName $scriptName -CommandText $scriptCommandText
	Write-Output $script

### 更新 T-SQL 脚本用于跨数据库执行  

以下 PowerShell 脚本可用于更新现有脚本的 T-SQL 命令文本。

设置以下变量以反映要设置的所需脚本定义：

	$scriptName = "Create a TestTable"
	$scriptUpdateComment = "Adding AdditionalInformation column to TestTable"
	$scriptCommandText = "
	IF NOT EXISTS (SELECT name FROM sys.tables WHERE name = 'TestTable')
	BEGIN
	CREATE TABLE TestTable(
		TestTableId INT PRIMARY KEY IDENTITY,
		InsertionTime DATETIME2
	);
	END
	GO

	IF NOT EXISTS (SELECT columns.name FROM sys.columns INNER JOIN sys.tables on columns.object_id = tables.object_id WHERE tables.name = 'TestTable' AND columns.name = 'AdditionalInformation')
	BEGIN
    ALTER TABLE TestTable
    ADD AdditionalInformation NVARCHAR(400);
	END
	GO

	INSERT INTO TestTable(InsertionTime, AdditionalInformation) VALUES (sysutcdatetime(), 'test');
	GO"

### 更新现有脚本的定义

	Set-AzureSqlJobContentDefinition -ContentName $scriptName -CommandText $scriptCommandText -Comment $scriptUpdateComment 

## 创建一个用于跨分片映射执行脚本的作业

以下 PowerShell 脚本可用于针对跨弹性缩放分片映射中每个分片的脚本执行启动作业。

设置以下变量以反映所需的脚本和目标：

	$jobName = "{Job Name}"
	$scriptName = "{Script Name}"
	$shardMapServerName = "{Shard Map Server Name}"
	$shardMapDatabaseName = "{Shard Map Database Name}"
	$shardMapName = "{Shard Map Name}"
	$credentialName = "{Credential Name}"
	$shardMapTarget = Get-AzureSqlJobTarget -ShardMapManagerDatabaseName $shardMapDatabaseName -ShardMapManagerServerName $shardMapServerName -ShardMapName $shardMapName 
	$job = New-AzureSqlJob -ContentName $scriptName -CredentialName $credentialName -JobName $jobName -TargetId $shardMapTarget.TargetId
	Write-Output $job

## 执行作业 

以下 PowerShell 脚本可以用于执行现有的作业：

更新以下变量以反映要执行的所需作业名称：

	$jobName = "{Job Name}"
	$jobExecution = Start-AzureSqlJobExecution -JobName $jobName 
	Write-Output $jobExecution
 
## 检索单个作业执行的状态

使用 **Get-AzureSqlJobExecution** cmdlet 并设置 **JobExecutionId** 参数以查看作业执行的状态。

	$jobExecutionId = "{Job Execution Id}"
	$jobExecution = Get-AzureSqlJobExecution -JobExecutionId $jobExecutionId
	Write-Output $jobExecution

使用相同 **Get-AzureSqlJobExecution** cmdlet 搭配 **IncludeChildren** 参数，以查看子作业执行的状态，还就是针对作为作业目标的每个数据库的每个作业执行的特定状态。

	$jobExecutionId = "{Job Execution Id}"
	$jobExecutions = Get-AzureSqlJobExecution -JobExecutionId $jobExecutionId -IncludeChildren
	Write-Output $jobExecutions 

## 查看多个作业执行的状态

**Get-AzureSqlJobExecution** cmdlet 具有多个可选参数，用于显示多个作业执行（已通过提供的参数进行筛选）。下面演示了 Get-AzureSqlJobExecution 的一些可能的用法：

检索所有处于活动状态的顶级作业执行：

	Get-AzureSqlJobExecution

检索所有顶级作业执行，包括非活动的作业执行：

	Get-AzureSqlJobExecution -IncludeInactive

检索已提供作业执行 ID 的所有子作业执行，包括非活动的作业执行：

	$parentJobExecutionId = "{Job Execution Id}"
	Get-AzureSqlJobExecution -AzureSqlJobExecution -JobExecutionId $parentJobExecutionId –IncludeInactive -IncludeChildren

检索使用计划/作业组合创建的所有作业执行，包括非活动的作业：

	$jobName = "{Job Name}"
	$scheduleName = "{Schedule Name}"
	Get-AzureSqlJobExecution -JobName $jobName -ScheduleName $scheduleName -IncludeInactive

检索以指定的分片映射为目标的所有作业，包括非活动的作业：

	$shardMapServerName = "{Shard Map Server Name}"
	$shardMapDatabaseName = "{Shard Map Database Name}"
	$shardMapName = "{Shard Map Name}"
	$target = Get-AzureSqlJobTarget -ShardMapManagerDatabaseName $shardMapDatabaseName -ShardMapManagerServerName $shardMapServerName -ShardMapName $shardMapName
	Get-AzureSqlJobExecution -TargetId $target.TargetId –IncludeInactive

检索以指定的自定义集合为目标的所有作业，包括非活动的作业：

	$customCollectionName = "{Custom Collection Name}"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
	Get-AzureSqlJobExecution -TargetId $target.TargetId –IncludeInactive
 
检索特定作业执行内的作业任务执行的列表：

	$jobExecutionId = "{Job Execution Id}"
	$jobTaskExecutions = Get-AzureSqlJobTaskExecution -JobExecutionId $jobExecutionId
	Write-Output $jobTaskExecutions 

检索作业任务执行详细信息：

以下 PowerShell 脚本可用于查看作业任务执行的详细信息，在调试执行失败时特别有用。

	$jobTaskExecutionId = "{Job Task Execution Id}"
	$jobTaskExecution = Get-AzureSqlJobTaskExecution -JobTaskExecutionId $jobTaskExecutionId
	Write-Output $jobTaskExecution

## 检索作业任务执行内的失败

JobTaskExecution 对象包括任务生命周期的属性以及 Message 属性。如果作业任务执行失败，生命周期属性将设为 *Failed*，且消息属性将设为生成的异常消息及其堆栈。如果作业不成功，必须查看给定操作不成功的作业任务的详细信息。

	$jobExecutionId = "{Job Execution Id}"
	$jobTaskExecutions = Get-AzureSqlJobTaskExecution -JobExecutionId $jobExecutionId
	Foreach($jobTaskExecution in $jobTaskExecutions) 
		{
		if($jobTaskExecution.Lifecycle -ne 'Succeeded')
    		{
        	Write-Output $jobTaskExecution
    		}
		}

## 等待作业执行完成

以下 PowerShell 脚本可用于等待作业任务完成：

	$jobExecutionId = "{Job Execution Id}"
	Wait-AzureSqlJobExecution -JobExecutionId $jobExecutionId 

## 创建自定义执行策略

弹性数据库作业支持创建可在启动作业时应用的自定义执行策略。
  
执行策略当前允许定义：

* 名称：执行策略的标识符。
* 作业超时：弹性数据库作业取消作业之前的总时间。
* 初始重试间隔：第一次重试之前等待的间隔。
* 最大重试间隔：要使用的重试间隔上限。
* 重试间隔回退系数：用于计算每两次重试的下一个间隔系数。使用以下公式：(初始重试间隔) * Math.pow((间隔回退指数),(重试次数) - 2)。 
* 最大尝试次数：重试在作业中执行的最大次数。

默认的执行策略使用以下值：

* 名称：默认执行策略
* 作业超时：1 周
* 初始重试间隔：100 毫秒
* 最大重试间隔：30 分钟
* 重试间隔系数：2
* 最大尝试次数：2,147,483,647

创建所需的执行策略：

	$executionPolicyName = "{Execution Policy Name}"
	$initialRetryInterval = New-TimeSpan -Seconds 10
	$jobTimeout = New-TimeSpan -Minutes 30
	$maximumAttempts = 999999
	$maximumRetryInterval = New-TimeSpan -Minutes 1
	$retryIntervalBackoffCoefficient = 1.5
	$executionPolicy = New-AzureSqlJobExecutionPolicy -ExecutionPolicyName $executionPolicyName -InitialRetryInterval $initialRetryInterval -JobTimeout $jobTimeout -MaximumAttempts $maximumAttempts -MaximumRetryInterval $maximumRetryInterval -RetryIntervalBackoffCoefficient $retryIntervalBackoffCoefficient
	Write-Output $executionPolicy

### 更新自定义执行策略

更新要更新的所需执行策略：

	$executionPolicyName = "{Execution Policy Name}"
	$initialRetryInterval = New-TimeSpan -Seconds 15
	$jobTimeout = New-TimeSpan -Minutes 30
	$maximumAttempts = 999999
	$maximumRetryInterval = New-TimeSpan -Minutes 1
	$retryIntervalBackoffCoefficient = 1.5
	$updatedExecutionPolicy = Set-AzureSqlJobExecutionPolicy -ExecutionPolicyName $executionPolicyName -InitialRetryInterval $initialRetryInterval -JobTimeout $jobTimeout -MaximumAttempts $maximumAttempts -MaximumRetryInterval $maximumRetryInterval -RetryIntervalBackoffCoefficient $retryIntervalBackoffCoefficient
	Write-Output $updatedExecutionPolicy
 
## 取消作业

弹性数据库作业支持取消作业请求。如果弹性数据库作业检测到当前正在执行作业的取消请求，它将尝试停止作业。

弹性数据库作业可通过两种不同的方式执行取消：

1. 取消当前正在执行的任务：如果在任务正在执行时检测到取消，将在当前正在执行的任务层面尝试取消。例如：当尝试取消时，如果有某个长时间运行的查询当前正在执行，将尝试取消该查询。
2. 取消任务重试：如果控制线程在启动任务执行之前检测到取消，控制线程将避免启动该任务，并将请求声明为已取消。

如果针对父作业请求了作业取消，则会对父作业及其所有子作业执行取消请求。
 
若要提交取消请求，请使用 **Stop-AzureSqlJobExecution** cmdlet 并设置 **JobExecutionId** 参数。

	$jobExecutionId = "{Job Execution Id}"
	Stop-AzureSqlJobExecution -JobExecutionId $jobExecutionId

## 根据名称和作业的历史记录删除作业

弹性数据库作业支持异步删除作业。可将某个作业标记为待删除，系统将在作业的作业执行都已完成后，删除该作业及其所有作业历史记录。系统不会自动取消处于活动状态的作业执行。

必须调用 Stop-AzureSqlJobExecution 来取消处于活动状态的作业执行。

若要触发作业删除，请使用 **Remove-AzureSqlJob** cmdlet 并设置 **JobName** 参数。

	$jobName = "{Job Name}"
	Remove-AzureSqlJob -JobName $jobName
 
## 创建自定义数据库目标
自定义数据库目标可以在弹性数据库作业中定义，可用于直接执行或包含在自定义数据库组内。由于**弹性数据库池**尚不能通过 PowerShell API 直接支持，你只需创建自定义数据库目标和自定义数据库集合目标，其中包含池中的所有数据库。

设置以下变量以反映所需的数据库信息：

	$databaseName = "{Database Name}"
	$databaseServerName = "{Server Name}"
	New-AzureSqlJobDatabaseTarget -DatabaseName $databaseName -ServerName $databaseServerName 

## 创建自定义数据库集合目标
可以自定义数据库集合目标，以跨多个已定义数据库目标执行。创建数据库组之后，数据库可与自定义集合目标相关联。

设置以下变量以反映所需的自定义集合目标配置：

	$customCollectionName = "{Custom Database Collection Name}"
	New-AzureSqlJobTarget -CustomCollectionName $customCollectionName 

### 将数据库添加到自定义数据库集合目标

数据库目标可以与自定义数据库集合目标相关联，以创建数据库组。每当创建以自定义数据库集合目标为目标的作业时，都会在执行时扩展为以关联到该组的数据库为目标。

将所需的数据库添加到特定的自定义集合：

	$serverName = "{Database Server Name}"
	$databaseName = "{Database Name}"
	$customCollectionName = "{Custom Database Collection Name}"
	Add-AzureSqlJobChildTarget -CustomCollectionName $customCollectionName -DatabaseName $databaseName -ServerName $databaseServerName 

#### 查看自定义数据库集合目标内的数据库

使用 **Get-AzureSqlJobTarget** cmdlet 检索自定义数据库集合目标内的子数据库。
 
	$customCollectionName = "{Custom Database Collection Name}"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
	$childTargets = Get-AzureSqlJobTarget -ParentTargetId $target.TargetId
	Write-Output $childTargets

### 创建作业以跨自定义数据库集合目标执行脚本

使用 **New-AzureSqlJob** cmdlet 可以针对自定义数据库集合目标定义的数据库组创建作业。弹性数据库作业将作业扩展为多个子作业（每个子作业映射到与自定义数据库集合目标关联的数据库），并确保脚本针对每个数据库执行。同样，重要的是脚本具有幂等处理重试的弹性。

	$jobName = "{Job Name}"
	$scriptName = "{Script Name}"
	$customCollectionName = "{Custom Collection Name}"
	$credentialName = "{Credential Name}"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
	$job = New-AzureSqlJob -JobName $jobName -CredentialName $credentialName -ContentName $scriptName -TargetId $target.TargetId
	Write-Output $job

## 跨数据库收集数据

**弹性数据库作业**支持跨数据库组执行查询，并将结果发送到指定的数据库表。可以在事实之后查询数据表，以查看每个数据库的查询结果。这提供了跨多个数据库执行查询的异步机制。例如其中一个数据库暂时不可用的失败案例是通过重试自动处理。

如果不存在与返回的结果集架构相符的指定目标表，则自动创建该表。如果脚本执行返回多个结果集，弹性数据库作业只将第一个结果集发送到提供的目标表。

以下 PowerShell 脚本可用于执行脚本，将其结果收集到指定的表。此脚本假设已创建一个可输出单个结果集的 T-SQL 脚本，并且已创建自定义的数据库集合目标。

设置以下项以反映所需的脚本、凭据和执行目标：

	$jobName = "{Job Name}"
	$scriptName = "{Script Name}"
	$executionCredentialName = "{Execution Credential Name}"
	$customCollectionName = "{Custom Collection Name}"
	$destinationCredentialName = "{Destination Credential Name}"
	$destinationServerName = "{Destination Server Name}"
	$destinationDatabaseName = "{Destination Database Name}"
	$destinationSchemaName = "{Destination Schema Name}"
	$destinationTableName = "{Destination Table Name}"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName

### 创建和启动用于数据库收集方案的作业
	$job = New-AzureSqlJob -JobName $jobName -CredentialName $executionCredentialName -ContentName $scriptName -ResultSetDestinationServerName $destinationServerName -ResultSetDestinationDatabaseName $destinationDatabaseName -ResultSetDestinationSchemaName $destinationSchemaName -ResultSetDestinationTableName $destinationTableName -ResultSetDestinationCredentialName $destinationCredentialName -TargetId $target.TargetId
	Write-Output $job
	$jobExecution = Start-AzureSqlJobExecution -JobName $jobName
	Write-Output $jobExecution

## 使用作业触发器创建作业执行计划

以下 PowerShell 脚本可用于创建重复计划。此脚本使用分钟间隔，但是 New-AzureSqlJobSchedule 也支持 -DayInterval、-HourInterval、-MonthInterval 和 -WeekInterval 参数。可以通过传递 -OneTime 来创建仅执行一次的计划。

创建新计划：

	$scheduleName = "Every one minute"
	$minuteInterval = 1
	$startTime = (Get-Date).ToUniversalTime()
	$schedule = New-AzureSqlJobSchedule -MinuteInterval $minuteInterval -ScheduleName $scheduleName -StartTime $startTime 
	Write-Output $schedule

### 创建作业触发器，使作业按时间计划执行

可以定义作业触发器，使作业根据时间计划执行。以下 PowerShell 脚本可用于创建作业触发器。

设置以下变量以对应于所需的作业和计划：

	$jobName = "{Job Name}"
	$scheduleName = "{Schedule Name}"
	$jobTrigger = New-AzureSqlJobTrigger -ScheduleName $scheduleName –JobName $jobName
	Write-Output $jobTrigger

### 删除计划的关联，以停止按计划执行作业

若要通过作业触发器中断作业重复执行，可以删除作业触发器。使用 **Remove-AzureSqlJobTrigger** cmdlet 删除作业触发器，以停止作业根据计划执行。

	$jobName = "{Job Name}"
	$scheduleName = "{Schedule Name}"
	Remove-AzureSqlJobTrigger -ScheduleName $scheduleName -JobName $jobName

### 检索绑定到时间计划的作业触发器

以下 PowerShell 脚本可用于获取并显示注册到特定时间计划的作业触发器。

	$scheduleName = "{Schedule Name}"
	$jobTriggers = Get-AzureSqlJobTrigger -ScheduleName $scheduleName
	Write-Output $jobTriggers

### 检索绑定到作业的作业触发器 

以下 PowerShell 脚本可用于获取并显示包含已注册作业的计划。

	$jobName = "{Job Name}"
	$jobTriggers = Get-AzureSqlJobTrigger -JobName $jobName
	Write-Output $jobTriggers

## 创建数据层应用程序部署 (DACPAC) 以跨数据库执行

弹性数据库作业可用于将数据层应用程序 (DACPAC) 部署到一组数据库。若要创建 DACPAC，请参阅此文档。对于要将 DACPAC 部署到数据库组的弹性数据库作业，DACPAC 必须能够访问该服务。建议将创建的 DACPAC 上载到 Azure 存储空间，并创建 DACPAC 的已签名 URI。

以下 PowerShell 脚本可用于将 DACPAC 插入弹性数据库作业：

	$dacpacUri = "{Uri}"
	$dacpacName = "{Dacpac Name}"
	$dacpac = New-AzureSqlJobContent -DacpacUri $dacpacUri -ContentName $dacpacName 
	Write-Output $dacpac

### 更新数据层应用程序部署 (DACPAC) 以跨数据库执行

弹性数据库作业中的现有已注册 DACPAC 可以更新为指向新的 URI。以下 PowerShell 脚本可用于更新现有已注册 DACPAC 上的 DACPAC URI：

	$dacpacName = "{Dacpac Name}"
	$newDacpacUri = "{Uri}"
	$updatedDacpac = Set-AzureSqlJobDacpacDefinition -ContentName $dacpacName -DacpacUri $newDacpacUri
	Write-Output $updatedDacpac

## 创建一个作业用于跨数据库应用数据层应用程序部署 (DACPAC)

在弹性数据库作业中创建 DACPAC 之后，可以创建一个作业用于跨数据库组应用 DACPAC。以下 PowerShell 脚本可用于创建跨自定义数据库集合的 DACPAC 作业：

	$jobName = "{Job Name}"
	$dacpacName = "{Dacpac Name}"
	$customCollectionName = "{Custom Collection Name}"
	$credentialName = "{Credential Name}"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
	$job = New-AzureSqlJob -JobName $jobName -CredentialName $credentialName -ContentName $dacpacName -TargetId $target.TargetId
	Write-Output $job 

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-powershell/cmd-prompt.png
[2]: ./media/sql-database-elastic-jobs-powershell/portal.png
<!--anchors-->

<!---HONumber=74-->