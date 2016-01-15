<properties linkid="" urlDisplayName="" pageTitle="MySQL Service Questions – Microsoft Azure Cloud" metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="12/01/2015"/>
# Security Consulting
> [AZURE.SELECTOR]
- [All FAQs](/documentation/articles/mysql-database-tech-faq)
- [Service Consulting](/documentation/articles/mysql-database-serviceinquiry)
- [Connection Issues](/documentation/articles/mysql-database-connectioninquiry)
- [Security Consulting](/documentation/articles/mysql-database-securityinquiry)
- [Compatibility Issues](/documentation/articles/mysql-database-compatibilityinquiry)

### **I noticed that the database server address is a public endpoint. Does this mean that the access request will go through the Internet before reaching the server if my app visits a database in the same data center?**

No. The Azure data center’s network routing will deduce that this is one of its own addresses and directly route the request to the IP address via the data center’s internal network. This way of routing is secure, so there is no need to worry about third parties monitoring query requests or results. However, if your app and database are not in the same data center, the database query requests and results will go through the Internet. In such situations, we recommend that SSL be used to ensure the privacy of data transfers. For more detailed information on SSL connections, please refer to [Use SSL to Securely Access MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/).

<!---HONumber=Acom_0104_2016_MySql-->