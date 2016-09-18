<properties linkid="" urlDisplayName="" pageTitle="MySQL Database on Azure服务概述 - Azure 微软云" metaKeywords="Azure云,技术文档,文档与资源,MySQL,数据库,入门指南,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="MySQL Database on Azure服务概述列举了MySQL Database on Azure云数据库服务的各项特性与优势。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />  

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-tech-overview/)
- [English](/documentation/articles/mysql-database-enus-tech-overview/)

# MySQL Database on Azure服务概述 #

## 什么是 MySQL Database on Azure？ ##
MySQL Database on Azure是我们在Azure平台上推出的MySQL云数据库服务。它是一种关系型数据库服务，全面兼容MySQL协议，为用户提供了一个全托管的性能稳定、可快速部署、高可用、高安全性的数据库服务。同时，为了适应用户不同的性能需求，MySQL on Azure服务目前提供六个不同的性能版本，性能从低到高按倍数提高。

## MySQL Database on Azure有哪些优势？ ##
MySQL Database on Azure服务是一款全托管的数据库平台服务，与用户在本地自建数据库或用虚机搭建的方案相比，用户无需关心基础设施的细节，从而更好地专注于业务逻辑的开发。

### MySQL Database on Azure具有以下核心优势： ###

#### 支持弹性扩展与收缩，简化购买体验 ####

您不再需要反复计算数据库服务器的配置需求，您也不再需要为如何调节各方面的配置以得到最好的数据库性能而烦恼，MySQL Database on Azure服务提供了多个不同性能的版本，您可以根据当前的性能需求选择一个相匹配的版本，后续再根据应用的实际运行情况进行弹性扩展或收缩。并且，如果您的业务属于读多写少的场景，我们建议您创建配置只读实例，进行读写分离，让您的业务可以更好地横向扩展。具体请参考：[创建和使用只读实例](https://www.azure.cn/documentation/articles/mysql-database-read-replica)。

#### 提供完善的监控体验 ####

MySQL Database on Azure为您提供多达20多种监控指标，支持您实时监控各项数据库性能指标以及历史数据，让您更好地了解使用情况，及时发现问题并进行调整。同时支持开启下载数据库日志，以便数据库管理员更好地管理和优化数据库。具体请参考：[监控管理MySQL Databse on Azure](https://www.azure.cn/documentation/articles/mysql-database-operation-monitoring-metrics)。

#### 数据高度可靠和安全 ####

MySQL Database on Azure基于Azure的地域冗余存储来提供数据的高可靠性，在同一个区域中自动同步复制3个本地副本，同时在超过一千公里外的区域通过异步的方式复制3个异地副本，保证数据的高度可靠。同时，MySQL on Azure提供完善的备份恢复方案。MySQL Database on Azure还提供防火墙功能，可将您的客户端公网IP地址（也可以是IP地址段）加入到白名单中；此外，支持通过SSL链接访问数据库。关于SSL的配置，请参考：[使用SSL安全访问MySQL Database on Azure](https://www.azure.cn/documentation/articles/mysql-database-ssl-connection)。具体而言，在备份还原方面我们为您提供**完全备份**、**增量备份**以及**数据库还原**功能：



- **完全备份——**MySQL Database on Azure每天在您指定的时间自动完全备份您的MySQL数据库并可保留30天；此外您也可以自己做即时的手动完全备份。备份所占用的存储都是完全免费的，您无需为此支付任何额外费用！
- **增量备份——**MySQL Database on Azure会自动备份过去七天的数据库增量部分，以实现基于任意时间点的回滚。
- **数据库还原——**您可以将数据库还原到过去七天中的任意时间点、也可以还原到一个完全备份。除了还原到当前的数据库实例，MySQL Database on Azure也支持您恢复到新实例。

（请参考：[MySQL Database on Azure备份和恢复](https://www.azure.cn/documentation/articles/mysql-database-point-in-time-restore)，详细了解备份与恢复方案。）

#### 提供高可用，支持灾备恢复，保障业务连续性 ####

MySQL Database on Azure内置高可用架构设计，全面保障数据库服务器的可用性达到99.9%以上，可以做到快速故障转移。同时，MySQL Database on Azure为您提供灾备恢复方案，保障您的业务连续性，具体内容请参考：[MySQL on Azure业务连续性方案](https://www.azure.cn/documentation/articles/mysql-database-business-continuity-disaster-recovery)。

#### 快速部署，无需人工维护基础架构层 ####

MySQL Database on Azure帮助您轻松快速地在一两分钟内在云端部署一个具备高可用的MySQL数据库服务器，提供自动软件修补功能，让您无需对数据库服务进行任何基础架构的维护。请参考入门教程：[创建使用你的第一个MySQL云数据库服务器](https://www.azure.cn/documentation/articles/mysql-database-get-started)，了解如何快速部署MySQL数据库服务器。

#### 支持熟悉的平台和工具 ####

MySQL Database on Azure兼容MySQL协议，支持MySQL 5.5和MySQL 5.6以及 MySQL 5.7版本。您可以用常见的支持MySQL的平台与技术进行开发与集成，也可以用您熟悉的管理工具，如Workbench、Navicat等进行操作。

#### 支持异地数据库同步，满足企业构建混合云的需求 ####

MySQL Database on Azure支持异地数据库同步的功能，用户可以将Azure以外的MySQL实例设为主库，把运行在Azure上的实例设为从库，通过标准的MySQL主从数据同步的方式同步数据到Azure以更好地满足企业各种混合云场景的需求。同时，数据库同步的功能也可以帮助您实现更完善更灵活的云端迁移体验。具体配置步骤请参考： [将数据库从本地同步复制到MySQL Database on Azure](https://www.azure.cn/documentation/articles/mysql-database-data-replication)。

#### 支持自动化开发 ####

MySQL Database on Azure支持您利用PowerShell（请参考：[使用PowerShell快速创建MySQL云数据库](https://www.azure.cn/documentation/articles/mysql-database-etoe-powershell)）或REST-API（请参考：[REST-API相关技术文档](https://www.azure.cn/documentation/articles/mysql-database-api-createserver)）直接进行自动化部署开发。