<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="12/22/2015"/>

#All FAQs

> [AZURE.SELECTOR]
- [All FAQs](/documentation/articles/mysql-database-tech-faq)
- [Service consulting](/documentation/articles/mysql-database-serviceinquiry)
- [Connection issues](/documentation/articles/mysql-database-connectioninquiry)
- [Security consulting](/documentation/articles/mysql-database-securityinquiry)
- [Compatibility Issues](/documentation/articles/mysql-database-compatibilityinquiry)

+ [Service consulting](#step1)
+ [Connection issues](#step2)
+ [Security consulting](#step3)
+ [Client compatibility issues](#step4)

##**Service consulting**<a id="step1"></a> 
 
### **Is the amount of storage space for data backups limited?**
  
Data backups do not count towards your storage limit.

### **Is there a limit to the number of databases on a single server?**

Users can create multiple databases on a single MySQL server, and there is no limit to the number of databases that can be created. However, as multiple databases share server resources, the performance requirements will rise as the number of databases grows. We recommend that you create multiple MySQL servers.
	
### **What are the current limitations of MySQL Database on Azure?**
	
Find out more about the [service limitations of MySQL Database on Azure](/documentation/articles/mysql-database-operation-limitation/).

### **Why doesn’t MySQL Database on Azure support databases in MyISAM format?**

The product R&D team studied and analyzed this issue on several occasions, but ultimately decided not to support it. The main reasons for this are as follows:

1. MyISAM has flaws in terms of data integrity protection, and these flaws could cause database data to be damaged or even lost. Moreover, because these flaws are primarily due to design faults, there is no way to repair them without breaking compatibility.
2. Because the MyISAM input/output (I/O) operations are not the optimal solution for storage on Azure, MyISAM offers little performance advantage over InnoDB.
3. MyISAM requires a large amount of manual repair work if data is damaged, and it cannot be adapted to a platform as a service (PaaS) operating model.
4. The cost of migrating from MyISAM to InnoDB is minimal. In most applications, migration requires only changes to the table creation code.
5. The development of MySQL is also moving towards InnoDB. MySQL 5.7 (the latest version) doesn’t use MyISAM at all, and the system database has also been transitioned to InnoDB.

### **Why is the default size for new, empty database servers set to 530 MB? Why is the displayed amount of storage space used for the database larger than the amount of storage space actually used?**
	
For performance reasons, we use two 256 MB log files for new database instance configurations. Consequently, the storage space usage figure that you see in the Azure portal includes the size of the log files. However, the size of the log files will not change during the usage process.
	
### **Can I set permissions via the command line?**

Yes. Because the [Azure portal](https://manage.windowsazure.cn/) and Windows PowerShell support setting read/write permissions for the entire database only when you create users or databases, you can use the “grant” command to fine-tune user permission settings.
  
  
## **Connection issues**<a id="step2"></a> 

### **I can’t connect to MySQL Database on Azure after I create a database. What should I do?**

Here are solutions to some common causes:

1. Confirm that you have added your current IP address to the whitelist. This process can be completed on the Azure portal by selecting Configure and adding the current IP.
2. Confirm that the username you enter on the client side is in the instance name%username format. If you have checked both points above but still can’t connect to the database, contact 21Vianet for technical support.

### **Why do timeouts frequently occur when connecting to the database?**

This is caused by limitations in Azure Traffic Manager. We recommend that you manually set the server parameter in the Azure portal or in Windows PowerShell to any value between 60 and 240 seconds (s). We suggest a value of 120 s. For instances created from October onward, we have already adjusted the default value to 120 s with a selectable range of 60 to 240 s, so you do not need to manually change the value. (This change only works on instances created from October onward)
	
### **Do I have too few concurrent connections for MySQL Database on Azure?**
	
To ensure that connections are fully and effectively used, we recommend that you use connection pooling or persistent connections to connect to the database. See [How to connect efficiently to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/).

### **Java Database Connectivity (JDBC) reports an IllegalArgumentException when it connects to MySQL on Azure, with an error message showing "URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ...".**

This occurs because the percent symbol (%) in the database user name was not correctly transferred to the connection string’s URL. We recommend that you use the DriverManager.getConnection (URL, user name, password) function and input the user name and password into a transfer where the function automatically processes certain special characters. Alternatively, you can perform a manual transfer by replacing the "%" in the URL string with "%25".



## **Security issues**<a id="step3"></a> 

### **I noticed that the database server address is a public endpoint. Does this mean that the access request goes through the Internet before it reaches the server if my app visits a database in the same datacenter?**

No. The Azure datacenter’s network routing deduces that this is one of its own addresses and directly routes the request to the IP address via the datacenter’s internal network. This way of routing is more secure, so there is less concern about third parties monitoring query requests or results. However, if your app and database are not in the same datacenter, then database query requests and results do go through the Internet. In this situation, we recommend that you use Secure Sockets Layer (SSL) to ensure the privacy of data transfers. For more detailed information on SSL connections, please refer to [Use SSL to securely access MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/).


##**Client compatibility issues**<a id="step4"></a> 
MySQL Database on Azure uses MySQL Community Edition and is compatible with common MySQL management tools. During the actual operation and maintenance of MySQL Database on Azure, we have discovered certain compatibility issues with some versions of some clients. These issues are summarized below

### **Connection problems when you use Workbench 6.3.5 to connect to MySQL Database on Azure**

Because Workbench 6.3.5 selects SSL connections by default, you should configure the SSL certificate for use. See [Use SSL to securely access MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/) for the steps required to do this. Alternatively, we suggest that you choose a version earlier than 6.3.5. Doing so will solve the problem.

### **Connection problems when you use SQLyog to connect to MySQL Database on Azure**
If the user name is longer than 16 characters, this client automatically truncates the user name to the first 16 characters. This causes connection issues. We recommend that you use the latest version of the SQLyog client or a different MySQL management client such as MySQL Workbench.

<!---HONumber=Acom_0104_2016_MySql-->