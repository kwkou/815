<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date="11/25/2015"/>

#连接问题
> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry)

### **创建数据库后连接不上MySQL on Azure?**

一些常见原因:

1. 请确认您是否把当前IP地址加入白名单，添加过程可在管理门户上选择配置进行当前IP的添加。
2. 请确认您在客户端输入的用户名是以实例名%用户名的格式。如上述两者均确认,仍出现数据库无法连接的情况,请联系世纪互联寻求技术支持。

### **为什么数据库连接时常出现超时中断?**

这是由于Azure流量管理器的限制引起的。我们建议您在管理门户或PowerShell中将服务器的参数手动设置为60-240s之间的任意数值，推荐120s。对于10月后创建的实例，我们已经将缺省值调整为120s，可选范围为60-240s，您无需手动更改。（此项调整仅对10月后创建的实例有效）
	
### **MySQL Database on Azure 并发连接量不够用?**
	
为了保证连接可以高效的得到充分利用，我们建议您使用连接池(connection pool)或是长连接(persistent connection)连接数据库。查看[如何高效连接到MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/);

### **JDBC连接MySQL on Azure报IllegalArgumentException，错误信息显示“URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ...”。**

这是由于数据库用户名中包含的百分号‘%’在连接字符串的URL中没有被正确地转义。建议使用DriverManager.getConnection(url, username, password)函数并把用户名和密码传入由函数自动处理一些特殊字符的转义。或者可以用‘%25’代替URL字符串中的‘%’进行手工转义。
