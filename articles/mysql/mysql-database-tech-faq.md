<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="1/10/2017" wacn.date="1/10/2017" wacn.lang="cn" />

#全部常见问题

> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq/)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry/)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry/)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry/)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry/)
- [主从复制问题](/documentation/articles/mysql-database-readreplicainquiry/)

+ [服务咨询](#step1)
+ [连接问题](#step2)
+ [安全性咨询](#step3)
+ [客户端兼容性问题](#step4)
+ [主从复制问题](#step5)
+ [其他问题](#step6)

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

### **MySQL 5.7 都提供哪些功能？这些功能是否我在MySQL Database on Azure上都可以使用？**

MySQL on Azure 全面兼容MySQL社区版本，关于5.7的新功能，请参考MySQL 5.7 Release Notes了解更多。MySQL on Azure兼容上述绝大功能的更新，但下述功能尚不支持：

-	暂不支持5.7中关于Replication功能的改进 （由于MySQL on Azure的主从同步复制功能并未直接采用MySQL Replication技术，而是在其基础上针对MySQL on Azure服务进行了一定的改良）。
-	暂不支持5.7中关于InnoDB Buffer Pool Online Resize的功能。
-	暂不支持Query Rewrite Plugin。
-	暂不支持InnoDB Transparent Page Level Compression
-   暂不支持password expiration

### **如何在管理门户上创建MySQL 5.7的实例？**

在管理门户上点击左下角“”新建“”，选择MySQL Database on Azure，进行快速创建，在MySQL 版本中选择5.7。
![MySQL57创建][2]

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

workbench 6.3.5默认选择SSL连接, 且会使用“TLS-DHE-RSA-WITH-AES-256-CBC-SHA”进行加密，但我们的代理服务器端目前暂不支持识别，因此会造成无法利用workbench 6.3.5连接MySQL Database on Azure的问题。作为workaround，请您配置SSL证书使用，具体步骤如下所示：

1. [点击链接](https://www.wosign.com/root/WS_CA1_NEW.crt)，下载证书文件。
2. 在workbench 6.3.5中，“SSL CA File”一栏指定该证书文件的位置，“SSL Cipher”一栏填写AES256-SHA，如下图所示。
![Workbench6.3.5SSL连接方法][1]

此外，您还可以选择6.3.5以前的版本，老版本没有这个问题。

### **用SQLyog连接MySQL on Azure, 出现连接问题**
当user name超过16个字符时，该客户端会自动截取前16个字符，造成连接的问题。建议您采用最新版本的SQLyog客户端，或其他MySQL的管理客户端，如MySQL workbench。

## **主从复制问题**<a id="step5"></a>
### **我想更改主实例的性能版本，比如从MS4升到MS5，或者MS6降到MS5。从属实例的版本会随着更新吗？**
 
从属实例的性能版本不会随之改变，但用户可以根据需要另行改变从属实例的性能版本。

### **我想使用主从复制功能，但是我的数据库实例现在是5.5版本，我该怎么做？**

主从复制只支持5.6及以上版本。可以先将数据库实例手动升级到5.6或5.7版本，然后再使用复制功能。

### **我的主从实例之间有时有很大的延迟（例如超过五分钟），这是什么原因造成的？**

主从之间的大延迟可能是由以下几种情况之一导致：

1. 如果用户没有设置主键（Primary Key），复制时，需要对从属实例进行全表扫描，会严重影响效率。个别全表更新操作甚至会导致从属实例阻塞；
2. 当一个大数据量的更新发生时，从属实例同样需要花费很多时间来执行这个更新，这个时候就阻塞了下一个复制，从而产生延迟；
3. 只读实例配置太低，无法和主实例一样快速执行更新；
4. 从属实例上有大量的读操作，从而影响了同步执行主实例上产生的更新。

### **如何消除主从实例间的大延迟？**

1. 数据库表一定要设置主键（Primary Key），否则会导致复制慢甚至复制被阻塞的情况。
2. 提升只读实例的性能版本可以直接提升只读实例的吞吐能力（throughput），这样做一方面可以更快地执行更新，加速同步过程，另一方面也可以快速响应读操作。
3. 如果某个查询语句对于时间很敏感，无论怎么缩短延迟都达不到要求，那么可以将这个查询放到主库上执行。但这就需要用户在程序中指定特定查询语句在特定实例上执行。

## **其他问题**<a id="step6"></a> 

## 关于mysql.exe命令行工具的已知问题：
### status命令返回错误的服务器版本号
mysql.exe命令行工具连接MySQL Database on Azure数据库后，若使用status命令查看服务器版本，则会返回错误的版本号——比如您创建的服务器是5.7，但status命令返回的版本号却是5.5，如下图所示。
![status命令返回错误结果][3]
如需使用mysql.exe命令行工具查看服务器版本号，请使用命令“show variables like '%version%';”（不包含双引号），如下图所示。
![show命令返回正确结果][4]

## 关于数据库迁移的常见问题：
###导入TRIGGER, PROCEDURE, VIEW, FUNCTION, 或EVENT过程中报”Access denied; you need (at least one of) the SUPER privilege(s) for this operation” 错误。
检查报错的语句有否使用DEFINER并使用非当前用户，比如DEFINER=`user`@`host`， 如果这样的话MySQL是要求SUPER权限来执行该语句，由于MySQL Database on Azure不提供用户SUPER权限（参考[服务限制](/documentation/articles/mysql-database-operation-limitation/) ），导致运行该语句失败。您只需要把DEFINER从该语句删掉而使用缺省的当前用户就可以了。

###MYSQL Database on Azure 的管理门户只支持对用户设置整个数据库的读写权限，如果我现在的数据库有对用户权限更细化的设置，迁移会成功吗？
没有问题，虽然我们的管理门户 以及PowerShell /REST API在创建用户或数据库时只支持对整个数据库设置读写权限，但您可以用“grant”语句对用户权限进行更细化的设置。

###如何将数据库从5.5升级到5.6？
目前我们暂不支持一键升级，替代办法是用户可以通过mysqldump将数据从源5.5实例中导出，创建新的5.6数据库服务器实例，再将数据进行导入，测试兼容性通过后，可将应用迁移至5.6实例。
如果用户的源数据库是生产环境或不能接受任何停机时间，可以在源数据库上手动创建快照，恢复到全新实例后，以恢复后的实例进行迁移升级，以减少对源数据库的影响。

## 关于Emoji的支持问题：
###如何让新建的数据库支持Emoji表情符号？
在MySQL Database on Azure上创建数据库时，如无特殊指定，则创建的数据库默认采用UTF8字符集。由于MySQL上的UTF8字符集最多支持3字节编码，因此采用4字节编码的Emoji表情符号数据无法插入表中。为了让新建立的数据库支持Emoji表情符号，请按以下步骤操作：

**1. 更改MySQL Server端的配置**

在管理门户中，进入MySQL Database on Azure的“配置”页面，将character_set_server配置为utf8mb4，并保存。

![更改MySQL Server端的配置][5]

**2. 创建字符集为utf8mb4的数据库**

在数据库页面创建数据库时，选择“utf8mb4”作为该数据库的字符集。

![创建字符集为utf8mb4的数据库][6]

###如何让已有数据库支持Emoji表情符号？
针对已有字符集为非utf8mb4的数据库，无法通过将字符集设为utf8mb4直接实现Emoji的支持，但可以采用创建新表并将原表数据导入新表的方式间接实现对Emoji的支持，具体步骤如下（省略命令行）：

**1. 更改MySQL Server端的配置（同上文）**

**2. 更改已有数据库的字符集设置**

选中该数据库，单击页面下方的“编辑”按钮，然后选择utf8mb4为其字符集。

![更改已有数据库的字符集设置][7]

**3. 创建新表**

推荐使用DBMS工具（比如workbench）创建新表，假设新表名为table_b（原表名为table_a），注意table_b表结构必须与table_a完全一致。

**4. 转移原表数据至新表**

推荐使用MySQL语句：`INSERT INTO table_b (…) SELECT … FROM table_a;`

**5. DROP原表**

推荐使用MySQL语句：`DROP TABLE table_a;`

**6. 重命名新表为原表名称**

推荐使用MySQL语句：`RENAME table_b TO table_a;`

至此，无需对应用进行改动，便实现了数据库中某一张表的Emoji支持。

<!--Image references-->

[1]: ./media/mysql-database-compatibilityinquiry/SSL.png
[2]: ./media/mysql-database-serviceinquiry/mysql57.png
[3]: ./media/mysql-database-otherinquiry/statusbug1.png
[4]: ./media/mysql-database-otherinquiry/statusbug2.png
[5]: ./media/mysql-database-otherinquiry/emoji1.png
[6]: ./media/mysql-database-otherinquiry/emoji2.png
[7]: ./media/mysql-database-otherinquiry/emoji3.png