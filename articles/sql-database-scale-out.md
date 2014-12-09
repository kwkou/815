<properties linkid="manage-services-how-to-scale-a-sqldb" urlDisplayName="How to scale" pageTitle="如何缩放 SQL Database - Azure" metaKeywords="" description="了解用于在 Azure 中缩放 SQL Database 的选项。" metaCanonical="" services="sql-database" documentationCenter="" title="如何缩放 SQL Database 解决方案" authors="" solutions="" manager="" editor="" />

# 如何缩放 SQL Database 解决方案

在 Azure 上，数据库可伸缩性是横向扩展的同义词，在横向扩展时，工作负载将跨数据中心内的多个商品服务器重新分配。横向扩展是一种解决数据容量或性能问题的策略。一个具有高速增长趋势的超大型数据库无论是供部分用户还是供大量用户访问，最终都将需要横向扩展策略。

通过联合可在 Azure 上以最佳方式实现横向扩展。SQL Database 联合基于水平分片，其中一个或多个表按行进行拆分，并被划分到多个联合成员中。

联合并不是解决每个可伸缩性问题的唯一方法。有时，可根据数据的特征或应用程序要求采用更简单的方法。下表按复杂性顺序显示了可能的解决方案。

## 增加数据库的大小

数据库按照固定大小创建，该固定大小受每个版本强加的最大大小制约。对于 Web 版，您可以将数据库大小最大增加到 5 GB。对于企业版，最大数据库大小为 150 GB。增加数据容量的最有效方式是更改版本和最大大小：

     ALTER DATABASE school MODIFY (EDITION = 'Business', MAXSIZE=10GB);

## 使用多个数据库和分配用户

在有限的情况下，您可以创建数据库的副本，然后跨每个数据库分配登录名和用户。在选择联合方式之前，这是一个重新分配工作负载的常见方法。此方法不仅适用于短期使用并且随后会合并到您保存到本地的主要数据库中的数据库，还适用于提供只读数据的解决方案。

## 使用联合

使用 SQL Database 中的联合方法可大大提高可伸缩性和性能。数据库中的一个或多个表按行进行拆分，并被划分到多个数据库（联合成员）中。这种水平分区通常称为“分片”。如果您需要实现缩放、提高性能或管理容量，则该方法在此类主要情形中很有用。

企业版支持联合。有关详细信息，请参阅 [SQL Database 中的联合][SQL Database 中的联合]和 [SQL Database 联合教程 - DBA][SQL Database 联合教程 - DBA]。

## 考虑其他存储形式。

请记住，Azure 支持多种数据存储形式，包括表存储和 Blob 存储。如果您不需要相关功能，则表或 Blob 存储可能会更经济。

  [SQL Database 中的联合]: http://msdn.microsoft.com/zh-cn/library/azure/hh597452.aspx
  [SQL Database 联合教程 - DBA]: http://msdn.microsoft.com/zh-cn/library/azure/hh778416.aspx
