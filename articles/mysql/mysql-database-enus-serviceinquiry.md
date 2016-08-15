<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

# Service consulting
> [AZURE.SELECTOR]
- [All issues](/documentation/articles/mysql-database-enus-tech-faq/)
- [Service consulting](/documentation/articles/mysql-database-enus-serviceinquiry/)
- [Connection issues](/documentation/articles/mysql-database-enus-connectioninquiry/)
- [Security consulting](/documentation/articles/mysql-database-enus-securityinquiry/)
- [Compatibility issues](/documentation/articles/mysql-database-enus-compatibilityinquiry/)

### **Is the amount of storage space for data backups limited?**
  
Data backups do not count towards your storage limit.

### **Is there a limit to the number of databases on a single server?**

Users can create multiple databases on a single MySQL server, and there is no limit to the number of databases that can be created. However, multiple databases share server resources, and performance requirements rise as the number of databases grows. We recommend that you create multiple MySQL servers.
	
### **What are the current limitations of MySQL Database on Azure?**
	
For details, see [service limitations of MySQL Database on Azure](/documentation/articles/mysql-database-operation-limitation/).

### **Why doesn’t MySQL Database on Azure support databases in MyISAM format?**

The reasons include the following:

- MyISAM has flaws in terms of data integrity protection, and these flaws could cause database data to be damaged or even lost. Moreover, because these flaws are primarily due to design faults, there is no way to repair them without breaking compatibility.
- Because the MyISAM input/output (I/O) operations are not the optimal solution for storage on Azure, MyISAM offers little performance advantage over InnoDB.
- MyISAM requires significant manual repair work if data are damaged, and it cannot be adapted to a platform as a service (PaaS) operating model.
- The cost of migrating from MyISAM to InnoDB is minimal. In most applications, migration requires only changes to the table creation code.
- The development of MySQL is also moving towards InnoDB. For example, MySQL 5.7 doesn’t use MyISAM at all, and the system database has also been transitioned to InnoDB.

### **Why is the default size for new, empty database servers set to 530 MB? Why is the displayed amount of storage space used for the database larger than the amount of storage space actually used?**
	
For performance reasons, we use two 256 MB log files for new database instance configurations. Consequently, the storage space usage figure that you see in the Azure portal includes the size of the log files. However, the size of the log files will not change during usage.
	
### **Does MYSQL Database on Azure support setting privileges via the command line?**

Yes. While our [Management Portal](https://manage.windowsazure.cn/) and the PowerShell command line only support setting read/write privileges for the entire database when creating users or databases, you can use the **grant** command to fine-tune user privilege settings.

### **What system time does MySQL Database on Azure currently use? How can I change it?**
MySQL on Azure currently defaults to using UTC (Coordinated Universal Time) as the system time. Users can configure offsets using the Management Portal or PowerShell to update the time. For more information see [Change time zones on MySQL on Azure](/documentation/articles/mysql-database-timezone-config/).

<!---HONumber=Acom_0218_2016_MySql-->