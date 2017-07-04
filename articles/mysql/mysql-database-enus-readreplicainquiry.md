<properties
    linkid=""
    urlDisplayName=""
    pageTitle="MySQL Service Questions – Microsoft Azure Cloud"
    metaKeywords="Azure Cloud, technical documentation, documents and resources, MySQL, database, FAQs,Data secure access,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ"
    description="Provides quick answers for common technical questions encountered by users when using MySQL Database on Azure. Contact technical support if you have any further questions."
    metaCanonical=""
    services="MySQL"
    documentationCenter="Services"
    authors="v-chenyh"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="mysql_en"
    ms.author="v-chenyh"
    ms.topic="article"
    ms.date="1/10/2017"
    wacn.date="1/10/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-readreplicainquiry/)
- [English](/documentation/articles/mysql-database-enus-readreplicainquiry/)

#<a name=""></a>Master-slave replication issues
> [AZURE.SELECTOR]
- [All issues](/documentation/articles/mysql-database-enus-tech-faq/)
- [Service consulting](/documentation/articles/mysql-database-enus-serviceinquiry/)
- [Connection issues](/documentation/articles/mysql-database-enus-connectioninquiry/)
- [Security consulting](/documentation/articles/mysql-database-enus-securityinquiry/)
- [Compatibility issues](/documentation/articles/mysql-database-enus-compatibilityinquiry/)
- [Master-slave replication issues](/documentation/articles/mysql-database-enus-readreplicainquiry/)

## <a name="ms4ms5ms6ms5"></a>**I want to change the performance version of the master instance. For example, I might want to go up from MS4 to MS5 or go down from MS6 to MS5. Will the version for slave instances be updated as well?** 

A: The performance version for slave instances will not change with that of the master instance. However, you can change the performance version of slave instances separately.

## <a name="55"></a>**I would like to use the master-slave replication function, but my database instance is on version 5.5. What should I do?** 

Master-slave replication only supports version 5.6 and above. You can manually upgrade the database implementation to version 5.6 or 5.7 first, and then use the replicate function.

## <a name=""></a>**There is sometimes a very high latency between my master and slave instances (e.g. more than five minutes). What causes this?** 

High latency between master and slave instances can be caused by any of the following situations:

1. If you have not set up a primary key, replication will require a full table scan of the slave instance, which can severely affect performance. Individual full table update operations can even cause the slave instance to jam.
2. The slave instance also needs a lot of time to perform the update when a large amount of data is being updated, which can jam replication and cause delays.
3. If the read-only instance configuration is too low, it will not be possible to perform updates as fast as on the master instance.
4. Large amounts of read operations from the slave instance will affect the simultaneous implementation of updates generated on the master instance.

## <a name=""></a>**How can I eliminate high latency between the master and slave instances?** 

1. It is essential to set up a primary key for the database. Not doing so may result in replication slowing down or even jamming up.
2. Upgrading the performance version of read-only instances can directly increase throughput for read-only instances. This allows updates to be performed more quickly and speeds up the synchronization process, while also enabling rapid responses for read operations.
3. If a particular query phrase is very time-sensitive, it will fail to meet requirements no matter how much it is shortened, so you can run the query on the master database. However, this process means that you need to specify a particular query phrase to run on a particular instance.

<!--HONumber=May17_HO3-->