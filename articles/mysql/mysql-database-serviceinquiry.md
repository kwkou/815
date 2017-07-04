<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,数据备份存储限额,Azure MYISAM,数据库服务器默认大小,权限设置,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

#服务咨询
> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq/)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry/)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry/)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry/)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry/)

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

### **MySQL Database on Azure 现在使用什么系统时间？ 如何变更？**
MySQL on Azure目前默认采用UTC 协调世界时作为系统时间System， 用户可以通过在管理门户上或PowerShell等途径配置补偿值(offsite)来进行时间更新。具体配置方法请参考：[MySQL on Azure上的时区更改](/documentation/articles/mysql-database-timezone-config/).

### **MySQL 5.7 都提供哪些功能？这些功能是否我在MySQL Database on Azure上都可以使用？**

MySQL on Azure 全面兼容MySQL社区版本，关于5.7的新功能，请参考MySQL 5.7 Release Notes了解更多。MySQL on Azure兼容上述绝大功能的更新，但下述功能尚不支持：

-	暂不支持5.7中关于Replication功能的改进 （由于MySQL on Azure的主从同步复制功能并未直接采用MySQL Replication技术，而是在其基础上针对MySQL on Azure服务进行了一定的改良）。
-	暂不支持5.7中关于InnoDB Buffer Pool Online Resize的功能。
-	暂不支持Query Rewrite Plugin。
-	暂不支持InnoDB Transparent Page Level Compression
-   暂不支持password expiration

### **如何在管理门户上创建MySQL 5.7的实例？**

在管理门户上点击左下角“”新建“”，选择MySQL Database on Azure，进行快速创建，在MySQL 版本中选择5.7。
![MySQL57创建][1]

### **我已经有MySQL on Azure的数据库实例，但是5.5 或5.6 版本，我应该如何升级到5.7？**

建议您在升级前了解5.7版本的改动，测试完应用端以及数据库兼容性后再进行版本升级。如果您是生产环境，建议在业务量较低时段对数据库进行升级，步骤如下：

Step 1. (可选) 如果您担心升级后兼容性问题对生产环境产生影响，可以利用数据库恢复功能将当前数据库恢复到新的实例，具体恢复方式参见MySQL Database on Azure备份和恢复。推荐使用立即备份功能对当前数据库服务器进行快照，再对该快照在另一全新实例上进行恢复。

Step 2. 将当前数据库服务器数据导出：
您可以使用MySQL 管理工具，如MySQL Workbench，或MySQL utility: mysqldump 对当前数据库服务器进行数据库导出：mysqldump --databases <数据库名> --single-transaction --order-by-primary -r <备份文件名> --routines -h<服务器地址> -P<端口号> –u<用户名> -p<密码> .

Step 3. 在管理门户上，创建5.7实例并创建账户：
在管理门户上点击左下角“”新建“”，选择MySQL Database on Azure，进行快速创建，在MySQL 版本中选择5.7。并将原数据库实例中的账户，通过管理门户进行创建。

Step 4. 将数据导入到5.7实例
可以使用MySQL 常用管理工具Workbench 或者使用SQL语句： source <备份文件名>; 进行数据导入。
完成上述步骤后，您可以将应用层的连接字符串切换到新的服务器开始使用。

**注意： 考虑到兼容性，建议不要进行跨版本升级，如果您使用的实例是5.5 版本，请先按照此方法升级到5.6。再将5.6实例升级到5.7。**


<!--Image references-->

[1]: ./media/mysql-database-serviceinquiry/mysql57.png