<properties linkid="" urlDisplayName="" pageTitle="Understand the service limitations of MySQL Database on Azure – Azure cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, service restrictions and limitations, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="This article helps you understand the service limitations of MySQL Database on Azure during the period of this public preview version. Contact technical support if you have any questions about specific operations." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-operation-limitation/)
- [English](/documentation/articles/mysql-database-enus-operation-limitation/)

#Service limitations of MySQL Database on Azure

MySQL Database on Azure currently has the following limitations. If you have any questions, contact technical support.


1.	As MySQL Database on Azure uses customized user account authentication plug-ins, user accounts can only be created in the Azure portal. Users cannot directly create database user accounts using the SQL command line.
2.	You can create databases only by using the management portal. Directly creating databases by using the SQL command line is not supported. 
3.	Users do not have shutdown permissions, so users cannot shut down MySQL database servers.
4.	Users do not have super permissions, and so cannot change any global variables, for example. [Find out more about MySQL 5.5 user permissions](https://dev.mysql.com/doc/refman/5.5/en/privileges-provided.html)
5.	Users do not have file permissions. [Find out more about MySQL 5.5 user permissions](https://dev.mysql.com/doc/refman/5.5/en/privileges-provided.html).
6.	The MySQL built-in system table does not support write permissions, and read/write permissions are currently not supported for the following tables:

	* columns_priv
	* db
	* general_log
	* host
	* ndb_binlog_index
	* plugin
	* procs_priv
	* servers
	* slow_log
	* tables_priv
	* user
	* proxies_priv

7.	Data replication currently supports only data syncing between local and cloud servers, a feature that makes it easier for users to build hybrid cloud scenarios.
8.	The MyISAM storage engine is currently not supported.



<!--HONumber=81-->