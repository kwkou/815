<properties
    pageTitle="Azure DocumentDB 的常见用例和 NoSQL 方案 | Azure"
    description="了解 DocumentDB 最常见的五个用例：用户生成的内容、事件日志记录、目录数据、用户首选项数据和物联网 (IoT)。"
    services="documentdb"
    author="h0n"
    manager="jhubbard"
    editor=""
    documentationcenter=""
    translationtype="Human Translation" />
<tags
    ms.assetid="eca68a58-1a8c-4851-8cf8-6e4d2b889905"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/23/2017"
    wacn.date="05/08/2017"
    ms.author="hawong"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="d8877316521e0210c3d88c86e69b9a10b53dd743"
    ms.lasthandoff="04/28/2017" />

# <a name="common-documentdb-use-cases"></a>常见 DocumentDB 用例
本文概述了 DocumentDB 的几个常见用例。  本文中的建议可以作为使用 DocumentDB 开发应用程序的一个起点。   

阅读本文后，将能够回答以下问题： 

- DocumentDB 的常见用例有哪些？
- 对零售移动应用程序使用 DocumentDB 有什么好处？
- 将 DocumentDB 用作物联网 (IoT) 系统的数据存储空间有什么好处？
- 对 Web 和移动应用程序使用 DocumentDB 有什么好处？

## <a name="common-use-cases-for-documentdb"></a>DocumentDB 的常见用例
Azure DocumentDB 是一种通用型 NoSQL 数据库，广泛用于应用程序和用例。 它对于需要低延迟毫秒级响应时间或需要快速伸缩的任何应用程序都是一个不错的选择。 下面介绍了 DocumentDB 的几个属性，这些属性使其非常适用于高性能应用程序。

- DocumentDB 采用本机方式对数据进行分区，以实现高可用性和可伸缩性。
- DocumentDB 采用由 SSD 提供支持的存储，具有低延迟毫秒级响应时间。
- DocumentDB 支持最终、会话和受限停滞等一致性级别，从而实现较低的价格/性能比。 
- DocumentDB 提供了灵活的数据友好型定价模式，独立测量存储和吞吐量。
- DocumentDB 保留的吞吐量模型使你可以考虑读取/写入数量而不考虑基础硬件的 CPU/内存/IOP。
- DocumentDB 的设计允许扩展到每天约数十亿个请求的大规模请求量。

这些特性对于需要低响应时间和需要处理大量读取和写入操作的 Web、移动、游戏和 IoT 应用程序尤为有益。 

## <a name="iot-and-telematics"></a>IoT 和远程信息处理
IoT 用例在引入、处理和存储数据方法方面通常具有相同的模式。  首先，这些系统需要引入各处设备传感器中的激增数据。 其次，这些系统可以处理和分析流式传输数据，以获得实时见解。 然后将数据存档到闲置存储进行批量分析。 Azure 提供可应用于 IoT 用例的多种服务：Azure 事件中心、Azure DocumentDB、Azure 流分析、Azure 通知中心、Azure 机器学习、Azure HDInsight 和 PowerBI。 

![Azure DocumentDB IoT 参考体系结构](./media/documentdb-use-cases/documentdb-iot.png)

由于 Azure 事件中心可以低延迟引入高吞吐量数据，因此它可以引入激增数据。 可以将需要进行实时分析的引入数据传输至 Azure 流分析，以进行实时分析。 可以将数据加载到 DocumentDB 中进行即席查询。 将数据加载到 DocumentDB 后，即可对其进行查询。  DocumentDB 中的数据可以用作实时分析中的参考数据。 此外，通过将 DocumentDB 数据连接到 HDInsight 进行 Pig、Hive 或 Map/Reduce 作业，可以进一步优化和处理数据。  经过优化的数据随后被加载回 DocumentDB 用于报告。   

有关使用 DocumentDB、EventHubs 和 Storm 的 IoT 解决方案示例，请参阅 [GitHub 上的 hdinsight-storm-examples 存储库](https://github.com/hdinsight/hdinsight-storm-examples/)。

有关 Azure IoT 产品/服务的详细信息，请参阅 [创建物联网](http://www.microsoft.com/en-us/server-cloud/internet-of-things.aspx)。 

## <a name="retail-and-marketing"></a>零售和营销
零售业中常使用 DocumentDB 存储目录数据。 目录数据使用方案涉及存储和查询人员、地点、产品等实体的一组属性。  目录数据的部分示例包括用户帐户、产品目录、IoT 的设备注册表和物料系统清单。  此数据的属性可能会随时间的推移而发生变化和更改以满足应用程序要求。  

以汽车部件供应商的产品目录为例。 除所有部件共有的常见属性外，每个部件可能都具有其自己的属性。  而且，某个特定部件的属性还可能会在发布新型号之后的第二年发生变化。  作为 JSON 文档存储，DocumentDB 支持灵活的架构和分层数据，因此它非常适用于存储产品目录数据。

![Azure DocumentDB 零售目录参考体系结构](./media/documentdb-use-cases/documentdb-retail-catalog.png)

 此外，存储在 DocumentDB 中的数据可以与 HDInsight 集成，通过 Pig、Hive 或 Map/Reduce 作业进行大数据分析。 有关 DocumentDB 的 Hadoop 连接器的详细信息，请参阅使用 [DocumentDB 和 HDInsight 运行 Hadoop 作业](/documentation/articles/documentdb-run-hadoop-with-hdinsight/)。

 ![Azure DocumentDB 零售订单参考体系结构](./media/documentdb-use-cases/documentdb-retail-orders.png)
## <a name="gaming"></a>游戏
数据库层是游戏应用程序的关键组件。 新式游戏可在移动/控制台客户端执行图形处理，但需依赖云传输自定义的个性化内容，例如游戏中的统计数据、社交媒体集成以及高分排行榜。 游戏通常需要单毫秒的读取和写入延迟，从而带来具有吸引力的游戏体验。 新游戏启动和功能更新期间，游戏数据库需要具备很快的速度，并且要能够处理请求速率中的大量峰值。

DocumentDB 用于 [Next Games](http://www.nextgames.com/) 推出的 [The Walking Dead: No Man's Land（行尸走肉：无人地带）](https://azure.microsoft.com//blog/the-walking-dead-no-mans-land-game-soars-to-1-with-azure-documentdb/)和 [Halo 5: Guardians（光环 5：守护者）](https://azure.microsoft.com/zh-cn/blog/how-halo-5-guardians-implemented-social-gameplay-using-azure-documentdb/)等游戏。 针对游戏开发人员，DocumentDB 具备以下优势：

- DocumentDB 允许弹性提高或降低性能。 此优势使游戏可以通过执行单个 API 调用同时处理从几十到数百万不等的游戏配置文件和统计数据的更新。
- DocumentDB 支持毫秒读取和写入，从而有助于避免在玩游戏期间出现任何延迟。
- DocumentDB 的自动索引可对多个不同属性进行实时筛选，例如通过内部玩家 ID 或其 GameCenter、Facebook、Google ID 或基于玩家公会成员身份的查询对玩家进行定位。 如果不构建复杂的索引或分区基础结构，就不可能实现这种操作。
- 采用灵活的架构，就更容易实现游戏内聊天消息、玩家公会成员身份、已完成的挑战、高分排行榜以及社交图谱等社交功能。
- DocumentDB 作为托管的平台即服务 (PaaS) 只需最少的设置和管理工作即可实现快速迭代和缩短上市时间。

![Azure DocumentDB 游戏参考体系结构](./media/documentdb-use-cases/documentdb-gaming-architecture.png)

## <a name="web-and-mobile-applications"></a>Web 和移动应用程序
DocumentDB 通常用于 Web 和移动应用程序内部，尤其适用于社交互动建模、与第三方服务集成和积累丰富的个性化体验。 可以使用 DocumentDB SDK 构建使用常用 [Xamarin 框架](/documentation/articles/documentdb-mobile-apps-with-xamarin/)的丰富 iOS 和 Android 应用程序。  

### <a name="social-applications"></a>社交应用程序
DocumentDB 的一个常见用例就是存储和查询 Web 和移动应用程序的用户生成内容 (UGC)，尤其是社交媒体应用程序。 UGC 的部分示例包括聊天会话、推文、博客文章、评级和评论。 通常情况下，社交媒体应用程序中的 UGC 混合了不受刚性结构约束的自由格式文本、属性、标记和关系。 可将聊天、评论和帖子等内容存储在 DocumentDB 中，而无需转换或复杂对象关系映射层。  可以轻易添加或修改数据属性以满足开发人员遍历应用程序代码的要求，从而促进快速开发。  

与第三方社交网络集成的应用程序必须响应这些网络中不断更改的架构。 由于 DocumentDB 中的数据默认自动编制索引，因此可以随时查询数据。 因此，这些应用程序可以根据其各自的需求灵活地检索投影。

许多社交应用程序在全球范围内运行并表现出不可预测的使用模式。 因此，应用程序层能根据用量需求缩放时，缩放数据存储的灵活性至关重要。  可通过在 DocumentDB 帐户之下添加额外的数据分区来进行扩大。  也可在多个区域中创建额外的 DocumentDB 帐户。 有关 DocumentDB 服务区域可使用性的信息，请参阅 [Azure 区域](https://azure.microsoft.com/regions/#services)。

![Azure DocumentDB Web 应用参考体系结构](./media/documentdb-use-cases/documentdb-web.png)

### <a name="personalization"></a>个性化
如今，新式应用程序都具有复杂的视图和体验。 它们通常采用动态设计，迎合对用户首选项或情绪以及品牌塑造的需求。 因此，应用程序需要能够有效地检索个性化设置，以便快速呈现 UI 元素和体验。 

JSON 是一种用于呈现 UI 布局数据的有效格式，它不仅轻量而且可由 JavaScript 轻松理解。 DocumentDB 提供可调的一致性级别，可以实现快速读取和低延迟写入。 因此，在 DocumentDB 中将包括个性化设置的 UI 布局数据存储为 JSON 文档是获取网络数据的一种有效方法。

## <a name="next-steps"></a>后续步骤
若要开始使用 DocumentDB，可创建一个[帐户](/pricing/1rmb-trial/)，然后了解 DocumentDB 并查找所需信息。 

或者，如果想要了解有关使用 DocumentDB 的客户的详细信息，可以参阅下面的客户案例：

- [Sulekha.com](https://customers.microsoft.com/en-US/story/sulekha-uses-azure-documentdb-to-connect-customers-and-businesses-across-india)。 Sulekha 使用 Azure DocumentDB 连接整个印度的客户和企业。
- [NewOrbit](https://customers.microsoft.com/en-US/story/neworbit-takes-flight-with-azure-documentdb)。 NewOrbit 使用了 Azure DocumentDB。
- [Affinio](https://customers.microsoft.com/en-US/doclink/affinio-switches-from-aws-to-azure-documentdb-to-harness-social-data-at-scale)。 Affinio 从 AWS 改用 Azure DocumentDB 以大规模处理社交数据。
- [Next Games](https://azure.microsoft.com//blog/the-walking-dead-no-mans-land-game-soars-to-1-with-azure-documentdb/)。 The Walking Dead: No Man’s Land（行尸走肉：无人之地）游戏在 DocumentDB 的支持下飙升至排行榜第 1 名。
- [Halo（光环）](https://azure.microsoft.com/zh-cn/blog/how-halo-5-guardians-implemented-social-gameplay-using-azure-documentdb/)。 Halo 5（光环 5）使用 Azure DocumentDB 实现社交游戏玩法。
- [Cortana Analytics Gallery](https://azure.microsoft.com/zh-cn/blog/cortana-analytics-gallery-a-scalable-community-site-built-on-azure-documentdb/)。 Cortana Analytics Gallery - 构建在 Azure DocumentDB 基础之上的可扩展社区网站。
- [Breeze](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18602)。 领先集成商使用灵活的云技术在几分钟内为跨国公司提供全球见解。
- [News Republic](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18639)。 让新闻更智能，以便为订阅的市民提供有目的性的信息。 
- [SGS International](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18653)。 为了在全球范围内保持颜色一致，大品牌求救于 SGS。 而 SGS 采用 Azure。
- [Telenor](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18608)。 全球领先企业 Telenor 使用云以初创企业的速度发展。 
- [XOMNI](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18667)。 未来的存储依赖于数据的快速搜索和轻松流动。
- [Nucleo](https://customers.microsoft.com/story/azure-based-software-platform-breaks-down-barriers-bet)。 基于 Azure 的软件平台打破企业和客户之间的阻碍
- [Weka](https://customers.microsoft.com/story/weka-smart-fridge-improves-vaccine-management-so-more-people-can-be-protected-against-diseases)。 Weka 智能冰箱可改善疫苗管理，保护更多人免受疾病侵扰
- [Orange Tribes](https://customers.microsoft.com/en-US/story/theres-more-to-that-food-app-than-meets-the-eye-or-the-mouth)。 使用这款食品应用，得到的不仅仅是视觉或味觉上的满足。
- [Real Madrid](https://customers.microsoft.com/en-US/story/real-madrid-brings-the-stadium-closer-to-450-million-f)。 皇家马德里足球俱乐部借助 Microsoft 云拉近体育场与全球 4.5 亿粉丝之间的距离。
- [Tuku](https://customers.microsoft.com/en-US/story/tuku-makes-car-buying-fun-with-help-from-azure-services)。 TUKU 借助 Azure 服务增加购买汽车的乐趣

<!---Update_Description: wording update -->