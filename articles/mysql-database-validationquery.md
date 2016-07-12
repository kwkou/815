<properties linkid="" urlDisplayName="" pageTitle="如何在客户端配置验证机制确认长连接有效性- Azure 微软云" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，连接池，connection pool, Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="
配置验证机制，保障数据库的访问速度" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-validationquery/)
- [English](/documentation/articles/mysql-database-enus-validationquery/)

# 如何在客户端配置验证机制确认长连接有效性<sup style="color: #a5ce00; font-weight: bold; text-transform: uppercase; font-family: '微软雅黑'; font-size: 20px;" class="wa-previewTag"></sup>


在[如何高效连接到MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/)一文中谈到，为了更好地使用数据库的连接资源，我们推荐您使用连接池或长连接的方法进行数据库访问。但需要注意的是连接池或长连接也存在时效性。这是因为服务器会设置超时机制，如果一个连接在一段时间内处于闲置状态，服务器就会关闭这个链接，以释放不必要的资源占用。这就造成了如果客户端长时间在idle状态，再次访问数据库较慢的问题，相当于客户端与服务器间重新建立了连接请求。因此，为了保障在使用过程中，连接的有效性，本文以Tomcat JDBC 连接池为例，介绍如何在客户端配置验证机制确认长连接的有效性。

通过设定testOnBorrow参数，在有新的请求时，如果连接池中有闲置的可用连接，在返回这个闲置连接之前，连接池会自动验证这个连接的有效性，如果有效，直接返回，如果无效，连接池会回收这个无效连接，重新建立一个新的有效连接并返回。这样会有效地保障数据库的访问速度。

具体设置用户可参考[JDBC Connection Pool官方介绍文档](https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Common_Attributes)。主要需要配置以下三个参数： TestOnBorrow (设为ture), ValidationQuery (设为 SELECT 1), ValidationQueryTimeout (设为1)，具体示例代码如下：



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



