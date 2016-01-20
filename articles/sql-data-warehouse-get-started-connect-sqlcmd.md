<properties
   pageTitle="入门：连接到 Azure SQL 数据仓库 | Windows Azure"
   description="开始连接到 SQL 数据仓库并运行一些查询。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="twounder"
   manager=""
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="10/20/2015"
   wacn.date="01/20/2016"/>

# 使用 SQLCMD 进行连接和查询

> [AZURE.SELECTOR]
- [Visual Studio](/documentation/articles/sql-data-warehouse-get-started-connect)
- [SQLCMD](/documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd)

本演练说明如何使用 sqlcmd.exe 实用程序在短时间内连接和查询 Azure SQL 数据仓库数据库。在本演练中，你将：

+ 安装必备软件
+ 连接到包含 AdventureWorksDW 示例数据库的数据库
+ 对示例数据库执行查询  

## 先决条件

+ [sqlcmd.exe](https://msdn.microsoft.com/zh-cn/library/azure/ms162773.aspx) - 若要下载 sqlcmd.exe，请参阅[适用于 SQL Server 的 Microsoft 命令行实用程序 11](http://www.microsoft.com/zh-cn/download/details.aspx?id=36433)。

## 获取完全限定的 Azure SQL 服务器名称

若要连接到数据库，你需要服务器的完整名称 (****servername**.database.chinacloudapi.cn*)，该名称中包含要连接到的数据库。


若要查找完全限定的服务器名称，请执行以下操作。

1. 通过以下命令查找我们需要的服务器信息。本示例使用 AdventureWorksDW 示例数据库,Group1 资源组。
```
PS C:\> Get-AzureRmSqlServer -ResourceGroupName "Group1" -DatabaseName "AdventureWorksDW"  
```

## 使用 sqlcmd 连接到 SQL 数据仓库

若要在使用 sqlcmd 时连接到 SQL 数据仓库的特定实例，需要打开命令提示符并输入 **sqlcmd** 后接 SQL 数据仓库数据库的连接字符串。连接字符串需包含以下参数：

+ **用户 (-U)：**采用 `<`用户`>` 格式的服务器用户
+ **密码 (-P)：**与用户关联的密码
+ **服务器 (-S)：**采用 `<`服务器名称`>`.database.chinacloudapi.cn 格式的服务器
+ **数据库 (-D)：**数据库名称
+ **启用带引号的标识符 (-I)：**必须启用带引号的标识符才能连接到 SQL 数据仓库实例。

因此，若要连接到 SQL 数据仓库实例，需输入以下信息：

```
C:\>sqlcmd -S <Server Name>.database.chinacloudapi.cn -d <Database> -U <User> -P <Password> -I
```

## 运行示例查询

连接后，可以对实例发出任何支持的 Transact-SQL 语句。

```
C:\>sqlcmd -S <Server Name>.database.chinacloudapi.cn -d <Database> -U <User> -P <Password> -I
1> SELECT COUNT(*) FROM dbo.FactInternetSales;
2> GO
3> QUIT
```

有关 sqlcmd 的更多信息，请参阅 [sqlcmd 文档](https://msdn.microsoft.com/zh-cn/library/azure/ms162773.aspx)。


## 后续步骤

既然你可以执行连接和查询，接下来请尝试[使用 PowerBI 进行连接][]。

[使用 PowerBI 进行连接]: /documentation/articles/sql-data-warehouse-integrate-power-bi
 

<!--Image references-->
[1]: ./media/sql-data-warehouse-get-started-connect/get-server-name.png

<!---HONumber=Mooncake_1207_2015-->
