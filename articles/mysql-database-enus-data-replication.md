<properties linkid="" urlDisplayName="" pageTitle="How to Configure Data Sync to Replicate to MySQL Database on Azure – Azure Cloud" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,服务限制,数据复制，Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Helps you to understand how to use the data sync function to replicate local MySQL instances to the cloud." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="09/16/2015"/>

#How to Configure Data Sync to Replicate to MySQL Database on Azure
> [AZURE.SELECTOR]
- [Chinese Version](/documentation/articles/mysql-database-data-replication)
- [English Version](/documentation/articles/mysql-database-enus-data-replication)

MySQL Database on Azure supports slave server mode and standard MySQL data replication. You can use this feature to automatically sync database data from a MySQL server running locally or in other locations, to a server running on MySQL Database on Azure.

##Configuration steps
1.	Confirm that the system variable lower_case_table_names on the master MySQL server is set to 1. If this is not the case, then you must set it to 1. This is because MySQL database replication requires the value of this parameter to be consistent between the master and slave servers, and this parameter is set to 1 on MySQL On Azure. mysql> SET GLOBAL lower_case_table_names = 1;
2.	Set the master server to read-only mode: mysql> FLUSH TABLES WITH READ LOCK; mysql> SET GLOBAL read_only = ON;
3.	Run the SQL command “show master status” on the master server, in order to ascertain the current binary log file name and offset. The results returned should be similar to this: ![Return to results](./media/mysql-database-data-replication/packet.png)

4.	Export the databases for all users on the master server, for example by using the mysqldump tool. mysqldump --databases <database name> --single-transaction --order-by-primary -r <backup file name> --routines -h<server address> -P<port number> –u<username> -p<password> Note: Databases built into MySQL servers, including the mysql library and test library, do not need to be exported.
5.	Once the database has been exported, change the master MySQL server setting back to read/write mode: mysql> SET GLOBAL read_only = OFF; mysql> UNLOCK TABLES;  
6.	Create an account on the master MySQL server for data replication use, and set up the permissions. CREATE USER '<your user>'@'%' IDENTIFIED BY '<your password>'; GRANT REPLICATION SLAVE ON *.* TO '<your user>'@'%';
7.	Log into the Azure management portal and create a new MySQL server on MySQL Database on Azure.
8.	Create individual databases for all users on the master server on the newly created MySQL server.
9.	Create the required user accounts on the newly created MySQL server. This is necessary because user account information cannot be replicated.
10.	Import the user database data exported from the master server into the newly created MySQL server. If the database file is very large, we recommend that you upload the file to a virtual machine on Azure, and then import it into the MySQL server from the virtual machine. The virtual machine should be in the same data center as the newly created MySQL server. The specific steps are listed below.

	1) Upload the mysql.exe tool to the virtual machine.

	2) Upload the file exported form the database on to the virtual machine. If the backup file is very large, you can compress it before uploading.

	3) Log into the virtual machine, and connect to the newly created MySQL server using mysql.exe: mysql -h<server address> -P<port number> –u<username> -p<password>.

	4) Run the following SQL command to import data within the backup file. source <backup file name>.

	5) Repeat c) -> f), until all the data in the user databases have been imported into the MySQL server.

11.	Making the newly created MySQL server the slave server

	1) Select the newly created MySQL server, and click on the “Replicate” page.

	2) Change the role to “Slave server,” and then enter the master server parameters.

	i. For the master server binary log file name and offset, please enter the results obtained in step 2.

	ii. If you are using SSL links, please select the enable option in the locations using the SSL links. Next, open the master server CA certificate and copy the entire contents into the input box of the master server CA certificate. 3) Click on save once all the details are correctly configured.

>[AZURE.NOTE]**Note: We strongly recommend using SSL to ensure that your data is secure.**

![Configuration process](./media/mysql-database-data-replication/replicationsetting.png)


12.	Once the configuration is successful, the replication status at the bottom should say “replicating”. ![Configuration process](./media/mysql-database-data-replication/replicationstatus.png)

>[AZURE.NOTE] **Note: Once the replication role of the MySQL server is set to slave server, the server will be in read-only mode. - Once the replication role of the MySQL server is set to slave server, none of the master server parameters on the replication page will be editable, except for the role. If there is an input error, you must set the replication role to disabled and then reconfigure the slave server parameters. - We recommend setting the binlog_format for the master server to Mixed or Row, in order to avoid causing data replication errors due to the use of unsafe statements such as sysdate ().**


##Data replication restrictions
1. Changes on the master server to accounts and permissions are not replicated. If you created an account on the master server and this account needs to access the slave server, then you will need to create the same account yourself on MySQL Database on Azure.

2. The master and slave server versions must be the same. For example, both must be MySQL 5.5, or both must be MySQL 5.6.

##Solving data replication errors
If replication stops because it encounters a problem of any kind, the replication status will change to “replication error.” You can find details of the error by looking at the replication error field. Common causes of replication errors include: The value of the max_allowed_packet parameter on the slave server being less than the value of the same parameter on the master server. This parameter determines the maximum permitted DML size for MySQL servers. If the value of the parameter is smaller on the slave server than on the master server, some DMLs may run successfully on the master side, but fail to run on the slave server, causing an error. Please ensure that the max_allowed_packet values are consistent between the master and slave servers.

- When the replication role is changed to slave server, there is a master server parameter input error. This makes it impossible for the slave server to connect to the master server.

- Data is consistent between the master and slave servers. For example, replication attempts to insert a record into the slave server that already exists. There are several possible causes of this error:

	1) Some DMLs on the master server were not recorded into the binary log file. For example, SET sql_log_bin=0 is executed before the DML is executed on the master server.

	2) Before the replication role is changed to the slave server, faulty write operations are performed on it.

	3) There are input errors for the binary log file name or offset when changing the replication role to the slave server.

If data replication errors do occur, please solve them using the following process:

1.	Use the Azure management portal to change the replication role of the MySQL to disabled. This will put the MySQL in read-only mode.

2.	You can then determine the cause of the error by looking at the replication error field, and resolve the issue. For example, you can set a max_allowed_packet value consistent with that on the master server, and change the record on the slave server that is causing the replication failure.

3.	Use the Azure management portal to change the replication role of the MySQL back to the slave server.


	1) The master server binary log file name and offset are the master server binary file name and offset that were previously replicated and executed. If there were previously no input errors with the binary log file name or offset, we do not recommend making any changes.

	2) For security reasons, the master server password and master server CA certificate that were previously entered will not be displayed at this time. If you do not make any changes, MySQL will continue to use the previous password and CA certificate.

	3) The other master server parameter fields will show the corresponding parameter values that were previously entered. If there are no errors, you do not need to make any changes.

