<properties linkid="" urlDisplayName="" pageTitle="了解服务层和版本 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,性能,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,ASDB基准" description="针对服务层和不同版本的性能介绍,为您选择MySQL 数据库 on Azure提供了详细的参考。我们按照ASDB基准,提供了不同版本的测试数据供您参考。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-performance-guidance-asdb-test-result/)
- [English](/documentation/articles/mysql-database-enus-performance-guidance-asdb-test-result/)

#了解服务层和版本

##ASDB (Azure SQL 数据库 Benchmark)测试结果参考 
ASDB（Azure SQL 数据库 Benchmark）是Microsoft为了量化提高云计算资源配置如何转换为更高数据库性能而设计的一种数据库测试基准。该基准包括了几乎所有在线事务处理（OLTP）工作负载中发生的基本操作。尽管该基准是基于云计算设计的，但数据库架构、数据填充和事务在设计上广泛代表了 OLTP 工作负载中最常用的基本元素。

##ASDB基准的特性 
ASDB基准的基本特性包括：

- ASDB基准使用的数据库包括了六个表。其中两个表是固定大小，即行数是不变的，三个是缩放性表，表的行数与数据库达到的性能成比例，还有一个是增长性表，其大小在运行基准的过程中会进行数据插入和删除。

- 表的结构包含混合形式的数据类型，包括整数、数字、字符和日期/时间，同时包含主键和辅助键，但不包含任何外键。例如，表之间没有任何引用的完整性约束。

- 数据库的大小与并发链接和最大性能成比例。例如，有100 个并发链接的数据库可以实现的最大事务率为 100 TPS，这是因为基准应用程序有步调控制以保证每个链接平均每秒最多完成不超过1个事务。促成更高的 TPS 率需要有更多的并发链接和更大的数据库。

- ASDB基准包括了九个不同的事务类型。每个事务类型侧重于数据库引擎和系统硬件中的某一特定的一组系统特性，不同事务类型的侧重点都不一样。事务利用四个基本的 SQL DML 操作（SELECT、UPDATE、INSERT 和 DELETE）来测试数据库引擎以及基础的 CPU、内存、IO 和网络等硬件资源。

有关ASDB基准更详细的说明，请参阅 [ASDB 基准概述](https://msdn.microsoft.com/zh-cn/library/azure/dn741327.aspx#Benchmark_summary)。

##ASDB基准下的测试结果

以下表格显示了每个版本在ASDB基准下可以稳定持续的达到的性能和测试时相应的并发链接数。每次测试在稳定状态下至少运行一个小时以上。

<table border="1" cellspacing="0" cellpadding="0" width="477">
  <tr>
    <td width="47" valign="top"><p><strong>版本</strong><strong> </strong></p></td>
    <td width="195" valign="top"><p align="center"><strong>事</strong><strong>务</strong><strong>率</strong><strong> (TPS)</strong></p></td>
    <td width="122" valign="top"><p align="center"><strong>测试数据库大小</strong><strong> </strong></p></td>
    <td width="113" valign="top"><p align="center"><strong>并发链接数</strong><strong> </strong></p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 1</strong></p></td>
    <td width="195" valign="top"><p align="center">10</p></td>
    <td width="122" valign="top"><p align="center">1.7GB</p></td>
    <td width="113" valign="top"><p align="center">12</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 2</strong></p></td>
    <td width="195" valign="top"><p align="center">20</p></td>
    <td width="122" valign="top"><p align="center">3.0GB</p></td>
    <td width="113" valign="top"><p align="center">22</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 3</strong></p></td>
    <td width="195" valign="top"><p align="center">60</p></td>
    <td width="122" valign="top"><p align="center">8.9GB</p></td>
    <td width="113" valign="top"><p align="center">65</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 4</strong></p></td>
    <td width="195" valign="top"><p align="center">200</p></td>
    <td width="122" valign="top"><p align="center">28.6GB</p></td>
    <td width="113" valign="top"><p align="center">210</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 5</strong></p></td>
    <td width="195" valign="top"><p align="center">300</p></td>
    <td width="122" valign="top"><p align="center">43.6GB</p></td>
    <td width="113" valign="top"><p align="center">320</p></td>
  </tr>
  <tr>
    <td width="47" valign="top"><p align="center"><strong>MS 6</strong></p></td>
    <td width="195" valign="top"><p align="center">450</p></td>
    <td width="122" valign="top"><p align="center">73.6GB</p></td>
    <td width="113" valign="top"><p align="center">470</p></td>
  </tr>
</table>

>[AZURE.NOTE]必须知道，与所有基准一样，ASDB 只提供代表性和指导性的结果。使用基准应用程序实现的事务率与使用其他应用程序实现的事务率将会不同。该基准包括不同事务类型的集合，这些事务类型是针对包含一系列表和数据类型的架构运行的。尽管该基准会执行所有 OLTP 工作负载共有的相同基本操作，但它不代表任何特定类别的数据库或应用程序。该基准的目标是针对上调或下调性能级别时预期的数据库相对性能提供合理的指导。事实上，数据库具有不同的大小和复杂性，会遇到不同的工作负载混合形式，并且会以不同的方式做出响应。例如，IO 密集型应用程序可能很快就会达到 IO 阈值，而内存密集型应用程序可能很快就会达到内存限制。在负载增加的情况下，不保证任何特定的数据库会像ASDB基准结果中所示的那样表现。
