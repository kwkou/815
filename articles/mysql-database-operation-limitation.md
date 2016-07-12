<properties linkid="" urlDisplayName="" pageTitle="了解MySQL 数据库 on Azure服务限制- Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,服务限制,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="帮助您了解目前MySQL 数据库 on Azure 公共预览版期间的服务限制。如果您对某些操作存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-operation-limitation/)
- [English](/documentation/articles/mysql-database-enus-operation-limitation/)

#了解MySQL 数据库 on Azure服务限制

MySQL 数据库 on Azure目前有如下几点限制，如果您有任何疑问，欢迎联系技术支持。


1.	由于MySQL 数据库 on Azure应用了定制的用户账号认证plugin，用户账号的创建只能在管理门户上完成，用户不能直接通过SQL命令行创建新的数据库用户账号。
2.	只能通过管理门户创建数据库，不支持SQL命令行直接创建数据库。 
3.	用户没有shutdown权限，用户无法shutdown MySQL数据库服务器。
4.	用户没有Super权限，列如用户不能更改任意一个全局变量。[了解更多MySQL 5.5 用户权限](https://dev.mysql.com/doc/refman/5.5/en/privileges-provided.html)
5.	用户没有File 权限，[了解更多MySQL 5.5 用户权限](https://dev.mysql.com/doc/refman/5.5/en/privileges-provided.html)。
6.	MySQL built-in system table 不支持写权限，目前以下table不支持读写权限。

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

7.	目前Data Replication仅支持本地、云服务器间的数据同步，方便用户构建混合云场景
8.	目前不支持MyISAM存储引擎


