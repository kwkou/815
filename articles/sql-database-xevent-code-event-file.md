<properties 
	pageTitle="SQL 数据库的 XEvent 事件文件代码 | Azure" 
	description="提供一个双阶段代码示例的 PowerShell 和 Transact-SQL，该示例演示 Azure SQL 数据库的扩展事件中的事件文件目标。完成此方案部分必须用到 Azure 存储空间。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor="" 
	tags=""/>


<tags 
	ms.service="sql-database" 
	ms.date="04/28/2016" 
	wacn.date="05/23/2016"/>


# SQL 数据库中扩展事件的事件文件目标代码


你需要一个完整的代码示例来可靠捕获和报告扩展事件的信息。


在 Microsoft SQL Server 中，[事件文件目标](http://msdn.microsoft.com/zh-cn/library/ff878115.aspx)用于将事件输出存储在本地硬盘驱动器文件中。但是，此类文件并不适用于 Azure SQL 数据库。我们改为使用 Azure 存储空间服务来支持事件文件目标。


本主题演示了一个两阶段代码示例：


- PowerShell：用于在云中创建 Azure 存储空间容器。

- Transact-SQL：
 - 将 Azure 存储空间容器分配到事件文件目标。
 - 创建和启动事件会话，等等。


## 先决条件


- Azure 帐户和订阅。你可以注册[试用版](/pricing/1rmb-trial)。


- 可以在其中创建表的任何数据库。
 - 你可以选择快速[创建一个 **AdventureWorksLT** 演示数据库](/documentation/articles/sql-database-get-started/)。


- SQL Server Management Studio (ssms.exe) 2015 年 8 月预览版或更高版本。可从以下位置下载最新的 ssms.exe：
 - 标题为[下载 SQL Server Management Studio](http://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 的主题。
 - [直接指向下载位置的链接。](http://go.microsoft.com/fwlink/?linkid=616025)
 - Azure 建议定期更新 ssms.exe。在某些情况下，ssms.exe 将每个月更新。


- 你必须安装 [Azure PowerShell 模块](http://go.microsoft.com/?linkid=9811175)。
 - 这些模块提供 **New-AzureStorageAccount** 等命令。


## 阶段 1：Azure 存储空间容器的 PowerShell 代码


此 PowerShell 是两阶段代码示例的第 1 阶段。

脚本开头的命令将清除以前可能运行后的结果，并且设计为可重复运行。



1. 将 PowerShell 脚本粘贴到 Notepad.exe 等简单文本编辑器，并将脚本保存到扩展名为 **.ps1** 的文件。

2. 以管理员身份启动 PowerShell ISE。

3. 在提示符下键入 <br/>`Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser`<br/>，然后按 Enter。

4. 在 PowerShell ISE 中打开你的 **.ps1** 文件。运行该脚本。

5. 该脚本会先启动新的窗口让你登录 Azure。
 - 如果你想要重复运行脚本而不中断会话，可以很方便地选择注释掉 **Add-AzureAccount** 命令。


		![在准备运行脚本之前，必须准备好已装有 Azure 模块的 PowerShell ISE。][30_powershell_ise]


			## TODO: Before running, find all 'TODO' and make each edit!
			
			#--------------- 1 -----------------------
			
			
			# You can comment out or skip this Add-AzureAccount command after the first run.
			# Current PowerShell environment retains the successful outcome.
			
			'Expect a pop-up window in which you log in to Azure.'
			
			
			Add-AzureRmAccount -EnvironmentName AzureChinaCloud
			
			#-------------- 2 ------------------------
			
			
			'
			TODO: Edit the values assigned to these variables, especially the first few!
			'
			
			$subscriptionName       = 'YOUR_SUBSCRIPTION_NAME'
			$policySasExpiryTime = '2016-01-28T23:44:56Z'
			$policySasStartTime  = '2015-08-01'
			
			
			$storageAccountName     = 'gmstorageaccountxevent'
			$storageAccountLocation = 'China North'
			$contextName            = 'gmcontext'
			$containerName          = 'gmcontainerxevent'
			$policySasToken      = 'gmpolicysastoken'
			
			
			# Leave this value alone, as 'rwl'.
			$policySasPermission = 'rwl'
			
			#--------------- 3 -----------------------
			
			
			# The ending display lists your Azure subscriptions.
			# One should match the $subscriptionName value you assigned
			#   earlier in this PowerShell script. 
			
			'Choose an existing subscription for the current PowerShell environment.'
			
			
			Select-AzureSubscription -SubscriptionName $subscriptionName
			
			
			#-------------- 4 ------------------------
			
			
			'
			Clean-up the old Azure Storage Account after any previous run, 
			before continuing this new run.'
			
			
			If ($storageAccountName)
			{
			    Remove-AzureStorageAccount -StorageAccountName $storageAccountName
			}
			
			#--------------- 5 -----------------------
			
			[System.DateTime]::Now.ToString()
			
			'
			Create a storage account. 
			This might take several minutes, will beep when ready.
			  ...PLEASE WAIT...'
			
			New-AzureStorageAccount `
			    -StorageAccountName $storageAccountName `
			    -Location           $storageAccountLocation
			
			[System.DateTime]::Now.ToString()
			
			[System.Media.SystemSounds]::Beep.Play()
			
			
			'
			Get the primary access key for your storage account.
			'
			
			
			$primaryAccessKey_ForStorageAccount = `
			    (Get-AzureStorageKey `
			        -StorageAccountName $storageAccountName).Primary
			
			"`$primaryAccessKey_ForStorageAccount = $primaryAccessKey_ForStorageAccount"
			
			'Azure Storage Account cmdlet completed.
			Remainder of PowerShell .ps1 script continues.
			'
			
			#--------------- 6 -----------------------
			
			
			# The context will be needed to create a container within the storage account.
			
			'Create a context object from the storage account and its primary access key.
			'
			
			$context = New-AzureStorageContext `
			    -StorageAccountName $storageAccountName `
			    -StorageAccountKey  $primaryAccessKey_ForStorageAccount
			
			
			'Create a container within the storage account.
			'
			
			
			$containerObjectInStorageAccount = New-AzureStorageContainer `
			    -Name    $containerName `
			    -Context $context
			
			
			'Create a security policy to be applied to the SAS token.
			'
			
			New-AzureStorageContainerStoredAccessPolicy `
			    -Container  $containerName `
			    -Context    $context `
			    -Policy     $policySasToken `
			    -Permission $policySasPermission `
			    -ExpiryTime $policySasExpiryTime `
			    -StartTime  $policySasStartTime 
			
			'
			Generate a SAS token for the container.
			'
			Try
			{
			    $sasTokenWithPolicy = New-AzureStorageContainerSASToken `
			        -Name    $containerName `
			        -Context $context `
			        -Policy  $policySasToken
			}
			Catch 
			{
			    $Error[0].Exception.ToString()
			}
			
			#-------------- 7 ------------------------
			
			
			'Display the values that YOU must edit into the Transact-SQL script next!:
			'
			
			"storageAccountName: $storageAccountName"
			"containerName:      $containerName"
			"sasTokenWithPolicy: $sasTokenWithPolicy"
			
			'
			REMINDER: sasTokenWithPolicy here might start with "?" character, which you must exclude from Transact-SQL.
			'
			
			'
			(Later, return here to delete your Azure Storage account. See the preceding - Remove-AzureStorageAccount -StorageAccountName $storageAccountName)'
			
			'
			Now shift to the Transact-SQL portion of the two-part code sample!'
			
			# EOFile






		记下 PowerShell 脚本结束时输出的几个命名值。必须将这些值编辑成阶段 2 中使用的 Transact-SQL 脚本。


## 阶段 2：使用 Azure 存储空间容器的 Transact-SQL 代码


- 在此代码示例的第 1 阶段，你已运行 PowerShell 脚本来创建 Azure 存储空间容器。
- 接下来在第 2 阶段，以下 Transact-SQL 脚本必须使用该容器。


脚本开头的命令将清除以前可能运行后的结果，并且设计为可重复运行。


PowerShell 脚本在结束时输出了几个命名值。你必须编辑 Transact-SQL 脚本以使用这些值。在 Transact-SQL 脚本中查找 **TODO** 以找到编辑点。


1. 打开 SQL Server Management Studio (ssms.exe)。

2. 连接到你的 Azure SQL 数据库。

3. 单击打开新的查询窗格。

4. 将以下 Transact-SQL 脚本粘贴到查询窗格中。

5. 在脚本中查找每个 **TODO** 并进行适当的编辑。

6. 保存然后运行该脚本。


		---- TODO: First, run the PowerShell portion of this two-part code sample.
		---- TODO: Second, find every 'TODO' in this Transact-SQL file, and edit each.
		
		---- Transact-SQL code for Event File target on Azure SQL Database.
		
		
		SET NOCOUNT ON;
		
		GO
		
		
		----  Step 1.  Establish one little table, and  ---------
		----  insert one row of data.
		
		
		IF EXISTS
			(SELECT * FROM sys.objects
				WHERE type = 'U' and name = 'gmTabEmployee')
		BEGIN
			DROP TABLE gmTabEmployee;
		END
		GO
		
		
		CREATE TABLE gmTabEmployee
		(
			EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
			EmployeeId           int                not null  identity(1,1),
			EmployeeKudosCount   int                not null  default 0,
			EmployeeDescr        nvarchar(256)          null
		);
		GO
		
		
		INSERT INTO gmTabEmployee ( EmployeeDescr )
			VALUES ( 'Jane Doe' );
		GO
		
		
		------  Step 2.  Create key, and  ------------
		------  Create credential (your Azure Storage container must already exist).
		
		
		IF NOT EXISTS
			(SELECT * FROM sys.symmetric_keys
				WHERE symmetric_key_id = 101)
		BEGIN
			CREATE MASTER KEY ENCRYPTION BY PASSWORD = '0C34C960-6621-4682-A123-C7EA08E3FC46' -- Or any newid().
		END
		GO
		
		
		IF EXISTS
			(SELECT * FROM sys.database_scoped_credentials
				-- TODO: Assign AzureStorageAccount name, and the associated Container name.
				WHERE name = 'https://gmstorageaccountxevent.blob.core.chinacloudapi.cn/gmcontainerxevent')
		BEGIN
			DROP DATABASE SCOPED CREDENTIAL
				-- TODO: Assign AzureStorageAccount name, and the associated Container name.
				[https://gmstorageaccountxevent.blob.core.chinacloudapi.cn/gmcontainerxevent] ;
		END
		GO
		
		
		CREATE
			DATABASE SCOPED
			CREDENTIAL
				-- use '.blob.',   and not '.queue.' or '.table.' etc.
				-- TODO: Assign AzureStorageAccount name, and the associated Container name.
				[https://gmstorageaccountxevent.blob.core.chinacloudapi.cn/gmcontainerxevent]
			WITH
				IDENTITY = 'SHARED ACCESS SIGNATURE',  -- "SAS" token.
				-- TODO: Paste in the long SasToken string here for Secret, but exclude any leading '?'.
				SECRET = 'sv=2014-02-14&sr=c&si=gmpolicysastoken&sig=EjAqjo6Nu5xMLEZEkMkLbeF7TD9v1J8DNB2t8gOKTts%3D'
			;
		GO
		
		
		------  Step 3.  Create (define) an event session.  --------
		------  The event session has an event with an action,
		------  and a has a target.
		
		IF EXISTS
			(SELECT * from sys.database_event_sessions
				WHERE name = 'gmeventsessionname240b')
		BEGIN
			DROP
				EVENT SESSION
					gmeventsessionname240b
			    ON DATABASE;
		END
		GO
		
		
		CREATE
			EVENT SESSION
				gmeventsessionname240b
			ON DATABASE
		
			ADD EVENT
				sqlserver.sql_statement_starting
					(
					ACTION (sqlserver.sql_text)
					WHERE statement LIKE 'UPDATE gmTabEmployee%'
					)
			ADD TARGET
				package0.event_file
					(
					-- TODO: Assign AzureStorageAccount name, and the associated Container name.
					-- Also, tweak the .xel file name at end, if you like.
					SET filename =
						'https://gmstorageaccountxevent.blob.core.chinacloudapi.cn/gmcontainerxevent/anyfilenamexel242b.xel'
					)
			WITH
				(MAX_MEMORY = 10 MB,
				MAX_DISPATCH_LATENCY = 3 SECONDS)
			;
		GO
		
		
		------  Step 4.  Start the event session.  ----------------
		------  Issue the SQL Update statements that will be traced.
		------  Then stop the session.
		
		------  Note: If the target fails to attach,
		------  the session must be stopped and restarted.
		
		ALTER
			EVENT SESSION
				gmeventsessionname240b
			ON DATABASE
			STATE = START;
		GO
		
		
		SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;
		
		UPDATE gmTabEmployee
			SET EmployeeKudosCount = EmployeeKudosCount + 2
			WHERE EmployeeDescr = 'Jane Doe';
		
		UPDATE gmTabEmployee
			SET EmployeeKudosCount = EmployeeKudosCount + 13
			WHERE EmployeeDescr = 'Jane Doe';
		
		SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
		GO
		
		
		ALTER
			EVENT SESSION
				gmeventsessionname240b
			ON DATABASE
			STATE = STOP;
		GO
		
		
		-------------- Step 5.  Select the results. ----------
		
		SELECT
				*, 'CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS!' as [CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS],
				CAST(event_data AS XML) AS [event_data_XML]  -- TODO: In ssms.exe results grid, double-click this cell!
			FROM
				sys.fn_xe_file_target_read_file
					(
						-- TODO: Fill in Storage Account name, and the associated Container name.
						'https://gmstorageaccountxevent.blob.core.chinacloudapi.cn/gmcontainerxevent/anyfilenamexel242b',
						null, null, null
					);
		GO
		
		
		-------------- Step 6.  Clean up. ----------
		
		DROP
			EVENT SESSION
				gmeventsessionname240b
			ON DATABASE;
		GO
		
		DROP DATABASE SCOPED CREDENTIAL
			-- TODO: Assign AzureStorageAccount name, and the associated Container name.
			[https://gmstorageaccountxevent.blob.core.chinacloudapi.cn/gmcontainerxevent]
			;
		GO
		
		DROP TABLE gmTabEmployee;
		GO
		
		PRINT 'Use PowerShell Remove-AzureStorageAccount to delete your Azure Storage account!';
		GO



&nbsp;


如果当你运行脚本时无法附加目标，则你必须停止再重新启动事件会话：


	 
	ALTER EVENT SESSION ... STATE = STOP;
	GO
	ALTER EVENT SESSION ... STATE = START;
	GO
 


## 输出


完成 Transact-SQL 脚本后，请单击 **event\_data\_XML** 列标题下的单元格。此时将显示一个 **<event>** 元素，其中显示了一个 UPDATE 语句。

下面是测试期间生成的一个 **<event>** 元素：

 
	<event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T19:18:45.420Z">
	  <data name="state">
	    <value>0</value>
	    <text>Normal</text>
	  </data>
	  <data name="line_number">
	    <value>5</value>
	  </data>
	  <data name="offset">
	    <value>148</value>
	  </data>
	  <data name="offset_end">
	    <value>368</value>
	  </data>
	  <data name="statement">
	    <value>UPDATE gmTabEmployee
	    SET EmployeeKudosCount = EmployeeKudosCount + 2
	    WHERE EmployeeDescr = 'Jane Doe'</value>
	  </data>
	  <action name="sql_text" package="sqlserver">
	    <value>
	
	SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;
	
	UPDATE gmTabEmployee
	    SET EmployeeKudosCount = EmployeeKudosCount + 2
	    WHERE EmployeeDescr = 'Jane Doe';
	
	UPDATE gmTabEmployee
	    SET EmployeeKudosCount = EmployeeKudosCount + 13
	    WHERE EmployeeDescr = 'Jane Doe';
	
	SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
	</value>
	  </action>
	</event>


 

## 转换代码示例以在 SQL Server 上运行


假设你要在 Microsoft SQL Server 上运行上述 Transact-SQL 示例。


- 为了方便，你想要将 Azure 存储空间容器完全替换为一个简单文件（例如 **C:\\myeventdata.xel**）。该文件将写入托管 SQL Server 的计算机的本地硬盘驱动器上。


- 你不需要为 **CREATE MASTER KEY** 和 **CREATE CREDENTIAL** 使用任何类型的 Transact-SQL 语句。


- 在 **CREATE EVENT SESSION** 语句的 **ADD TARGET** 子句中，将对 **filename=** 分配的 Http 值替换为完整路径字符串（例如 **C:\\myfile.xel**）。
 - 此操作不涉及任何 Azure 存储帐户。


## 详细信息


有关 Azure SQL 数据库中扩展事件的主要主题是：

- [SQL 数据库中的扩展事件](/documentation/articles/sql-database-xevent-db-diff-from-svr/) - 有关 Azure SQL 数据库中扩展事件的主要主题。
 - 对比 Azure SQL 数据库与 Microsoft SQL Server 的扩展事件的不同方面。


- [SQL 数据库中扩展事件的环形缓冲区目标代码](/documentation/articles/sql-database-xevent-code-ring-buffer/) - 提供一个可以快速方便上手的辅助代码示例，但该示例主要适用于简单测试，而对于大型活动则不够可靠。


有关 Azure 存储空间服务中帐户和容器的详细信息，请参阅：

- [如何通过 .NET 使用 Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)
- [命名和引用容器、Blob 与元数据](http://msdn.microsoft.com/zh-cn/library/azure/dd135715.aspx)
- [使用根容器](http://msdn.microsoft.com/zh-cn/library/azure/ee395424.aspx)


<!--
Image references.
-->

[30_powershell_ise]: ./media/sql-database-xevent-code-event-file/event-file-powershell-ise-b30.png

<!---HONumber=Mooncake_0118_2016-->
