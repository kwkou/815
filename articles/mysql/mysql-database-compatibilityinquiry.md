<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,兼容性问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="10/10/2016" wacn.date="10/10/2016" wacn.lang="cn" />
#客户端兼容性问题
> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq/)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry/)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry/)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry/)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry/)

MySQL Database on Azure采用MySQL社区版本，兼容MySQL常见的管理工具。在实际运维当中，我们也发现对于某些客户端的某个版本存在一定的兼容性问题，归纳如下：

### **用workbench 6.3.5连接MySQL on Azure, 出现连接问题**

workbench 6.3.5默认选择SSL连接, 且会使用“TLS-DHE-RSA-WITH-AES-256-CBC-SHA”进行加密，但我们的代理服务器端目前暂不支持识别，因此会造成无法利用workbench 6.3.5连接MySQL Database on Azure的问题。作为workaround，请您配置SSL证书使用，具体步骤如下所示：

1. [点击链接](https://www.wosign.com/root/WS_CA1_NEW.crt)，下载证书文件。
2. 在workbench 6.3.5中，“SSL CA File”一栏指定该证书文件的位置，“SSL Cipher”一栏填写AES256-SHA，如下图所示。
![Workbench6.3.5SSL连接方法][1]

此外，您还可以选择6.3.5以前的版本，老版本没有这个问题。

### **用SQLyog连接MySQL on Azure, 出现连接问题**
当user name超过16个字符时，该客户端会自动截取前16个字符，造成连接的问题。建议您采用最新版本的SQLyog客户端，或其他MySQL的管理客户端，如MySQL workbench。


<!--Image references-->

[1]: ./media/mysql-database-compatibilityinquiry/SSL.png