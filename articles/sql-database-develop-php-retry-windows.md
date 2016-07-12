<properties
	pageTitle="用于连接到 SQL 数据库的 PHP 重试逻辑 | Azure"
	description="演示一个示例 PHP 程序，该程序可以通过暂时性故障处理从 Windows 客户端连接到 Azure SQL 数据库，并与客户端所需的软件组件建立链接。"
	services="sql-database"
	documentationCenter=""
	authors="meet-bhagdev"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="03/18/2016"
	wacn.date="05/16/2016"/>


# 使用 Windows 上的 PHP 和暂时性故障处理连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


本主题演示了如何从 Windows 运行的、以 PHP 编写的客户端应用程序连接到 Azure SQL 数据库。

## 步骤 1：配置开发环境

[AZURE.INCLUDE [sql-database-develop-includes-prerequisites-php-windows](../includes/sql-database-develop-includes-prerequisites-php-windows.md)]

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。


## 步骤 3：获取连接详细信息

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## 步骤 4：连接和查询

设计该演示程序的目的是在尝试连接期间发生暂时性故障时导致重试。但是，在执行查询命令期间发生暂时性故障时会导致程序丢弃连接并创建新的连接，然后重试查询命令。我们既不建议也不反对这种设计。演示程序演示了一些可供你使用的设计弹性。

<br>此代码示例的长度多半是因为捕获异常的逻辑。[此处](/documentation/articles/sql-database-develop-php-simple-windows/)提供了此 Program.cs 文件的简短版本。
<br>Main 方法在 Program.cs 中。调用堆栈的运行如下：
* Main 调用 ConnectAndQuery。
* ConnectAndQuery 调用 EstablishConnection。
* EstablishConnection 调用 IssueQueryCommand。

[sqlsrv\_query()](http://php.net/manual/en/function.sqlsrv-query.php) 函数可用于针对 SQL 数据库从查询中检索结果集。此函数实际上可接受任何查询和连接对象，并返回可使用 [sqlsrv\_fetch\_array()](http://php.net/manual/zh/function.sqlsrv-fetch-array.php) 循环访问的结果集。

	<?php
		// Variables to tune the retry logic.  
		$connectionTimeoutSeconds = 30;  // Default of 15 seconds is too short over the Internet, sometimes.
		$maxCountTriesConnectAndQuery = 3;  // You can adjust the various retry count values.
		$secondsBetweenRetries = 4;  // Simple retry strategy.
		$errNo = 0;
		$serverName = "tcp:yourdatabase.database.chinacloudapi.cn,1433";
		$connectionOptions = array("Database"=>"AdventureWorks",
		   "Uid"=>"yourusername", "PWD"=>"yourpassword", "LoginTimeout" => $connectionTimeoutSeconds);
		$conn;
		$errorArr = array();
		for ($cc = 1; $cc <= $maxCountTriesConnectAndQuery; $cc++)
		{
		    $errorArr = array();
		    $ctr = 0;
		    // [A.2] Connect, which proceeds to issue a query command.
		    $conn = sqlsrv_connect($serverName, $connectionOptions);  
		    if( $conn == true)
		    {
		        echo "Connection was established";
		        echo "<br>";

		        $tsql = "SELECT [CompanyName] FROM SalesLT.Customer";
		        $getProducts = sqlsrv_query($conn, $tsql);
		        if ($getProducts == FALSE)
		            die(FormatErrors(sqlsrv_errors()));
		        $productCount = 0;
		        $ctr = 0;
		        while($row = sqlsrv_fetch_array($getProducts, SQLSRV_FETCH_ASSOC))
		        {   
		            $ctr++;
		            echo($row['CompanyName']);
		            echo("<br/>");
		            $productCount++;
		            if($ctr>10)
		                break;
		        }
		        sqlsrv_free_stmt($getProducts);
		        break;
		    }
		    // Adds any the error codes from the SQL Exception to an array.
		    else {  
		        if( ($errors = sqlsrv_errors() ) != null) {
		            foreach( $errors as $error ) {
		                $errorArr[$ctr] = $error['code'];
		                $ctr = $ctr + 1;
		            }
		        }
		        $isTransientError = TRUE;
		        // [A.4] Check whether sqlExc.Number is on the whitelist of transients.
		        $isTransientError = IsTransientStatic($errorArr);
		        if ($isTransientError == TRUE)  // Is a static persistent error...
		        {
		            echo("Persistent error suffered, SqlException.Number==". $errorArr[0].". Program Will terminate.");
		            echo "<br>";
		            // [A.5] Either the connection attempt or the query command attempt suffered a persistent SqlException.
		            // Break the loop, let the hopeless program end.
		            exit(0);
		        }
		        // [A.6] The SqlException identified a transient error from an attempt to issue a query command.
		        // So let this method reloop and try again. However, we recommend that the new query
		        // attempt should start at the beginning and establish a new connection.
		        if ($cc >= $maxCountTriesConnectAndQuery)
		        {
		            echo "Transient errors suffered in too many retries - " . $cc ." Program will terminate.";
		            echo "<br>";
		            exit(0);
		        }
		        echo("Transient error encountered.  SqlException.Number==". $errorArr[0]. " . Program might retry by itself.");  
		        echo "<br>";
		        echo $cc . " Attempts so far. Might retry.";
		        echo "<br>";
		        // A very simple retry strategy, a brief pause before looping. This could be changed to exponential if you want.
		        sleep(1*$secondsBetweenRetries);
		    }
		    // [A.3] All has gone well, so let the program end.
		}
		function IsTransientStatic($errorArr) {
		    $arrayOfTransientErrorNumbers = array(4060, 10928, 10929, 40197, 40501, 40613);
		    $result = array_intersect($arrayOfTransientErrorNumber, $errorArr);
		    $count = count($result);
		    if($count >= 0) //change to > 0 later.
		        return TRUE;
		}
	?>

## 后续步骤

有关 PHP 安装和用法的详细信息，请参阅[使用 PHP 访问 SQL Server 数据库](http://technet.microsoft.com/zh-cn/library/cc793139.aspx)。

<!---HONumber=Mooncake_0509_2016-->
