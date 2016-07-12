<properties 
	pageTitle="使用 PowerShell 创建和管理弹性数据库作业" 
	description="使用 PowerShell 管理 Azure SQL 数据库池" 
	services="sql-database" documentationCenter=""  
	manager="jhubbard" 
	authors="ddove"/>

<tags 
	ms.service="sql-database" 
	ms.date="05/03/2016" 
	wacn.date="06/14/2016" />

# 使用 PowerShell 创建和管理 SQL 数据库弹性数据库作业（预览版）

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-elastic-jobs-powershell/)



使用适用于**弹性数据库作业**的 PowerShell API（预览版），可以定义要针对哪组数据库执行哪些脚本。本文说明如何使用 PowerShell cmdlet 创建和管理**弹性数据库作业**。请参阅[弹性作业概述](/documentation/articles/sql-database-elastic-jobs-overview/)。

## 先决条件
* Azure 订阅。若要获取试用版，请参阅[试用](/pricing/1rmb-trial)。
* 使用弹性数据库工具创建的一组数据库。请参阅[弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/)。
* Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。
* **弹性数据库作业** PowerShell 包：请参阅[安装弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-service-installation/)

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
该作业可以跨数据库组执行 Transact-SQL (T-SQL) 脚本或应用 DACPAC。提交需要跨数据库组执行的作业时，该作业将“扩展”到子作业中，其中的每个作业会针对组中的单一数据库执行请求的执行操作。
 
有两种可以创建的组类型：

* [分片映射](/documentation/articles/sql-database-elastic-scale-shard-map-management/)组：提交以分片映射为目标的作业时，该作业会通过查询分片映射来确定其当前的分片集，然后为分片映射中的每个分片创建子作业。
* 自定义集合组：一种自定义数据库集。当作业以自定义集合为目标时，它会针对当前位于自定义集合中的每个数据库创建子作业。

## 设置弹性数据库作业连接

在使用作业 API 之前，需设置作业控制数据库连接。如果运行此 cmdlet，则会在安装弹性数据库作业时触发一个弹出式凭据窗口，请求用户提供已创建的用户名/密码。本主题中提供的所有示例都假设已经执行第一个步骤。

建立与弹性数据库作业的连接：

	Use-AzureSqlJobConnection -CurrentAzureSubscription 

## 弹性数据库作业中的已加密凭据

数据库凭据可以插入密码已加密的作业控制数据库。必须存储凭据，使作业可以在稍后执行（使用作业计划）。
 
加密是通过创建为安装脚本一部分的证书来进行的。安装脚本创建证书并将其上载到 Azure 云服务，以解密已存储的加密密码。Azure 云服务稍后在作业控制数据库内存储公钥，让 PowerShell API 或 Azure 管理门户界面加密提供的密码，而不需要在本地安装证书。
 
凭据密码经过加密后，无法供只能通过只读方式访问弹性数据库作业对象的用户访问。不过，具有弹性数据库作业对象读-写访问权限的恶意用户可能会提取密码。凭据设计为跨作业执行重复使用。在建立连接时，凭据将传递到目标数据库。适用于每个凭据的目标数据库目前并没有受到限制，因此恶意用户可以针对自己所控制的数据库来添加数据库目标。然后，该用户可以启动一个针对该数据库的作业，从而获取凭据的密码。

弹性数据库作业安全方面的最佳实践包括：

* 将 API 的使用限制为受信任的个人。
* 凭据应该具有执行作业任务所需的最低权限。有关详细信息，请参阅这篇[授权和权限](https://msdn.microsoft.com/zh-cn/library/bb669084.aspx) SQL Server MSDN 文章。

### 跨数据库创建执行作业所需的加密凭据

创建新的加密凭据时，[**Get-Credential cmdlet**](https://technet.microsoft.com/zh-cn/library/hh849815.aspx) 会提示用户输入可以传递给 [**New-AzureSqlJobCredential cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346063.aspx) 的用户名和密码。

	$credentialName = "{Credential Name}"
	$databaseCredential = Get-Credential
	$credential = New-AzureSqlJobCredential -Credential $databaseCredential -CredentialName $credentialName
	Write-Output $credential

### 更新凭据

当密码更改时，请使用 [**Set-AzureSqlJobCredential cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346062.aspx) 并设置 **CredentialName** 参数。

	$credentialName = "{Credential Name}"
	Set-AzureSqlJobCredential -CredentialName $credentialName -Credential $credential 

## 定义弹性数据库分片映射目标

若要针对分片集（使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)创建）中的所有数据库执行作业，请将分片映射用作数据库目标。本示例要求使用弹性数据库客户端库创建分片应用程序。请参阅[弹性数据库工具示例入门](/documentation/articles/sql-database-elastic-scale-get-started/)。

###使用示例应用程序创建分片映射管理器

本示例将创建分片映射管理器以及多个分片，然后将数据插入分片。

1. 生成并运行**弹性数据库工具入门**示例应用程序。一直执行到[下载和运行示例应用](/documentation/articles/sql-database-elastic-scale-get-started/#Getting-started-with-elastic-database-tools)部分中的步骤 7。在步骤 7 结束时，你将看到以下命令提示符：

	![命令提示符][1]

2.  在命令窗口中键入“1”，然后按 **Enter**。这会创建分片映射管理器，并将两个分片添加到服务器。然后键入“3”并按 **Enter**；重复该操作四次。这会在你的分片中插入示例数据行。
  
3.  [Azure 管理门户](https://manage.windowsazure.cn)应会在 v12 服务器中显示三个新的数据库：

	![Visual Studio 确认][2]

使用 [**New-AzureSqlJobCredential cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346063.aspx) 创建分片映射目标。必须将分片映射管理器数据库设置为数据库目标，然后将特定分片映射指定为目标。

	$shardMapCredentialName = "{Credential Name}"
	$shardMapDatabaseName = "{ShardMapDatabaseName}" #example: ElasticScaleStarterKit_ShardMapManagerDb
	$shardMapDatabaseServerName = "{ShardMapServerName}"
	$shardMapName = "{MyShardMap}" #example: CustomerIDShardMap
	$shardMapDatabaseTarget = New-AzureSqlJobTarget -DatabaseName $shardMapDatabaseName -ServerName $shardMapDatabaseServerName
	$shardMapTarget = New-AzureSqlJobTarget -ShardMapManagerCredentialName $shardMapCredentialName -ShardMapManagerDatabaseName $shardMapDatabaseName -ShardMapManagerServerName $shardMapDatabaseServerName -ShardMapName $shardMapName
	Write-Output $shardMapTarget

## 创建 T-SQL 脚本用于跨数据库执行

创建要执行的 T-SQL 脚本时，强烈建议将其构建为[幂等](https://en.wikipedia.org/wiki/Idempotence)模式，以便灵活地应对各种故障。每当执行发生失败时，不论失败的分类，弹性数据库作业将重试执行脚本。

使用 [**New-AzureSqlJobContent cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346085.aspx) 创建并保存执行脚本，然后设置 **-ContentName** 和 **-CommandText** 参数。

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
如果 T-SQL 脚本是在文件中定义的，请使用以下语句来导入脚本：

	$scriptName = "My Script Imported from a File"
	$scriptPath = "{Path to SQL File}"
	$scriptCommandText = Get-Content -Path $scriptPath
	$script = New-AzureSqlJobContent -ContentName $scriptName -CommandText $scriptCommandText
	Write-Output $script

### 更新可以跨数据库执行的 T-SQL 脚本  

此 PowerShell 脚本更新现有脚本的 T-SQL 命令文本。

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

此 PowerShell 脚本可以启动一个作业，以便跨弹性缩放分片映射中的每个分片来执行脚本。

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

此 PowerShell 脚本执行现有作业：

更新以下变量以反映要执行的所需作业名称：

	$jobName = "{Job Name}"
	$jobExecution = Start-AzureSqlJobExecution -JobName $jobName 
	Write-Output $jobExecution
 
## 检索单个作业的执行状态

使用 [**Get-AzureSqlJobExecution cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346058.aspx) 并设置 **JobExecutionId** 参数以查看作业的执行状态。

	$jobExecutionId = "{Job Execution Id}"
	$jobExecution = Get-AzureSqlJobExecution -JobExecutionId $jobExecutionId
	Write-Output $jobExecution

使用相同 **Get-AzureSqlJobExecution** cmdlet 搭配 **IncludeChildren** 参数，以查看子作业执行的状态，还就是针对作为作业目标的每个数据库的每个作业执行的特定状态。

	$jobExecutionId = "{Job Execution Id}"
	$jobExecutions = Get-AzureSqlJobExecution -JobExecutionId $jobExecutionId -IncludeChildren
	Write-Output $jobExecutions 

## 查看多次执行作业时的状态

[**Get-AzureSqlJobExecution cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346058.aspx) 具有多个可选参数，用于显示多次作业执行情况（已通过提供的参数进行筛选）。下面演示了 Get-AzureSqlJobExecution 的一些可能的用法：

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

## 检索执行作业任务过程中出现的失败

**JobTaskExecution 对象**包括任务生命周期的属性以及消息属性。如果作业任务执行失败，生命周期属性将设为 Failed，且消息属性将设为生成的异常消息及其堆栈。如果作业不成功，必须查看给定操作不成功的作业任务的详细信息。

	$jobExecutionId = "{Job Execution Id}"
	$jobTaskExecutions = Get-AzureSqlJobTaskExecution -JobExecutionId $jobExecutionId
	Foreach($jobTaskExecution in $jobTaskExecutions) 
		{
		if($jobTaskExecution.Lifecycle -ne 'Succeeded')
    		{
        	Write-Output $jobTaskExecution
    		}
		}

## 等待作业执行完毕

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
	$executionPolicy = New-AzureSqlJobExecutionPolicy -ExecutionPolicyName $executionPolicyName -InitialRetryInterval $initialRetryInterval -JobTimeout $jobTimeout -MaximumAttempts $maximumAttempts -MaximumRetryInterval $maximumRetryInterval 
	-RetryIntervalBackoffCoefficient $retryIntervalBackoffCoefficient
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
 
若要提交取消请求，请使用 [**Stop-AzureSqlJobExecution cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346053.aspx) 并设置 **JobExecutionId** 参数。

	$jobExecutionId = "{Job Execution Id}"
	Stop-AzureSqlJobExecution -JobExecutionId $jobExecutionId

## 以异步方式删除作业和作业历史记录

弹性数据库作业支持异步删除作业。可将某个作业标记为待删除，系统将在作业的作业执行都已完成后，删除该作业及其所有作业历史记录。系统不会自动取消处于活动状态的作业执行。

调用 [**Stop-AzureSqlJobExecution**](https://msdn.microsoft.com/zh-cn/library/mt346053.aspx) 即可取消正在执行的作业。

若要触发作业删除操作，请使用 [**Remove-AzureSqlJob cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346083.aspx) 并设置 **JobName** 参数。

	$jobName = "{Job Name}"
	Remove-AzureSqlJob -JobName $jobName
 
## 创建自定义数据库目标

你可以定义各种可以直接执行或包括到自定义数据库组中的自定义数据库目标。例如，由于**弹性数据库池**尚不能通过 PowerShell API 直接使用，你可以创建自定义数据库目标和自定义数据库集合目标，其中包含池中的所有数据库。

设置以下变量以反映所需的数据库信息：

	$databaseName = "{Database Name}"
	$databaseServerName = "{Server Name}"
	New-AzureSqlJobDatabaseTarget -DatabaseName $databaseName -ServerName $databaseServerName 

## 创建自定义数据库集合目标

使用 [**New-AzureSqlJobTarget**](https://msdn.microsoft.com/zh-cn/library/mt346077.aspx) 可以自定义数据库集合目标，从而让执行操作跨多个已定义数据库目标。创建数据库组之后，数据库可与自定义集合目标相关联。

设置以下变量以反映所需的自定义集合目标配置：

	$customCollectionName = "{Custom Database Collection Name}"
	New-AzureSqlJobTarget -CustomCollectionName $customCollectionName 

### 将数据库添加到自定义数据库集合目标

若要将数据库添加到特定的自定义集合，请使用 **Add-AzureSqlJobChildTarget** cmdlet。

	$serverName = "{Database Server Name}"
	$databaseName = "{Database Name}"
	$customCollectionName = "{Custom Database Collection Name}"
	Add-AzureSqlJobChildTarget -CustomCollectionName $customCollectionName -DatabaseName $databaseName -ServerName $databaseServerName 

#### 查看自定义数据库集合目标内的数据库

使用 [**Get-AzureSqlJobTarget**](https://msdn.microsoft.com/zh-cn/library/mt346077.aspx) cmdlet 检索自定义数据库集合目标内的子数据库。
 
	$customCollectionName = "{Custom Database Collection Name}"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
	$childTargets = Get-AzureSqlJobTarget -ParentTargetId $target.TargetId
	Write-Output $childTargets

### 创建作业以跨自定义数据库集合目标执行脚本

使用 [**New-AzureSqlJob**](https://msdn.microsoft.com/zh-cn/library/mt346078.aspx) cmdlet 可以针对自定义数据库集合目标定义的数据库组创建作业。弹性数据库作业将作业扩展为多个子作业（每个子作业映射到与自定义数据库集合目标关联的数据库），并确保脚本针对每个数据库执行。同样，重要的是脚本具有幂等处理重试的弹性。

	$jobName = "{Job Name}"
	$scriptName = "{Script Name}"
	$customCollectionName = "{Custom Collection Name}"
	$credentialName = "{Credential Name}"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
	$job = New-AzureSqlJob -JobName $jobName -CredentialName $credentialName -ContentName $scriptName -TargetId $target.TargetId
	Write-Output $job

## 跨数据库收集数据

你可以使用作业跨一组数据库来执行查询，然后将结果发送到特定的表。可以在事实之后查询数据表，以查看每个数据库的查询结果。这提供了跨多个数据库执行查询的异步方法。可通过重试自动处理失败的尝试。

如果不存在指定的目标表，则自动创建该表。新表与返回的结果集的架构相符。如果脚本返回多个结果集，弹性数据库作业只将第一个结果集发送到目标表。

以下 PowerShell 脚本可用于执行脚本，将其结果收集到指定的表。此脚本假设已创建一个可输出单个结果集的 T-SQL 脚本，并且已创建自定义的数据库集合目标。

此脚本使用 [**Get-AzureSqlJobTarget**](https://msdn.microsoft.com/zh-cn/library/mt346077.aspx) cmdlet。设置脚本、凭据和执行目标的参数：

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

### 创建和启动用于数据收集方案的作业

此脚本使用 [**Start-AzureSqlJobExecution**](https://msdn.microsoft.com/zh-cn/library/mt346055.aspx) cmdlet。
 
	$job = New-AzureSqlJob -JobName $jobName 
	-CredentialName $executionCredentialName 
	-ContentName $scriptName 
	-ResultSetDestinationServerName $destinationServerName 
	-ResultSetDestinationDatabaseName $destinationDatabaseName 
	-ResultSetDestinationSchemaName $destinationSchemaName 
	-ResultSetDestinationTableName $destinationTableName 
	-ResultSetDestinationCredentialName $destinationCredentialName 
	-TargetId $target.TargetId
	Write-Output $job
	$jobExecution = Start-AzureSqlJobExecution -JobName $jobName
	Write-Output $jobExecution

## 计划作业执行触发器

以下 PowerShell 脚本可用于创建重复计划。此脚本使用分钟间隔，但是 [**New-AzureSqlJobSchedule**](https://msdn.microsoft.com/zh-cn/library/mt346068.aspx) 也支持 -DayInterval、-HourInterval、-MonthInterval 和 -WeekInterval 参数。可以通过传递 -OneTime 来创建仅执行一次的计划。

创建新计划：

	$scheduleName = "Every one minute"
	$minuteInterval = 1
	$startTime = (Get-Date).ToUniversalTime()
	$schedule = New-AzureSqlJobSchedule 
    -MinuteInterval $minuteInterval 
	-ScheduleName $scheduleName 
	-StartTime $startTime 
	Write-Output $schedule

### 触发按时间计划执行的作业

可以定义作业触发器，使作业根据时间计划执行。以下 PowerShell 脚本可用于创建作业触发器。

使用 [New-AzureSqlJobTrigger](https://msdn.microsoft.com/zh-cn/library/mt346069.aspx)，根据所需作业和计划设置以下变量：

	$jobName = "{Job Name}"
	$scheduleName = "{Schedule Name}"
	$jobTrigger = New-AzureSqlJobTrigger
	-ScheduleName $scheduleName
	–JobName $jobName
	Write-Output $jobTrigger

### 删除计划的关联，以停止按计划执行作业

若要通过作业触发器中断作业重复执行，可以删除作业触发器。使用 [**Remove-AzureSqlJobTrigger cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346070.aspx) 删除作业触发器，以停止按计划执行作业。

	$jobName = "{Job Name}"
	$scheduleName = "{Schedule Name}"
	Remove-AzureSqlJobTrigger 
	-ScheduleName $scheduleName 
	-JobName $jobName

### 检索绑定到时间计划的作业触发器

以下 PowerShell 脚本可用于获取并显示注册到特定时间计划的作业触发器。

	$scheduleName = "{Schedule Name}"
	$jobTriggers = Get-AzureSqlJobTrigger -ScheduleName $scheduleName
	Write-Output $jobTriggers

### 检索绑定到作业的作业触发器 

使用 [Get-AzureSqlJobTrigger](https://msdn.microsoft.com/zh-cn/library/mt346067.aspx) 来获取并显示包含已注册作业的计划。

	$jobName = "{Job Name}"
	$jobTriggers = Get-AzureSqlJobTrigger -JobName $jobName
	Write-Output $jobTriggers

## 创建可以跨数据库执行的数据层应用程序 (DACPAC)

若要创建 DACPAC，请参阅[数据层应用程序](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx)。若要部署 DACPAC，请使用 [New-AzureSqlJobContent cmdlet](https://msdn.microsoft.com/zh-cn/library/mt346085.aspx)。DACPAC 必须可供服务访问。建议将创建的 DACPAC 上载到 Azure 存储空间，并创建 DACPAC 的[共享访问签名](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。

	$dacpacUri = "{Uri}"
	$dacpacName = "{Dacpac Name}"
	$dacpac = New-AzureSqlJobContent -DacpacUri $dacpacUri -ContentName $dacpacName 
	Write-Output $dacpac

### 更新可以跨数据库执行的数据层应用程序 (DACPAC)

弹性数据库作业中的现有已注册 DACPAC 可以更新为指向新的 URI。使用 [**Set-AzureSqlJobContentDefinition cmdlet**](https://msdn.microsoft.com/zh-cn/library/mt346074.aspx) 在现有的已注册 DACPAC 基础上更新 DACPAC URI：

	$dacpacName = "{Dacpac Name}"
	$newDacpacUri = "{Uri}"
	$updatedDacpac = Set-AzureSqlJobDacpacDefinition -ContentName $dacpacName -DacpacUri $newDacpacUri
	Write-Output $updatedDacpac

## 创建一个作业，以便跨数据库应用数据层应用程序 (DACPAC)

在弹性数据库作业中创建 DACPAC 之后，可以创建一个作业用于跨数据库组应用 DACPAC。以下 PowerShell 脚本可用于创建跨自定义数据库集合的 DACPAC 作业：

	$jobName = "{Job Name}"
	$dacpacName = "{Dacpac Name}"
	$customCollectionName = "{Custom Collection Name}"
	$credentialName = "{Credential Name}"
	$target = Get-AzureSqlJobTarget 
	-CustomCollectionName $customCollectionName
	$job = New-AzureSqlJob 
	-JobName $jobName 
	-CredentialName $credentialName 
	-ContentName $dacpacName -TargetId $target.TargetId
	Write-Output $job 

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-powershell/cmd-prompt.png
[2]: ./media/sql-database-elastic-jobs-powershell/portal.png
<!--anchors-->

<!---HONumber=Mooncake_0530_2016-->
