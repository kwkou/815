<properties
    pageTitle="使用 .NET (C#) 连接 Azure SQL 数据库 | Azure"
    description="使用本快速入门教程中的示例代码可以生成一个包含 C# 代码并由云中强大的 Azure SQL 数据库关系数据库支持的现代应用程序。"
    services="sql-database"
    documentationcenter=""
    author="stevestein"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="7faca033-24b4-4f64-9301-b4de41e73dfd"
    ms.service="sql-database"
    ms.custom="development"
    ms.workload="drivers"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="03/16/2017"
    wacn.date="04/17/2017"
    ms.author="sstein"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="09348ed8bfa798556982e31b0d2a693d0af07e8f"
    ms.lasthandoff="04/07/2017" />

# <a name="azure-sql-database-use-net-c-to-connect-and-query-data"></a>Azure SQL 数据库：使用 .NET (C#) 进行连接和数据查询

使用 [C# 和 ADO.NET](https://msdn.microsoft.com/zh-cn/library/kb9s9ks0.aspx) 连接和查询 Azure SQL 数据库。 本指南详述了如何使用 C# 连接到 Azure SQL 数据库，然后执行查询、插入、更新和删除语句。

此快速入门使用以下某个快速入门中创建的资源作为其起点：

- [创建 DB - 门户](/documentation/articles/sql-database-get-started-portal/)
- [创建 DB - CLI](/documentation/articles/sql-database-get-started-cli/)
- [创建 DB - PowerShell](/documentation/articles/sql-database-get-started-powershell/) 

在开始之前，请确保已为 C# 配置开发环境。 请参阅 [Install Visual Studio Community for free](https://www.visualstudio.com/)（免费安装 Visual Studio Community），或者安装 [适用于 SQL Server 的 ADO.NET 驱动程序](https://www.microsoft.com/net/download)。

## <a name="connect-to-database-and-query-data"></a>连接到数据库并查询数据

在 Azure 门户预览中获取连接字符串。 请使用连接字符串连接到 Azure SQL 数据库。

1. 登录 [Azure 门户预览](https://portal.azure.cn/)。
2. 从左侧菜单中选择“SQL 数据库”，然后单击“SQL 数据库”页上的数据库。 
3. 在数据库的“概要”窗格中，找到并单击“显示数据库连接字符串”。
4. 复制 **ADO.NET** 连接字符串。

    <img src="./media/sql-database-connect-query-dotnet/connection-strings.png" alt="connection strings" style="width: 780px;" />

5. 打开 Visual Studio，创建一个控制台应用程序。
6. 将 ```using System.Data.SqlClient``` 添加到代码文件 ([System.Data.SqlClient namespace](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.aspx))。 

7. 使用 [SqlCommand.ExecuteReader](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlcommand.executereader.aspx) 和 [SELECT](https://msdn.microsoft.com/zh-cn/library/ms189499.aspx) Transact-SQL 语句查询 Azure SQL 数据库中的数据。

        string strConn = "<connection string>";
        using (var connection = new SqlConnection(strConn))
        {
        
        connection.Open();

        SqlCommand selectCommand = new SqlCommand("", connection);
        selectCommand.CommandType = CommandType.Text;

        selectCommand.CommandText = @"SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
            FROM [SalesLT].[ProductCategory] pc
            JOIN [SalesLT].[Product] p
            ON pc.productcategoryid = p.productcategoryid";

        SqlDataReader reader = selectCommand.ExecuteReader();

        while (reader.Read())
        {
            // show data
            Console.WriteLine($"{reader.GetString(0)}\t{reader.GetString(1)}");
        }
        reader.Close();
        }

## <a name="insert-data"></a>插入数据

使用 [SqlCommand.ExecuteNonQuery](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlcommand.executenonquery.aspx) 和 [INSERT](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx) Transact-SQL 语句将数据插入 Azure SQL 数据库。

    SqlCommand insertCommand = new SqlCommand("", connection);
    insertCommand.CommandType = CommandType.Text;
    insertCommand.CommandText = @"INSERT INTO[SalesLT].[Product]
                ( [Name]
                , [ProductNumber]
                , [Color]
                , [StandardCost]
                , [ListPrice]
                , [SellStartDate]
                )
    VALUES
    (
                @Name,
                @ProductNumber,
                @Color,
                @StandardCost,
                @ListPrice,
                @SellStartDate)";

    insertCommand.Parameters.Add("@Name", SqlDbType.Text);
    insertCommand.Parameters.Add("@ProductNumber", SqlDbType.Int);
    insertCommand.Parameters.Add("@Color", SqlDbType.Text);
    insertCommand.Parameters.Add("@StandardCost", SqlDbType.Decimal);
    insertCommand.Parameters.Add("@ListPrice", SqlDbType.Decimal);
    insertCommand.Parameters.Add("@SellStartDate", SqlDbType.Date);

    insertCommand.Parameters["@Name"].Value = "BrandNewProduct";
    insertCommand.Parameters["@ProductNumber"].Value = 200989;
    insertCommand.Parameters["@Color"].Value = "Blue";
    insertCommand.Parameters["@StandardCost"].Value = 75;
    insertCommand.Parameters["@ListPrice"].Value = 80;
    insertCommand.Parameters["@SellStartDate"].Value = "3/15/2017";

    int newrows = insertCommand.ExecuteNonQuery();
    Console.WriteLine($"Inserted {newrows.ToString()} row(s).");

## <a name="update-data"></a>更新数据

使用 [SqlCommand.ExecuteNonQuery](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlcommand.executenonquery.aspx) 和 [UPDATE](https://msdn.microsoft.com/zh-cn/library/ms177523.aspx) Transact-SQL 语句更新 Azure SQL 数据库中的数据。

    SqlCommand updateCommand = new SqlCommand("", connection);
    updateCommand.CommandType = CommandType.Text;
    updateCommand.CommandText = @"UPDATE SalesLT.Product SET ListPrice = @ListPrice WHERE Name = @Name";
    updateCommand.Parameters.Add("@Name", SqlDbType.Char);
    updateCommand.Parameters.Add("@ListPrice", SqlDbType.Decimal);
    updateCommand.Parameters["@ListPrice"].Value = 500;
    updateCommand.Parameters["@Name"].Value = "BrandNewProduct";

    int updatedrows = updateCommand.ExecuteNonQuery();
    Console.WriteLine($"Updated {updatedrows.ToString()} row(s).");

## <a name="delete-data"></a>删除数据

使用 [SqlCommand.ExecuteNonQuery](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlcommand.executenonquery.aspx) 和 [DELETE](https://msdn.microsoft.com/zh-cn/library/ms189835.aspx) Transact-SQL 语句删除 Azure SQL 数据库中的数据。

    SqlCommand deleteCommand = new SqlCommand("", connection);
    deleteCommand.CommandType = CommandType.Text;
    deleteCommand.CommandText = @"DELETE FROM SalesLT.Product WHERE Name = @Name";
    deleteCommand.Parameters.Add("@Name", SqlDbType.Char);
    deleteCommand.Parameters["@Name"].Value = "BrandNewProduct";

    int deletedrows = deleteCommand.ExecuteNonQuery();
    Console.WriteLine($"Deleted {deletedrows.ToString()} row(s).");

## <a name="complete-script"></a>完整脚本

以下脚本在单个代码块中包含所有前述步骤。

    using System;
    using System.Data;
    using System.Data.SqlClient;

    namespace ConsoleApplication1
    {
        class Program
        {
            static void Main(string[] args)
            {

                string strConn = "<connection string>";

                using (var connection = new SqlConnection(strConn))
                {
                    connection.Open();

                    Console.WriteLine("Query data example:");
                    Console.WriteLine("\n=========================================\n");

                    SqlCommand selectCommand = new SqlCommand("", connection);
                    selectCommand.CommandType = CommandType.Text;

                    selectCommand.CommandText = @"SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
                       FROM [SalesLT].[ProductCategory] pc
                       JOIN [SalesLT].[Product] p
                       ON pc.productcategoryid = p.productcategoryid";

                    SqlDataReader reader = selectCommand.ExecuteReader();

                    while (reader.Read())
                    {
                        // show data columns
                        Console.WriteLine($"{reader.GetString(0)}\t{reader.GetString(1)}");
                    }
                    reader.Close();
                    Console.WriteLine("\nPress any key to continue ...");
                    Console.ReadLine();

                    Console.WriteLine("\nInsert data example:");
                    Console.WriteLine("=========================================\n");
                    SqlCommand insertCommand = new SqlCommand("", connection);
                    insertCommand.CommandType = CommandType.Text;
                    insertCommand.CommandText = @"INSERT INTO[SalesLT].[Product]
                              ( [Name]
                              , [ProductNumber]
                              , [Color]
                              , [StandardCost]
                              , [ListPrice]
                              , [SellStartDate]
                              )
                    VALUES
                    (
                                @Name,
                                @ProductNumber,
                                @Color,
                                @StandardCost,
                                @ListPrice,
                                @SellStartDate)";

                    insertCommand.Parameters.Add("@Name", SqlDbType.Text);
                    insertCommand.Parameters.Add("@ProductNumber", SqlDbType.Int);
                    insertCommand.Parameters.Add("@Color", SqlDbType.Text);
                    insertCommand.Parameters.Add("@StandardCost", SqlDbType.Decimal);
                    insertCommand.Parameters.Add("@ListPrice", SqlDbType.Decimal);
                    insertCommand.Parameters.Add("@SellStartDate", SqlDbType.Date);

                    insertCommand.Parameters["@Name"].Value = "BrandNewProduct";
                    insertCommand.Parameters["@ProductNumber"].Value = 200989;
                    insertCommand.Parameters["@Color"].Value = "Blue";
                    insertCommand.Parameters["@StandardCost"].Value = 75;
                    insertCommand.Parameters["@ListPrice"].Value = 80;
                    insertCommand.Parameters["@SellStartDate"].Value = "7/1/2016";

                    int newrows = insertCommand.ExecuteNonQuery();
                    Console.WriteLine($"Inserted {newrows.ToString()} row(s).");
                    Console.WriteLine("\nPress any key to continue ...");
                    Console.ReadLine();

                    Console.WriteLine("\nUpdate data example:");
                    Console.WriteLine("======================\n");
                    SqlCommand updateCommand = new SqlCommand("", connection);
                    updateCommand.CommandType = CommandType.Text;
                    updateCommand.CommandText = @"UPDATE SalesLT.Product SET ListPrice = @ListPrice WHERE Name = @Name";
                    updateCommand.Parameters.Add("@Name", SqlDbType.Char);
                    updateCommand.Parameters.Add("@ListPrice", SqlDbType.Decimal);
                    updateCommand.Parameters["@ListPrice"].Value = 500;
                    updateCommand.Parameters["@Name"].Value = "BrandNewProduct";

                    int updatedrows = updateCommand.ExecuteNonQuery();
                    Console.WriteLine($"Updated {updatedrows.ToString()} row(s).");
                    Console.WriteLine("\nPress any key to continue ...");
                    Console.ReadLine();

                    Console.WriteLine("\nDelete data example:");
                    Console.WriteLine("======================\n");
                    SqlCommand deleteCommand = new SqlCommand("", connection);
                    deleteCommand.CommandType = CommandType.Text;
                    deleteCommand.CommandText = @"DELETE FROM SalesLT.Product WHERE Name = @Name";
                    deleteCommand.Parameters.Add("@Name", SqlDbType.Char);
                    deleteCommand.Parameters["@Name"].Value = "BrandNewProduct";

                    int deletedrows = deleteCommand.ExecuteNonQuery();
                    Console.WriteLine($"Deleted {deletedrows.ToString()} row(s).");
                    Console.WriteLine("\nPress any key to continue ...");
                    Console.ReadLine();
                }
            }
        }
    }

## <a name="next-steps"></a>后续步骤

- 有关 .NET 文档，请参阅 [.NET 文档](https://docs.microsoft.com/dotnet/)。
- 有关使用 Visual Studio Code 查询和编辑数据的信息，请参阅 [Visual Studio Code](https://code.visualstudio.com/docs)。