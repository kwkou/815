<properties
    pageTitle="JDBC 如何连接 SQL Azure 数据库"
    description="JDBC 如何连接 SQL Azure 数据库"
    service=""
    resource="sqldatabase"
    authors="Yu Tao"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="SQL Database, JDBC, Java"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="sql-database-aog"
    ms.date=""
    wacn.date="05/16/2017" />

# JDBC 如何连接 SQL Azure 数据库

使用 JAVA 代码连接 Azure SQL Database 时产生了 SSL 错误，对于此问题大多数用户都是因为不知如何编写 JDBC 连接字符串而产生的，以下为相关示例代码，供您参考：

    package sqldbtest;

    import java.sql.*;
    import com.microsoft.sqlserver.jdbc.*;

    public class sqldbconn {

        public static void main(String[] args) {
            // TODO Auto-generated method stub
            String connectionString =
                        "jdbc:sqlserver://[server name].database.chinacloudapi.cn:1433;"
                        + "database=[database name];"
                        + "user==[database name]@ [server name];"
                        + "password=xxxxxxxxxxx;"
                        + "encrypt=true;"
                        + "trustServerCertificate=true;"
                        + "hostNameInCertificate=*.database.chinacloudapi.cn;"
                        + "loginTimeout=30;";

            // Declare the JDBC objects.
            Connection connection = null;
            Statement statement = null;
            ResultSet resultSet = null;
            PreparedStatement prepsInsertPerson = null;
            PreparedStatement prepsUpdateAge = null;

            try {
                // INSERT two rows into the table.
                // ...
                // TRANSACTION and commit for an UPDATE.
                // ...
                // SELECT rows from the table.
                // ...
                                connection = DriverManager.getConnection(connectionString);
                System.out.println("Successful");
                System.out.println("1");
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

[AZURE.NOTE] 完善连接字符串 **connectionString** 中相关参数，根据给出的格式进行修改。