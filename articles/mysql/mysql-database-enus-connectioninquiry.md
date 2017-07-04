<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Azure Cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql_en" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="en" />

#Connection issues
> [AZURE.SELECTOR]
- [All FAQs](/documentation/articles/mysql-database-enus-tech-faq/)
- [Service consulting](/documentation/articles/mysql-database-enus-serviceinquiry/)
- [Connection issues](/documentation/articles/mysql-database-enus-connectioninquiry/)
- [Security consulting](/documentation/articles/mysql-database-enus-securityinquiry/)
- [Compatibility Issues](/documentation/articles/mysql-database-enus-compatibilityinquiry/)

### **I can’t connect to MySQL Database on Azure after I create a database. What should I do?**

Here are solutions to some common causes:

1. Confirm that you have added your current IP address to the whitelist. This process can be completed on the Azure portal by selecting Configure and adding the current IP.
2. Confirm that the user name you’ve entered on the client side is in the instance name%username format. If you have checked both of the points above but still can’t connect to the database, contact 21Vianet for technical support.

### **Why do timeouts frequently occur when I connect to the database?**

This is caused by limitations in Azure traffic manager. We recommend that you manually set the server parameter in the Azure portal or in Windows PowerShell to any value between 60 and 240 seconds (s). We suggest a value of 120 s. For instances created from October onward, we have already adjusted the default value to 120 s with a selectable range of 60 to 240 s, so you do not need to manually change the value. (This change only works on instances created from October onward)
	
### **Do I have too few concurrent connections for MySQL Database on Azure?**
	
To ensure that connections are fully and effectively used, we recommend that you use connection pooling or persistent connections to connect to the database. See [How to connect efficiently to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/).

### **Java Database Connectivity (JDBC) reports an IllegalArgumentException when it connects to MySQL on Azure, with an error message showing "URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ...".**

This occurs because the percent symbol (%) in the database user name was not correctly transferred to the connection string’s URL. We recommend that you use the DriverManager.getConnection (URL, user name, password) function and input the user name and password into a transfer where the function automatically processes certain special characters. Alternatively, you can perform a manual transfer by replacing the "%" in the URL string with "%25".

<!---HONumber=Acom_0104_2016_MySql-->