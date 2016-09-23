<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,数据安全访问,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="09/23/2016" wacn.date="09/23/2016" wacn.lang="cn" />

#主从复制问题

> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq/)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry/)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry/)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry/)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry/)
- [主从复制问题](/documentation/articles/mysql-database-readreplicainquiry/)

## **我想更改主实例的性能版本，比如从MS4升到MS5，或者MS6降到MS5。从属实例的版本会随着更新吗？**
 
从属实例的性能版本不会随之改变，但用户可以根据需要另行改变从属实例的性能版本。

## **我想使用主从复制功能，但是我的数据库实例现在是5.5版本，我该怎么做？**

主从复制只支持5.6及以上版本。可以先将数据库实例手动升级到5.6或5.7版本，然后再使用复制功能。

## **我的主从实例之间有时有很大的延迟（例如超过五分钟），这是什么原因造成的？**

主从之间的大延迟可能是由以下几种情况导致：

1. 当一个大数据量的更新发生时，从属实例同样需要花费很多时间来执行这个更新，这个时候就阻塞了下一个复制，从而产生延迟；
2. 只读实例配置太低，无法和主实例一样快速执行更新；
3. 从属实例上有大量的读操作，从而影响了同步执行主实例上产生的更新。

## **如何消除主从实例间的大延迟？**

提升只读实例的性能版本可以直接提升只读实例的吞吐能力（throughput），这样做一方面可以更快地执行更新，加速同步过程，另一方面也可以快速响应读操作。此外，如果某个查询语句对于时间很敏感，无论怎么缩短延迟都达不到要求，那么可以将这个查询放到主库上执行。但这就需要用户在程序中指定特定查询语句在特定实例上执行。