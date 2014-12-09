<properties linkid="develop-dotnet-performance" urlDisplayName="Performance" pageTitle="Performance best practices - Azure" metaKeywords="Azure optimization, Azure best practice performance" description="Learn about best practices for performance in Azure." metaCanonical="" services="cloud-services,sql-database,storage,service-bus,virtual-network" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />

# 有关 Azure 应用程序性能的最佳实践

本指南提供了有关优化 Azure 应用程序性能时应遵循的最佳实践和技术的说明性指导。

请注意，使用 Azure 有许多优势：性能只是其中之一。本白皮书中的建议主要关注性能。在其他场景中，性能不是如此重要：例如，你可能希望利用将物理硬件管理负载转移到 Azure 的功能，而“即付即用”功能可能尤其吸引人。本白皮书不尝试评估性能优先级较低的场景。

## 概述

性能可定义为[“完成的有用工作量与使用的时间和资源的比较”][“完成的有用工作量与使用的时间和资源的比较”]。

该定义从两个方面阐释了性能的含义：指标和资源。性能指标是为满足业务需求而必须实现的数量，其中包括响应时间、吞吐量、可用性等。性能还包括达到给定级别的性能指标所需的资源使用级别。由于成本几乎总是一项业务要求，而资源耗费金钱，因此，性能意味着尽可能高效地使用资源。

### 性能生命周期

你可以在应用程序生命周期中的两个不同时间点影响性能：

-   设计期间，你可以在此时做出可影响性能的基本体系结构决策；以及
-   运行时，你可以在此时发现瓶颈、执行监视和测量等。

这两项活动都是必需的。本白皮书主要关注设计阶段，因为在运行时较难修复体系结构错误。

#### 应用程序建模

为你的应用程序的最重要的客户应用场景生成模型非常重要。在这里，“重要”表示对性能具有最大影响。确定这些场景和所需的应用程序活动将使你能够执行概念证明测试。

#### 概念证明性能测试

完整的端到端性能测试是设计和部署应用程序期间的一个关键步骤。Azure 应用程序由许多部分组成，其中可能包括定制的组件以及 Microsoft 提供的组件。Microsoft 无法对这些组件的每种可能组合执行性能测试。因此，对你的应用程序执行完整而恰当的性能测试是任何部署的一个关键步骤。

根据你生成的应用程序模型，为确保你的应用程序满足可伸缩性和延迟方面的性能要求，接下来你应尽快对你的应用程序执行概念证明测试，并对其执行负载测试以验证应用程序体系结构。验证你的初始体系结构和假设极其重要。你不希望发现你的应用程序在运行后无法承受预期负载！Visual Studio 提供了用于执行负载测试的工具，如 [Azure 中的 Visual Studio 负载测试概述][Azure 中的 Visual Studio 负载测试概述]中所述。

### Azure 性能的不同之处

Azure 应用程序中可实现的最显著性能改进来自资源的横向扩展和分区。在 Azure 中生成可扩展应用程序需要利用按资源的以下物理分区进行的资源横向扩展：SQL Database、存储、计算节点等。此分区方式允许并行执行应用程序任务，并且因为 Azure 可提供整个数据中心的资源并为你处理物理分区，因此它也是获得高性能的基础。实现此级别的总体性能需要使用适当的横向扩展设计模式。

自相矛盾的是，在从整体上为应用程序实现此高性能时，Azure 中的每个单独操作的性能不如本地的相应操作高，因为增加了网络延迟，并且由于故障转移操作等增强了可靠性。但通过恰当使用分区资源而实现的并行执行能力远远弥补了单独性能的不足。

### 所需的设计活动

一些性能因素随你的应用场景而变化。下一章将讨论以下事项：对资源进行扩展和分区、适当地定位数据以及优化 Azure 服务的使用。

此后的一章讨论与任何 Azure 应用程序相关的性能因素：网络延迟、暂时性连接等。

## 设计云环境中的性能

在设计 Azure 应用程序或将本地应用程序迁移到 Azure 时，你必须考虑依赖应用场景的以下方面：

-   资源横向扩展和分区：这是实现并行性从而获得高性能的基本机制。
-   数据体系结构：对你的应用程序数据的不同部分使用哪种数据存储
-   单独的 Azure 服务优化

Azure 之所以具有最大性能优势，是因为它能够对资源进行横向扩展和分区，从而能够大规模并行执行各项活动。在考虑大型 Azure SQL Database 时，这相当明显，并且对于可能成为瓶颈的任何资源，也是如此。

Azure 可提供以下数据存储选择，因此，能否做出正确的选择对性能具有巨大影响：

-   Azure SQL Database

<!-- * Azure Caching -->

-   Azure 表存储
-   Azure Blob 存储
-   Azure 驱动器
-   Azure 队列
-   Azure Service Bus 中转消息传送
-   “大数据”存储解决方案，例如 Hadoop

由于具体要求不同，我们将根据以下场景讨论如何执行这些操作：

-   使用 SQL Database 的 Azure 云服务
-   大量使用存储队列的 Azure 云服务
-   使用 MySQL 作为后端数据库的 Azure 网站

<!-- * "Big Data" applications  -->

-   使用 MySQL 后端数据库的应用程序

### 场景：云服务中的 SQL Database

优秀数据库设计的大多数原则仍适用于 Azure SQL Database。有大量介绍如何设计有效的 SQL Server 或 Azure SQL Database 架构的材料。有关 SQL Database 架构设计的若干参考包括：

-   [数据库设计和建模基础][数据库设计和建模基础]
-   [数据库设计方法][数据库设计方法]
-   [数据库设计][数据库设计]

有两项关键设计活动与 Azure 不同：

-   适当地定位数据：适当时，这可能需要将一些关系数据移动到 Azure Blob 或 Azure 表中。
-   确保尽可能实现可伸缩性：决定是否以及如何对你的数据进行分区。

#### 数据体系结构：从 SQL Database 中移出数据

应将通常驻留在本地 SQL Server 中的一些数据移动到 Azure 中的其他位置。

##### 将数据移动到 Azure Blob 中

不应将 Blob 数据（如图像或文档）存储在 SQL Database 中，而应将其存储在 Azure Blob 存储中。虽然此类数据通常存储在本地 SQL Server 中，但在云中，使用 Blob 存储服务将更加经济。通常，你可以使用 Blob 存储标识符替换指向 blob 数据的外键以维护检索 blob 数据的功能，并且将需要修改引用该数据的查询。

##### 将 SQL 表移动到 Azure 表存储

在决定是否使用 Azure 表存储时，你必须考虑成本和性能。使用表存储比使用 SQL Database 存储相同数据经济得多。不过，你必须仔细考虑数据要在多大程度上使用 SQL 的关系功能（例如联接、筛选、查询等）。如果数据很少使用此类功能，则非常适合将数据存储在 Azure 表中。

你可以考虑表存储的一种常见设计模式涉及一个包含许多行的表，例如常见的 AdventureWorks 示例数据库中的 Customers 表，其中许多列没有被大多数客户使用，而只是被一小部分客户使用。将这些列拆分到第二个表（可能名为 CustomerMiscellany）中是一种常见的设计模式，在这种模式下，客户和第二个表之间具有可选的 1 对 0 关系。你可以考虑将第二个表移动到表存储。你必须评估表的大小和访问模式在这种情况下是否经济高效。

有关表存储的更多讨论，请参阅：

-   [Azure 表存储和 Azure SQL Database - 比较与对照][Azure 表存储和 Azure SQL Database - 比较与对照]
-   [Azure 表存储性能注意事项][Azure 表存储性能注意事项]
-   [SQL Database 和 Azure 表存储][SQL Database 和 Azure 表存储]
-   [通过批处理 Azure 表存储插入操作来改进性能（可能为英文页面）][通过批处理 Azure 表存储插入操作来改进性能（可能为英文页面）]，其中讨论了一些性能结果。
-   [SQL Database 性能和弹性指南][SQL Database 性能和弹性指南]

#### 数据分区

最常用的分区资源之一是数据。如果你要创建 Azure 云服务，则应考虑使用 SQL Database 中通过联合提供的内置分片。

有关 SQL Database 联合的概述，请参阅 [SQL Database 中的联合][SQL Database 中的联合]。

##### 有关 SQL 联合的设计任务

在 Azure 云服务中使用 SQL Database 联合需要对经典设计原则进行一些修改。不过，在设计 SQL Database 联合时，必须从针对本地 SQL Server 数据库的优秀设计中的大多数任务着手。有两个主要设计任务将决定：

-   对哪些表进行联合；以及

-   将非联合表放在何处。

若要决定对哪些表进行联合，你需要查看你的数据库架构并确定相关聚合表。聚合是一组表，其中的所有表通过其外键按一对多的关系相关联（聚合的根除外）。

例如，在已知的 AdventureWorks 示例数据库中，一种可能的聚合是集合 {Customer、Order、OrderLine 和可能的其他内容}。另一种可能的聚合是 {Supplier、Product、OrderLine、Order}。

每个聚合是联合的候选项。你必须评估希望增加哪个位置的大小，并且还必须查看你的应用程序的工作负载：“非常符合”联合方案的查询（即，该查询不需要来自多个联合成员的数据）将运行良好。不太符合的查询将需要应用层中的逻辑，因为 SQL Database 当前不支持跨数据库联接。

若要查看检查用于联合的 AdventureWorks 数据库，以及分步演示设计中相关注意事项的设计分析示例，请参阅[使用联合完成数据库设计的优先扩展方法：第 1 部分 – 选择联合和联合键][使用联合完成数据库设计的优先扩展方法：第 1 部分 – 选择联合和联合键]。

在你决定要联合哪些表后，你必须将聚合根表的主键作为列添加到每个相关表中。

在决定对哪些表进行联合后，另一个问题是引用表以及其他数据库对象的位置。在[使用联合完成数据库设计的优先扩展方法：第 2 部分 – 为联合添加批注并部署架构][使用联合完成数据库设计的优先扩展方法：第 2 部分 – 为联合添加批注并部署架构]中，详细讨论了此主题。在[第 2 部分][第 2 部分]中对执行更高级的查询进行了介绍。

##### 自己动手分区

有许多演示数据分区方法的示例。如果你决定不使用联合对你的 SQL Database 实例进行分区，则必须选择适合你的应用程序的分区方法。下面是一些示例：

-   在发布联合技术之前，[如何使用 SQL Database 进行分片][如何使用 SQL Database 进行分片]一文包含此方面的详细信息。
-   [SQL Server 和 SQL Database 分片库][SQL Server 和 SQL Database 分片库]

##### 对其他资源进行分区

你可以对除 SQL Database 之外的其他资源进行分区。例如，你可能希望对应用程序服务器进行分区并将它们专用于特定数据库。让我们假定你的应用程序包含 N 台应用程序服务器，还包含 N 个数据库。如果允许每台应用程序服务器访问每个数据库，则需要建立 N 的平方个数据库连接，在某些情况下，这可能会达到 Azure 的硬限制。但是，如果你仅允许每台应用程序服务器访问几个数据库，将显著减少所用连接数。

根据你的应用程序，你可以对其他资源应用类似推论。

<!-- #### Caching ####  The Azure Caching Service provides distributed elastic memory for caching things like ASP.net session state, or commonly referenced values from SQL Database reference tables. Because the objects are in distributed memory, there is a considerable performance gain possible. Because Azure handles the caching infrastructure, there is little development cost in implementing it.   Plan to provide enough caching capacity so that you can cache frequently accessed objects. In SQL Database there are frequently reference tables used to convert numeric codes into longer descriptive character strings. These tables often include data such as Country and City names, valid Postal Code values, names of Departments within your company, etc. For smaller tables it may make sense to store the entire table in cache, for others you might only store the most frequently used values. The performance gain comes in multi-join queries that involve this data: for each value that is found in the cache, several disk accesses are saved. A good introduction and discussion of performance and caching in Azure is [Introducing the Azure Caching Service](http://go.microsoft.com/fwlink/?LinkId=252680). A more recent blog post on the subject is at [Windows #Azure Caching Performance Considerations](http://go.microsoft.com/fwlink/?LinkId=252681). -->

#### 场景：在 Azure 应用程序中使用队列

此场景的示例利用 StreamInsight 来使用稍后将处理的消息填充队列。

Azure 队列用于传递消息，临时分离子系统并提供负载平衡和负荷量。

Azure 具有两种备选队列技术：Azure 存储队列和 Service Bus。

Azure 存储队列提供了大队列大小、进度跟踪等功能。Service Bus 提供了发布/订阅、与 Windows Communication Foundation (WCF) 的完全集成、自动重复检测、有保证的先入先出 (FIFO) 传送等功能。

有关这两种技术的更完整的详细比较，请参阅 [Azure 队列和 Azure Service Bus 队列 - 比较与对照][Azure 队列和 Azure Service Bus 队列 - 比较与对照]。

有关 Service Bus 性能的讨论，请参阅[使用 Service Bus 中转消息传送改善性能的最佳实践][使用 Service Bus 中转消息传送改善性能的最佳实践]。
<!-- #### Scenario: "Big Data" Applications ####  "Big Data" is often found as a by-product of another system or application. Examples include:   * Web logs   * Other diagnostic, audit, and monitoring files   * Oil company seismic logs   * Click-data and other information left by people traversing the Internet   "Big Data" can be identified by the following criteria:   * Size (typically, hundreds of terabytes or larger)   * Type: non-relational, variable schema, files in a file system   The data is generally not suited for processing in a relational database.   There are four major kinds of non-SQL data storage:   * Key-value   * Document   * Graph   * Column-Family   Azure provides direct support for Hadoop, and also enables use of other technologies. For information about Azure HDInsight Service, see:   * [Big Data](/en-us/solutions/big-data/)  * [Azure HDInsight Service](/zh-cn/documentation/services/hdinsight/) * [Getting Started with Azure HDInsight Service](/zh-cn/documentation/articles/hdinsight-get-started/)  For some discussion of issues involved with various noSQL storage methods, see:   * [Getting Acquainted with NoSQL on Azure](http://go.microsoft.com/fwlink/?LinkId=252729)  * [AggregateOrientedDatabase](http://go.microsoft.com/fwlink/?LinkID=252731) * [PolyglotPersistence](http://go.microsoft.com/fwlink/?LinkId=252732)  -->

#### 其他 Azure 单个服务性能优化

有许多单独 Azure 服务的功能和设置会显著影响性能，因此必须对其进行分析。

##### Azure 驱动器

你可以通过 Azure 驱动器来模拟现有应用程序的硬盘驱动器使用情况，Azure 驱动器由 Windows Blob 提供支持，因而在单台计算机发生故障时可供持久使用。

##### 本地存储

虽然本地存储在计算机发生故障时不具有持久性，但它可用于保存经常访问的信息，或保存将在其他地方使用的中间结果。这种方式经济高效，因为无需支付任何费用。
<!-- ##### Azure Access Control Service (ACS) #####  The two main factors affecting ACS resource usage, and thus performance, are the token size, and encryption. Further discussion is at [ACS Performance Guidelines](http://go.microsoft.com/fwlink/?LinkId=252747).  -->

##### 序列化

序列化在性能优化过程中的作用不明显，但在一些应用场景中，它可显著减少网络流量。有关序列化大小如何因协议而异的示例，请参阅 [Azure Web 应用程序和序列化][Azure Web 应用程序和序列化]中演示的减少情形。

如果要移动的数据量是性能问题，可使用最小的可用序列化。在序列化性能不理想时，可考虑使用自定义或非 Microsoft 第三方序列化格式。通常，概念证明测试是关键。

### 使用 mySQL 的 Azure 网站

以下链接提供了有关 MySQL 的性能建议：

-   在 [][]<http://mysql.com></a> 上搜索 *performance* 可找到许多资源。
-   此外还可以查看 [][1]<http://forums.mysql.com/list.php?24></a> 上的论坛等其他资源。

## 设计共享系统

Azure 旨在运行多个并发应用程序，复制它们以在多台计算机上进行故障转移。这将以多种方式影响应用程序性能：

-   暂时性连接

-   限制最大使用率的资源节流

-   网络延迟

-   服务的物理位置

这些注意事项适用于所有应用程序体系结构，因为它们由 Azure 数据中心的物理基础结构决定。有关详细讨论，请参阅 [SQL Database 性能和弹性指南（可能为英文页面）][SQL Database 性能和弹性指南（可能为英文页面）]。

### 网络延迟

Azure 是共享资源的基于服务的平台，这意味着会定期发生两种类型的延迟或中断。第一种是通过 Internet 发出请求和接收响应所花的时间。由于这些请求和响应在返回到客户端之前可能经过任何数量的路由器，因此，超时和断开连接比在本地固定网络中更频繁。第二种是共享资源系统（如 Azure）创建数据的备份版本以实现数据持久性，以及替换请求并将其重新路由到任何已删除实例所花的时间。这些延迟和失败对了解如何补偿以满足应用程序中的任何性能要求很重要。现实级别的负载测试将为你提供有关你遇到的延迟的详细信息。

应考虑可能将出现更多延迟，因为云数据中心实际上可能比你的本地服务器更遥远。

### 服务的物理位置

如果可能，请将同一数据中心内的不同节点或应用层放置在一起。否则，网络延迟和成本将更大。

例如，应将 Web 应用程序与它所访问的 SQL Database 实例放在同一数据中心，而不要将其放置在其他数据中心或本地。

### 暂时性连接

你的应用程序必须能够处理放弃的连接。放弃的连接对云体系结构来说是不可避免的且是固有的（例如，替换死节点、拆分 SQL Database 中的联合成员等操作）。立即处理此问题的最佳框架是[瞬时故障处理应用程序块（可能为英文页面）][瞬时故障处理应用程序块（可能为英文页面）]。

### 限制

在服务领域，资源可能非常精细，并且你只需为所使用的资源付费。不过，所有资源都有大小、速度或吞吐量的最低保证 – 例如，如果你需要超过指定大小的数据库，则这很重要 – 而且，在一些情况下，还对重要服务具有外部限制。因为 Azure 应用程序与其他应用程序运行在共享环境中，所以 Azure 应用你必须考虑的许多资源限制。如果你超过资源使用限制，则进一步请求该资源将导致异常。

### 物理容量

性能规划的一个重要事项是容量规划：如果你没有提供足够的各种存储，则你的应用程序可能根本无法运行。同样地，内存或处理器容量不足会显著降低你的应用程序的执行速度。

Azure 明显减少了容量规划所涉及的工作，因为许多旧活动（尤其是获取和设置计算机）已发生显著变化。在 Azure 中，容量规划不再关注计算的物理元素，而是完成更高级别的抽象工作，询问需要多少以下对象：

-   计算节点
-   Blob 存储
-   表存储
-   队列等

并且由于 Azure 具有可伸缩性，初始容量决策不是一成不变的：增加（或减少）Azure 资源比较容易。虽然如此，但进行准确的容量规划依然很重要，因为这将确保在应用程序投入生产时，没有试用期和与容量相关的错误。

对于其资源需求随时间显著波动的应用程序，请考虑使用[自动缩放应用程序块][自动缩放应用程序块]。此应用程序块允许你为角色实例的增加和减少设置规则。定义了两种规则：

-   约束规则，即，按一天的时间设置实例的最大/最小数

-   反应规则，它们在满足某个条件（如 CPU 使用百分比）时生效

你还可以定义自定义规则。有关详细信息，请参阅[自动缩放应用程序块（可能为英文页面）][自动缩放应用程序块]。

容量规划本身是一门完整的学科知识，本白皮书假设你已掌握它。有关 Azure 中容量规划的详细讨论，请参阅[针对 Service Bus 队列和主题的容量规划][针对 Service Bus 队列和主题的容量规划]。

## 运行时的性能监视和调整

即使是最精心的设计也无法保证在运行时不出现性能问题，因此需要持续监视应用程序性能，以验证是否已达到所需的性能指标，并处理没有达到这些指标的情况。即使是设计良好的应用程序也可能遇到意外事件，例如使用率指数增长或运行时环境可能发生变化，这些情况会导致性能问题，需要对其进行调整。通常，识别并消除瓶颈是此过程的重要部分。

能够在运行时解决性能问题需要完成一些内置日志记录和正确处理异常的前期工作，以便在可能出现问题时排查这些问题。有关此方面的完善解决方法，请参阅[有关开发 Azure 应用程序的问题排查最佳实践][有关开发 Azure 应用程序的问题排查最佳实践]。

可借助适当工具监视每项 Azure 服务的持续性能。另外，应将日志记录工具内置到应用程序中，以提供排查并解决性能问题所需的详细信息。

### SQL Database

请注意，SQL 事件探查器目前在 Azure 中不可用。可使用多种解决方法来获取所需的性能信息。开发期间的一个替代方法是在本地版本的数据库中执行初始测试，此时可使用 SQL 事件探查器。

你还可以使用 SET STATISTICS Transact-SQL 命令，并使用 SQL Server Management Studio 查看查询生成的执行计划，因为对高效查询进行编码是提升性能的关键。有关详细讨论以及如何执行此操作的分步说明，请参阅[了解 SQL Database 的性能][了解 SQL Database 的性能]。另一种有趣的方法是分析 [SQL Database 和本地 SQL Server][SQL Database 和本地 SQL Server] 之间的性能。

有关动态管理视图的两个主题为：

-   [使用动态管理视图监视 SQL Database][使用动态管理视图监视 SQL Database]
-   [SQL Database 可使用 DMV 分析你是否缺少 SQL 事件探查器][SQL Database 可使用 DMV 分析你是否缺少 SQL 事件探查器]

### 分析资源和工具

许多第三方非 Microsoft 工具可用于分析 Azure 性能：

-   [Cerebrata][Cerebrata]
-   [SQL Server 和 SQL Database 性能测试：Enzo SQL 基线（可能为英文页面）][SQL Server 和 SQL Database 性能测试：Enzo SQL 基线（可能为英文页面）]

其他资源

-   [SQL Database 性能和弹性指南][SQL Database 性能和弹性指南（可能为英文页面）]
-   [SQL Database][SQL Database]
-   [存储][存储]
-   [联网][联网]
-   [Service Bus][Service Bus]

<!-- * [Azure Planning - A Post-decision Guide to Integrate Azure in Your Environment](http://go.microsoft.com/fwlink/?LinkId=252884)  -->

  [“完成的有用工作量与使用的时间和资源的比较”]: http://go.microsoft.com/fwlink/?LinkId=252650
  [Azure 中的 Visual Studio 负载测试概述]: http://www.visualstudio.com/get-started/load-test-your-app-vs
  [数据库设计和建模基础]: http://go.microsoft.com/fwlink/?LinkId=252675
  [数据库设计方法]: http://go.microsoft.com/fwlink/?LinkId=252676
  [数据库设计]: http://go.microsoft.com/fwlink/?LinkId=252677
  [Azure 表存储和 Azure SQL Database - 比较与对照]: http://msdn.microsoft.com/zh-cn/library/jj553018.aspx
  [Azure 表存储性能注意事项]: http://go.microsoft.com/fwlink/?LinkId=252663
  [SQL Database 和 Azure 表存储]: http://go.microsoft.com/fwlink/?LinkId=252664
  [通过批处理 Azure 表存储插入操作来改进性能（可能为英文页面）]: http://go.microsoft.com/fwlink/?LinkID=252665
  [SQL Database 性能和弹性指南]: http://go.microsoft.com/fwlink/?LinkId=221876
  [SQL Database 中的联合]: http://go.microsoft.com/fwlink/?LinkId=252668
  [使用联合完成数据库设计的优先扩展方法：第 1 部分 – 选择联合和联合键]: http://go.microsoft.com/fwlink/?LinkId=252671
  [使用联合完成数据库设计的优先扩展方法：第 2 部分 – 为联合添加批注并部署架构]: http://blogs.msdn.com/b/cbiyikoglu/archive/2012/04/12/scale-first-approach-to-database-design-with-federations-part-2-annotating-schema-for-federations.aspx
  [第 2 部分]: http://blogs.msdn.com/b/cbiyikoglu/archive/2012/01/19/fan-out-querying-in-federations-part-ii-summary-queries-fanout-queries-with-top-ordering-and-aggregates.aspx
  [如何使用 SQL Database 进行分片]: http://go.microsoft.com/fwlink/?LinkId=252678
  [SQL Server 和 SQL Database 分片库]: http://go.microsoft.com/fwlink/?LinkId=252679
  [Azure 队列和 Azure Service Bus 队列 - 比较与对照]: http://msdn.microsoft.com/zh-cn/library/hh767287.aspx
  [使用 Service Bus 中转消息传送改善性能的最佳实践]: http://msdn.microsoft.com/zh-cn/library/hh528527.aspx
  [Azure Web 应用程序和序列化]: http://go.microsoft.com/fwlink/?LinkId=252749
  []: http://go.microsoft.com/fwlink/?LinkId=252775
  [1]: http://go.microsoft.com/fwlink/?LinkId=252776
  [SQL Database 性能和弹性指南（可能为英文页面）]: http://go.microsoft.com/fwlink/?LinkID=252666
  [瞬时故障处理应用程序块（可能为英文页面）]: http://go.microsoft.com/fwlink/?LinkID=236901
  [自动缩放应用程序块]: http://go.microsoft.com/fwlink/?LinkId=252873
  [针对 Service Bus 队列和主题的容量规划]: http://go.microsoft.com/fwlink/?LinkId=252875
  [有关开发 Azure 应用程序的问题排查最佳实践]: http://go.microsoft.com/fwlink/?LinkID=252876
  [了解 SQL Database 的性能]: http://go.microsoft.com/fwlink/?LinkId=252877
  [SQL Database 和本地 SQL Server]: http://go.microsoft.com/fwlink/?LinkId=252878
  [使用动态管理视图监视 SQL Database]: http://go.microsoft.com/fwlink/?LinkId=236195
  [SQL Database 可使用 DMV 分析你是否缺少 SQL 事件探查器]: http://go.microsoft.com/fwlink/?LinkId=252879
  [Cerebrata]: http://go.microsoft.com/fwlink/?LinkId=252880
  [SQL Server 和 SQL Database 性能测试：Enzo SQL 基线（可能为英文页面）]: http://enzosqlbaseline.codeplex.com/
  [SQL Database]: http://azure.microsoft.com/zh-cn/services/sql-database/
  [存储]: http://go.microsoft.com/fwlink/?LinkId=246933
  [联网]: http://go.microsoft.com/fwlink/?LinkId=252882
  [Service Bus]: http://go.microsoft.com/fwlink/?LinkId=246934
