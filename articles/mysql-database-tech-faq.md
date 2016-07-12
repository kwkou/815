<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

#全部常见问题

> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq/)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry/)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry/)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry/)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry/)

+ [服务咨询](#step1)
+ [连接问题](#step2)
+ [安全性咨询](#step3)
+ [客户端兼容性问题](#step4)
+ [其他问题](#step5)

##**服务咨询**<a id="step1"></a> 
 
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
MySQL on Azure目前默认采用UTC 协调世界时作为系统时间System， 用户可以通过在管理门户上或PowerShell等途径配置补偿值(offsite)来进行时间更新。具体配置方法请参考：[MySQL on Azure上的时区配置](/documentation/articles/mysql-database-timezone-config/).

  
## **连接问题**<a id="step2"></a> 

### **创建数据库后连接不上MySQL on Azure?**

一些常见原因:

1. 请确认您是否把当前IP地址加入白名单，添加过程可在管理门户上选择配置进行当前IP的添加。
2. 请确认您在客户端输入的用户名是以实例名%用户名的格式。如上述两者均确认,仍出现数据库无法连接的情况,请联系世纪互联寻求技术支持。

### **为什么数据库连接时常出现超时中断?**

这是由于Azure流量管理器的限制引起的。我们建议您在管理门户或PowerShell中将服务器的参数手动设置为60-240s之间的任意数值，推荐120s。对于10月后创建的实例，我们已经将缺省值调整为120s，可选范围为60-240s，您无需手动更改。（此项调整仅对10月后创建的实例有效）
	
### **MySQL Database on Azure 并发连接量不够用?**
	
为了保证连接可以高效的得到充分利用，我们建议您使用连接池(connection pool)或是长连接(persistent connection)连接数据库。查看[如何高效连接到MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/);

###**在设置连接池后，MySQL Database on Azure 访问时快时慢？**

这是因为服务器端会设置超时机制，如果一个连接在一段时间内处于闲置状态，服务器就会关闭这个链接，以释放不必要的资源占用。因此，为了更好的维护连接池，保障MySQL数据库的访问速度，用户需要在客户端配置验证机制，用以进行连接有效性的验证，保证分配的连接都是有效的。以Tomcat JDBC Connection Pool为例，用户可以参考[JDBC Connection Pool官方介绍文档](https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Common_Attributes)中testOnBorrow的设定。

### **JDBC连接MySQL on Azure报IllegalArgumentException，错误信息显示“URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ...”。**

这是由于数据库用户名中包含的百分号‘%’在连接字符串的URL中没有被正确地转义。建议使用DriverManager.getConnection(url, username, password)函数并把用户名和密码传入由函数自动处理一些特殊字符的转义。或者可以用‘%25’代替URL字符串中的‘%’进行手工转义。



## **安全问题**<a id="step3"></a> 

### **我看到数据库服务器的地址是一个公共的endpoint?我的应用在同一个数据中心访问数据库时?访问请求是否会先经过Internet再到服务器地址?**

不会，Azure数据中心的网络路由会解析到这是一个自己的地址，会直接通过数据中心的内部网络路由到那个IP地址。而且这样的路由是安全的，不用担心查询请求和结果会被第三方监听到。但是如果您的应用和数据库不在同一个数据中心里，数据库查询请求和结果会经过Internet，在这种情况下建议用户用SSL来保证数据传递的私密性。关于SSL链接的详细信息参考[SSL安全访问MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/)。


##**客户端兼容性问题**<a id="step4"></a> 
MySQL Database on Azure采用MySQL 社区版本，兼容MYSQL常见的管理工具。在实际运维当中，我们也发现对于某些客户端的某个版本存在一定的兼容性问题，归纳如下：

### **用workbench 6.3.5连接MySQL on Azure, 出现连接问题**

workbench 6.3.5默认选择SSL连接, 且会使用“TLS-DHE-RSA-WITH-AES-256-CBC-SHA”进行加密，但是，我们的代理服务器断，目前暂不支持识别，因此会造成无法利用workbench 6.3.5连接MySQL on Azure的问题。作为workaround, 请您配置SSL证书使用，步骤参见[SSL安全访问MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/)，并且，您需要在SSL_Cipher中指明AES256-SHA，具体如下图所示。或者建议您可以选择6.3.5以前的版本，该问题将得以解决。
![Workbench6.3.5连接方法][1]

### **用SQLyog连接MySQL on Azure, 出现连接问题**
当user name超过16个字符时，该客户端会自动截取前16个字符，造成连接的问题。建议您采用最新版本的SQLyog客户端，或其他MySQL的管理客户端，如MySQL workbench。

## **其他问题**<a id="step5"></a> 

## 关于数据库迁移的常见问题：
###导入TRIGGER, PROCEDURE, VIEW, FUNCTION, 或EVENT过程中报”Access denied; you need (at least one of) the SUPER privilege(s) for this operation” 错误。
检查报错的语句有否使用DEFINER并使用非当前用户，比如DEFINER=`user`@`host`， 如果这样的话MySQL是要求SUPER权限来执行该语句，由于MySQL Database on Azure不提供用户SUPER权限（参考[服务限制](/documentation/articles/mysql-database-operation-limitation/) ），导致运行该语句失败。您只需要把DEFINER从该语句删掉而使用缺省的当前用户就可以了。

###MYSQL Database on Azure 的管理门户只支持对用户设置整个数据库的读写权限，如果我现在的数据库有对用户权限更细化的设置，迁移会成功吗？
没有问题，虽然我们的管理门户 以及PowerShell /REST API在创建用户或数据库时只支持对整个数据库设置读写权限，但您可以用“grant”语句对用户权限进行更细化的设置。

###如何将数据库从5.5升级到5.6？
目前我们暂不支持一键升级，替代办法是用户可以通过mysqldump将数据从源5.5实例中导出，创建新的5.6数据库服务器实例，再将数据进行导入，测试兼容性通过后，可将应用迁移至5.6实例。
如果用户的源数据库是生产环境或不能接受任何停机时间，可以在源数据库上手动创建快照，恢复到全新实例后，以恢复后的实例进行迁移升级，以减少对源数据库的影响。

<!--Image references-->

[1]: ./media/mysql-database-compatibilityinquiry/SSL.png