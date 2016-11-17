<properties
   pageTitle="连接到 Azure SQL 数据仓库 | Azure"
   description="如何查找连接到 Azure SQL 数据仓库所需的服务器名称和连接字符串"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="09/26/2016"
   wacn.date="10/31/2016"
   ms.author="sonyama;barbkess"/>  


# 连接到 Azure SQL 数据仓库

本文介绍首次连接到 SQL 数据仓库的操作方法。

## 查找服务器名称

连接到 SQL 数据仓库时，第一步是了解如何查找服务器名称。例如，以下示例中的服务器名称是 sample.database.chinacloudapi.cn。若要查找完全限定的服务器名称，请执行以下操作：

1. 转到 [Azure 门户][]。
2. 单击“SQL 数据库”
3. 单击要连接到的数据库。
4. 找到完整的服务器名称。

    ![完整服务器名称][1]  


## 支持的驱动程序和连接字符串

Azure SQL 数据仓库支持 [ADO.NET][]、[ODBC][]、[PHP][] 和 [JDBC][]。单击前述的某个驱动程序，查找最新版本和文档。若要自动生成通过 Azure 门户使用的驱动程序的连接字符串，可单击前述示例中的“显示数据库连接字符串”。下面也是一些示例，说明了每个驱动程序的连接字符串的样式。

> [AZURE.NOTE] 请考虑将连接超时值设置为 300 秒，以便连接可以经受住短时间内不可用。

### ADO.NET 连接字符串示例


    Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;


### ODBC 连接字符串示例


    Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;


### PHP 连接字符串示例


    Server: {your_server}.database.chinacloudapi.cn,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( "sqlsrv:server = tcp:{your_server}.database.chinacloudapi.cn,1433; Database = {your_database}", "{your_user_name}", "{your_password_here}");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( "Error connecting to SQL Server." );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array("UID" => "{your_user_name}", "pwd" => "{your_password_here}", "Database" => "{your_database}", "LoginTimeout" => 30, "Encrypt" => 1, "TrustServerCertificate" => 0);\r\n$serverName = "tcp:{your_server}.database.chinacloudapi.cn,1433";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);


### JDBC 连接字符串示例


    jdbc:sqlserver://yourserver.database.chinacloudapi.cn:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.chinacloudapi.cn;loginTimeout=30;


## 连接设置

SQL 数据仓库在连接和创建对象期间标准化一些设置。这些设置不能重写，其中包括：

| 数据库设置 | 值 |
| :--------------------- | :--------------------------- |
| [ANSI\_NULLS][] | 开 |
| [QUOTED\_IDENTIFIERS][] | 开 |
| [DATEFORMAT][] | mdy |
| [DATEFIRST][] | 7 |

## 后续步骤

若要使用 Visual Studio 进行连接和查询，请参阅 [Query with Visual Studio][]（使用 Visual Studio 进行查询）。若要详细了解身份验证选项，请参阅 [Authentication to Azure SQL Data Warehouse][]（通过 Azure SQL 数据仓库进行身份验证）。

<!--Articles-->

[Query with Visual Studio]: /documentation/articles/sql-data-warehouse-query-visual-studio/
[Authentication to Azure SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-authentication/

<!--MSDN references-->
[ADO.NET]: https://msdn.microsoft.com/zh-cn/library/e80y5yhx(v=vs.110).aspx
[ODBC]: https://msdn.microsoft.com/zh-cn/library/jj730314.aspx
[PHP]: https://msdn.microsoft.com/zh-cn/library/cc296172.aspx?f=255&MSPPError=-2147217396
[JDBC]: https://msdn.microsoft.com/zh-cn/library/mt484311(v=sql.110).aspx
[ANSI\_NULLS]: https://msdn.microsoft.com/zh-cn/library/ms188048.aspx
[QUOTED\_IDENTIFIERS]: https://msdn.microsoft.com/zh-cn/library/ms174393.aspx
[DATEFORMAT]: https://msdn.microsoft.com/zh-cn/library/ms189491.aspx
[DATEFIRST]: https://msdn.microsoft.com/zh-cn/library/ms181598.aspx

<!--Other-->
[Azure 门户预览]: https://portal.azure.cn

<!--Image references-->

[1]: ./media/sql-data-warehouse-connect-overview/get-server-name.png

<!---HONumber=Mooncake_1024_2016-->