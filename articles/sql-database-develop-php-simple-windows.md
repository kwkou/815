<properties
	pageTitle="在 Windows 上使用 PHP 连接到 SQL Database"
	description="演示一个示例 PHP 程序，该程序可从 Windows 客户端连接到 Azure SQL Database，并与客户端所需的软件组件建立链接。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="php"
	ms.topic="article"
	ms.date="04/17/2015"
	wacn.date="05/25/2015" 
	ms.author="genemi"/>


# 在 Windows 上使用 PHP 连接到 SQL Database


<!--
Original content written by Luiz Fernando Santos, then edited by GeneMi, HackaDoc2, 2015-04-14.
-->


本主题演示了如何从 Windows 运行的、以 PHP 编写的客户端应用程序连接到 Azure SQL Database。


## 先决条件


若要运行本主题中所述的 PHP 代码示例，你的客户端计算机必须已安装以下软件项：


- [Microsoft Drivers for PHP for Microsoft SQL Server](http://www.microsoft.com/zh-CN/download/details.aspx?id=20098)（SQLSRV32.EXE 包含最新的数据）
- [Microsoft SQL Server Native Client 11.0](http://www.microsoft.com/zh-CN/download/details.aspx?id=36434)
- 某些安装需要使用 [Web 平台安装程序](http://www.microsoft.com/web/downloads/platform.aspx)来完成。有关详细信息，请参阅下一节。


### 使用 Web 平台安装程序安装


1. 下载并运行 [Web 平台安装程序](http://www.microsoft.com/web/downloads/platform.aspx)，然后选择所需的模块。
2. 使用 [Web 平台安装程序](http://www.microsoft.com/web/downloads/platform.aspx)安装以下组件：
 - IIS 或 IIS Express.<br/>例如，可通过在网站中搜索以下关键字找到 [IIS Express 8.0 下载](http://www.microsoft.com/zh-cn/download/details.aspx?id=34679)：IIS Express 下载。
 - PHP for Windows 版本 5.5 或更高，32 位。
3. 通过在 Dynamic Extensions 节下添加一个条目来编辑 PHP.INI 文件，如下所示：<br/>**extension=php_sqlsrv_55_nts.dll**


## 连接到 SQL Database 数据库


此 **OpenConnection** 函数将在其后面的所有函数的靠近顶部位置调用。


	function OpenConnection()
	{
		try
		{
			$serverName = "tcp:myserver.database.chinacloudapi.cn,1433";

			$connectionOptions = array("Database"=>"AdventureWorks",
				"Uid"=>"MyUser", "PWD"=>"MyPassword");

			$conn = sqlsrv_connect($serverName, $connectionOptions);

			if($conn == false)
				die(FormatErrors(sqlsrv_errors()));
		}
		catch(Exception $e)
		{
			echo("Error!");
		}
	}


## 将值插入表中


	function InsertData()
	{
		try
		{
			$conn = OpenConnection();

			$tsql = "INSERT INTO [SalesLT].[Customer]
					   ([NameStyle],[FirstName],[MiddleName],[LastName],
					   [PasswordHash],[PasswordSalt],[rowguid],[ModifiedDate])
					   VALUES (?,?,?,?,?,?,?,?)";

			$params = array("0", "Luiz", "F", "Santos",
				"L/Rlwxzp4w7RWmEgXX+/A7cXaePEPcp+KwQhl2fJL7w=",
				"1KjXYs4=", "88949BFE-0BB4-4148-AFC2-DD8C8DAE4CD6",
				date("Y-m-d"));

			$insertReview = sqlsrv_prepare($conn, $tsql, $params);
			if($insertReview == FALSE)
				die(FormatErrors( sqlsrv_errors()));

			if(sqlsrv_execute($insertReview) == FALSE)
				die(FormatErrors(sqlsrv_errors()));

			sqlsrv_free_stmt($insertReview);

			sqlsrv_close($conn);
		}
		catch(Exception $e)
		{
			echo("Error!");
		}
	}


## 从表中删除值


	function DeleteData()
	{
		try
		{
			$conn = OpenConnection();

			$tsql = "DELETE FROM [SalesLT].[Customer] WHERE FirstName=? AND LastName=?";
			$params = array($_REQUEST['FirstName'], $_REQUEST['LastName']);

			$deleteReview = sqlsrv_prepare($conn, $tsql, $params);
			if($deleteReview == FALSE)
				die(FormatErrors(sqlsrv_errors()));

			if(sqlsrv_execute($deleteReview) == FALSE)
				die(FormatErrors(sqlsrv_errors()));

			sqlsrv_free_stmt($deleteReview);

			sqlsrv_close($conn);
		}
		catch(Exception $e)
		{
			echo("Error!");
		}
	}


## 从表中选择值


	function ReadData()
	{
		try
		{
			$conn = OpenConnection();

			$tsql = "SELECT [CompanyName] FROM SalesLT.Customer";

			$getProducts = sqlsrv_query($conn, $tsql);

			if ($getProducts == FALSE)
				die(FormatErrors(sqlsrv_errors()));

			$productCount = 0;

			while($row = sqlsrv_fetch_array($getProducts, SQLSRV_FETCH_ASSOC))
			{
				echo($row['CompanyName']);
				echo("<br/>");
				$productCount++;
			}

			sqlsrv_free_stmt($getProducts);

			sqlsrv_close($conn);
		}
		catch(Exception $e)
		{
			echo("Error!");
		}
	}


## 事务


	function Transactions()
	{
		try
		{
			$conn = OpenConnection();

			if (sqlsrv_begin_transaction($conn) == FALSE)
				die(FormatErrors(sqlsrv_errors()));

			/* Initialize parameter values. */
			$orderId = 43659;
			$qty = 5;
			$productId = 709;
			$offerId = 1;
			$price = 5.70;

			/* Set up and execute the first query. */
			$tsql1 = "INSERT INTO SalesLT.SalesOrderDetail
			   (SalesOrderID,OrderQty,ProductID,SpecialOfferID,UnitPrice)
			   VALUES (?, ?, ?, ?, ?)";
			$params1 = array($orderId, $qty, $productId, $offerId, $price);
			$stmt1 = sqlsrv_query($conn, $tsql1, $params1);

			/* Set up and execute the second query. */
			$tsql2 = "UPDATE Production.ProductInventory SET Quantity = (Quantity - ?)
					  WHERE ProductID = ?";
			$params2 = array($qty, $productId);
			$stmt2 = sqlsrv_query( $conn, $tsql2, $params2 );

			/* If both queries were successful, commit the transaction. */
			/* Otherwise, rollback the transaction. */
			if($stmt1 && $stmt2)
			{
					sqlsrv_commit($conn);
					echo "Transaction was committed.\n";
			}
			else
			{
					sqlsrv_rollback($conn);
					echo "Transaction was rolled back.\n";
			}

			/* Free statement and connection resources. */
			sqlsrv_free_stmt( $stmt1);
			sqlsrv_free_stmt( $stmt2);

			sqlsrv_close($conn);
		}
		catch(Exception $e)
		{
			echo("Error!");
		}
	}


## 存储过程


	function ExecuteSProc()
	{
		try
		{
			$conn = OpenConnection();

			$tsql = "EXEC [dbo].[uspLogError]";

			$execReview = sqlsrv_prepare($conn, $tsql);
			if($execReview == FALSE)
				die(FormatErrors( sqlsrv_errors()));

			if(sqlsrv_execute($execReview) == FALSE)
				die(FormatErrors(sqlsrv_errors()));

			sqlsrv_free_stmt($execReview);

			sqlsrv_close($conn);
		}
		catch(Exception $e)
		{
			echo("Error!");
		}
	}


## 深入阅读


有关 PHP 安装和用法的详细信息，请参阅[使用 PHP 访问 SQL Server 数据库](http://technet.microsoft.com/zh-cn/library/cc793139.aspx)。


<!--
FWLink 533699 points to TechNet...
https://technet.microsoft.com/en-us/library/cc793139(v=sql.90).aspx,
 which instead should be...
http://technet.microsoft.com/library/cc793139.aspx
.
Regardless, we should use direct URL, not indirect, in this type of one-time case. It might be different if this link were being placed into 25 different topics.
-->

<!--HONumber=55-->