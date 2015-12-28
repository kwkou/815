<<<<<<< HEAD
<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />
=======
<properties linkid="" urlDisplayName="" pageTitle="MySQL technical FAQ – Microsoft Azure cloud" metakeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQ, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />
>>>>>>> origin/mysql-en

<tags ms.service="mysql" ms.date="" wacn.date="12/22/2015"/>

#全部常见问题

<<<<<<< HEAD
> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry)

+ [服务咨询](#step1)
+ [连接问题](#step2)
+ [安全性咨询](#step3)
+ [客户端兼容性问题](#step4)

##**服务咨询**<a id="step1"></a> 
 
### **数据备份占用存储限额吗?**
  
数据备份不会占用您存储限额。
=======
 
- Is the amount of storage space for data backups limited? 
	Data backups do not count towards your storage limit.

- Is there a limit to the number of databases on a single server? 
	Users can create multiple databases on a single MySQL server, and there is no limit to the number of databases that can be created. However, as multiple databases share server resources, the performance requirements will rise as the number of databases grows. We recommend that you create multiple MySQL servers.
>>>>>>> origin/mysql-en

### **一个服务器是否限制数据库的数量?**

在一个MySQL服务器中，用户可创建多个数据库，数量上没有限制，但是多个数据库会共享服务器资源，如数据库数量较多，性能需求较高，建议创建多个MySQL服务器。
	
### **MySQL Database on Azure目前有哪些限制?**
	
<<<<<<< HEAD
了解更多[MySQL Database on Azure服务限制](/documentation/articles/mysql-database-operation-limitation/)

### **为什么MySQL Database on Azure不支持MYISAM格式的数据库?**

产品研发团队在多次研究和分析以后，做出了不支持的决定。考虑的因素主要有以下几点:
=======
	Confirm that you have added your current IP address to the whitelist. This process can be completed on the Azure portal by selecting Configure and adding the current IP. Also confirm that the username you enter on the client side is in the instance name%username format. If you have checked both of the points above but are still unable to connect to the database, contact 21Vianet for technical support.
 	

- What limitations are there for MySQL Database on Azure? 
	Find out more about the [Service limitations of MySQL database on Azure](/documentation/articles/mysql-database-operation-limitation).
>>>>>>> origin/mysql-en

1. MYISAM对数据完整性的保护存在缺陷，而这些缺陷会导致数据库数据的损坏甚至丢失。并且这些缺陷由于很多是设计的问题，无法在不破坏兼容性的前提下修复。
2. MYISAM对于I/O的操作对于Azure的存储不是最优化的方案,导致MYISAM的性能相对于InnoDB优势不大。
3. MYISAM在出现数据损害情况下,需要很多手工修复,无法适应一个PaaS服务的运营方式。
4. MYISAM向InnoDB的迁移代价不大,大多数应用仅需要改动建表的代码。
5. MySQL的发展也是在向InnoDB转移,在最新的5.7中MySQL可以完全不是MyISAM,系统的数据库也被转移到了InnoDB。

### **为什么新建的空的数据库服务器默认大小为530M? 为什么数据库显示使用存储空间大于实际使用的存储空间?**
	
出于性能考虑，我们为新创建的数据库实例配置使用两个256M的日志文件。因此您在管理门户中看到的存储空间使用统计包括了日志文件的大小。但是日志文件大小在使用过程中不会改变。
	
<<<<<<< HEAD
### **MYSQL Database on Azure 是否支持用户通过命令行设置权限**

支持,虽然我们的[管理门户](https://manage.windowsazure.cn/) 以及PowerShell 命令行在创建用户或数据库时只支持对整个数据库设置读写权限，但你可以用“grant”命令对用户权限进行更细化的设置。
  
  
## **连接问题**<a id="step2"></a> 

### **创建数据库后连接不上MySQL on Azure?**

一些常见原因:

1. 请确认您是否把当前IP地址加入白名单，添加过程可在管理门户上选择配置进行当前IP的添加。
2. 请确认您在客户端输入的用户名是以实例名%用户名的格式。如上述两者均确认,仍出现数据库无法连接的情况,请联系世纪互联寻求技术支持。

### **为什么数据库连接时常出现超时中断?**
=======
	To ensure that connections are fully and effectively used, we recommend that you use connection pooling or persistent connections to connect to the database. See [How to connect efficiently to MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool).
>>>>>>> origin/mysql-en

这是由于Azure流量管理器的限制引起的。我们建议您在管理门户或PowerShell中将服务器的参数手动设置为60-240s之间的任意数值，推荐120s。对于10月后创建的实例，我们已经将缺省值调整为120s，可选范围为60-240s，您无需手动更改。（此项调整仅对10月后创建的实例有效）
	
<<<<<<< HEAD
### **MySQL Database on Azure 并发连接量不够用?**
=======
	Yes. While the [Azure portal](https://manage.windowsazure.cn/) only supports setting read/write permissions for the entire database when you create users or databases, you can use the “grant” command to fine-tune user permission settings.

- Why doesn’t MySQL Database on Azure support databases in MYISAM format? The product R&D team studied and analyzed this issue on several occasions, but ultimately decided not to support it. The main reasons for this are as follows:
	1. MyISAM has flaws in terms of data integrity protection, and these flaws could cause database data to be damaged or even lost. Moreover, because these flaws are primarily due to design faults, there is no way to repair them without breaking compatibility.
	2. As MyISAM’s I/O operations are not the optimal solution for storage on Azure, MyISAM consequently offers little performance advantage over InnoDB.
	3. MyISAM requires a large amount of manual repair work if data is damaged. It cannot be adapted to a PaaS service operating model.
	4. The cost of migrating from MyISAM to InnoDB is minimal, and simply requires changes to the table creation code in most applications.
	5. The development of MySQL has also moved towards InnoDB. The latest version (5.7) doesn’t use MyISAM at all, and the system database has also been transitioned to InnoDB.
>>>>>>> origin/mysql-en
	
为了保证连接可以高效的得到充分利用，我们建议您使用连接池(connection pool)或是长连接(persistent connection)连接数据库。查看[如何高效连接到MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/);

### **JDBC连接MySQL on Azure报IllegalArgumentException，错误信息显示“URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ...”。**

这是由于数据库用户名中包含的百分号‘%’在连接字符串的URL中没有被正确地转义。建议使用DriverManager.getConnection(url, username, password)函数并把用户名和密码传入由函数自动处理一些特殊字符的转义。或者可以用‘%25’代替URL字符串中的‘%’进行手工转义。



## **安全问题**<a id="step3"></a> 

### **我看到数据库服务器的地址是一个公共的endpoint?我的应用在同一个数据中心访问数据库时?访问请求是否会先经过Internet再到服务器地址?**

不会，Azure数据中心的网络路由会解析到这是一个自己的地址，会直接通过数据中心的内部网络路由到那个IP地址。而且这样的路由是安全的，不用担心查询请求和结果会被第三方监听到。但是如果您的应用和数据库不在同一个数据中心里，数据库查询请求和结果会经过Internet，在这种情况下建议用户用SSL来保证数据传递的私密性。关于SSL链接的详细信息参考[SSL安全访问MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/)。


##**客户端兼容性问题**<a id="step4"></a> 
MySQL Database on Azure采用MySQL 社区版本，兼容MYSQL常见的管理工具。在实际运维当中，我们也发现对于某些客户端的某个版本存在一定的兼容性问题，归纳如下：

### **用workbench 6.3.5连接MySQL on Azure, 出现连接问题**

<<<<<<< HEAD
workbench 6.3.5默认选择SSL连接, 请您配置SSL证书使用，步骤参见[SSL安全访问MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/)。或者建议您可以选择6.3.5以前的版本，该问题将得以解决。

### **用SQLyog连接MySQL on Azure, 出现连接问题**
当user name超过16个字符时，该客户端会自动截取前16个字符，造成连接的问题。建议您采用最新版本的SQLyog客户端，或其他MySQL的管理客户端，如MySQL workbench。
=======
	For performance reasons, we use two 256 MB log files for new database instance configurations. Consequently, the storage space usage figure that you see in the Azure portal includes the size of the log files. However, the size of the log files will not change during the usage process.


<!--HONumber=81-->
>>>>>>> origin/mysql-en
