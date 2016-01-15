<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="11/25/2015"/>

# Service Consulting
> [AZURE.SELECTOR]
- [All FAQs](/documentation/articles/mysql-database-tech-faq)
- [Service Consulting](/documentation/articles/mysql-database-serviceinquiry)
- [Connection Issues](/documentation/articles/mysql-database-connectioninquiry)
- [Security Consulting](/documentation/articles/mysql-database-securityinquiry)
- [Compatibility Issues](/documentation/articles/mysql-database-compatibilityinquiry)

### **Is the amount of storage space for data backups limited?**
  
Data backups do not count towards your storage limit.

### **Is there a limit to the number of databases on a single server?**

Users can create multiple databases on a single MySQL server, and there is no limit to the number of databases that can be created. However, as multiple databases share server resources, the performance requirements will rise as the number of databases grows. We recommend that you create multiple MySQL servers.
	
### **What are the current limitations of MySQL Database on Azure?**
	
Find out more about the [Service limitations of MySQL database on Azure](/documentation/articles/mysql-database-operation-limitation/).

### **Why doesn’t MySQL Database on Azure support databases in MyISAM format?**

The product R&D team studied and analyzed this issue on several occasions, but ultimately decided not to support it. The main reasons for this are as follows:

1. MyISAM has flaws in terms of data integrity protection, and these flaws could cause database data to be damaged or even lost. Moreover, because these flaws are primarily due to design faults, there is no way to repair them without breaking compatibility.
2. As MyISAM’s I/O operations are not the optimal solution for storage on Azure, MyISAM consequently offers little performance advantage over InnoDB.
3. MyISAM requires a large amount of manual repair work if data are damaged, and cannot be adapted to a PaaS service operating model.
4. The cost of migrating from MyISAM to InnoDB is minimal, and simply requires changes to the table creation code in most applications.
5. The development of MySQL is also moving towards InnoDB. The latest version (5.7) of MySQL doesn’t use MyISAM at all, and the system database has also been transitioned to InnoDB.

### **Why is the default size for new, empty database servers set to 530 MB? Why is the displayed amount of storage space used for the database larger than the amount of storage space actually used?**
	
For performance reasons, we use two 256 MB log files for new database instance configurations. Consequently, the storage space usage figure that you see in the Azure portal includes the size of the log files. However, the size of the log files will not change during the usage process.
	
### **Does MySQL Database on Azure support users setting permissions via the command line?**

Yes. While our [Management Portal](https://manage.windowsazure.cn/) and PowerShell only support setting read/write permissions for the entire database when creating users or databases, you can use the “grant” command to fine-tune user permission settings.

<!---HONumber=Acom_0104_2016_MySql-->