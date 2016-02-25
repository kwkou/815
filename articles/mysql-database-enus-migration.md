<properties linkid="" urlDisplayName="" pageTitle="Migrate a database to MySQL Database on Azure – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, connection pool, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Using SSL encryption to access databases helps ensure that your access is secure. This article explains how to download and configure SSL certificates. MySQL Database on Azure currently supports the use of public keys to perform encryption and verification on the server side." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="01/21/2016"/>

# Migrate a database to MySQL Database on Azure
This article summarizes the methods for migrating user databases to MySQL Database on Azure and issues that must be considered.

##Migratability of applications
We recommend that you evaluate the migratability of your application database before you migrate.

MySQL Database on Azure is compatible with MySQL 5.5 and MySQL 5.6, so the vast majority of applications will run successfully on MySQL Database on Azure without any changes. Of course, to run your application even better on MySQL Database on Azure, we recommend that applications have a database reconnection mechanism to ensure a good level of fault tolerance and avoid the application crashing as a result of temporary inability to connect to the database. This is because even highly available cloud databases inevitably encounter situations where failover or server maintenance causes temporary inability to connect to the database. We also recommend using connection pooling and persistent connections to access the database whenever possible, particularly for applications with higher performance requirements. See [How to efficiently connect to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool) for details.

Another point that is important to note is that MySQL Database on Azure does not support the old MyISAM engine. You may refer to “Why doesn’t MySQL Database on Azure support databases in MyISAM format?” in the [FAQs](/documentation/articles/mysql-database-serviceinquiry). In most situations, you can use the database normally by simply replacing the MyISAM database engine with InnoDB in the table creation code.

##Solution 1: Migration based on database import/export
If your system can accept a relatively long period of downtime (e.g. 1-2 hours) caused by migration, you can use the relatively simple import/export method to carry out database migration. Specific procedure

1\.Sign in to the Azure Management Portal, create a new MySQL server on MySQL Database on Azure and complete the necessary configuration steps, such as the daily backup time. For the specific steps involved, see http://www.windowsazure.cn/documentation/articles/mysql-database-get-started#step1.


2\. Use the Azure Management Portal to create the target database that you want to migrate to on the newly created MySQL server. For the specific steps involved, see http://www.windowsazure.cn/documentation/articles/mysql-database-get-started#step4.


3\. If you have multiple database accounts that need to access the original database, you must create the corresponding accounts on the new database server using Azure Management Portal.


4\. If the database is relatively large (i.e. over 1 GB), we recommend that you prepare a VM in the same Azure data center, so that you can transfer the data to the VM first and then import it into the DB.


5\. On Azure, complete the deployment of the application’s components other than the database, such as websites.


6\. Once all the preparatory work is complete, start the migration. We recommend that you first close the application or run it in read-only mode (if supported), as this will avoid the creation of new data during the migration process.


7\. Export the application database from the current database server to a file. You can use a tool that you are familiar with, such as mysqldump or Workbench. The example below uses mysqldump to export a database.


	mysqldump --databases <database name> --single-transaction --order-by-primary -r <backup file name> --routines -h<server address> -P<port number> –u<user name> -p<password> 


8\. If the database file is relatively large, we recommend that you first transfer the database file to a VM (which should be in the same data center) on Azure. You can do this using a data transfer tool that you are familiar with, such as FTP or AzCopy. This method prevents the entire database transfer process from failing if the Internet connection drops out. If the backup file is very large, you can compress it before uploading.


9\. Import the database data into the target database. You can use a tool that you are familiar with, such as mysql.exe or Workbench. The example below uses mysql.exe to import the database.



9\.1 Connect to the newly created MySQL server on your client using mysql.exe (note: if you are not importing the data from a VM on Azure, you need to add the client to the IP whitelist):

	mysql -h<server address> -P<port number> –u<user name> -p<password> 

9\.2 Import the data from the SQL command line:
	
	source <backup file name>; 


10\. Direct the newly deployed application to the migrated database and complete the remaining application migration steps.


##Solution 2: Migration based on data synchronization


The database migration method above is relatively simple, but has the disadvantage that it involves a relatively long period of downtime, and also requires you to be highly proficient at completing every step of the migration process and able to predict the amount of time required very accurately, otherwise it will have a major impact on your continuity of service. If you cannot accept downtime during the migration process, such as for a company’s web applications, or you would like to complete a smooth migration in stages, we recommend that you use the data synchronization method to synchronize a previously generated database to Azure. MySQL Database on Azure provides this function, which will allow your application and current database to continue working unaffected during synchronization.


Recommended migration process:

1\. Synchronize the database to MySQL Database on Azure. You can configure the database server running on MySQL Database on Azure as a slave server. For the specific steps involved in configuring and synchronizing databases, see Configure SQL Data Sync to replicate to MySQL Database on Azure.



>[AZURE.NOTE] **Note: You will need to open up external access to the current database server. We strongly recommend that you configure SSL and only permit external access using SSL.**

2\. Deploy the new application on Azure and direct it to the newly created database on Azure.


Warning: As the database on Azure is running in read-only mode at this time, the functionality of the application may be limited.

3\. Confirm that the database on Azure has reached a synchronized state. You can confirm the synchronization status based on the replication status and replication latency on the replication page.
![Migrate][1]

4\. Close the old application or make the application run in read-only mode (if it supports read-only mode).
	
5\. Stop the database synchronization replication. You just need to click on “Disable” and then save on the page below.
![Migrate][2]
>[AZURE.NOTE] **Note: This operation will restart the database server**

6\. Enable the new application.

## Common issues with database migration:
###An error message saying “Access denied; you need (at least one of) the SUPER privilege(s) for this operation” is reported during the TRIGGER, PROCEDURE, VIEW, FUNCTION, or EVENT import process.
Check whether the statement reporting the error uses DEFINER and uses users other than the current user, for example DEFINER=user@host. If this is the case, MySQL requires SUPER privileges to execute this statement, and as MySQL Database on Azure does not provide user SUPER privileges, (see [Service limitations](http://www.windowsazure.cn/documentation/articles/mysql-database-operation-limitation)), this will cause running this statement to fail. You just need to delete DEFINER from the statement and use the default current user.

###Given that MySQL Database on Azure’s Management Portal only supports configuring user read/write privileges for the entire database, will the migration still succeed if my existing database has more detailed user privilege settings?
Yes. Although our Management Portal and Windows PowerShell/REST API only supports setting read/write privileges for the entire database when you create users or databases, you can use the “grant” command to fine-tune user privilege settings.

<!--Image references-->

[1]: ./media/mysql-database-migration/migration1.png
[2]: ./media/mysql-database-migration/migration2.png

<!---HONumber=Acom_0218_2016_MySql-->