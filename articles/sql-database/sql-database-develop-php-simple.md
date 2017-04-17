<properties
    pageTitle="在 Windows上使用 PHP 连接到 SQL 数据库 | Azure"
    description="演示一个示例 PHP 程序，该程序可以从 Windows 客户端连接到 Azure SQL 数据库，并与客户端所需的软件组件建立链接。"
    services="sql-database"
    documentationcenter=""
    author="meet-bhagdev"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="4e71db4a-a22f-4f1c-83e5-4a34a036ecf3"
    ms.service="sql-database"
    ms.custom="development"
    ms.workload="drivers"
    ms.tgt_pltfrm="na"
    ms.devlang="php"
    ms.topic="article"
    ms.date="02/13/2017"
    wacn.date="04/17/2017"
    ms.author="meetb"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="11932dd0ab9d523a564c171fba3ffe606de2f2de"
    ms.lasthandoff="04/07/2017" />

# <a name="connect-to-sql-database-by-using-php"></a>使用 PHP 连接到 SQL 数据库
[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

本主题说明如何使用 PHP 来连接和查询 Azure SQL 数据库。 可在 Windows 或 Linux 上运行此示例。 


## <a name="step-1-create-a-sql-database"></a>步骤 1：创建 SQL 数据库
请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。  务必根据指南创建 **AdventureWorks 数据库模板**。 下面所示的示例只适用于 **AdventureWorks 架构**。 创建数据库后，请确保根据[入门页](/documentation/articles/sql-database-get-started/)中所述，通过启用防火墙规则来启用对 IP 地址的访问

## <a name="step-2-configure-development-environment"></a>步骤 2：配置开发环境

### <a name="linux-ubuntu"></a>**Linux (Ubuntu)**
打开终端并导航到你要在其中创建 python 脚本的目录。 输入以下命令安装**适用于 Linux 的 Microsoft ODBC 驱动程序**、**pdo_sqlsrv** 和 **sqlsrv**。 适用于 SQL Server 的 Microsoft PHP 驱动程序使用 Linux 上的 Microsoft ODBC 驱动程序连接到 SQL 数据库。

    sudo su
    curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
    curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
    exit
    sudo apt-get update
    sudo ACCEPT_EULA=Y apt-get install msodbcsql unixodbc-dev gcc g++ build-essential
    sudo pecl install sqlsrv pdo_sqlsrv
    sudo echo "extension= pdo_sqlsrv.so" >> `php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"`
    sudo echo "extension= sqlsrv.so" >> `php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"`

### <a name="windows"></a>**Windows**
- [从 Web 平台安装程序](https://www.microsoft.com/web/downloads/platform.aspx?lang=)安装 PHP 7.1.1 (x64) 
- 安装 [Microsoft ODBC 驱动程序 13.1](https://www.microsoft.com/download/details.aspx?id=53339)。 
- 下载[用于 SQL Server 的 Microsoft PHP 驱动程序](https://pecl.php.net/package/sqlsrv/4.1.6.1/windows)的非线程安全 dll，并将二进制文件放在 PHP\v7.x\ext 文件夹中。
- 然后通过添加对 dll 的引用来编辑 php.ini (C:\Program Files\PHP\v7.1\php.ini) 文件。 例如：
      
      extension=php_sqlsrv.dll
      extension=php_pdo_sqlsrv.dll

此时，应已向 PHP 注册 dll。

## <a name="step-3-run-sample-code"></a>步骤 3：运行示例代码
创建名为 **sql_sample.php** 的文件并在其中粘贴以下代码。 可以通过命令行使用以下命令运行此操作：

    php sql_sample.php

### <a name="connect-to-your-sql-database"></a>连接到 SQL 数据库
[sqlsrv connect](http://php.net/manual/en/function.sqlsrv-connect.php) 函数用于连接 SQL 数据库。

    <?php
    $serverName = "yourserver.database.chinacloudapi.cn";
    $connectionOptions = array(
        "Database" => "yourdatabase",
        "Uid" => "yourusername",
        "PWD" => "yourpassword"
        );
    //Establishes the connection
    $conn = sqlsrv_connect($serverName, $connectionOptions);
    if($conn)
        echo "Connected!"
    ?>

### <a name="execute-an-sql-select-statement"></a>执行 SQL SELECT 语句
[sqlsrv_query](http://php.net/manual/en/function.sqlsrv-query.php) 函数可用于针对 SQL 数据库从查询中检索结果集。 

    <?php
    $serverName = "yourserver.database.chinacloudapi.cn";
    $connectionOptions = array(
        "Database" => "yourdatabase",
        "Uid" => "yourusername",
        "PWD" => "yourpassword"
    );
    //Establishes the connection
    $conn = sqlsrv_connect($serverName, $connectionOptions);
    if($conn)
        echo "Connected!"
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
    function FormatErrors( $errors )
    {
        /* Display errors. */
        echo "Error information: ";
    
        foreach ( $errors as $error )
        {
            echo "SQLSTATE: ".$error['SQLSTATE']."";
            echo "Code: ".$error['code']."";
            echo "Message: ".$error['message']."";
        }
    }
    ?>

### <a name="insert-a-row-pass-parameters-and-retrieve-the-generated-primary-key"></a>插入一行，传递参数，然后检索生成的主键
在 SQL 数据库中，可以使用 [IDENTITY](https://msdn.microsoft.com/zh-cn/library/ms186775.aspx) 属性和 [SEQUENCE](https://msdn.microsoft.com/zh-cn/library/ff878058.aspx) 对象自动生成[主键值](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)。 

    <?php
    $serverName = "yourserver.database.chinacloudapi.cn";
    $connectionOptions = array(
        "Database" => "yourdatabase",
        "Uid" => "yourusername",
        "PWD" => "yourpassword"
    );
    //Establishes the connection
    $conn = sqlsrv_connect($serverName, $connectionOptions);
    if($conn)
        echo "Connected!"
    $tsql = "INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server 1', 'SQL Server 2', 0, 0, getdate())";  
    //Insert query  
    $insertReview = sqlsrv_query($conn, $tsql);  
    if($insertReview == FALSE)  
        die(FormatErrors( sqlsrv_errors()));  
    echo "Product Key inserted is :";  
    while($row = sqlsrv_fetch_array($insertReview, SQLSRV_FETCH_ASSOC))  
    {     
        echo($row['ProductID']);  
    }  
    sqlsrv_free_stmt($insertReview);  
    sqlsrv_close($conn);  
    function FormatErrors( $errors )
    {
        /* Display errors. */
        echo "Error information: ";
        foreach ( $errors as $error )
        {
            echo "SQLSTATE: ".$error['SQLSTATE']."";
            echo "Code: ".$error['code']."";
            echo "Message: ".$error['message']."";
        }
    }
    ?>

## <a name="next-steps"></a>后续步骤
* 参阅 [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/)
* 有关 [Microsoft PHP Driver for SQL Server](https://github.com/Microsoft/msphpsql/) 的详细信息
* [提出问题](https://github.com/Microsoft/msphpsql/issues)。

## <a name="additional-resources"></a>其他资源
* [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* 浏览所有 [SQL 数据库的功能](/home/features/sql-database/)。
<!--Update_Description: add detailed steps for operations on Linux and Windows environment-->