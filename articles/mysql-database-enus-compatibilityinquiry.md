<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="12/22/2015"/>
#Client compatibility issues
> [AZURE.SELECTOR]
- [All FAQs](/documentation/articles/mysql-database-tech-faq)
- [Service consulting](/documentation/articles/mysql-database-serviceinquiry)
- [Connection issues](/documentation/articles/mysql-database-connectioninquiry)
- [Security consulting](/documentation/articles/mysql-database-securityinquiry)
- [Compatibility Issues](/documentation/articles/mysql-database-compatibilityinquiry)

MySQL Database on Azure uses MySQL Community Edition and is compatible with common MySQL management tools. During the actual operation and maintenance of MySQL Database on Azure, we have discovered certain compatibility issues with some versions of some clients. These issues are summarized below

### **Connection problems when you use Workbench 6.3.5 to connect to MySQL on Azure**

Because Workbench 6.3.5 selects SSL connections by default, you should configure the SSL certificate for use. See [Use SSL to securely access MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/) for the steps required to do this. Alternatively, we suggest that you choose a version earlier than 6.3.5 to solve the problem.

### **Connection problems when you use SQLyog to connect to MySQL on Azure**
If the user name is longer than 16 characters, this client automatically truncates the user name to the first 16 characters. This causes connection issues. We recommend that you use the latest version of the SQLyog client or a different MySQL management client such as MySQL Workbench.

<!---HONumber=Acom_0104_2016_MySql-->