<properties
	pageTitle="分析平台：Apache Storm 与流分析之间的比较 | Azure"
	description="使用 Apache Storm 与流分析之间的比较获取有关选择云分析平台的指导。了解功能和区别。"
	keywords="分析平台, 分析平台, 云分析平台, storm 比较"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="stream-analytics"
	ms.date="05/03/2016"
	wacn.date="06/20/2016"/>


# 帮助你选择流分析平台：Apache Storm 与 Azure 流分析的比较

使用 Apache Storm 与 Azure 流分析之间的这种比较获取有关选择云分析平台的指导。了解流分析与 Apache Storm（Azure HDInsight 上的一种托管服务）的价值主张，以便为你的业务用例选择适当的解决方案。

这两种分析平台都具有 PaaS 解决方案的优势，但几项主要功能却存在各种差异。下面列出了这些服务的功能和限制，以帮助你选择所需的解决方案来实现目标。

## Storm 与流分析的比较：常规功能 ##
<table border="1" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    <strong>Azure 流分析</strong>
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    <strong>Apache Storm on HDInsight</strong>
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>开放源</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    否，Azure 流分析是一种 Microsoft 专有产品/服务。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    是，Apache Storm 是一种由 Apache 授权的技术。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>由 Microsoft 提供支持</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    是
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    是
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>硬件要求</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    无硬件要求。Azure 流分析是一种 Azure 服务。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    无硬件要求。Apache Storm 是一种 Azure 服务。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>顶层单位</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    使用 Azure 流分析，客户可以部署和监视流式处理作业。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    在 HDInsight 上使用 Apache Storm，客户可以部署和监视整个群集，而群集可以托管多个 Storm 作业和其他工作负荷（包括批处理）。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>价格</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    流分析通过处理的数据量以及流式处理单位的数目（作业每小时运行的）来定价。
                </p>
                <p>
                    <a href="/home/features/stream-analytics/pricing/">更多定价信息可在此处找到。</a>
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    就 Apache Storm on HDInsight 来说，采购单位是根据群集计算的，而收费则根据群集所运行的时间来计算，与部署的作业无关。
                </p>
                <p>
                    <a href="/home/features/hdinsight/pricing/">更多定价信息可在此处找到。</a>
                </p>
            </td>
        </tr>
    </tbody>
</table>
## 在每个分析平台上创作 ##
<table border="1" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    <strong>Azure 流分析</strong>
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    <strong>Apache Storm on HDInsight</strong>
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>功能：SQL DSL</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    是，提供易用的 SQL 语言支持。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    否，用户必须使用 Java C# 编写代码或使用 Trident API。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>功能：临时运算符</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    默认支持开窗聚合和临时联接。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    临时运算符必须由用户来实现。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>开发体验</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    通过 Azure 门户获得对示例数据的交互式创作和调试体验。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    就 .NET 来说，用户可通过 Visual Studio 获得开发、调试和监视体验；就 Java 和其他语言来说，开发人员可以使用自己选择的 IDE。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>调试支持</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    流分析提供基本的作业状态和操作日志来帮助进行调试，但目前用户无法灵活地选择包括在日志中的内容以及内容的多少（即详细模式）。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    提供详细日志是为了方便进行调试。可以通过两种方式向用户提供日志，一种是通过 Visual Studio，另一种是由用户通过 RDP 进入群集来访问日志。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>支持 UDF（用户定义函数）</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    目前不支持 UDF。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    UDF 可以使用 C#、Java 或用户选择的语言进行编写。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>可扩展 - 自定义代码</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    没有在流分析中提供可扩展代码支持。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    是，可以在 Storm 上使用 C#、Java 或其他支持的语言来编写自定义代码。
                </p>
            </td>
        </tr>
    </tbody>
</table>
## 数据源和输出 ##
<table border="1" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    <strong>Azure 流分析</strong>
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    <strong>Apache Storm on HDInsight</strong>
                </p>
            </td>
        </tr>
		<tr>
            <td width="174" valign="top">
				<p>
				 <strong>输入数据源</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>支持的输入源为 Azure 事件中心和 Azure Blob。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    提供的连接器适用于事件中心、Service Bus、Kafka 等。不支持的连接器可通过自定义代码来实现。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>输入数据格式</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    支持的输入格式为 Avro、JSON、CSV。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    任何格式均可通过自定义代码来实现。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>Outputs</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    流式处理作业可能有多个输出。支持的输出：Azure 事件中心、Azure Blob 存储、Azure 表、Azure SQL DB 和 PowerBI。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    拓扑中支持多种输出，每种输出都可以有进行下游处理的自定义逻辑。Storm 默认提供的连接器适用于 PowerBI、Azure 事件中心、Azure Blob 存储、Azure DocumentDB、SQL 和 HBase。不支持的连接器可通过自定义代码来实现。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>数据编码格式</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    流分析要求使用 UTF-8 数据格式。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    任何数据编码格式均可通过自定义代码来实现。
                </p>
            </td>
        </tr>
    </tbody>
</table>
## 管理和操作 ##
<table border="1" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    <strong>Azure 流分析</strong>
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    <strong>Apache Storm on HDInsight</strong>
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>作业部署模型</strong>
                </p>
                <p>
                    - <strong>Azure 门户</strong>
                </p>
                <p>
                    - <strong>Visual Studio</strong>
                </p>
                <p>
                    - <strong>PowerShell</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    部署是通过 Azure 门户、PowerShell 和 REST API 实现的。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    部署是通过 Azure 门户、PowerShell、Visual Studio 和 REST API 实现的。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>监视（统计信息、计数器等）</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    监视是通过 Azure 门户和 REST API 实现的。
                </p>
                <p>
                    用户还可以配置 Azure 警报。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    监视是通过 Storm UI 和 REST API 实现的。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>可伸缩性</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    每个作业的流式处理单位数。每个流式处理单位的处理速度高达 1 MB/秒。默认的最大值为 50 个单位。可通过调用来提高限制。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    HDI Storm 群集中的节点数。节点数没有限制（最高限制由 Azure 配额来定义）。可通过调用来提高限制。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>数据处理限制</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    用户可以提高或降低流式处理单位数来提高数据处理速度或优化成本。
                </p>
                <p>
                    提高到 1 GB/秒
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    用户可以根据需要来增大或缩小群集。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>停止/恢复</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    停止，然后从上次停止的位置恢复。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    停止，然后根据水印从上次停止的位置恢复。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>服务和框架更新</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    自动修补，不停机。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    自动修补，不停机。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>通过确保服务高度可用并确保 SLA 来保证业务连续性</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    SLA：99.9% 的运行时间
                </p>
                <p>
                    从故障自动恢复
                </p>
                <p>
                    内置有状态临时运算符恢复功能。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    SLA：99.9% 的 Storm 群集运行时间。Apache Storm 是一种容错性的流式处理平台，不过，客户必须负责确保其流式处理作业的不间断运行。
                </p>
            </td>
        </tr>
    </tbody>
</table>
## 高级功能 ##
<table border="1" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    <strong>Azure 流分析</strong>
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    <strong>Apache Storm on HDInsight</strong>
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>延迟到达和无序事件处理</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    内置可配置的策略，方便重新排序、删除事件或调整事件时间。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    用户必须实现相关逻辑来处理这种情况。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>引用数据</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    Azure Blob（带有最大大小为 100 MB 的内存中查找缓存）提供引用数据。该服务管理引用数据的刷新。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    数据大小没有限制。连接器适用于 HBase、DocumentDB、SQL Server 和 Azure。不支持的连接器可通过自定义代码来实现。
                </p>
                <p>
                    引用数据的刷新必须由自定义代码处理。
                </p>
            </td>
        </tr>
        <tr>
            <td width="174" valign="top">
                <p>
                    <strong>集成机器学习</strong>
                </p>
            </td>
            <td width="204" valign="top">
                <p>
                    可以在 ASA 作业创建过程<a href="http://blogs.msdn.com/b/streamanalytics/archive/2015/05/24/real-time-scoring-of-streaming-data-using-machine-learning-models.aspx">（专用预览）</a>中通过将已发布的 Azure 机器学习模型配置为各种功能来完成。
                </p>
            </td>
            <td width="246" valign="top">
                <p>
                    通过 Storm Bolt 提供。
                </p>
            </td>
        </tr>
    </tbody>
</table>

<!---HONumber=Mooncake_0328_2016-->