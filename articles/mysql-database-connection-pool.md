<properties linkid="" urlDisplayName="" pageTitle="Connect efficiently to MySQL Database on Azure – Microsoft Azure cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pool, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Making sensible use of connection pooling to access MySQL Database on Azure will optimize performance. This article explains how to use connection pooling to more effectively access MySQL Database on Azure and provides sample code that uses Java and PHP as examples for your reference." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="07/27/2015"/>
# Connect efficiently to MySQL Database on Azure<sup style="color: #a5ce00; font-weight: bold; text-transform: uppercase; font-family: '微软雅黑'; font-size: 20px;" class="wa-previewTag"></sup>



Database connections are a limited resource, so making sensible use of connection pooling to access MySQL Database on Azure will optimize performance. This article explains how to use connection pooling or persistent connections to more effectively access MySQL Database on Azure.

## Access databases that use connection pooling (recommended)##
As MySQL Database on Azure performs more authentication tasks when it establishes connections, the result can be connections that require even greater overhead than the local database. Managing database connections can have a significant impact on the performance of the application as a whole. To optimize the performance of your program, the goals should be to reduce the number of times connections are established and to avoid putting the time for establishing connections in key code paths. We strongly recommend that you use database connection pooling or persistent connections to connect to MySQL Database on Azure. Database connection pooling handles the creation, management, and allocation of database connections. When a program requests a database connection, it prioritizes the allocation of existing idle database connections, rather than the creation of a new connection. Once the program has finished using the database connection, the connection is recovered in preparation for further use, rather than simply being closed down.

This article explains this further by providing a [piece of sample code that uses Java as an example](http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/documents/MySQLConnectionPool.java) for your reference. You can also refer to [Apache common DBCP](http://commons.apache.org/proper/commons-dbcp/) to find out more.

## Access databases that use persistent connections (recommended)##
We recommend that you use persistent connections in PHP. The concept of persistent connections is similar to that of connection pooling. It is important to note that PHP currently has three types of drivers. While MySQLi does not support persistent connection, the other two types of drivers do.

Read [Use PDO to establish persistent connections](http://php.net/manual/en/pdo.connections.php); read [Use MySQL engines to establish persistent connections](http://php.net/manual/en/function.mysql-pconnect.php).

Replacing short connections with persistent connections requires only minor changes to the code, but has a major effect in terms of improving performance in many typical application scenarios.

## Access databases by using wait and retry mechanisms with short connections##
Given resource limitations, we strongly recommend that you use database pooling or persistent connections to access databases. However, if you do use short connections and experience connection failures when approaching the upper limit on the number of concurrent connections, we recommend that you try connecting multiple times. You can set an appropriate wait time, with a shorter wait time after the first attempt. After this, waiting for random events can be performed multiple times.


<!--HONumber=81-->