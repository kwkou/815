<properties linkid="" urlDisplayName="" pageTitle="如何高效连接到MySQL Database on Azure- Azure 微软云" metaKeywords="Azure 云，技术文档，文档与资源，MySQL,数据库，连接池，connection pool, Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="
合理的利用连接池访问MySQL Database on Azure会优化性能。本文介绍如何使用连接池有效地访问MySQL Database on Azure，并给出以JAVA和PHP为例的示例代码供参考。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-connection-pool/)
- [English](/documentation/articles/mysql-database-enus-connection-pool/)

# 如何高效连接到MySQL Database on Azure<sup style="color: #a5ce00; font-weight: bold; text-transform: uppercase; font-family: '微软雅黑'; font-size: 20px;" class="wa-previewTag"></sup>

数据库连接是一种有限的资源，合理的利用连接池访问MySQL Database on Azure会优化性能。本文介绍如何使用连接池或长连接高效效地访问MySQL Database on Azure。

## 通过连接池访问数据库 （推荐）##
由于在发起连接时，MySQL Database on Azure会做更多的验证工作，导致发起连接的开销相对于本地数据库更大。对于数据库连接的管理能够显著影响到整个应用程序的性能。为了使您的程序能够达到性能最优，目标是降低发起连接的次数，把发起连接的时间不放在关键的代码路径上。我们强烈建议您使用数据库连接池(connection pool)或长连接(persistent connection)连接MySQL Database on Azure。 
数据库连接池负责建立，管理和分配数据库连接。 当应用程序申请一个数据库连接时，它优先分配一个现有的空闲的数据库连接，而不是重新建立一个。当数据库连接使用完毕后，它会回收该连接以备再次使用，而不是直接关闭该连接。

为了更好地说明，本文提供[以JAVA为例的一段示例代码](http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/documents/MySQLConnectionPool.java )，供参考，您也可以参考[Apache common DBCP](http://commons.apache.org/proper/commons-dbcp/)来了解更多。

>[AZURE.NOTE]**服务器端会设置超时机制，如果一个连接在一段时间内处于闲置状态，服务器就会关闭这个链接，以释放不必要的资源占用。因此为了保障在您使用时，您的长链接的有效性，请设置验证机制，具体配置可参考[如何在客户端配置验证机制确认长连接有效性](/documentation/articles/mysql-database-validationquery/)**

## 通过长连接访问数据库 （推荐）##
PHP中建议您使用长连接，长连接的概念与连接池的概念类似。需要注意的是PHP目前有三种驱动，除Mysqli外，其他两种驱动均支持Persistent Connection. 

查看[利用PDO建立长连接](http://php.net/manual/en/pdo.connections.php)； 
查看[利用MySQL引擎建立长连接](http://php.net/manual/en/function.mysql-pconnect.php)。

将短连接修改至长连接对于代码的改动不大，但是对于性能的提高在很多典型应用场景有很大的作用。

## 通过等待重试机制短连接访问数据库##
基于资源的有效性，我们强烈推荐您使用连接池或是长连接访问数据库。但如果您使用短连接，在并发连接数接近上限，连接出现失败的情况下，我们建议您尝试多次连接，可以适当设定等待时间，初次等待时间可以较短，后可以进行多次的随机事件的等待。

