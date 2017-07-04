<properties linkid="" urlDisplayName="" pageTitle="Configure a verification mechanism on the client to confirm the effectiveness of persistent connections – Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pool, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Configure a verification mechanism to ensure database access speeds" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-validationquery/)
- [English](/documentation/articles/mysql-database-enus-validationquery/)

# Verify the effectiveness of persistent connections<sup style="color: #a5ce00; font-weight: bold; text-transform: uppercase; font-family: '微软雅黑'; font-size: 20px;" class="wa-previewTag"></sup>


To make better use of the database’s connection resources, use the connection pooling or persistent connection methods to access the database. For more information, see [How to connect efficiently to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/). However, time-efficiency is also an issue for connection pooling and persistent connections. The server configures a timeout mechanism, closing a connection that has been in an idle state for some time, in order to free up resources. When the client accesses the database again, this is equivalent to creating a new connection request between the client and the server. To ensure the effectiveness of connections during the process of using them, configure a verification mechanism on the client. For example, as shown below, you can use Tomcat JDBC connection pooling to configure this verification mechanism.

By setting the TestOnBorrow parameter, when there is a new request, the connection pool automatically verifies the effectiveness of any available connections that are idle before returning the idle connections. If such a connection is effective, it will be directly returned; if it is not effective, the connection pool will withdraw the connection, create a new, effective connection, and return it. This ensures the speed of access to the database.

For the specific settings, see the [JDBC Connection Pool official introduction document](https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Common_Attributes). You mainly need to set the following three parameters: TestOnBorrow (set to true), ValidationQuery (set to SELECT 1), and ValidationQueryTimeout (set to 1). Specific sample code is shown below:



 	public class SimpleTestOnBorrowExample {
      	public static void main(String[] args) throws Exception {
          	PoolProperties p = new PoolProperties();
          	p.setUrl("jdbc:mysql://localhost:3306/mysql");
          	p.setDriverClassName("com.mysql.jdbc.Driver");
          	p.setUsername("root");
          	p.setPassword("password");
                // The indication of whether objects will be validated by the idle object evictor (if any). 
                // If an object fails to validate, it will be dropped from the pool. 
                // NOTE - for a true value to have any effect, the validationQuery or validatorClassName parameter must be set to a non-null string. 
          	p.setTestOnBorrow(true); 
                
                // The SQL query that will be used to validate connections from this pool before returning them to the caller.
                // If specified, this query does not have to return any data, it just can't throw a SQLException.
          	p.setValidationQuery("SELECT 1");
                
                // The timeout in seconds before a connection validation queries fail. 
                // This works by calling java.sql.Statement.setQueryTimeout(seconds) on the statement that executes the validationQuery. 
                // The pool itself doesn't timeout the query, it is still up to the JDBC driver to enforce query timeouts. 
                // A value less than or equal to zero will disable this feature.
          	p.setValidationQueryTimeout(1);
                // set other usefull pool properties.
          	DataSource datasource = new DataSource();
          	datasource.setPoolProperties(p);

         	 Connection con = null;
          	try {
            	con = datasource.getConnection();
            	// execute your query here
          	} finally {
           	 if (con!=null) try {con.close();}catch (Exception ignore) {}
          	}
      	}
  	}

<!---HONumber=Acom_0218_2016_MySql-->