<properties
	pageTitle="DocumentDB 的一致性级别 | Azure"
	description="DocumentDB 提供四种一致性级别来帮助你在最终一致性、可用性和延迟之间做出取舍。"
	keywords="最终一致性, documentdb, azure, Microsoft azure"
	services="documentdb"
	authors="mimig1"
	manager="jhubbard"
	editor="cgronlun"
	documentationCenter=""/>

<tags
	ms.service="documentdb"
	ms.date="06/15/2016"
	wacn.date="06/30/2016"/>
# DocumentDB 中的一致性级别

DocumentDB 是从无到有开发出来的，其设计考虑到了全局分布。它旨在提供可预测的低延迟保证、99.99%的可用性 SLA，以及多个完善定义的宽松一致性模型。目前，DocumentDB 提供四种一致性级别：非常一致性、受限停滞一致性、会话一致性和最终级别。除了其他 NoSQL 数据库提供的**非常一致性**和**最终一致性**模型以外，DocumentDB 还提供两个谨慎代码化和操作化的一致性模型 – **受限停滞一致性**和**会话一致性**，并已根据真实用例验证其有效性。总而言之，这四个一致性级别可让你在一致性、可用性和延迟之间做出合理的取舍。

## 一致性的范围

一致性的粒度归并为单个用户请求。写入请求可以对应于插入、替换、更新插入或删除事务（执行或不执行关联的前置或后置触发器）。或者，写入请求可以对应于针对某个分区中的多个文档运行的 JavaScript 存储过程的事务执行。与写入操作一样，读取/查询事务也归并为单个用户请求。用户可能需要在跨越多个分区的大型结果集中分页，但每个读取事务归并为单个页面，并从单个分区内部进行。

## 一致性级别

你可以配置数据库帐户的默认一致性级别，以应用于数据库帐户下的所有集合（跨所有数据库）。所有对用户定义的资源发出的读取和查询，默认都会使用数据库帐户上指定的默认一致性级别。不过，你可以指定 [[x-ms-consistency-level]](https://msdn.microsoft.com/library/azure/mt632096.aspx) 请求标头，来放宽特定读取/查询请求的一致性级别。如下所述，DocumentDB 复制协议支持四种类型的一致性级别，这些级别可在特定的一致性保证与性能之间提供明确的折衷。

![DocumentDB 提供多个妥善定义的（宽松）一致性模型供你选择][1]

**非常一致性**：

- 非常一致性提供[可线性化](https://aphyr.com/posts/313-strong-consistency-models)保证，即保证读取操作返回文档的最新版本。 
- 非常一致性保证写入只有在副本的多数仲裁一直提交的情况下才可见。写入要么由主要副本和辅助副本仲裁一直同步提交，要么被中止。读取始终由多数读取仲裁确认；客户端绝不会看到未提交或不完整的写入，而且始终保证读取最新确认的写入。 
- 配置为使用非常一致性的 DocumentDB 帐户不能将多个 Azure 区域与其 DocumentDB 帐户相关联。 
- 具有非常一致性的读取操作的开销（从消耗的[请求单位数](/documentation/articles/documentdb-request-units/)来讲）高于会话一致性和最终一致性，但与受限停滞一致性相同。
 

**有限过期一致性**：

- 有限过期一致性保证读取操作滞后于写入操作最多 K 个文档版本或前缀或 t 个时间间隔。 
- 因此，如果选择有限过期，则可以通过两种方式配置“过期”： 
    - 读取操作比写入操作滞后的文档版本 K 数
    - 时间间隔 t 
- 有限过期提供全局整体顺序，但在“停滞窗口”中除外。请注意，“停滞窗口”内部和外部的区域中提供单调读取保证。 
- 与会话或最终一致性相比，受限停滞一致性提供更强的一致性保证。对于全球分布式应用程序，如果你想要获得非常一致性，同时希望获得 99.99% 的可用性和低延迟，则我们建议你使用受限停滞。 
- 配置了有限过期一致性的 DocumentDB 帐户可将任意数量的 Azure 区域与其 DocumentDB 帐户相关联。 
- 具有非常一致性的读取操作的开销（从消耗的 RU 来讲）高于会话一致性和最终一致性，但与非常一致性相同。

**会话一致性**：

- 与强一致性和受限停滞一致性级别所提供的全局一致性模型不同，会话一致性归并为客户端会话。 
- 对于涉及到设备或用户会话的所有方案，会话一致性非常合适，因为它提供单调读取、单调写入和读取自己的写入 (RYW) 保证。 
- 会话一致性为会话提供可预测的一致性和最大的读取吞吐量，同时提供最低延迟的写入和读取。 
- 配置了会话一致性的 DocumentDB 帐户可将任意数量的 Azure 区域与其 DocumentDB 帐户相关联。 
- 具有会话一致性级别的读取操作的开销（从消耗的 RU 来讲）小于非常一致性和受限停滞一致性，但大于最终一致性
 

**最终一致性**：

- 最终一致性保证在没有再收到任何写入的情况下，组中的副本最终只剩一个。 
- 最终一致性是最弱的一致性形式，客户端可能会获取比之前看到的值还要旧的值。
- 最终一致性提供最差的读取一致性，但是读取和写入的延迟均为最低。
- 配置了最终一致性的 DocumentDB 帐户可将任意数量的 Azure 区域与其 DocumentDB 帐户相关联。 
- 具有最终一致性级别的读取操作的开销（从消耗的 RU 来讲）在所有 DocumentDB 一致性级别中是最低的。


## 一致性保证

下表列出了对应于四种一致性级别的不同一致性保证。

| 保证 | 强 | 有限过期性 | 会话 | 最终 |
|----------------------------------------------------------|-------------------------------------------------|------------------------------------------------------------------------------------------------|--------------------------------------------------|--------------------------------------------------|
| **全局整体顺序** | 是 | 是，在“停滞窗口”外部 | 否，部分“会话”顺序 | 否 |
| **一致前缀保证** | 是 | 是 | 是 | 是 |
| **单调读取** | 是 | 是，在停滞窗口外部的区域之间以及区域内部，始终都可保证 | 是，针对给定的会话 | 否 |
| **单调写入** | 是 | 是 | 是 | 是 |
| **读取自己的写入** | 是 | 是 | 是（在写入区域中） | 否 |


## 配置默认的一致性级别

1.  在 [Azure 门户预览](https://portal.azure.cn/)的跳转栏中，单击“DocumentDB 帐户”。

2. 在“DocumentDB 帐户”边栏选项卡中，选择要修改的数据库帐户。

3. 在“帐户”边栏选项卡中，如果“所有设置”边栏选项卡尚未打开，请单击最上方命令栏中的“设置”图标。

4. 在“所有设置”边栏选项卡中，单击“功能”下的“默认一致性”条目。

	![屏幕截图：突出显示“设置”图标和默认一致性条目](./media/documentdb-consistency-levels/database-consistency-level-1.png)

5. 在“默认一致性”边栏选项卡中，选择新的一致性级别，然后单击“确定”。

	![突出显示一致性级别和“确定”按钮的屏幕截图](./media/documentdb-consistency-levels/database-consistency-level-2.png)

## 查询的一致性级别

默认情况下，对于用户定义的资源，查询的一致性级别与读取的一致性级别相同。默认情况下，每次在集合中插入、替换或删除文档时同步更新索引。这个行为让查询能够使用与文档读取相同的一致性级别。虽然 DocumentDB 针对写入进行了优化，且支持文档写入，以及同步索引维护和提供一致的查询服务，但你也可以配置某些集合，使其索引延迟更新。延迟索引编制可大大提高写入性能，非常适合工作负荷主要具有大量读取操作的批量引入方案。

索引模式|	读取|	查询  
-------------|-------|---------
一致（默认值）|	有强、有限过期、会话或最终可供选择|	有强、有限过期、会话或最终可供选择|
延迟|	有强、有限过期、会话或最终可供选择|	最终  

与读取请求一样，你可以指定 [x-ms-consistency-level](https://msdn.microsoft.com/library/azure/mt632096.aspx) 请求标头，来降低特定查询请求的一致性级别。

## 后续步骤

如果你想详细了解一致性级别和平衡方案，建议参阅下列资源：

-	Doug Terry。Replicated Data Consistency explained through baseball（借助棒球解释复制数据一致性）（视频）。   
[https://www.youtube.com/watch?v=gluIh8zd26I](https://www.youtube.com/watch?v=gluIh8zd26I)
-	Doug Terry。Replicated Data Consistency explained through baseball（借助棒球解释重复数据一致性）。   
[http://research.microsoft.com/pubs/157411/ConsistencyAndBaseballReport.pdf](http://research.microsoft.com/pubs/157411/ConsistencyAndBaseballReport.pdf)
-	Doug Terry。Session Guarantees for Weakly Consistent Replicated Data（弱一致性重复数据的会话保证）。   
[http://dl.acm.org/citation.cfm?id=383631](http://dl.acm.org/citation.cfm?id=383631)
-	Daniel Abadi。Consistency Tradeoffs in Modern Distributed Database Systems Design: CAP is only part of the story”（现代分布式数据库系统设计中的一致性平衡方案：CAP 只是冰山一角）。   
[http://computer.org/csdl/mags/co/2012/02/mco2012020037-abs.html](http://computer.org/csdl/mags/co/2012/02/mco2012020037-abs.html)
-	Peter Bailis、Shivaram Venkataraman、Michael J. Franklin、Joseph M. Hellerstein、Ion Stoica。Probabilistic Bounded Staleness (PBS) for Practical Partial Quorums（实用部分仲裁的概率性有限过期性 (PBS)）   
[http://vldb.org/pvldb/vol5/p776\_peterbailis\_vldb2012.pdf](http://vldb.org/pvldb/vol5/p776_peterbailis_vldb2012.pdf)
-	Werner Vogels。Eventual Consistent - Revisited（最终一致 - 重新访问）。    
[http://allthingsdistributed.com/2008/12/eventually\_consistent.html](http://allthingsdistributed.com/2008/12/eventually_consistent.html)


[1]: ./media/documentdb-consistency-levels/consistency-tradeoffs.png
<!---HONumber=Mooncake_0627_2016-->