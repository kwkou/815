<properties linkid="" urlDisplayName="" pageTitle="MySQL FAQ - Azure 微软云" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，常见问题，FAQ" description="针对用户在使用MySQL Database on Azure中遇到的一些常见技术问题，提供快速解答。如果您仍存有疑问，欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="" wacn.date=""/>

#技术FAQ
- 公共预览版是否有SLA的保证？
	与Azure其他服务一样，根据[世纪互联有关 Azure 的在线服务标准协议](/support/legal/subscription-agreement)，MySQL Database on Azure在公共预览阶段暂不提供SLA保证。尽管如此，MySQL Database on Azure公共预览版的设计实现和运维都是严格以99.9%的高可用性为标准。 - 数据备份占用存储限额吗？
	数据备份不会占用您存储限额。- 一个服务器是否限制数据库的数量？
	在一个MySQL服务器中，用户可创建多个数据库，数量上没有限制，但是多个数据库会共享服务器资源，如数据库数量较多，性能需求较高，建议创建多个MySQL服务器。 - MySQL Database on Azure目前有哪些限制？
	了解更多[MySQL Database on Azure服务限制](/documentation/articles/mysql-database-operation-limitation/)- MySQL Database on Azure 并发连接量不够用？
	
	为了保证连接可以高效的得到充分利用，我们建议您使用连接池(connection pool)或是长连接(persistent connection)连接数据库。查看[如何高效连接到MySQL Database on Azure](/documentation/articles/mysql-database-connection-pool/);

- MYSQL Database on Azure 是否支持用户通过命令行设置权限？
	
	支持，虽然我们的[管理门户](https://manage.windowsazure.cn/)创建用户或数据库时只支持对整个数据库设置读写权限，但你可以用“grant”命令对用户权限进行更细化的设置。
- 为什么MySQL Database on Azure不支持MYISAM格式的数据库？
	产品研发团队在多次研究和分析以后，做出了不支持的决定。考虑的因素主要有以下几点：
	1. MYISAM对数据完整性的保护存在缺陷，而这些缺陷会导致数据库数据的损坏甚至丢失。并且这些缺陷由于很多是设计的问题，无法在不破坏兼容性的前提下修复。
	2. MYISAM对于I/O的操作对于Azure的存储不是最优化的方案，导致MYISAM的性能相对于InnoDB优势不大。
	3. MYISAM在出现数据损害情况下，需要很多手工修复，无法适应一个PaaS服务的运营方式。
	4. MYISAM向InnoDB的迁移代价不大，大多数应用仅需要改动建表的代码。
	5. MySQL的发展也是在向InnoDB转移，在最新的5.7中，MySQL可以完全不是MyISAM，系统的数据库也被转移到了InnoDB。
