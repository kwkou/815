<properties 
	pageTitle="使用 CSharp 在 Azure SQL 数据库 上执行 CREATE ASSEMBLY"
	description="提供 CSharp 源代码，用于在先将 DLL 文件编码成包含长十六进制数的字符串后，向 Azure SQL 数据库 发出 CREATE ASSEMBLY 语句。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="07/14/2015" 
	wacn.date="09/15/2015"/>


# 使用 CSharp 在 Azure SQL 数据库 上执行 CREATE ASSEMBLY

本主题提供可以用来向 Azure SQL 数据库发出 [CREATE ASSEMBLY](http://msdn.microsoft.com/zh-cn/library/ms189524.aspx) 语句的 C# 代码示例。对于 SQL 数据库，FROM 子句不能接受托管数据库的本地计算机上的简单格式路径。替代的做法是先将程序集 DLL 的二进制位编码成包含十六进制数的长字符串。然后，将该字符串指定为 FROM 子句中的值。


### 先决条件


若要学习本主题，必须已了解以下信息：


- [CLR 表值函数](https://msdn.microsoft.com/zh-cn/library/ms131103.aspx)<br/>说明 CREATE ASSEMBLY Transact-SQL 语句如何与本地 Microsoft SQL Server 的其他语句配合工作。


## A.总体方法


1. 根据需要使用 DROP FUNCTION 和 DROP ASSEMBLY 来清理前一次运行。
2. 请记住从自己的代码编译的 Microsoft.NET Framework 程序集 DLL 文件所在位置。需要在下一步中提供该位置。 
3. 运行要对其指定本主题中的 C# 源代码的 EXE。告诉该 EXE 你的 DLL 文件位于哪个位置。
 - 将二进制 DLL 编码成包含十六进制数的长字符串。
 - 发出 CREATE ASSEMBLY 语句，并在 FROM 子句中指定该十六进制字符串。
4. 执行 [CREATE FUNCTION](http://msdn.microsoft.com/zh-cn/library/ms186755.aspx) 以引用程序集中的方法。
5. 执行 T-SQL SELECT 语句以调用并测试你的函数。


上面的列表未提到...<br/> **execute sp\_configure 'clr enabled', 1;**<br/> ...因为 Azure SQL 数据库不需要此语句，但 Microsoft SQL Server 需要它。


如果需要重新运行，请执行以下 T-SQL 代码删除函数和程序集：


    DROP FUNCTION fnCompareStringsCaseSensitive;
    DROP ASSEMBLY CreateAssemblyFunctions3;


## B.T-SQL 函数要引用的简单程序集 DLL


可将本部分中的简单 C# 代码示例编译到程序集 DLL 文件中。


此代码示例包含稍后要在 T-SQL CREATE FUNCTION 语句中引用的 **CompareCaseSensitiveNet** 方法。请注意，已使用名为 **SqlFunction** 的 .NET 属性修饰了该方法。用此属性修饰的方法可由 T-SQL 作为函数调用。


	using           System;   // C#
	using SDSqTyp = System.Data.SqlTypes;
	using MSqServ = Microsoft.SqlServer.Server;
	
	namespace CreateAssemblyFunctions3
	{
	    public class SqlFunctionMethodsForSprocs
	    {
	        /// <summary> This method is referenced
	        /// by a T-SQL CREATE FUNCTION statement. </summary>
	        [MSqServ.SqlFunction(IsDeterministic = true, IsPrecise = true)]
	        static public SDSqTyp.SqlInt32 CompareCaseSensitiveNet(string strA, string strB)
	        {
	            return String.Compare(strA, strB, false);
	        }
	    }
	}


## C.发出 CREATE ASSEMBLY 的 EXE 的 C&#x23; 代码示例


运行从此 C# 示例生成的 EXE 时，将发生以下序列：


1. 从命令行运行的 EXE 调用 **Main** 方法。
2. Main 调用 **ObtainHexStringOfAssembly** 方法。
 - 该方法输出一个 SqlString，它将程序集存储为十六进制数。
3. Main 调用 **SubmitCreateAssemblyToAzureSqlDb** 方法。
 - 主要输入为 SqlString。
 - 输出是发送到 Azure SQL 数据库 的 CREATE ASSEMBLY 调用。


			using             System;   // C#
			using SDat      = System.Data;
			using SDSClient = System.Data.SqlClient;
			using SGlo      = System.Globalization;
			using SIo       = System.IO;
			using STex      = System.Text;
			
			namespace CreateAssemblyFromHexString6
			{
			    /// <summary>
			    /// Run the Main method on your local computer console, so it can issue a
			    /// a CREATE ASSEMBLY statement to your Azure SQL 数据库 server:
			    /// </summary>
			    class Program
			    {
			        /// <summary>
			        /// Calls the methods in the proper sequence.
			        /// </summary>
			        /// <param name="args">
			        /// Parameters for the cmd.exe command line
			        ///    run of CreateAssemblyFromHexString6.exe:
			        /// args[0] = FullDirPathFileNameOfAssembly.
			        /// args[1] = AssemblyName.
			        ///    For the CREATE ASSEMBLY assemblyName statement.
			        /// args[2] = Azure SQL 数据库 - ServerName, including a suffix like .database.chinacloudapi.cn .
			        /// args[3] = Azure SQL 数据库 - DatabaseName.
			        /// args[4] = Azure SQL 数据库 - LoginName.
			        /// args[5] = Azure SQL 数据库 - Password for login.
			        ///    (Better if from .config file.)
			        /// </param>
			        static int Main(string[] args)
			        {
			            int returnCode = 1; // Only 0 (zero) means Good Success.
			            string
			                fullPathNameAssemblyFile,
			                assemblyName,
			                AzureSqlDbServerName,
			                AzureSqlDbDatabaseName,
			                AzureSqlDbLoginName,
			                AzureSqlDbPassword;
			
			            try
			            {
			                fullPathNameAssemblyFile = args[0];
			                assemblyName             = args[1];
			                AzureSqlDbServerName     = args[2];
			                AzureSqlDbDatabaseName   = args[3];
			                AzureSqlDbLoginName      = args[4];
			                AzureSqlDbPassword       = args[5];
			
			                string hexStringOfAssembly = Program
			                    .ObtainHexStringOfAssembly(fullPathNameAssemblyFile);
			
			                Program.SubmitCreateAssemblyToAzureSqlDb(
			                    hexStringOfAssembly,
			                    assemblyName,
			                    AzureSqlDbServerName,
			                    AzureSqlDbDatabaseName,
			                    AzureSqlDbLoginName,
			                    AzureSqlDbPassword);
			
			                returnCode = 0;
			            }
			            catch (Exception ex)
			            {
			                Console.WriteLine("Bad, Failure.");
			                throw ex;
			            }
			            Console.WriteLine("Good, Success.");
			            return returnCode;
			        }
			
			        /// <summary> Encodes the binary bits of the assembly DLL into a 
			        /// string containing a hexadecimal number. </summary>
			        /// <param name="fullPathToAssembly"
			        /// >Full directory path plus the file name, to the .DLL file.</param>
			        /// <returns>A string containing a hexadecimal number that encodes
			        /// the binary bits of the .DLL file.</returns>
			        static private string ObtainHexStringOfAssembly
			            (string fullPathToAssembly)
			        {
			            STex.StringBuilder sbuilder = new STex.StringBuilder("0x");
			            using (SIo.FileStream fileStream = new SIo.FileStream(
			                fullPathToAssembly,
			                SIo.FileMode.Open, SIo.FileAccess.Read, SIo.FileShare.Read)
			                )
			            {
			                int byteAsInt;
			                while (true)
			                {
			                    byteAsInt = fileStream.ReadByte();
			                    if (-1 >= byteAsInt) { break; }
			                    sbuilder.Append(byteAsInt.ToString
			                        ("X2", SGlo.CultureInfo.InvariantCulture));
			                }
			            }
			            return sbuilder.ToString();
			        }
			
			        /// <summary>
			        /// Sends a Transact-SQL CREATE ASSEMBLY FROM hexString.
			        /// </summary>
			        static private void SubmitCreateAssemblyToAzureSqlDb(
			            string hexStringOfAssembly,
			            string assemblyName,
			            string AzureSqlDbServerName,
			            string AzureSqlDbDatabaseName,
			            string AzureSqlDbLoginName,
			            string AzureSqlDbPassword)
			        {
			            string sqlCreateAssembly = "CREATE ASSEMBLY [" + assemblyName
			                + "] FROM " + hexStringOfAssembly + ";";
			            STex.StringBuilder sbuilderConnection = new STex.StringBuilder();
			
			            sbuilderConnection.Append("Server=tcp:");
			            sbuilderConnection.Append(AzureSqlDbServerName);
			            sbuilderConnection.Append(",1433;");
			
			            sbuilderConnection.Append("Database=");
			            sbuilderConnection.Append(AzureSqlDbDatabaseName);
			            sbuilderConnection.Append(";");
			
			            sbuilderConnection.Append("Trusted_Connection=False;");
			            sbuilderConnection.Append("Connection Timeout=30;");
			
			            sbuilderConnection.Append("User ID=");
			            sbuilderConnection.Append(AzureSqlDbLoginName);
			            sbuilderConnection.Append(";");
			
			            sbuilderConnection.Append("Password=");
			            sbuilderConnection.Append(AzureSqlDbPassword);
			            sbuilderConnection.Append(";");
			
			            using (SDSClient.SqlConnection sqlConnection =
			                new SDSClient.SqlConnection())
			            {
			                SDSClient.SqlCommand sqlCommand;
			                sqlConnection.ConnectionString = sbuilderConnection.ToString();
			                sqlCommand = sqlConnection.CreateCommand();
			                sqlCommand.CommandType = SDat.CommandType.Text;
			                sqlCommand.CommandText = sqlCreateAssembly;
			                sqlConnection.Open();
			                sqlCommand.ExecuteNonQuery();
			            }
			            return;
			        }
			    }
			}


### C.1 编译引用和版本


当我们编译和测试 EXE 工具的示例代码时，我们使用了以下程序：


- Visual Studio 2013 Update 4
 - 我们的项目模板类型是简单的控制台应用程序。
- .NET Framework 4.5


我们的 Visual Studio 项目引用了以下要编译的程序集：


- Microsoft.CSharp
- System
- System.Core
- System.Data
- System.Data.DataSetExtensions
- System.Xml
- System.Xml.Linq


### C.2 用于运行 EXE 的命令行


以下代码块显示了从控制台中运行 EXE 所要输入的的命令行示例。为方便查看，命令行中的参数已经过人为包装。


	CreateAssemblyFromHexString6.exe
		C:\my\bin\debug\CreateAssemblyFunctions3.dll
		CreateAssemblyFunctions3
		myazuresqldbsvr2.database.chinacloudapi.cn
		myazuresqldbdab4
		myazurelogin
		Mypassword123


为方便解释，此示例传递了密码作为命令行参数。更好的设计是让 C# 代码从 CONFIG 文件中获取密码。


## D.运行 CREATE FUNCTION 语句


将程序集存储在 Azure SQL 数据库 服务器中后，你必须运行的 Transact-SQL CREATE FUNCTION 语句以引用程序集中的方法。


以下 Transact-SQL 代码块包含一组无关紧要的 SELECT 语句，以证明数据库系统包含程序集和函数的记录。最后，有 SELECT 语句调用了函数。


	SELECT a11.*, am2.*
		FROM           sys.assemblies       AS a11
		    INNER JOIN sys.assembly_modules AS am2 ON am2.assembly_id = a11.assembly_id
		WHERE a11.name = 'CreateAssemblyFunctions3';
	GO
	
	CREATE FUNCTION fnCompareStringsCaseSensitive
	    (@strA nvarchar(128), @strB nvarchar(128))
	    returns int
	    AS EXTERNAL NAME
	        CreateAssemblyFunctions3
	            .[CreateAssemblyFunctions3.SqlFunctionMethodsForSprocs]
	                .CompareCaseSensitiveNet;
	GO
	SELECT * FROM sys.objects WHERE name = 'fnCompareStringsCaseSensitive';
	
	 -- Use the new function.
	SELECT dbo.fnCompareStringsCaseSensitive('BLUE', 'blue') as returnedValue;
	GO


上面的 Transact-SQL 代码块以调用新函数的 SELECT 语句结尾。可以将该 SELECT 语句放入存储过程。


<!-- EndOfFile -->

<!---HONumber=69-->