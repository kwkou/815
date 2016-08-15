<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="v-chuw" solutions="" manager="RongYu" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-point-in-time-restore/)
- [English](/documentation/articles/mysql-database-enus-point-in-time-restore/)

# Back up and restore a MySQL database on Azure

The **MySQL Database on Azure** option on the Azure portal supports full backups and incremental backups. Full backups are also known as snapshot backups. Space that's used for backups does not count toward the storage limit of the database. You can restore from any time point in the last seven days or from any specific snapshot backup that's stored in the system.

## **Full backups**

**MySQL Database on Azure** automatically backs up your MySQL database every day at the time that you specified. This is a full backup. Full backups are retained for 30 days.

You can also instantly perform a manual full backup. You can generate a total of 10 manual full backups. Manual full backups are not automatically removed. When you reach the limit, you will need to delete a manual full backup before you can create a new manual full backup.

Here's the instant manual backup process:

1. Sign in to the [Azure Management Portal](http://manage.windowsazure.cn/), and click on **MYSQL DATABASE ON AZURE** in the services list in the left pane.
2.	In the server list, click the database server that you want to back up to open the server interface, then click **Backup**.
3.	Click on **Instant backup**, and confirm the operation.

## **Incremental backups**

**MySQL Database on Azure** automatically backs up newly added sections or changes to the database for the last seven days in order to enable rollback based on any point in time. You don't need to do anything for this to happen.

## **Restore the database to any point in time**

MySQL Database on Azure supports restoring to any time point in the last 7 days.

Process for restoring to a particular time point:

1. Sign in to the [Azure Management Portal](http://manage.windowsazure.cn/), and click **MYSQL DATABASE ON AZURE** in the services list in the left pane.
2. In the server list, click the database server that you want to restore to open the server interface, and then click on **DASHBOARD**.
![Restore to any point in time][1]
3. Click the **RESTORE** button in the task menu at the bottom. A dialogue box will pop up.
4. The **RESTORE TYPE** is already set to **Rollback to a point in time** by default. Select the time point, the target database server instance, and the version that you want to restore. Click on the **Finish** button.
![Restore to any point in time][2]
5. Go back to the server list page to confirm that the server was successfully restored and that the status is **Running**. The data recovery process might take from a few minutes to several hours, depending on the time point that you choose and the amount of data that was added to the database server that day.

You can access the new server instance using the original server instance account. However, you will need to change the server nameprefix in the account to the new server name.

## **Restore a full backup to a new instance**

With **MySQL Database on Azure**, you can restore a full backup to a new instance. The specific procedure for this is as follows:

1.	Sign in to the [Azure Management Portal](http://manage.windowsazure.cn/), and click **MYSQL DATABASE ON AZURE** in the services list in the left pane.
2.	In the server list, click the database server that you want to restore to open the server interface, and then click **BACKUPS**.
3.	Select the full backup that you want to restore, and click the **RESTORE** button.
![Restore a full backup to a new instance][3]
4.	In the dialogue box that opens, the **RESTORE TYPE** is set to **Rollback to a full backup** by default, the **SELECTED BACKUP** is automatically set to the full backup that you selected, and **RESTORE TO** is set to **New Server** by default. Enter the name and version of the new server instance, and then click the **Finish** button.
![Restore a full backup to a new instance][4]
5.	Go back to the server list to confirm that the server was successfully created and that the status is **Running**. This process is generally completed within two minutes. You can access the new server instance by using the original server instance account. However, you will need to change the server name prefix in the account to the new server name.

## **Restore a full backup to the original instance**

With **MySQL Database on Azure**, you can restore a full backup to the original instance. The specific procedure for this is as follows:

1.	Sign in to the [Azure Management Portal](http://manage.windowsazure.cn/), and click **MYSQL DATABASE ON AZURE** in the services list in the left pane.
2.	In the server list, click the database server that you want to restore to open the server interface, then click **BACKUPS**.
3.	Select the full backup that you want to restore, and click the **RESTORE** button.
4.	Select **Current Service** from the **RESTORE TO** options in the dialogue box that pops up. At the same time, you must confirm that **I understand it will take a few minutes to restore the server, during which the database server could not be accessed. ** Click the **Finish** button.
![Restore a full backup to the original instance][5]
5.	Go back to the server list to confirm that the status of the server is **Running**. This indicates that the server was successfully restored.

<!--Image references-->

[1]: ./media/mysql-database-point-in-time-restore/Restore1-en.jpg
[2]: ./media/mysql-database-point-in-time-restore/Restore2-en.jpg
[3]: ./media/mysql-database-point-in-time-restore/Restore3-en.jpg
[4]: ./media/mysql-database-point-in-time-restore/Restore4-en.jpg
[5]: ./media/mysql-database-point-in-time-restore/Restore5-en.jpg

<!---HONumber=Acom_0418_2016_MySql-->
