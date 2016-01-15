<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="11/25/2015"/>

# Connection Issues
> [AZURE.SELECTOR]
- [All FAQs](/documentation/articles/mysql-database-tech-faq)
- [Service Consulting](/documentation/articles/mysql-database-serviceinquiry)
- [Connection Issues](/documentation/articles/mysql-database-connectioninquiry)
- [Security Consulting](/documentation/articles/mysql-database-securityinquiry)
- [Compatibility Issues](/documentation/articles/mysql-database-compatibilityinquiry)

### **Are you unable to connect to MySQL Database on Azure after creating a database?**

Here are some common causes:

1. Confirm that you have added your current IP address to the whitelist. This process can be completed on the Azure portal by selecting Configure and adding the current IP.
2. Confirm that the user name you’ve entered on the client side is in the instance name%username format. If you have checked both of the points above but are still unable to connect to the database, contact 21Vianet for technical support.

### **Why do timeouts frequently occur when connecting to the database?**

This is caused by limitations in the Azure traffic manager. We recommend that you manually set the server parameter in the Management Portal or PowerShell to any value between 60 and 240s. We suggest a value of 120s. For instances created from October onwards, we have already adjusted the default value to 120s with a selectable range of 60-240s, so you do not need to manually change the value. (This change only works on instances created from October onward)
	
### **Do I have an insufficient number of concurrent connections for MySQL Database on Azure?**
	
To ensure that connections are fully and effectively used, we recommend that you use connection pooling or persistent connections to connect to the database. See [How to Connect Efficiently to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/).

### **JDBC reports an IllegalArgumentException when connecting to MySQL on Azure, with an error message showing “URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ...”.**

This occurs because the percent (%) symbol contained in the database username was not correctly transferred to the connection string’s URL. We recommend using the DriverManager.getConnection (URL, username, password) function and inputting the username and password into a transfer where the function automatically processes certain special characters. Alternatively, you can perform a manual transfer by replacing the “%” in the URL string with “%25”.

<!---HONumber=Acom_0104_2016_MySql-->