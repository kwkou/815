<properties linkid="" urlDisplayName="" pageTitle="Understanding the Service Limitations of MySQL Database on Azure – Microsoft Azure Cloud" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,服务限制,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Helps you to understand the service limitations of MySQL Database on Azure during the period of this public preview version. Please contact technical support if you have any questions about specific operations." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="04/29/2015"/>

#Understanding the Service Limitations of MySQL Database on Azure

MySQL Database on Azure currently has the following limitations. If you have any questions, please contact technical support.


1.	As MySQL Database on Azure uses customized user account authentication plugins, user accounts can only be created in the management portal. Users cannot directly create database user accounts using the SQL command line.
2.	You can only create databases using the management portal; directly creating databases with the SQL command line is not supported. 
3.	Users do not have shutdown **permissions**, so users cannot shut down MySQL database servers.
4.	Users do not have super permissions, and so cannot change any global variables, for example. [Find Out More About MySQL 5.5 User Permissions](https://dev.mysql.com/doc/refman/5.5/en/privileges-provided.html).
5.	Users do not have file permissions. [Find Out More About MySQL 5.5 User Permissions](https://dev.mysql.com/doc/refman/5.5/en/privileges-provided.html).
6.	The MySQL built-in system table does not support write permissions, and read-write permissions are currently not supported for the following tables:

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

7.	Data replication currently only supports data syncing between local and cloud servers, a feature that makes it easier for users to build hybrid cloud scenarios.
8.	The MyISAM storage engine is currently not supported.


