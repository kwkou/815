<properties linkid="" urlDisplayName="" pageTitle="Connect efficiently to MySQL Database on Azure – Azure cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pool, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Making sensible use of connection pooling to access MySQL Database on Azure will optimize performance. This article explains how to use connection pooling to more effectively access MySQL Database on Azure and provides sample code that uses Java and PHP as examples for your reference." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-connection-pool/)
- [English](/documentation/articles/mysql-database-enus-connection-pool/)

# Connect efficiently to MySQL Database on Azure<sup style="color: #a5ce00; font-weight: bold; text-transform: uppercase; font-family: 'Arial'; font-size: 20px;" class="wa-previewTag"></sup>


Database connections are a limited resource, so making sensible use of connection pooling to access MySQL Database on Azure optimizes performance.

## Access databases that use connection pooling (recommended)##
Managing database connections can have a significant impact on the performance of the application as a whole. To optimize the performance of your program:
- Reduce the number of times connections are established.
- Avoid specifying the time for establishing connections in key code paths.
- Use database connection pooling or persistent connections to connect to MySQL Database on Azure.

Database connection pooling handles the creation, management, and allocation of database connections. When a program requests a database connection, it prioritizes the allocation of existing idle database connections, rather than the creation of a new connection. Once the program has finished using the database connection, the connection is recovered in preparation for further use, rather than simply being closed down.

For your reference, see this [piece of sample code that uses Java as an example](http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/documents/MySQLConnectionPool.java). And for more information, see the [Apache common DBCP](http://commons.apache.org/proper/commons-dbcp/).

>[AZURE.NOTE]**The server configures a timeout mechanism, closing a connection that has been in an idle state for some time, in order to free up resources. Be sure to set up the authentication system to ensure the effectiveness of persistent connections when you are using them. For more information, see: [How to configure authentication systems on the client side to ensure the effectiveness of persistent connections.](/documentation/articles/mysql-database-validationquery/)**

## Access databases that use persistent connections (recommended)##
Use persistent connections in PHP. The concept of persistent connections is similar to that of connection pooling. It is important to note that PHP currently has three types of drivers. While the MySQLi driver type does not support persistent connection, the other two types of drivers do.

Replacing short connections with persistent connections requires only minor changes to the code, but has a major effect in terms of improving performance in many typical application scenarios.

For more information, see [Use PDO to create persistent connections](http://php.net/manual/en/pdo.connections.php) and  [Use the MySQL engine to create persistent connections](http://php.net/manual/en/function.mysql-pconnect.php).

## Access databases by using wait and retry mechanisms with short connections##
If you use short connections and experience connection failures when approaching the upper limit on the number of concurrent connections, you can try connecting multiple times. You can set an appropriate wait time, with a shorter wait time after the first attempt. After this, waiting for random events can be performed multiple times.

<!---HONumber=Acom_0218_2016_MySql-->