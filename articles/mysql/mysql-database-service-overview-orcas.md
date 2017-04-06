<properties linkid="" urlDisplayName="" pageTitle="Azure Database for MySQL服务概述 - Azure 微软云" metaKeywords="Azure云,技术文档,文档与资源,MySQL,数据库,入门指南,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="Azure Database for MySQL服务概述列举了MySQL Database on Azure云数据库服务的各项特性与优势。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />  

<tags ms.service="mysql" ms.date="04/06/2017" wacn.date="04/06/2017" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-service-overview-orcas/)

# About Azure Database for MySQL

Azure Database for MySQL是我们在Azure平台上推出的MySQL云数据库服务。它是一种关系型数据库服务，全面兼容MySQL协议，为用户提供了一个全托管的性能稳定、可快速部署、高可用、高安全性的数据库服务。同时，为了适应用户不同的性能需求，Azure Database for MySQL服务目前提供基础和标准两种不同的服务层级，共四个性能版本，性能从低到高按倍数提高。

## Azure Database for MySQL有哪些优势？
Azure Database for MySQL服务是一款全托管的数据库平台服务。与用户在本地自建数据库或用虚机搭建的方案相比，用户无需关心基础设施的细节，从而更好地专注于业务逻辑的开发。

## 支持弹性扩展与收缩，简化购买体验

您不再需要反复计算数据库服务器的配置需求，您也不再需要为如何调节各方面的配置以得到最好的数据库性能而烦恼，Azure Database for MySQL服务提供了多个不同性能的版本，您可以根据当前的性能需求选择一个相匹配的版本，后续再根据应用的实际运行情况进行弹性扩展或收缩。

## 提供完善的监控体验

Azure Database for MySQL为您提供多达20多种监控指标，支持您实时监控各项数据库性能指标以及历史数据，让您更好地了解使用情况，及时发现问题并进行调整。同时支持开启下载数据库日志，以便数据库管理员更好地管理和优化数据库。具体请参考：监控管理Azure Database for MySQL。

## 提供高可用性和安全性

Azure Database for MySQL内置高可用架构设计，全面保障数据库服务器的可用性达到99.99%以上，可以做到快速故障转移。同时，Azure Database for MySQL提供增量备份恢复方案以及防火墙功能，可将您的客户端公网IP地址（也可以是IP地址段）加入到白名单中；此外，支持通过SSL链接访问数据库。关于SSL的配置，请参考：使用SSL安全访问Azure Database for MySQL。我们为您提供增量备份以及数据库还原功能，系统会自动备份过去35天的数据库增量部分（基础版为7天），以实现基于任意时间点的回滚。您可以将数据库还原到过去35天中的任意时间点（基础版为7天）。（请参考：Azure Database for MySQL备份和恢复，详细了解备份与恢复方案。）

## 快速部署，无需人工维护基础架构

Azure Database for MySQL帮助您轻松快速地在一两分钟内在云端部署一个具备高可用的MySQL数据库服务器，提供自动软件修补功能，让您无需对数据库服务进行任何基础架构的维护。请参考入门教程：创建使用你的第一个MySQL云数据库服务器，了解如何快速部署MySQL数据库服务器。

## 支持熟悉的平台和工具

Azure Database for MySQL兼容MySQL协议，支持MySQL 5.6以及 MySQL 5.7版本。您可以用兼容MySQL的平台与技术进行开发与集成，也可以用您熟悉的管理工具，如mysql.exe、Workbench等进行操作。

## 后续步骤

### 了解服务层和版本

了解Azure Database for MySQL的不同性能版本。

### Azure Database for MySQL数据库入门

帮助您快速了解如何使用Azure Database for MySQL。