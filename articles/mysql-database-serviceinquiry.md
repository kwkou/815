<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="11/25/2015"/>

#服务咨询
> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry)

### **数据备份占用存储限额吗?**
  
数据备份不会占用您存储限额。

### **一个服务器是否限制数据库的数量?**

在一个MySQL服务器中，用户可创建多个数据库，数量上没有限制，但是多个数据库会共享服务器资源，如数据库数量较多，性能需求较高，建议创建多个MySQL服务器。
	
### **MySQL Database on Azure目前有哪些限制?**
	
了解更多[MySQL Database on Azure服务限制](/documentation/articles/mysql-database-operation-limitation/)

### **为什么MySQL Database on Azure不支持MYISAM格式的数据库?**

产品研发团队在多次研究和分析以后，做出了不支持的决定。考虑的因素主要有以下几点:

1. MYISAM对数据完整性的保护存在缺陷，而这些缺陷会导致数据库数据的损坏甚至丢失。并且这些缺陷由于很多是设计的问题，无法在不破坏兼容性的前提下修复。
2. MYISAM对于I/O的操作对于Azure的存储不是最优化的方案,导致MYISAM的性能相对于InnoDB优势不大。
3. MYISAM在出现数据损害情况下,需要很多手工修复,无法适应一个PaaS服务的运营方式。
4. MYISAM向InnoDB的迁移代价不大,大多数应用仅需要改动建表的代码。
5. MySQL的发展也是在向InnoDB转移,在最新的5.7中MySQL可以完全不是MyISAM,系统的数据库也被转移到了InnoDB。

### **为什么新建的空的数据库服务器默认大小为530M? 为什么数据库显示使用存储空间大于实际使用的存储空间?**
	
出于性能考虑，我们为新创建的数据库实例配置使用两个256M的日志文件。因此您在管理门户中看到的存储空间使用统计包括了日志文件的大小。但是日志文件大小在使用过程中不会改变。
	
### **MYSQL Database on Azure 是否支持用户通过命令行设置权限**

支持,虽然我们的[管理门户](https://manage.windowsazure.cn/) 以及PowerShell 命令行在创建用户或数据库时只支持对整个数据库设置读写权限，但你可以用“grant”命令对用户权限进行更细化的设置。