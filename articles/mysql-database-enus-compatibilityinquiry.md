<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="01/12/2015"/>
#Client compatibility issues
> [AZURE.SELECTOR]
- [All issues](/documentation/articles/mysql-database-tech-faq)
- [Service consulting](/documentation/articles/mysql-database-serviceinquiry)
- [Connection issues](/documentation/articles/mysql-database-connectioninquiry)
- [Security consulting](/documentation/articles/mysql-database-securityinquiry)
- [Compatibility issues](/documentation/articles/mysql-database-compatibilityinquiry)

MySQL Database on Azure uses MySQL Community Edition and is compatible with common MySQL management tools. During the actual operation and maintenance of MySQL Database on Azure, we have discovered certain compatibility issues with some versions of some clients. These issues are summarized below

### **Connection problems when you use Workbench 6.3.5 to connect to MySQL Database on Azure**

By default, Workbench 6.3.5 selects SSL connection and will use “TLS-DHE-RSA-WITH-AES-256-CBC-SHA” for encryption. However, our proxy server does not currently support identification, which causes the problem where it is impossible to connect to MySQL on Azure using Workbench 6.3.5. As a workaround, please configure the SSL certificate for use. See [SSL secure access to MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/) for the steps involved. You must also specify AES256-SHA in SSL\_Cipher, as shown in the diagram below. Alternatively, we suggest that you choose a version earlier than 6.3.5. Doing so will solve the problem.
![Workbench 6.3.5 connection methods][1]

### **Connection problems when you use SQLyog to connect to MySQL Database on Azure**
If the user name is longer than 16 characters, this client automatically truncates the user name to the first 16 characters. This causes connection issues. We recommend that you use the latest version of the SQLyog client or a different MySQL management client such as MySQL Workbench.



[1]: ./media/mysql-database-compatibilityinquiry/SSL.png

<!---HONumber=Acom_0218_2016_MySql-->