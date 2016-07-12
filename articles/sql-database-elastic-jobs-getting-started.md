<properties
	pageTitle="弹性数据库作业入门"
	description="如何使用弹性数据库作业"
	services="sql-database"
	documentationCenter=""  
	manager="jeffreyg"
	authors="sidneyh"/>

<tags
	ms.service="sql-database"
	ms.date="02/23/2016"
	wacn.date="04/06/2016" />

# 弹性数据库作业入门

Azure SQL 数据库的弹性数据库作业（预览版）可让你跨多个数据库可靠执行 T-SQL 脚本，同时自动重试并提供最终完成保证。有关弹性数据库作业功能的详细信息，请参阅[功能概述页](/documentation/articles/sql-database-elastic-jobs-overview/)。

本主题对[弹性数据库工具入门](/documentation/articles/sql-database-elastic-scale-get-started/)中的示例做了延伸。完成本主题后，你将学会如何创建和管理用于管理一组相关数据库的作业。

## 先决条件

下载并运行[弹性数据库工具示例入门](/documentation/articles/sql-database-elastic-scale-get-started/)。

## 使用示例应用程序创建分片映射管理器

在此处，你将创建分片映射管理器以及多个分片，然后将数据插入分片。如果你的分片中设置了分片数据，则你可以跳过下面的步骤，直接转到下一部分。

1. 生成并运行**弹性数据库工具入门**示例应用程序。一直执行到[下载和运行示例应用](/documentation/articles/sql-database-elastic-scale-get-started/#Getting-started-with-elastic-database-tools)部分中的步骤 7。在步骤 7 结束时，你将看到以下命令提示符：

	![命令提示符][1]

2.  在命令窗口中键入“1”，然后按 **Enter**。这会创建分片映射管理器，并将两个分片添加到服务器。然后键入“3”并按 **Enter**；重复该操作四次。这会在你的分片中插入示例数据行。

3.  [Azure 管理门户](https://manage.windowsazure.cn)应会在 v12 服务器中显示三个新的数据库：

	![Visual Studio 确认][2]

	现在，我们将创建一个自定义数据库集合，用于反映分片映射中的所有数据库。这样，我们便可以创建和执行用于跨分片添加新表的作业。

我们通常会使用 **New-AzureSqlJobTarget** cmdlet 来创建分片映射目标。必须将分片映射管理器数据库设置为数据库目标，然后将特定分片映射指定为目标。而我们的做法是枚举服务器中的所有数据库，并将这些数据库添加到 master 数据库除外的其他新自定义集合。

## 创建自定义集合并将服务器中的所有数据库添加到 master 除外的自定义集合目标。


	$customCollectionName = "dbs_in_server"
	New-AzureSqlJobTarget -CustomCollectionName $customCollectionName 
	$ResourceGroupName = "ddove_samples"
	$ServerName = "samples"
	$dbsinserver = Get-AzureRMSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName 
	$dbsinserver | %{
    $currentdb = $_.DatabaseName 
    $ErrorActionPreference = "Stop"
    Write-Output ""

    Try
    {
       New-AzureSqlJobTarget -ServerName $ServerName -DatabaseName $currentdb | Write-Output
    }
    Catch 
    {
        $ErrorMessage = $_.Exception.Message
        $ErrorCategory = $_.CategoryInfo.Reason
                
        if ($ErrorCategory -eq 'UniqueConstraintViolatedException')
        {
             Write-Host $currentdb "is already a database target." 
        }

        else
        {
            throw $_
        }
    
    }

    Try
    {
        if ($currentdb -eq "master")
        {
            Write-Host $currentdb "will not be added custom collection target" $CustomCollectionName "."
        }

        else
        {
            Add-AzureSqlJobChildTarget -CustomCollectionName $CustomCollectionName -ServerName $ServerName -DatabaseName $currentdb
            Write-Host $currentdb "was added to" $CustomCollectionName "."
        }

    }
    Catch 
    {
        $ErrorMessage = $_.Exception.Message
        $ErrorCategory = $_.CategoryInfo.Reason

        if ($ErrorCategory -eq 'UniqueConstraintViolatedException')
        {
             Write-Host $currentdb "is already in the custom collection target" $CustomCollectionName"."
        }

        else
        {
            throw $_
        }
    }
    $ErrorActionPreference = "Continue"
	}
	
## 创建 T-SQL 脚本用于跨数据库执行

	$scriptName = "NewTable"
	$scriptCommandText = "
	IF NOT EXISTS (SELECT name FROM sys.tables WHERE name = 'Test')
	BEGIN
		CREATE TABLE Test(
			TestId INT PRIMARY KEY IDENTITY,
			InsertionTime DATETIME2
		);
	END
	GO
	INSERT INTO Test(InsertionTime) VALUES (sysutcdatetime());
	GO"

	$script = New-AzureSqlJobContent -ContentName $scriptName -CommandText $scriptCommandText
	Write-Output $script

##创建作业以跨自定义数据库组执行脚本

	$jobName = "create on server dbs"
	$scriptName = "NewTable"
	$customCollectionName = "dbs_in_server"
	$credentialName = "ddove66"
	$target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
	$job = New-AzureSqlJob -JobName $jobName -CredentialName $credentialName -ContentName $scriptName -TargetId $target.TargetId
	Write-Output $job


##执行作业 

以下 PowerShell 脚本可以用于执行现有的作业：

更新以下变量以反映要执行的所需作业名称：

	$jobName = "create on server dbs"
	$jobExecution = Start-AzureSqlJobExecution -JobName $jobName 
	Write-Output $jobExecution
 
## 检索单个作业执行的状态

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

弹性数据库作业支持作业取消请求。如果弹性数据库作业检测到当前正在执行作业的取消请求，它将尝试停止作业。

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

以下 PowerShell 脚本可用于创建重复计划。此脚本使用一分钟间隔，但是 New-AzureSqlJobSchedule 也支持 -DayInterval、-HourInterval、-MonthInterval 和 -WeekInterval 参数。可以通过传递 -OneTime 来创建仅执行一次的计划。

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





## 将弹性数据库查询结果导入 Excel

 你可以将查询结果导入到 Excel 文件。

1. 启动 Excel 2013。
2. 	导航到“数据”功能区。
3. 	单击“从其他源”，然后单击“从 SQL Server”。

	![从其他源导入 Excel][5]
4. 	在“数据连接向导”中，键入服务器名称和登录凭据。然后，单击“下一步”。
5. 	在对话框“选择包含所需数据的数据库”中，选择 **ElasticDBQuery** 数据库。
6. 	在列表视图中选择“客户”表并单击“下一步”。然后单击“完成”。
7. 	在“导入数据”窗体中的“请选择该数据在工作簿中的显示方式”下，选择“表”，然后单击“确定”。

存储在不同分片中、来自“客户”表的所有行将填入 Excel 工作表。

## 后续步骤
现在，你可以使用 Excel 的数据功能。使用包含服务器名称、数据库名称和凭据的连接字符串，将 BI 和数据集成工具连接到弹性查询数据库。请确保支持将 SQL Server 用作工具的数据源。参考弹性查询数据库和外部表，就如同将要与你的工具连接的任何其他 SQL Server 数据库和 SQL Server 表。

### 成本
使用弹性数据库查询功能不会产生额外的费用。但是，目前此功能只能在用作终结点的高级数据库上使用，但分片可以是任何服务层。

有关价格信息，请参阅 [SQL 数据库定价详细信息](/home/features/sql-database/pricing/)。


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-query-getting-started/cmd-prompt.png
[2]: ./media/sql-database-elastic-query-getting-started/portal.png
[3]: ./media/sql-database-elastic-query-getting-started/tiers.png
[4]: ./media/sql-database-elastic-query-getting-started/details.png
[5]: ./media/sql-database-elastic-query-getting-started/exel-sources.png
<!--anchors-->

<!---HONumber=Mooncake_0328_2016-->
