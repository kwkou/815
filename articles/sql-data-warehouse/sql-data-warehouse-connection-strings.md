<!-- Temporarily comment out connect overview -->
<properties
   pageTitle="SQL 数据仓库的驱动程序 | Azure"
   description="SQL 数据仓库的连接字符串和驱动程序"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="10/31/2016"
   wacn.date="12/12/2016"
   ms.author="sonyama;barbkess"/>


# Azure SQL 数据仓库的驱动程序
用户可以使用多个不同的应用程序协议，例如 [ADO.NET][ADO.NET]、[ODBC][ODBC]、[PHP][PHP] 和 [JDBC][JDBC] 连接到 SQL 数据仓库。下面是每个协议的连接字符串的一些示例。可以使用 Azure 门户来生成连接字符串。若要使用 Azure 门户生成连接字符串，请导航到数据库边栏选项卡，在“概要”下单击“显示数据库连接字符串”。

## 示例 ADO.NET 连接字符串

    Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;

## 示例 ODBC 连接字符串

    Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;

## 示例 PHP 连接字符串

    Server: {your_server}.database.chinacloudapi.cn,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( "sqlsrv:server = tcp:{your_server}.database.chinacloudapi.cn,1433; Database = {your_database}", "{your_user_name}", "{your_password_here}");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( "Error connecting to SQL Server." );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array("UID" => "{your_user_name}", "pwd" => "{your_password_here}", "Database" => "{your_database}", "LoginTimeout" => 30, "Encrypt" => 1, "TrustServerCertificate" => 0);\r\n$serverName = "tcp:{your_server}.database.chinacloudapi.cn,1433";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);

## 示例 JDBC 连接字符串

    jdbc:sqlserver://yourserver.database.chinacloudapi.cn:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.chinacloudapi.cn;loginTimeout=30;

> [AZURE.NOTE] 请考虑将连接超时值设置为 300 秒，以便连接可以经受住短时间内不可用。

## 后续步骤
若要开始使用 Visual Studio 和其他应用程序查询数据仓库，请参阅[使用 Visual Studio 进行查询][Query with Visual Studio]。

<!--Image references-->


<!--Azure.com references-->
 [Query with Visual Studio]: /documentation/articles/sql-data-warehouse-query-visual-studio/
 
<!--MSDN references-->
[ADO.NET]: https://msdn.microsoft.com/zh-cn/library/e80y5yhx(v=vs.110).aspx
[ODBC]: https://msdn.microsoft.com/zh-cn/library/jj730314.aspx
[PHP]: https://msdn.microsoft.com/zh-cn/library/cc296172.aspx?f=255&MSPPError=-2147217396
[JDBC]: https://msdn.microsoft.com/zh-cn/library/mt484311(v=sql.110).aspx

<!--Other references-->

<!---HONumber=Mooncake_1205_2016-->