<properties linkid="develop-java-sql-azure" urlDisplayName="SQL Database" pageTitle="如何使用 SQL Azure (Java) - Azure 功能指南" metaKeywords="" description="了解如何通过 Java 代码使用 Azure SQL Database。 " metaCanonical="" services="sql-database" documentationCenter="Java" title="How to Use Azure SQL Database in Java" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" videoId="" scriptId="" />

# 如何通过 Java 使用 Azure SQL Database

以下步骤演示了如何通过 Java 使用 Azure SQL Database。显示命令行示例是为了简单起见，但对于在本地、Azure 或其他环境中托管的 Web 应用程序，采用的步骤非常相似。本指南介绍了如何从 [Azure 管理门户](https://manage.windowsazure.cn)创建服务器和数据库。

## 什么是 Azure SQL Database

Azure SQL Database 为 Azure 提供关系数据库管理系统并以 SQL Server 技术为基础。使用 SQL Database 实例，您可以轻松地配置关系数据库解决方案并将其部署到云中，并且利用能够为企业级可用性、可缩放性和安全性提供内置数据保护和自愈优势的分布式数据中心。

## 目录

-   [概念][]
-   [先决条件][]
-   [创建 Azure SQL Database][]
-   [确定 SQL Database 连接字符串][]
-   [允许访问一系列 IP 地址][]
-   [通过 Java 使用 Azure SQL Database][]
-   [通过代码与 Azure SQL Database 通信][]
-   [创建表][]
-   [在表上创建索引][]
-   [插入行][]
-   [检索行][]
-   [使用 WHERE 子句检索行][]
-   [检索行的计数][]
-   [更新行][]
-   [删除行][]
-   [检查表是否存在][]
-   [删除索引][]
-   [删除表][]
-   [在 Azure 部署中通过 Java 使用 SQL Database][]
-   [后续步骤][]

<h2><a id="concepts"></a>概念</h2>
因为 Azure SQL Database 是基于 SQL Server 技术构建的，所以通过 Java 访问 SQL Database 与通过 Java 访问 SQL Server 非常相似。您可以从本地开发应用程序（使用 SQL Server），然后通过仅更改连接字符串连接到 SQL Database。您可以对应用程序使用 SQL Server JDBC 驱动程序。但是，SQL Database 和 SQL Server 之间存在一些差别，它们可能会影响您的应用程序。有关详细信息，请参阅[指南和限制 (SQL Database)](http://msdn.microsoft.com/zh-cn/library/azure/ff394102.aspx)。

有关 SQL Database 的其他资源，请参阅[后续步骤][]部分。

<h2><a id="prerequisites"></a>先决条件</h2>

如果您打算通过 Java 使用 SQL Database，则需满足下面的先决条件。

* Java 开发人员工具包 (JDK) 1.6 版或更高版本。
* Azure 订阅，可从 <http://www.windowsazure.cn/zh-cn/pricing/overview/>. 获取
* 如果您使用的是 Eclipse，则将需要 Eclipse IDE for Java EE Developers Indigo 或更高版本。可从 <http://www.eclipse.org/downloads/> 下载。还需要 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）。在此插件的安装过程中，请确保包含了 Microsoft JDBC Driver 4.0 for SQL Server。有关详细信息，请参阅[安装 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）](http://msdn.microsoft.com/zh-cn/library/azure/hh690946.aspx)。
* 如果您使用的不是 Eclipse，则将需要 Microsoft JDBC Driver 4.0 for SQL Server，可从以下网址下载它 <http://www.microsoft.com/zh-cn/download/details.aspx?id=11774>.

<h2><a id="create_db"></a>创建 Azure SQL Database</h2>

通过 Java 代码使用 Azure SQL Database 之前，需要创建 Azure SQL Database 服务器。

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 单击"新建"****。

    ![Create new SQL database][create_new]

3. 单击"SQL database"****，然后单击"自定义创建"****。

    ![Create custom SQL database][create_new_sql_db]

4. 在"数据库设置"****对话框中，指定数据库名称。在本指南中，使用 **gettingstarted** 作为数据库名称。
5. 对于"服务器"****，选择"新建 SQL Database 服务器"****。对其他字段使用默认值。

    ![SQL database settings][create_database_settings]

6. 单击"下一步"箭头。	
7. 在"服务器设置"****对话框中，指定 SQL Server 登录名。在本指南中，使用了 **MySQLAdmin**。指定并确认密码。指定一个区域，并确保选中了"允许 Azure 服务访问服务器"****。

    ![SQL server settings][create_server_settings]

8. 单击"完成"按钮。

<h2><a id="determine_connection_string"></a>确定 SQL Database 连接字符串</h2>

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 单击"SQL Database"****。
3. 单击要使用的数据库。
4. 单击"显示连接字符串"****。
5. 突出显示"JDBC"****连接字符串的内容。

    ![Determine JDBC connection string][get_jdbc_connection_string]

6. 右键单击"JDBC"****连接字符串的突出显示内容，然后单击"复制"****。
7. 您现在可以将此值粘贴到代码文件中以创建以下形式的连接字符串。将 *your_server*（在两个位置中）替换为在上一步中复制的文本，并将 *your_password* 替换为创建 SQL Database 帐户时指定的密码值。（如果未使用 **gettingstarted** 和 **MySQLAdmin**，则还要分别替换分配给 **database=** 和 **user=** 的值。） 

		String connectionString =
		"jdbc:sqlserver://*your_server*.database.chinacloudapi.cn:1433" + ";" +
    	"database=gettingstarted" + ";" + 
    	"user=MySQLAdmin@*your_server*" + ";" +  
    	"password=*your_password*" + ";" +  
        "encrypt=true" + ";" +
        "hostNameInCertificate=*.database.chinacloudapi.cn" + ";" +
        "loginTimeout=30";

我们实际上会在本指南的后面使用此字符串，您目前知道了确定连接字符串的步骤。此外，根据您的应用程序需求，您可能不需要使用 **encrypt** 和 **hostNameInCertificate** 设置，并可能需要修改 **loginTimeout** 设置。

<h2><a id="specify_allowed_ips"></a>允许访问一系列 IP 地址</h2>
1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击"SQL Database"****。
3. 单击"服务器"****。
4. 单击要使用的服务器。
5. 单击"管理"****。
6. 单击"配置"****。
7. 在"允许的 IP 地址"****下，输入新 IP 规则的名称。指定 IP 地址的开始和结束范围。为方便起见，将会显示当前的客户端 IP 地址。以下示例允许使用单个客户端 IP 地址（您的 IP 将与此不同）。

    ![Allowed IP addresses dialog][allowed_ips_dialog]

8. 单击"完成"按钮。您指定的 IP 地址现在可以访问您的数据库服务器。

<h2><a id="use_sql_azure_in_java"></a>通过 Java 使用 Azure SQL Database</h2>

1. 创建一个 Java 项目。在本教程中，将该项目命名为 **HelloSQLAzure**。
2. 将名为 **HelloSQLAzure.java** 的 Java 类文件添加到该项目。
3. 将 **Microsoft JDBC Driver for SQL Server** 添加到生成路径。

   如果使用了 Eclipse：

    1. 在 Eclipse 的"项目资源管理器"中，右键单击"HelloSQLAzure"****项目并单击"属性"****。
    2. 在"属性"****对话框的左侧窗格中，单击"Java 生成路径"****。
    3. 单击"库"****选项卡，然后单击"添加库"****。
    4. 在"添加库"****对话框中，选择"Microsoft JDBC Driver 4.0 for SQL Server"****、单击"下一步"****，然后单击"完成"****。
    5. 单击"确定"****以关闭"属性"****对话框。

    如果您使用的不是 Eclipse，请将 Microsoft JDBC Driver 4.0 for SQL Server JAR 添加到您的类路径。若要了解相关信息，请参阅[使用 JDBC Driver](http://msdn.microsoft.com/zh-cn/library/ms378526.aspx)。

4. 在"HelloSQLAzure.java"****代码中，添加"import"语句，如下所示：

        import java.sql.*;
        import com.microsoft.sqlserver.jdbc.*;

5. 指定连接字符串。下面是一个示例。如上所述，将 *your_server*（在两个位置中）、*your_user* 和 *your_password* 替换为适合您的 SQL Database 服务器的值。

        String connectionString =
        	"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +
        		"database=master" + ";" + 
        		"user=your_user@your_server" + ";" +  
        		"password=your_password";

您现在已准备好添加与 SQL Database 服务器通信的代码。

<h2><a id="communicate_from_code"></a>通过代码与 Azure SQL Database 通信</h2>

本主题剩下的部分将显示执行以下操作的示例：

1. 连接到 SQL Database 服务器。
2. 定义 SQL 语句，例如，创建或删除表、插入/选择/删除行，等等。
3. 通过调用 **executeUpdate** 或 **executeQuery** 执行 SQL 语句。
4. 在适当的时候显示查询结果。

以下各节需要按顺序阅读（举例）。第一个代码段是一个完整示例；其他代码段将依赖于完整示例中的部分框架，例如 **import** 语句、**class** 和 **main** 声明、错误处理以及资源关闭。

<h2><a id="to_create_table"></a>创建表</h2>

以下代码演示了如何创建名为 **Person** 的表。

	import java.sql.*;
	import com.microsoft.sqlserver.jdbc.*;
	
	public class HelloSQLAzure {
	
	    public static void main(String[] args) 
	    {
	
			// Connection string for your SQL Database server.
			// Change the values assigned to your_server, 
			// your_user@your_server,
			// and your_password.
			String connectionString = 
				"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +
					"database=gettingstarted" + ";" + 
					"user=your_user@your_server" + ";" +  
					"password=your_password";
			
			// The types for the following variables are
			// defined in the java.sql library.
			Connection connection = null;  // For making the connection
			Statement statement = null;    // For the SQL statement
			ResultSet resultSet = null;    // For the result set, if applicable
			
			try
			{
			    // Ensure the SQL Server driver class is available.
			    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
			
			    // Establish the connection.
			    connection = DriverManager.getConnection(connectionString);
			
			    // Define the SQL string.
			    String sqlString = 
					"CREATE TABLE Person (" + 
			        	"[PersonID] [int] IDENTITY(1,1) NOT NULL," +
			            "[LastName] [nvarchar](50) NOT NULL," + 
			            "[FirstName] [nvarchar](50) NOT NULL)";
			
			    // Use the connection to create the SQL statement.
			    statement = connection.createStatement();
			
			    // Execute the statement.
			    statement.executeUpdate(sqlString);
			
			    // Provide a message when processing is complete.
			    System.out.println("Processing complete.");
			
			}
			// Exception handling
	        catch (ClassNotFoundException cnfe)  
	        {
	            
	            System.out.println("ClassNotFoundException " +
	                               cnfe.getMessage());
	        }
	        catch (Exception e)
	        {
	            System.out.println("Exception " + e.getMessage());
	            e.printStackTrace();
	        }
	        finally
	        {
	            try
	            {
	                // Close resources.
	                if (null != connection) connection.close();
	                if (null != statement) statement.close();
	                if (null != resultSet) resultSet.close();
	            }
	            catch (SQLException sqlException)
	            {
	                // No additional action if close() statements fail.
	            }
	        }
	        
	    }
	
	}
	

<h2><a id="to_create_index"></a>在表中创建索引</h2>

以下代码演示了如何使用 **PersonID** 列在 **Person** 表上创建名为 **index1** 的索引。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = 
			"CREATE CLUSTERED INDEX index1 " + "ON Person (PersonID)";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    statement.executeUpdate(sqlString);
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...



<h2><a id="to_insert_rows"></a>插入行</h2>

以下代码演示了如何将行添加到 **Person** 表。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = 
			"SET IDENTITY_INSERT Person ON " + 
	        	"INSERT INTO Person " + 
	            "(PersonID, LastName, FirstName) " + 
	            "VALUES(1, 'Abercrombie', 'Kim')," + 
	            	  "(2, 'Goeschl', 'Gerhard')," + 
	                  "(3, 'Grachev', 'Nikolay')," + 
	                  "(4, 'Yee', 'Tai')," + 
	                  "(5, 'Wilson', 'Jim')";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    statement.executeUpdate(sqlString);
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...

 
<h2><a id="to_retrieve_rows"></a>检索行</h2>

以下代码演示了如何从 **Person** 表中检索行。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = "SELECT TOP 10 * FROM Person";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    resultSet = statement.executeQuery(sqlString);
	
	    // Loop through the results
	    while (resultSet.next())
	    {
	        // Print out the row data
	        System.out.println(
	        	"Person with ID " + 
	        	resultSet.getString("PersonID") + 
	        	" has name " +
	        	resultSet.getString("FirstName") + " " +
	       		resultSet.getString("LastName"));
	        }
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...

 上面的代码从 **Person** 表中选择了前 10 行。如果要返回所有行，请将 SQL 语句修改为以下形式：

	String sqlString = "SELECT * FROM Person";

 
<h2><a id="to_retrieve_rows_using_where"></a>使用 WHERE 子句检索行</h2>

若要使用子句检索行，请使用上面所示的代码，只不过需要将 SQL 语句更改为包含一个子句。以下 SQL 语句包含一个用于检索其 **FirstName** 值等于 **Jim** 的行的子句。

	// Define the SQL string.
	String sqlString = "SELECT * FROM Person WHERE FirstName='Jim'";
	
还可以在检索计数、更新行或删除行时使用 WHERE 子句。

<h2><a id="to_retrieve_row_count"></a>检索行的计数</h2>

以下代码演示了如何从 **Person** 表中检索行的计数。
 
	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	// Define the SQL string.
	    String sqlString = "SELECT COUNT (PersonID) FROM Person";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    resultSet = statement.executeQuery(sqlString);
	
	    // Print out the returned number of rows.
	    while (resultSet.next())
	    {
	        System.out.println("There were " + 
	                         resultSet.getInt(1) +
	                         " rows returned.");
	    }
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...

<h2><a id="to_update_rows"></a>更新行</h2>

以下代码演示了如何更新行。在此示例中，将 **FirstName** 值为 **Jim** 的所有行的 **LastName** 值更改为 **Kim**。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = 
			"UPDATE Person " + "SET LastName = 'Kim' " + "WHERE FirstName='Jim'";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	    
	    // Execute the statement.
	    statement.executeUpdate(sqlString);
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}// Exception handling and resource closing not shown...

 

<h2><a id="to_delete_rows"></a>删除行</h2>

以下代码演示了如何删除行。在此示例中，将删除 **FirstName** 值为 **Jim** 的所有行。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = 
			"DELETE from Person " + 
				"WHERE FirstName='Jim'";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    statement.executeUpdate(sqlString);
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...
	
 
<h2><a id="to_check_table_existence"></a>检查表是否存在</h2>

以下代码演示了如何确定某个表是否存在。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = 
			"IF EXISTS (SELECT 1 " +
	        	"FROM sysobjects " + 
	            "WHERE xtype='u' AND name='Person') " +
	            "SELECT 'Person table exists.'" +
	            "ELSE  " +
	            "SELECT 'Person table does not exist.'";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    resultSet = statement.executeQuery(sqlString);
	
	    // Display the result.
	    while (resultSet.next())
	    {
	        System.out.println(resultSet.getString(1));
	    }
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...

<h2><a id="to_drop_index"></a>删除索引</h2>

以下代码演示了如何在 **Person** 表上删除名为 **index1** 的索引。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +
			"database=gettingstarted" + ";" +
			"user=your_user@your_server" + ";" +
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = 
			"DROP INDEX index1 " + 
	        	"ON Person";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    statement.executeUpdate(sqlString);
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...

 
<h2><a id="to_drop_table"></a>删除表</h2>

以下代码演示了如何删除名为 **Person** 的表。

	// Connection string for your SQL Database server.
	// Change the values assigned to your_server, 
	// your_user@your_server,
	// and your_password.
	String connectionString = 
		"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433" + ";" +  
			"database=gettingstarted" + ";" + 
			"user=your_user@your_server" + ";" +  
			"password=your_password";
	
	// The types for the following variables are
	// defined in the java.sql library.
	Connection connection = null;  // For making the connection
	Statement statement = null;    // For the SQL statement
	ResultSet resultSet = null;    // For the result set, if applicable
	
	try
	{
	    // Ensure the SQL Server driver class is available.
	    Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
	
	    // Establish the connection.
	    connection = DriverManager.getConnection(connectionString);
	
	    // Define the SQL string.
	    String sqlString = "DROP TABLE Person";
	
	    // Use the connection to create the SQL statement.
	    statement = connection.createStatement();
	
	    // Execute the statement.
	    statement.executeUpdate(sqlString);
	
	    // Provide a message when processing is complete.
	    System.out.println("Processing complete.");
	
	}
	// Exception handling and resource closing not shown...

<h2><a id="using_in_azure"></a>在 Azure 部署中通过 Java 使用 SQL Database</h2>

若要在 Azure 部署中通过 Java 使用 SQL Database，除了将 Microsoft JDBC Driver 4.0 for SQL Server 作为类路径中的库（如前面所述）之外，还需要将它与您的部署一起打包。


**在使用 Eclipse 的情况下打包 Microsoft JDBC Driver 4.0 SQL Server**

1. 在 Eclipse 的"项目资源管理器"中，右键单击您的项目并单击"属性"****。
2. 在"属性"****对话框的左侧窗格中，单击"部署程序集"****，然后单击"添加"****。
3. 在"新建程序集指令"****对话框中，单击"Java 生成路径项"****，然后单击"下一步"****。
4. 选择"Microsoft JDBC Driver 4.0 SQL Server"****，然后单击"完成"****。
5. 单击"确定"****以关闭"属性"****对话框。
6. 按照[使用 Azure Plugin for Eclipse with Java（由 Microsoft Open Technologies 提供）创建 Hello World 应用程序](http://msdn.microsoft.com/zh-cn/library/azure/hh690944.aspx)中所记录的步骤，将项目的 WAR 文件导出到 approot 文件夹，然后重新生成 Azure 项目。本主题还介绍了如何在计算仿真程序和 Azure 中运行应用程序。

**在未使用 Eclipse 的情况下打包 Microsoft JDBC Driver 4.0 SQL Server**

* 确保已将 Microsoft JDBC Driver 4.0 SQL Server 库包含在与 Java 应用程序相同的 Azure 角色中，并且已将其添加到您的应用程序的类路径。

<h2><a id="nextsteps"></a>后续步骤</h2>

若要了解有关 Microsoft JDBC Driver for SQL Server 的详细信息，请参阅 [JDBC Driver 概述](http://msdn.microsoft.com/zh-cn/library/ms378749.aspx)。若要了解有关 SQL Database 的详细信息，请参阅 [SQL Database 概述](http://msdn.microsoft.com/zh-cn/library/azure/ee336241.aspx)。

[概念]:#concepts
[先决条件]:#prerequisites
[创建 Azure SQL Database]:#create_db
[确定 SQL Database 连接字符串]:#determine_connection_string
[允许访问一系列 IP 地址]:#specify_allowed_ips
[通过 Java 使用 Azure SQL Database]:#use_sql_azure_in_java
[通过代码与 Azure SQL Database 通信]:#communicate_from_code
[创建表]:#to_create_table
[在表上创建索引]:#to_create_index
[插入行]:#to_insert_rows
[检索行]:#to_retrieve_rows
[使用 WHERE 子句检索行]:#to_retrieve_rows_using_where
[检索行的计数]:#to_retrieve_row_count
[更新行]:#to_update_rows
[删除行]:#to_delete_rows
[检查表是否存在]:#to_check_table_existence
[删除索引]:#to_drop_index
[删除表]:#to_drop_table
[在 Azure 部署中通过 Java 使用 SQL Database]:#using_in_azure
[后续步骤]:#nextsteps
[create_new]: ./media/sql-data-java-how-to-use-sql-database/WA_New.png
[create_new_sql_db]: ./media/sql-data-java-how-to-use-sql-database/WA_SQL_DB_Create.png
[create_database_settings]: ./media/sql-data-java-how-to-use-sql-database/WA_CustomCreate_1.png
[create_server_settings]: ./media/sql-data-java-how-to-use-sql-database/WA_CustomCreate_2.png
[get_jdbc_connection_string]: ./media/sql-data-java-how-to-use-sql-database/WA_SQL_JDBC_ConnectionString.png
[allowed_ips_dialog]: ./media/sql-data-java-how-to-use-sql-database/WA_Allowed_IPs.png
