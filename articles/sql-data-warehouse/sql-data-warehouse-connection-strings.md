<!-- Temporarily comment out connect overview -->
<properties
   pageTitle="Azure SQL 数据仓库的驱动程序 | Azure"
   description="SQL 数据仓库的连接字符串和驱动程序"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="06/16/2016"
   wacn.date="07/04/2016" />


# Azure SQL 数据仓库的驱动程序

> [AZURE.SELECTOR]
- [身份验证](/documentation/articles/sql-data-warehouse-authentication/)
- [驱动程序](/documentation/articles/sql-data-warehouse-connection-strings/)
<!-- - [概述](/documentation/articles/sql-data-warehouse-connect-overview/) -->

可以使用以下任一应用程序协议连接到 SQL 数据仓库：

- ADO.NET
- ODBC
- PHP
- JDBC 

下面是每个协议的连接字符串的一些示例。你可以使用 Azure 管理门户来帮助设置连接字符串。只需在 Azure 经典管理门户中导航到你的数据库。在“必备”下面，单击“显示数据库连接字符串”。

## 示例 ADO.NET 连接字符串

    Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;

## 示例 ODBC 连接字符串

    Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;

## 示例 PHP 连接字符串

    Server: {your_server}.database.chinacloudapi.cn,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( "sqlsrv:server = tcp:{your_server}.database.chinacloudapi.cn,1433; Database = {your_database}", "{your_user_name}", "{your_password_here}");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( "Error connecting to SQL Server." );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array("UID" => "{your_user_name}", "pwd" => "{your_password_here}", "Database" => "{your_database}", "LoginTimeout" => 30, "Encrypt" => 1, "TrustServerCertificate" => 0);\r\n$serverName = "tcp:{your_server}.database.chinacloudapi.cn,1433";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);

## 示例 JDBC 连接字符串

    jdbc:sqlserver://yourserver.database.chinacloudapi.cn:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.chinacloudapi.cn;loginTimeout=30;

<!--Image references-->

<!--Azure.com references-->


<!--MSDN references-->

<!--Other references-->

<!---HONumber=Mooncake_0627_2016-->