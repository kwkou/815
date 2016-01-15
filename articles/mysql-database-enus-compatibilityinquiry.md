<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="12/22/2015"/>
# Client Compatibility Issues
> [AZURE.SELECTOR]
- [All FAQs](/documentation/articles/mysql-database-tech-faq)
- [Service Consulting](/documentation/articles/mysql-database-serviceinquiry)
- [Connection Issues](/documentation/articles/mysql-database-connectioninquiry)
- [Security Consulting](/documentation/articles/mysql-database-securityinquiry)
- [Compatibility Issues](/documentation/articles/mysql-database-compatibilityinquiry)

MySQL Database on Azure uses MySQL Community Edition and is compatible with common MySQL management tools. During the actual operation and maintenance of MySQL Database on Azure, we have discovered certain compatibility issues with some versions of some clients. These issues are summarized below:

### **Connection problems occur when connecting to MySQL on Azure using Workbench 6.3.5**

As Workbench 6.3.5 selects SSL connections by default, please configure the SSL certificate for use. See [Use SSL to Securely Access MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/) for the steps required to do this. Alternatively, we suggest that you choose a version earlier than 6.3.5, as this will solve the problem.

### **Connection problems occur when connecting to MySQL on Azure using SQLyog**
If the user name is longer than 16 characters, this client automatically truncates the user name to the first 16 characters, causing connection issues. We recommend that you use the latest version of the SQLyog client or a different MySQL management client such as MySQL Workbench.

<!---HONumber=Acom_0104_2016_MySql-->