<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

# All FAQs

> [AZURE.SELECTOR]
- [All issues](/documentation/articles/mysql-database-enus-tech-faq/)
- [Service consulting](/documentation/articles/mysql-database-enus-serviceinquiry/)
- [Connection issues](/documentation/articles/mysql-database-enus-connectioninquiry/)
- [Security consulting](/documentation/articles/mysql-database-enus-securityinquiry/)
- [Compatibility issues](/documentation/articles/mysql-database-enus-compatibilityinquiry/)

+ [Service consulting](#step1)
+ [Connection issues](#step2)
+ [Security consulting](#step3)
+ [Client compatibility issues](#step4)
+ [Other issues](#step5)

## **Service consulting**<a id="step1"></a> 
 
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
MySQL on Azure currently defaults to using UTC (Coordinated Universal Time) as the system time. Users can configure offsets using the Management Portal or PowerShell to update the time. For more information, see [Time zone configuration on MySQL on Azure](/documentation/articles/mysql-database-timezone-config/).

  
## **Connection issues**<a id="step2"></a> 

### **I can’t connect to MySQL Database on Azure after I create a database. What should I do?**

Here are solutions to some common causes:

- Confirm that you have added your current IP address to the safe list. This process can be completed on the Azure portal by selecting **Configure** and adding the current IP.
- Confirm that the user name you’ve entered on the client side is in the instance name%username format. If you have checked both points above but still can’t connect to the database, contact 21Vianet for technical support.

### **Why do timeouts frequently occur when I connect to the database?**

This is caused by limitations in Azure Traffic Manager. You should manually set the server parameter in the Azure portal or in Windows PowerShell to any value between 60-240 seconds(120 recommended). For instances created from October onward, you don’t need to set the server parameter manually. We have already adjusted the default value to 120 seconds with a selectable range of 60-240.
	
### **Do I have too few concurrent connections for MySQL Database on Azure?**
	
To ensure that connections are fully and effectively used, use connection pooling or persistent connections to connect to the database. See [How to connect efficiently to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/).

### **Access to MySQL Database on Azure is sometimes fast and sometimes slow since I set up connection pooling.**

This is because the server configures a timeout mechanism, closing a connection that has been in an idle state for some time, in order to free up resources. For this reason, you need to configure a verification system on the client to better maintain connection pooling and ensure access speeds for MySQL databases. This is used to verify the effectiveness of connections and ensure that the allocated connections are all effective. For example, refer to the TestOnBorrow settings in the [JDBC Connection Pool official introduction document](https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Common_Attributes).

### **Java Database Connectivity (JDBC) reports an IllegalArgumentException when it connects to MySQL on Azure, with an error message showing "URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ...".**

This occurs because the percent symbol (%) in the database user name was not correctly transferred to the connection string’s URL. Use the DriverManager.getConnection (URL, user name, password) function and input the user name and password into a transfer where the function automatically processes certain special characters. Alternatively, you can perform a manual transfer by replacing the "%" in the URL string with "%25".



## **Security issues**<a id="step3"></a> 

### **I noticed that the database server address is a public endpoint. Does this mean that the access request goes through the Internet before it reaches the server if my app visits a database in the same datacenter?**

No. The Azure datacenter’s network routing deduces that this is one of its own addresses and directly routes the request to the IP address via the datacenter’s internal network. This way of routing is more secure, so there is less concern about third parties monitoring query requests or results. However, if your app and database are not in the same datacenter, then database query requests and results do go through the Internet. In this situation, we recommend that you use Secure Sockets Layer (SSL) to ensure the privacy of data transfers. For more detailed information on SSL connections, see [Use SSL to securely access MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/).


## **Client compatibility issues**<a id="step4"></a> 
MySQL Database on Azure uses MySQL Community Edition and is compatible with common MySQL management tools. During the actual operation and maintenance of MySQL Database on Azure, you may encounter compatibility issues with some versions of some clients. These issues are summarized below:

### **Connection problems when you use Workbench 6.3.5 to connect to MySQL Database on Azure**

By default, Workbench 6.3.5 selects SSL connection and uses “TLS-DHE-RSA-WITH-AES-256-CBC-SHA” for encryption. However, our proxy server does not currently support identification, which causes connectivity problems between MySQL on Azure and Workbench 6.3.5. As a workaround, please configure the SSL certificate for use. See [SSL secure access to MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/). You must also specify AES256-SHA in SSL\_Cipher, as shown in the picture below. Alternatively, choose a version earlier than 6.3.5. Alternatively, we suggest that you choose a version earlier than 6.3.5. Doing so will solve the problem.
![Workbench 6.3.5 connection methods][1]

### **Connection problems when you use SQLyog to connect to MySQL Database on Azure**
If the user name is longer than 16 characters, this client automatically truncates the user name to the first 16 characters. This causes connection issues. Use the latest version of the SQLyog client, or a different MySQL management client such as MySQL Workbench.

## **Other issues**<a id="step5"></a> 
## Common issues with database migration:
### An error message saying “Access denied; you need (at least one of) the SUPER privilege(s) for this operation” is reported during the TRIGGER, PROCEDURE, VIEW, FUNCTION, or EVENT import process.
Check whether the statement reporting the error uses DEFINER and uses users other than the current user (for example DEFINER=user`user`@`host`host). If this is the case, MySQL requires SUPER privileges to execute this statement. MySQL Database on Azure does not provide user SUPER privileges (see [Service limitations](http://www.windowsazure.cn/documentation/articles/mysql-database-operation-limitation/)), causing an error. To resolve this error, delete DEFINER from the statement and use the default current user.


<!--Image references-->

[1]: ./media/mysql-database-compatibilityinquiry/SSL.png

<!---HONumber=Acom_0218_2016_MySql-->