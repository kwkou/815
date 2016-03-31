<properties
	pageTitle="在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库"
	description="演示了一个可以用来连接到 Azure SQL 数据库的 Java 代码示例。该示例使用 JDBC，并在 Windows 客户端计算机上运行。"
	services="sql-database"
	documentationCenter=""
	authors="LuisBosquez"
	manager="jeffreyg"
	editor="genemi"/>


<tags
	ms.service="sql-database"
	ms.date="12/17/2015"
	wacn.date="01/15/2016"/>


# 在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库


> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-develop-dotnet-simple)
- [PHP](/documentation/articles/sql-database-develop-php-simple-windows)
- [Python](/documentation/articles/sql-database-develop-python-simple-windows)
- [Ruby](/documentation/articles/sql-database-develop-ruby-simple-windows)
- [Java](/documentation/articles/sql-database-develop-java-simple-windows)
- [Node.js](/documentation/articles/sql-database-develop-nodejs-simple-windows)


本主题演示了一个可以用来连接到 Azure SQL 数据库的 Java 代码示例。该 Java 示例依赖于 Java 开发工具包 (JDK) 版本 1.8。该示例将使用 JDBC 驱动程序连接到 Azure SQL 数据库。


## 先决条件

### 驱动程序和库

- [Microsoft JDBC Driver for SQL Server - SQL JDBC 4](http://www.microsoft.com/zh-cn/download/details.aspx?displaylang=en&id=11774)。
- 运行 [Java 开发工具包 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 的任何操作系统平台。

### SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started)，以了解如何创建数据库。

### SQL 表

本主题中的 Java 代码示例假设 Azure SQL 数据库数据库中已存在以下测试表。

<!--
Could this instead be a #tempPerson table, so that the Java code sample could be fully self-sufficient and be runnable (with automatic cleanup)?
-->


	CREATE TABLE Person
	(
		id         INT    PRIMARY KEY    IDENTITY(1,1),
		firstName  VARCHAR(32),
		lastName   VARCHAR(32),
		age        INT
	);


## 步骤 1：获取连接字符串

[AZURE.INCLUDE [sql-database-include-connection-string-jdbc-20-portalshots](../includes/sql-database-include-connection-string-jdbc-20-portalshots.md)]

> [AZURE.NOTE]如果你使用的是 JTDS JDBC 驱动程序，则需要将“ssl=require”添加到连接字符串的 URL，并需要设置以下 JVM 选项：“-Djsse.enableCBCProtection=false”。此 JVM 选项会禁用针对某个安全漏洞的修复程序，因此在设置此选项之前，请确保你了解涉及哪些风险。


## 步骤 2：编译 Java 代码示例


本部分包含 Java 代码示例的主干。代码中包含注释，指出你可以在何处复制并粘贴后续部分中提供的较小 Java 段。即使不在注释附近复制并粘贴代码段，本部分中的示例也可以编译和运行，不过，它只会连接，然后结束。你会看到的注释如下：


1. `// INSERT two rows into the table.`
2. `// TRANSACTION and commit for an UPDATE.`
3. `// SELECT rows from the table.`


下面是 Java 代码示例的主干。该示例包含 `SQLDatabaseTest` 类的 `main` 函数。


	import java.sql.*;
	import com.microsoft.sqlserver.jdbc.*;

	public class SQLDatabaseTest {

		public static void main(String[] args) {
			String connectionString =
				"jdbc:sqlserver://your_server.database.chinacloudapi.cn:1433;"
				+ "database=your_database;"
				+ "user=your_user@your_server;"
				+ "password=your_password;"
				+ "encrypt=true;"
				+ "trustServerCertificate=false;"
				+ "hostNameInCertificate=*.database.chinacloudapi.cn;"
				+ "loginTimeout=30;";

			// Declare the JDBC objects.
			Connection connection = null;
			Statement statement = null;
			ResultSet resultSet = null;
			PreparedStatement prepsInsertPerson = null;
			PreparedStatement prepsUpdateAge = null;

			try {
				connection = DriverManager.getConnection(connectionString);

				// INSERT two rows into the table.
				// ...

				// TRANSACTION and commit for an UPDATE.
				// ...

				// SELECT rows from the table.
				// ...
			}
			catch (Exception e) {
				e.printStackTrace();
			}
			finally {
				// Close the connections after the data has been handled.
				if (prepsInsertPerson != null) try { prepsInsertPerson.close(); } catch(Exception e) {}
				if (prepsUpdateAge != null) try { prepsUpdateAge.close(); } catch(Exception e) {}
				if (resultSet != null) try { resultSet.close(); } catch(Exception e) {}
				if (statement != null) try { statement.close(); } catch(Exception e) {}
				if (connection != null) try { connection.close(); } catch(Exception e) {}
			}
		}
	}


当然，若要实际运行上面的 Java 代码示例，你必须将实际值填入连接字符串以替换占位符：


- 你的服务器
- 你的数据库
- 你的用户
- 你的密码


## 步骤 3：插入行


此 Java 段将发出 Transact-SQL INSERT 语句，以在 Person 表中插入两行。一般顺序如下：


1. 使用 `Connection` 对象生成 `PreparedStatement` 对象。
 - 我们包含了参数 `Statement.RETURN_GENERATED_KEYS`，以便稍后可以获取自动为 **id** 键值生成的值。
2. 对 `PreparedStatement` 对象调用 `execute` 方法。
3. 使用 `PreparedStatement` 对象获取自动为主键生成的数字值。
 - 此值与 Person 表中 **id** 列上的 AUTO\_INCREMENT 规范相关


将此简短 Java 段复制并粘贴到主代码示例中注释 `// INSERT two rows into the table.` 所在的位置。


	// Create and execute an INSERT SQL prepared statement.
	String insertSql = "INSERT INTO Person (firstName, lastName, age) VALUES "
		+ "('Bill', 'Gates', 59), "
		+ "('Steve', 'Ballmer', 59);";

	prepsInsertPerson = connection.prepareStatement(
		insertSql,
		Statement.RETURN_GENERATED_KEYS);
	prepsInsertPerson.execute();
	// Retrieve the generated key from the insert.
	resultSet = prepsInsertPerson.getGeneratedKeys();
	// Iterate through the set of generated keys.
	while (resultSet.next()) {
		System.out.println("Generated: " + resultSet.getString(1));
	}


## 步骤 4：提交事务

以下 Java 代码段将发出 Transact-SQL UPDATE 语句，以增大 Person 表中每行的 `age` 值。一般顺序如下：


1. 调用 `setAutoCommit` 方法，以防止在数据库中自动提交更新。
2. 调用 `executeUpdate` 方法以在事务上下文中执行 UPDATE。
3. 调用 `commit` 方法以显式提交事务。


将此简短 Java 段复制并粘贴到主代码示例中注释 `// TRANSACTION and commit for an UPDATE.` 所在的位置。


	// Set AutoCommit value to false to execute a single transaction at a time.
	connection.setAutoCommit(false);

	// Write the SQL Update instruction and get the PreparedStatement object.
	String transactionSql = "UPDATE Person SET Person.age = Person.age + 1;";
	prepsUpdateAge = connection.prepareStatement(transactionSql);

	// Execute the statement.
	prepsUpdateAge.executeUpdate();

	//Commit the transaction.
	connection.commit();

	// Return the AutoCommit value to true.
	connection.setAutoCommit(true);


## 步骤 4：执行查询


此 Java 段执行 Transact-SQL SELECT 语句，以查看 Person 表中所有已更新的行。一般顺序如下：


1. 使用 `Connection` 对象生成 `Statement` 对象。
2. 使用 `Statement` 对象生成 `ResultSet` 对象。
3. 循环调用 `resultSet.next` 以循环访问所有返回的行。


将此简短 Java 段复制并粘贴到主代码示例中注释 `// SELECT rows from a table.` 所在的位置。


	// Create and execute a SELECT SQL statement.
	String selectSql = "SELECT firstName, lastName, age FROM dbo.Person";
	statement = connection.createStatement();
	resultSet = statement.executeQuery(selectSql);

	// Iterate through the result set and print the attributes.
	while (resultSet.next()) {
		System.out.println(resultSet.getString(2) + " "
			+ resultSet.getString(3));
	}

## 后续步骤

有关详细信息，请参阅 [Java 开发人员中心](/develop/java)。

<!---HONumber=Mooncake_0104_2016-->
