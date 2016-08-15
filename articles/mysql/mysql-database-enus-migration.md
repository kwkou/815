<properties linkid="" urlDisplayName="" pageTitle="Migrate a database to MySQL Database on Azure – Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pool, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Using SSL encryption to access databases helps ensure that your access is secure. This article explains how to download and configure SSL certificates. MySQL Database on Azure currently supports the use of public keys to perform encryption and verification on the server side." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-migration/)
- [English](/documentation/articles/mysql-database-enus-migration/)

# Migrate a database to MySQL Database on Azure

Before you migrate to MySQL Database on Azure, evaluate whether your application database can do so successfully.

MySQL Database on Azure is compatible with MySQL 5.5 and MySQL 5.6, so the vast majority of applications will run successfully on MySQL Database on Azure without any changes. Applications should have a database reconnection mechanism to ensure a good level of fault tolerance and avoid the application crashing as a result of temporary inability to connect to the database. This is because even highly available cloud databases inevitably encounter situations where failover or server maintenance causes temporary inability to connect to the database. You should also use connection pooling and persistent connections to access the database whenever possible, particularly for applications with higher performance requirements. For more information, see [How to efficiently connect to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/).

Note also that MySQL Database on Azure does not support MyISAM format. (For more details, see “Why doesn’t MySQL Database on Azure support databases in MyISAM format?” in the [FAQs](/documentation/articles/mysql-database-serviceinquiry/).) In most situations, you can use the database normally by simply replacing the MyISAM database engine with InnoDB in the table creation code.

## Solution 1: Migration based on database import/export
If your system can accept a relatively long period of downtime (for example, 1-2 hours) caused by migration, you can use the relatively simple import/export method to carry out database migration.

1\. Sign in to the Azure Management Portal, and create a new MySQL server on MySQL Database on Azure. Complete the necessary configuration steps, such as the daily backup time. For the specific steps involved, see: http://www.windowsazure.cn/documentation/articles/mysql-database-get-started/#step1.


2\. Use the Azure Management Portal to create the target database that you want to migrate to on the newly created MySQL server. For the specific steps involved, see http://www.windowsazure.cn/documentation/articles/mysql-database-get-started/#step4.


3\. If you have multiple database accounts that need to access the original database, you must create the corresponding accounts on the new database server using Azure Management Portal.


4\. If the database is relatively large (for example, more than 1 GB), prepare a virtual machine (VM) in the same Azure data center, so that you can transfer the data to the VM first and then import it into the DB.


5\. On Azure, complete the deployment of the application’s components other than the database, such as websites.


6\. Once all the preparatory work is complete, start the migration. First close the application or run it in read-only mode (if supported), as this will avoid the creation of new data during the migration process.


7\. Export the application database from the current database server to a file. You can use a tool that you are familiar with, such as mysqldump or Workbench. The following example uses mysqldump to export a database: mysqldump --databases <database name\> --single-transaction --order-by-primary -r <backup file name\> --routines -h<server address\> -P<port number\> –u<user name\> -p<password\> 


8\. If the database file is relatively large, first transfer the database file to a VM (which should be in the same data center) on Azure. You can do this using a data transfer tool that you are familiar with, such as FTP or AzCopy. This method prevents the entire database transfer process from failing if the Internet connection drops out. If the backup file is very large, you can compress it before uploading.


9\. Import the database data into the target database. You can use a tool that you are familiar with, such as mysql.exe or Workbench. The following example uses mysql.exe to import the database.


9\.1 Connect to the newly created MySQL server on your client using mysql.exe (note: if you are not importing the data from a VM on Azure, you need to add the client to the IP safe list): mysql -h<server address\> -P<port number\> –u<user name\> -p<password\>
 

9\.2 Import the data from the SQL command line: source <backup file name\>; 


10\. Direct the newly deployed application to the migrated database and complete the remaining application migration steps.


## Solution 2: Migration based on data synchronization


Database import and export is relatively simple, but involves a relatively long period of downtime. If you cannot accept downtime during the migration process, such as for a company’s web applications, or you would like to complete a smooth migration in stages, try the data synchronization method instead. MySQL Database on Azure provides this function, which will allow your application and current database to continue working unaffected during synchronization.


1\. Synchronize the database to MySQL Database on Azure. You can configure the database server running on MySQL Database on Azure as a subordinate server. For the specific steps involved in configuring and synchronizing databases, see Configure SQL Data Sync to replicate to MySQL Database on Azure.



>[AZURE.NOTE] **Note: You will need to open up external access to the current database server. We strongly recommend that you configure SSL and only permit external access using SSL.**

2\. Deploy the new application on Azure and direct it to the newly created database on Azure.


[AZURE.WARNING] As the database on Azure is running in read-only mode at this time, the functionality of the application may be limited.

3\. Confirm that the database on Azure has reached a synchronized state. You can confirm the synchronization status based on the replication status and latency on the replication page.
![Migrate][1]

4\. Close the old application or make the application run in read-only mode (if it supports read-only mode).
	
5\. Stop the database synchronization replication. Click **Disable**, and then save on the page below.
![Migrate][2]

>[AZURE.NOTE] **Note: This operation restarts the database server.**

6\. Enable the new application.

## Common issues with database migration:
### An error message saying “Access denied; you need (at least one of) the SUPER privilege(s) for this operation” is reported during the TRIGGER, PROCEDURE, VIEW, FUNCTION, or EVENT import process.
Check whether the statement reporting the error uses DEFINER and uses users other than the current user (for example DEFINER=user@host). If this is the case, MySQL requires SUPER privileges to execute this statement. MySQL Database on Azure does not provide user SUPER privileges (see [Service limitations](http://www.windowsazure.cn/documentation/articles/mysql-database-operation-limitation/)), causing an error. To resolve this error, delete DEFINER from the statement and use the default current user.

### Given that MySQL Database on Azure’s Management Portal only supports configuring user read/write privileges for the entire database, will the migration still succeed if my existing database has more detailed user privilege settings?
Yes. Although the Azure Management Portal and Windows PowerShell/REST API only supports setting read/write privileges for the entire database when you create users or databases, you can use the **grant** command to fine-tune user privilege settings.

<!--Image references-->

[1]: ./media/mysql-database-migration/migration1-en.png
[2]: ./media/mysql-database-migration/migration2-en.png

<!---HONumber=Acom_0218_2016_MySql-->