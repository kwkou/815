<properties linkid="manage-services-biztalk-services-editions-chart" urlDisplayName="Editions chart" pageTitle="Learn about features in BizTalk Services editions | Azure" metaKeywords="BizTalk Services, get started, Azure, editions" description="Compare the capabilities of the BizTalk Services editions: Free, Developer, Basic, Standard, and Premium." metaCanonical="" services="biztalk-services" documentationCenter="" title=" Basic" authors="mandia" solutions="" manager="paulettm" editor="cgronlun" />

# BizTalk 服务：版本图表

Azure BizTalk 服务提供以下版本：“免费”、“开发人员”、“基本”、“标准”和“高级”：

**免费（预览版）**：提供用于创建和管理混合连接的功能。使用混合连接可以轻松地将 Azure 网站连接到 SQL Server 等本地系统。

**开发人员版**：提供混合连接、EAI 和 EDI 消息处理功能，其中包括易于使用的贸易合作伙伴管理门户，并支持常见的 EDI 架构以及通过 X12 和 AS2 处理丰富的 EDI 内容。可创建通过任何 HTTP/S、REST、FTP、WCF 和 SFTP 协议连接云中服务以读写消息的常见 EAI 方案。利用与本地 LOB 系统的连接（通过随时可用的 SAP、Oracle eBusiness、Oracle DB、Siebel 和 SQL Server 适配器）。全都处于具有 Visual Studio 工具的以开发人员为中心的环境中以方便开发和部署。仅限开发和测试用途，不提供服务级别协议 (SLA)。

**基本版**：包括开发人员版的大多数功能，还增加了混合连接、EAI 桥、EDI 协议和 BizTalk 适配器包连接。另外，还提供高可用性，以及用于缩放服务级别协议 (SLA) 的选项。

**标准版**：包括基本版的所有功能，还增加了混合连接、EAI 桥、EDI 协议和 BizTalk 适配器包连接。另外，还提供高可用性，以及用于缩放服务级别协议 (SLA) 的选项。

**高级版**：包括标准版的所有功能，还增加了混合连接、EAI 桥、EDI 协议和 BizTalk 适配器包连接。另外，还提供存档和高可用性，以及用于缩放服务级别协议 (SLA) 的选项。

下表列出了差异：

<table>
<colgroup>
<col width="16%" />
<col width="16%" />
<col width="16%" />
<col width="16%" />
<col width="16%" />
<col width="16%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"></th>
<th align="left">免费（预览版）</th>
<th align="left">开发人员</th>
<th align="left">基本</th>
<th align="left">标准</th>
<th align="left">高级</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><strong>起价</strong></td>
<td align="left">请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服务定价</a>。</td>
<td align="left">请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服务定价</a>。</td>
<td align="left">请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服务定价</a>。</td>
<td align="left">请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服务定价</a>。</td>
<td align="left">请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服务定价</a>。</td>
</tr>
<tr class="even">
<td align="left"><strong>默认最低配置</strong></td>
<td align="left">1 个免费版单位</td>
<td align="left">1 个开发人员版单位</td>
<td align="left">1 个基本版单位</td>
<td align="left">1 个标准版单位</td>
<td align="left">1 个高级版单位</td>
</tr>
<tr class="odd">
<td align="left"><strong>缩放</strong></td>
<td align="left">无扩展</td>
<td align="left">无扩展</td>
<td align="left">可扩展，增量为 1 个基本版单位</td>
<td align="left">可扩展，增量为 1 个标准版单位</td>
<td align="left">可扩展，增量为 1 个高级版单位</td>
</tr>
<tr class="even">
<td align="left"><strong>允许的最大横向扩展</strong></td>
<td align="left">无扩展</td>
<td align="left">无扩展</td>
<td align="left">最多 8 个单位</td>
<td align="left">最多 8 个单位</td>
<td align="left">最多 8 个单位</td>
</tr>
<tr class="odd">
<td align="left"><strong>每个单位的 EAI 桥数</strong></td>
<td align="left">不包括</td>
<td align="left">25</td>
<td align="left">25</td>
<td align="left">125</td>
<td align="left">500</td>
</tr>
<tr class="even">
<td align="left"><strong>EDI，AS2</strong><br /><br /> 包括 TPM 协议</td>
<td align="left">不包括</td>
<td align="left">包括。每个单位 10 个协议。</td>
<td align="left">包括。每个单位 50 个协议。</td>
<td align="left">包括。每个单位 250 个协议。</td>
<td align="left">包括。每个单位 1000 个协议。</td>
</tr>
<tr class="odd">
<td align="left"><strong>每个单位的混合连接数</strong></td>
<td align="left">5</td>
<td align="left">5</td>
<td align="left">10</td>
<td align="left">50</td>
<td align="left">100</td>
</tr>
<tr class="even">
<td align="left"><strong>每个单位的混合连接数据传输 (GB)</strong></td>
<td align="left">5</td>
<td align="left">5</td>
<td align="left">50</td>
<td align="left">250</td>
<td align="left">500</td>
</tr>
<tr class="odd">
<td align="left"><strong>BizTalk 适配器服务与内部部署的 LOB 系统的连接数</strong></td>
<td align="left">不包括</td>
<td align="left">1 个连接</td>
<td align="left">2 个连接</td>
<td align="left">5 个连接</td>
<td align="left">25 个连接</td>
</tr>
<tr class="even">
<td align="left"><strong>支持的协议/系统：</strong>
<ul>
<li>HTTP</li>
<li>HTTPS</li>
<li>FTP</li>
<li>SFTP</li>
<li>WCF</li>
<li>Service Bus (SB)</li>
<li>Azure Blob</li>
<li>REST API</li>
</ul></td>
<td align="left">不包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
</tr>
<tr class="odd">
<td align="left"><strong>高可用性</strong><br /><br /> 有关服务级别协议 (SLA)，请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服务定价</a>。</td>
<td align="left">不包括</td>
<td align="left">不包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
</tr>
<tr class="even">
<td align="left"><strong>备份和还原</strong></td>
<td align="left">不包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
</tr>
<tr class="odd">
<td align="left"><strong>跟踪</strong></td>
<td align="left">不包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
</tr>
<tr class="even">
<td align="left"><strong>存档</strong><br /><br /> 包括接收不可否认 (NRR) 和下载跟踪的消息</td>
<td align="left">不包括</td>
<td align="left">包括</td>
<td align="left">不包括</td>
<td align="left">不包括</td>
<td align="left">包括</td>
</tr>
<tr class="odd">
<td align="left"><strong>使用自定义代码</strong></td>
<td align="left">不包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
</tr>
<tr class="even">
<td align="left"><strong>使用转换，包括自定义 XSLT</strong></td>
<td align="left">不包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
<td align="left">包括</td>
</tr>
</tbody>
</table>

**注意**
为了针对硬件故障进行恢复，高可用性意味着一个 BizTalk 单位中包含多个 VM。

## 常见问题

#### BizTalk 单位是什么？

“单位”是 Azure BizTalk 服务部署的原子级别。每个版本均附带一个具有不同的计算能力和内存的单元。例如，基本单位具有比开发人员版更多的计算，而标准版具有比基本版更多的计算，依此类推。当你缩放 BizTalk 服务时，将按“单位”进行缩放。

#### BizTalk 服务和 Azure BizTalk VM 之间的差别是什么？

BizTalk 服务提供一个真正的“平台即服务”(PaaS) 体系结构，用于在云中生成集成解决方案。使用 PaaS 模型，你可以将工作重心完全放在应用程序逻辑上，而将所有基础结构管理工作都留给 Microsoft 执行，包括：

-   无需管理或修补虚拟机
-   Microsoft 确保可用性
-   只需通过 Azure 管理门户请求更多或更少容量，按需控制缩放。

Azure 虚拟机上的 BizTalk Server 提供基础结构即服务 (IaaS) 体系结构。你可以完全像本地环境一样创建虚拟机和配置它们，从而更轻松地在云中运行现有应用程序而不必进行代码更改。使用 IaaS 时，仍由你负责配置和管理虚拟机（例如，安装软件和操作系统修补程序）以及相应地设计应用程序以实现高可用性。

如果你考虑生成新的集成解决方案以将基础结构管理工作减至最少，那么请使用 BizTalk 服务。如果你希望快速迁移现有 BizTalk 解决方案或寻找按需环境以开发和测试 BizTalk Server 应用程序，那么请在 Azure 虚拟机中使用 BizTalk Server。

#### BizTalk 适配器服务和混合连接之间的差别是什么？

BizTalk 适配器服务由 Azure BizTalk 服务使用。BizTalk 适配器服务使用 BizTalk 适配器包连接到本地业务线 (LOB) 系统。使用混合连接可以轻松方便地将 Azure 应用程序（例如网站和移动服务）连接到本地资源。

#### 在 BizTalk 服务中创建协议时，为什么桥数增加 2 而非 1？

每个协议都由两个不同的桥组成，一个发送端通信桥和一个接收端通信桥。

#### 达到桥数或协议数的配额限制时将出现什么情况？

你将无法部署任何新桥或创建任何新协议。为了部署更多桥，你需要纵向扩展更多个 BizTalk 服务单位，或升级到更高版本。

#### 如何从 BizTalk 服务的一个等级迁移到另一个等级？

使用备份和还原流程从一个等级迁移到另一个等级。仅支持某些迁移途径。请参考 [BizTalk 服务：备份和还原][BizTalk 服务：备份和还原]，了解有关支持的迁移途径的更多详细信息。

#### BizTalk 适配器服务是否包含在 BizTalk 服务内？我如何接收该软件？

包含在内，BizTalk 适配器服务以及 BizTalk 适配器包已包含在 Azure BizTalk 服务 SDK [下载][下载]中。

## 下一步

若要在 Azure 管理门户中设置 Azure BizTalk 服务，请转到 [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]。若要开始创建应用程序，请转到 [Azure BizTalk 服务][Azure BizTalk 服务]。

## 另请参阅

-   [BizTalk 服务：使用 Azure 管理门户进行设置][BizTalk 服务：使用 Azure 管理门户进行设置]
-   [BizTalk 服务：设置状态图表][BizTalk 服务：设置状态图表]
-   [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡][BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]
-   [BizTalk 服务：备份和还原][BizTalk 服务：备份和还原]
-   [BizTalk 服务：限制][BizTalk 服务：限制]
-   [BizTalk 服务：颁发者名称和颁发者密钥][BizTalk 服务：颁发者名称和颁发者密钥]
-   [如何开始使用 Azure BizTalk 服务 SDK][如何开始使用 Azure BizTalk 服务 SDK]

  [Azure BizTalk 服务定价]: http://go.microsoft.com/fwlink/p/?LinkID=304011
  [BizTalk 服务：备份和还原]: http://go.microsoft.com/fwlink/p/?LinkID=329873
  [下载]: http://www.microsoft.com/download/details.aspx?id=39087
  [BizTalk 服务：使用 Azure 管理门户进行设置]: http://go.microsoft.com/fwlink/p/?LinkID=302280
  [Azure BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=235197
  [BizTalk 服务：设置状态图表]: http://go.microsoft.com/fwlink/p/?LinkID=329870
  [BizTalk 服务：“仪表板”、“监视”和“缩放”选项卡]: http://go.microsoft.com/fwlink/p/?LinkID=302281
  [BizTalk 服务：限制]: http://go.microsoft.com/fwlink/p/?LinkID=302282
  [BizTalk 服务：颁发者名称和颁发者密钥]: http://go.microsoft.com/fwlink/p/?LinkID=303941
  [如何开始使用 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=302335
