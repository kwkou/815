<properties
   pageTitle="发生中断后在 SQL 数据仓库中恢复数据库 | Azure"
   description="发生中断后在 SQL 数据仓库中恢复数据库的步骤。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/09/2016"
   wacn.date="04/11/2016"/>

# 发生中断后在 SQL 数据仓库中恢复数据库

异地还原让你能够从异地冗余备份还原数据库以创建新数据库。可以在任意 Azure 区域中的任何服务器上创建数据库。由于它使用异地冗余备份作为其源，因此即使由于停电而无法访问数据库，也能够用其恢复数据库。除了发生中断后进行恢复，异地还原也可以用于其他情况，例如将数据库迁移或复制到不同的服务器或区域。


## 何时启动恢复
恢复操作在恢复时要求更改 SQL 连接字符串，并可能会导致数据永久丢失。因此，仅当中断的持续时间超过了应用程序的 RTO 时，才应执行恢复操作。使用以下数据点来声明有必要进行恢复：

- 数据库永久连接故障。
- Azure 门户显示了警报，指出区域中的某个事件会造成广泛的影响。


## 使用异地还原进行恢复
恢复数据库会从最新的异地冗余备份创建一个新的数据库。必须确保要恢复到的服务器具有足够的 DTU，可以容纳新数据库的容量。你可以通过[与支持人员联系][]来请求增加此配额。


### PowerShell
使用 Azure PowerShell 以编程方式执行数据库恢复。若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。可以通过运行 Get-Module -ListAvailable -Name Azure 来检查你的版本。本文基于  Azure PowerShell 版本 1.0.4。

若要恢复数据库，请使用 [Start-AzureSqlDatabaseRecovery][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的数据库的订阅。
4. 获取要恢复的数据库。
5. 创建对数据库的恢复请求。
6. 监视恢复进度。

```

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    Get-AzureRmSubscription
    Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"
    
    # Get the database you want to recover
    $Database = Get-AzureRmSqlRecoverableDatabase -ServerName "<YourServerName>" –DatabaseName "<YourDatabaseName>"
    
    # Recover database
    $RecoveryRequest = Start-AzureSqlDatabaseRestore -SourceServerName "<YourSourceServerName>" -SourceDatabase $Database -TargetDatabaseName "<NewDatabaseName>" -TargetServerName "<YourTargetServerName>"
    
    # Monitor progress of recovery operation
    Get-AzureSqlDatabaseOperation -ServerName "<YourTargetServerName>" –OperationGuid $RecoveryRequest.RequestID

```

请注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为上述 Powershell cmdlet 中的 -ServerName。

### REST API
使用 REST 以编程方式执行数据库恢复。

1. 使用[列出可恢复的数据库][]操作获取可恢复数据库的列表。
2. 使用[获取可恢复的数据库][]操作获取你要恢复的数据库。
3. 使用[创建数据库恢复请求][]操作创建恢复请求。
4. 使用[数据库操作状态][]操作跟踪恢复状态。



## 恢复后配置数据库
此清单可帮助你准备好将恢复的数据库投入生产。

1. **更新连接字符串**：验证客户端工具的连接字符串指向最近恢复的数据库。
2. **修改防火墙规则**：验证目标服务器上的防火墙规则，并确保已启用从客户端计算机或 Azure 到服务器以及最近恢复的数据库的连接。
3. **验证服务器登录名和数据库用户**：验证应用程序使用的所有登录名是否在托管已恢复数据库的服务器上存在。重新创建缺少的登录名，并向其授予对已恢复数据库的适当权限。 
4. **启用审核**：如果需要通过审核来访问数据库，你需要在恢复数据库后启用审核。

如果源数据库启用了 TDE，则已恢复的数据库将启用 TDE。


## 后续步骤
若要详细了解其他 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL 数据库业务连续性概述][]。


<!--Image references-->

<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[Finalize a recovered database]: /documentation/articles/sql-database-recovered-finalize/

<!--MSDN references-->
[Start-AzureSqlDatabaseRecovery]: https://msdn.microsoft.com/zh-cn/library/azure/dn720224.aspx
[列出可恢复的数据库]: http://msdn.microsoft.com/zh-cn/library/azure/dn800984.aspx
[获取可恢复的数据库]: http://msdn.microsoft.com/zh-cn/library/azure/dn800985.aspx
[创建数据库恢复请求]: http://msdn.microsoft.com/zh-cn/library/azure/dn800986.aspx
[数据库操作状态]: http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx

<!--Other Web references-->
[Azure 门户]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0405_2016-->