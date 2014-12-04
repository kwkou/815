<properties linkid="manage-services-biztalk-services-dashboard-monitor-scale-tabs" urlDisplayName="Dashboard, Monitor and Scale tabs" pageTitle="Dashboard, Monitor, and Scale in BizTalk Services | Azure" metaKeywords="BizTalk Services, Azure, dashboard, monitor, scale" description="Learn about the controls on the Management Portal tabs for BizTalk Services: Dashboard, Monitor, and Scale." metaCanonical="" services="biztalk-services" documentationCenter="" title=" Monitor and Scale tabs" authors="mandia" solutions="" manager="paulettm" editor="cgronlun" />




# BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡

首次打开 Azure 管理门户时，系统会自动为你打开**“所有项目”**选项卡。可以对**“所有项目”**选项卡上的列进行排序。若要查看 BizTalk 服务，请在**“所有项目”**选项卡上选择你的 BizTalk 服务，或者选择**“BIZTALK 服务”**选项卡，然后选择你的 BizTalk 服务名称。

此时会打开一个新窗口，其中包含以下选项卡：本主题介绍这些选项卡。

-   [快速入门][快速入门]

-   [仪表板][仪表板]

-   [监视][监视]

-   [缩放][缩放]

-   [配置][配置]

-   [混合连接][混合连接]

## <a name="QuickStart"></a>快速启动

你不一定能够使用列出的所有选项，这取决于所用的 BizTalk 服务版本。
<table border="1">

    <tr>
        <td><strong>获取工具</strong></td>

        <td>下载 BizTalk 服务 SDK 以在你的本地开发计算机上安装 Visual Studio 项目模板。这些模板创建将部署到你的 BizTalk 服务的 <strong>BizTalk 服务</strong>（桥接）和 <strong>BizTalk Service 项目</strong>（转换）Visual Studio 项目。
        <br/><br/>
		<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=302335">我如何开始使用 Azure BizTalk 服务 SDK </a> 和<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=241589">安装 Azure BizTalk 服务 SDK</a> 列出了要开始的步骤。
        </td>
    </tr>

    <tr>
		<td><strong>创建合作伙伴协议</strong></td>

        <td>打开在添加合作伙伴和创建 X12 和 AS2 EDI 协议的 Azure 上托管的 Azure BizTalk 服务门户。
        <br/><br/>
        <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=303653">在 BizTalk 服务门户上配置适用于 EDI 消息传递的组件</a>列出了要开始的步骤。
        </td>
    </tr>

<tr>
       <td><strong>详细了解 BizTalk 服务</strong>
       <td>转到学习中心以了解有关 Azure BizTalk 服务的更多信息。</td>
</tr>
</table>


在底部的任务栏中，你可以：


<table border="1">

<tr>
<td><strong>管理</strong></td>
<td>打开 Azure BizTalk 服务门户。BizTalk 服务门户是进行 EDI 配置的入口，包括添加合作伙伴和创建 X12 和 AS2 协议。<br /><br /> 这与<strong>“快速启动”</strong>选项卡上的<strong>“创建合作伙伴协议”</strong>相同。<br /><br /> <a href="http://go.microsoft.com/fwlink/p/?LinkID=303653">在 BizTalk 服务门户上配置 EDI 消息传送的组件</a>提供了有关 BizTalk 服务门户的详细信息。</td>
</tr>
<tr class="even">
<td><strong>连接信息</strong></td>
<td>在你选择“连接信息”时，将显示“访问控制命名空间”、“默认颁发者”和“默认密钥”。你可以复制这些值。<br /><br /> 你还可以打开访问控制管理门户。此访问控制管理门户与在左导航窗格中打开使用<strong>“Active Directory”</strong>选项相同。<br /><br /> <a href="http://go.microsoft.com/fwlink/p/?LinkID=285670">管理 ACS 命名空间</a>提供与访问控制管理门户有关的详细信息。</td>
</tr>

<td><strong>同步密钥</strong></td>
<td>当你创建存储帐户时，将自动创建主密钥和辅助密钥。这些密钥控制对存储帐户的访问。BizTalk 服务自动使用主密钥。<strong>“同步密钥”</strong>使用户可以在主密钥与辅助密钥之间切换，而不会中断 BizTalk 服务。<br/><br/> 例如，你希望 BizTalk 服务为存储帐户使用新的主密钥。为此，请按以下步骤操作：<br/><br/>
<ol>
<li>依次选择你的 BizTalk 服务和<strong>“同步密钥”</strong>。选择“辅助密钥”。当你执行此操作时，BizTalk 服务开始使用辅助密钥。</li>
<li>在 Azure 管理门户中，选择“存储帐户”和“重新生成主密钥”。请记住，你的 BizTalk 服务正在使用辅助密钥。</li>
<li>依次选择你的 BizTalk 服务和<strong>“同步密钥”</strong>。现在，选择“主密钥”。这是你重新生成的新主密钥。</li>
<li>在 Azure 管理门户中，选择“存储帐户”和“重新生成辅助密钥”。</li>
</ol>
<br/> 此过程称为“变换密钥”。其目的是让用户可以在主密钥与辅助密钥之间切换，而不会中断 BizTalk 服务。</td>
</tr>

<tr>
<td><strong>删除</strong></td>
<td>当你选择“删除”时，将删除你的 BizTalk 服务和部署到此服务的所有项目。</td>
</tr>
</table>

## <a name="Dashboard"></a>仪表板

当你选择 BizTalk 服务名称时，将显示“仪表板”选项卡。你不一定能够使用列出的所有选项，这取决于所用的 BizTalk 服务版本。

“仪表板”显示以下信息：

##### 使用概览：显示使用的混合连接数

另外还显示数据用量（以 GB 为单位）。

##### 度量值图表：显示性能度量值的固定列表。

这些度量值提供有关 BizTalk 服务的运行状况的实时值。你还可以为图表中显示的度量值指定**“相对”**或**“绝对”**值，以及时间范围**“间隔”**。

有关这些性能度量值的说明，请转到本主题中的[可用度量值][可用度量值]。

##### 速览：列出 BizTalk 服务属性：


<table border="1">

<tr>
<td><strong>更新跟踪数据库凭据/strong></td>
<td>更改用于登录到跟踪数据库的用户名和密码。</td>
</tr>
<tr>
<td><strong>更新 SSL 证书</strong></td>
<td>可以修改 BizTalk 服务以使用不同的 SSL 证书。当你<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=302280">创建 BizTalk 服务</a>时，系统将自动创建一个自签名 SSL 证书。</td>
</tr>
<tr>
<td><strong>下载证书</strong></td>
<td>可以将 BizTalk 服务使用的 SSL 证书下载到本地计算机。</td>
</tr>
<tr>
<td><strong>状态</strong></td>
<td>显示 BizTalk 服务的当前状态。请参阅 <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=329870">BizTalk 服务：服务状态图表</a>。 </td>
</tr>
<tr>
<td><strong>服务 URL</strong></td>
<td>BizTalk 服务的 URL。这与你在创建 BizTalk 服务时输入的<Strong>“域 URL”</Strong>相同。</td>
</tr>
<tr>
<td><strong>公用虚拟 IP (VIP) 地址</strong></td>
<td>分配给 BizTalk 服务的 IP 地址。它用于所有输入终结点，并且是出站流量的源地址。只要设置了 BizTalk 服务，则此 IP 地址就属于你的 BizTalk 服务。如果你删除 BizTalk 服务，则会将该 IP 地址分配给其他 BizTalk 服务。 </td>
</tr>
<tr>
<td><strong>ACS 命名空间</strong></td>
<td>通过 BizTalk 服务进行身份验证。</td>
</tr>
<tr>
<td><strong>版本</strong></td>
<td>列出你在创建 BizTalk 服务时输入的版本。</td>
</tr>
<tr>
<td><strong>位置</strong></td>
<td>显示托管 BizTalk 服务的地理区域。</td>
</tr>
<tr>
<td><strong>已创建</strong></td>
<td>显示创建 BizTalk 服务时的日期和时间。</td>
</tr>
<tr>
<td><strong>跟踪数据库</strong></td>
<td> 存储你的 BizTalk 服务所使用的跟踪表的 Azure SQL Database 名称。</td>
</tr>
<tr>
<td><strong>监视/存档存储</strong></td>
<td>存储 BizTalk 服务的监视输出的 Azure 存储帐户名称。</td>
</tr>
<tr>
<td><strong>订阅名称</strong></td>
<td>列出托管 BizTalk 服务的订阅。订阅控制对 Azure 管理门户的访问。</td>
</tr>
<tr>
<td><strong>订阅 ID</strong></td>
<td>当创建订阅时，将自动生成一个订阅 ID。当使用 REST API 时，你可能需要输入“订阅 ID”。</td>
</tr>
</table>

[BizTalk 服务：使用 Azure 管理门户进行设置][创建 BizTalk 服务]列出了创建 BizTalk 服务的步骤。

##### 任务栏中的“管理”、“连接信息”、“同步密钥”和“删除”：

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left"><strong>管理</strong></td>
<td align="left">打开 Azure BizTalk 服务门户。BizTalk 服务门户是进行 EDI 配置的入口，包括添加合作伙伴和创建 X12 和 AS2 协议。<br /><br /> 这与<strong>“快速启动”</strong>选项卡上的<strong>“创建合作伙伴协议”</strong>相同。<br /><br /> <a href="http://go.microsoft.com/fwlink/p/?LinkID=303653">在 BizTalk 服务门户上配置 EDI 消息传送的组件</a>提供了有关 BizTalk 服务门户的详细信息。</td>
</tr>
<tr class="even">
<td align="left"><strong>连接信息</strong></td>
<td align="left">显示“Access Control 命名空间”、“默认颁发者”和“默认密钥”值；可以复制这些值。<br /><br /> 你还可以打开访问控制管理门户。此访问控制管理门户与在左导航窗格中打开使用 Active Directory 选项相同。<br /><br /> <a href="http://go.microsoft.com/fwlink/p/?LinkID=285670">管理 ACS 命名空间</a>提供与访问控制管理门户有关的详细信息。</td>
</tr>
<tr class="odd">
<td align="left"><strong>同步密钥</strong></td>
<td align="left">当你创建存储帐户时，将自动创建主密钥和辅助密钥。这些密钥控制对存储帐户的访问。BizTalk 服务自动使用主密钥。<strong>“同步密钥”</strong>使用户可以在主密钥与辅助密钥之间切换，而不会中断 BizTalk 服务。<br /><br /> 例如，你希望 BizTalk 服务为存储帐户使用新的主密钥。为此，请按以下步骤操作：<br /><br />
<ol>
<li>依次选择你的 BizTalk 服务和<strong>“同步密钥”</strong>。选择“辅助密钥”。当你执行此操作时，BizTalk 服务开始使用辅助密钥。</li>
<li>在 Azure 管理门户中，选择“存储帐户”和“重新生成主密钥”。请记住，你的 BizTalk 服务正在使用辅助密钥。</li>
<li>依次选择你的 BizTalk 服务和<strong>“同步密钥”</strong>。现在，选择“主密钥”。这是你重新生成的新主密钥。</li>
<li>在 Azure 管理门户中，选择“存储帐户”和“重新生成辅助密钥”。</li>
</ol>
<br /> 此过程称为“变换密钥”。其目的是让用户可以在主密钥与辅助密钥之间切换，而不会中断 BizTalk 服务。</td>
</tr>
<tr class="even">
<td align="left"><strong>删除</strong></td>
<td align="left">删除你的 BizTalk 服务和部署到此服务的所有项目。</td>
</tr>
</tbody>
</table>

## <a name="Monitor"></a>监视

当你选择 BizTalk 服务名称时，将会出现“监视”选项卡，其中显示了以下信息。本部分的内容不适用于免费版。

##### 度量值图表：显示选定的性能度量值。

这些度量值提供有关 BizTalk 服务的运行状况的实时值。你可以选择要显示哪些性能度量值。最多可同时显示六个性能度量值。

你还可以为显示的度量值指定**“相对”**或**“绝对”**值，以及时间范围**“间隔”**。

##### 在图表中删除或显示度量值：

1.  选择**“监视”**选项卡。
2.  在任务栏中选择**“添加度量值”**：

    ![选择“添加度量值”][选择“添加度量值”]
3.  检查要显示的性能度量值。
4.  单击复选标记可返回到**“监视”**选项卡。
5.  选择度量值旁边的圆圈以便在图表中显示该度量值的值。

    例如，**“CPU 使用率”**度量值是灰显的；其输出不会显示在图表中：
	
    ![“CPU 使用率”度量值是灰显的][“CPU 使用率”度量值是灰显的]

    选择灰显的圆圈可以使**“CPU 使用率”**度量值在图表中显示其输出：

    ![已启用“CPU 使用率”度量值][已启用“CPU 使用率”度量值]

6.  若要从显示图表和列表中删除度量值，请选择任务栏中的**“删除度量值”**。若要将此度量值重新添回到列表，请在任务栏中选择**“添加度量值”**，检查此度量值，并单击复选标记以返回到**“监视”**选项卡。选择灰显的圆圈可启用该度量值。

## <a name="Metrics"></a>可用度量值

以下性能计数器和度量值可用：

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left"><strong>往返延迟</strong></td>
<td align="left">显示 BizTalk 服务跨所有桥处理一条消息所花的平均时间，单位为毫秒 (ms)（从收到消息开始，到完全处理此消息为止）。仅对已成功处理的消息计数。<br /><br /> 发生以下事件时，将创建时间戳：
<ul>
<li>消息进入网关</li>
<li>消息路由到目标</li>
<li>收到目标响应</li>
<li>目标确认响应发送到网关</li>
</ul>
<br /> 此度量值显示以下计算的结果：<br /><br /> [目标确认响应发送到网关] – [消息进入网关]</td>
</tr>
<tr class="even">
<td align="left"><strong>源故障数</strong></td>
<td align="left">显示 BizTalk 服务从源终结点拉取消息时失败的总消息数量。</td>
</tr>
<tr class="odd">
<td align="left"><strong>CPU 使用率</strong></td>
<td align="left">列出所有角色实例的平均 %Processor Time。</td>
</tr>
<tr class="even">
<td align="left"><strong>处理延迟</strong></td>
<td align="left">显示 BizTalk 服务跨所有桥处理一条消息所花的平均时间，单位为毫秒 (ms)（花在目标上的时间除外）。仅对已成功处理的消息计数。<br /><br /> 发生以下每个事件时，将创建时间戳：
<ul>
<li>消息进入网关</li>
<li>消息路由到目标</li>
<li>收到目标响应</li>
<li>目标确认响应发送到网关</li>
</ul>
<br />此度量值显示以下计算的结果：<br /><br /> [目标确认响应发送到网关] – [消息进入网关] – [收到目标响应] + [消息路由到目标]</td>
</tr>
<tr class="odd">
<td align="left"><strong>处理失败次数</strong></td>
<td align="left">显示 BizTalk 服务在某个时间间隔内跨所有桥进行处理时失败的消息总数。</td>
</tr>
<tr class="even">
<td align="left"><strong>发送的消息数</strong></td>
<td align="left">显示 BizTalk 服务在某个时间间隔内跨所有桥发送的总消息数。当从管道中发送的一条消息到达路由目标时，此度量值将加一。此度量值并不指示成功处理消息。<br /><br /> 在请求-答复方案中，当路由目标向管道发回一个接收确认时，此度量值将加一。</td>
</tr>
<tr class="odd">
<td align="left"><strong>收到的消息数</strong></td>
<td align="left">显示 BizTalk 服务在某个时间间隔内跨所有桥接收的总消息数。当管道收到一条新消息时，此度量值将加一。</td>
</tr>
<tr class="even">
<td align="left"><strong>正在处理的消息数</strong></td>
<td align="left">显示 BizTalk 服务在某个时间间隔内当前处理的总消息数。</td>
</tr>
<tr class="odd">
<td align="left"><strong>已处理的消息数</strong></td>
<td align="left">显示 BizTalk 服务在某个时间间隔内跨所有桥成功处理的总消息数。当管道成功收到一条消息且该消息成功路由到目标时，此度量值将加一。</td>
</tr>
</tbody>
</table>

## <a name="Scale"></a>缩放

在“缩放”选项卡上，你可以添加或减少 BizTalk 服务使用的单位数量。默认情况下配置了一个单位。可以添加额外的单位以扩展 BizTalk 服务。如果你扩大规模，则你是要增加吞吐量。资源量也增加，包括部署的桥数量、协议、LOB 连接和处理能力。例如，可以将规模从 1 个单位扩展到 2 个单位。在此情况下，你可以部署双倍的桥数、双倍的协议、双倍的 LOB 连接和双倍的处理能力。

某些 BizTalk 版本未提供“缩放”选项。在此情况下，允许一个“单位”。若要确定你所用版本可以扩展多少个单位，请参阅 [BizTalk 服务：版本图表][BizTalk 服务：版本图表]。

增大单位数可能影响定价。如果你增加单位数，则选择**“保存”**将显示一条消息，告诉你结算是否受影响。然后，你可以进行选择以继续操作。当你增大单位数后，BizTalk 服务的状态将从“有效”变为“正在更新”。在“正在更新”状态下，BizTalk 服务继续运行。

## <a name="Configure"></a>配置

将“备份状态”设置为“无”或“自动”。如果设置为“无”，则不自动创建备份。如果设置为“自动”，则需要配置备份位置、备份频率以及备份文件的保存时间长度。

[BizTalk 服务：备份和还原][BizTalk 服务：备份和还原]提供了详细信息。

**注意**：备份和还原不适用于混合连接。

## <a name="HybridConnections"></a>混合连接

使用混合连接可将网站或移动服务等 Azure 应用程序连接到使用静态 TCP 端口的本地资源，例如 SQL Server、MySQL、HTTP Web API 和大多数自定义 Web 服务。可以在 Azure 管理门户中的 BizTalk 服务内管理混合连接。

若要在 Azure 网站中创建混合连接，请参阅[混合连接：将 Azure 网站连接到本地资源][混合连接：将 Azure 网站连接到本地资源]。

若要在 Azure 移动服务中使用混合连接，请参阅 [Azure 移动服务和混合连接][Azure 移动服务和混合连接]。

若要在 Azure BizTalk 服务中创建或管理混合连接，请参阅[混合连接][1]。

## 下一步

在熟悉不同的选项卡后，你可以了解有关 Azure BizTalk 服务功能的详细信息：

-   [BizTalk 服务：限制][BizTalk 服务：限制]
-   [BizTalk 服务：颁发者名称和颁发者密钥][BizTalk 服务：颁发者名称和颁发者密钥]
-   [BizTalk 服务：备份和还原][BizTalk 服务：备份和还原]

## 另请参阅

-   [混合连接][1]
-   [BizTalk 服务：开发人员版、基本版、标准版和高级版图表][BizTalk 服务：版本图表]
-   [BizTalk 服务：使用 Azure 管理门户进行设置][创建 BizTalk 服务]
-   [BizTalk 服务：BizTalk 服务状态图表][BizTalk 服务：服务状态图表]
-   [如何开始使用 Azure BizTalk 服务 SDK][如何开始使用 Azure BizTalk 服务 SDK]

  [快速入门]: #QuickStart
  [仪表板]: #Dashboard
  [监视]: #Monitor
  [缩放]: #Scale
  [配置]: #Configure
  [混合连接]: #HybridConnections
  [在 BizTalk 服务门户上配置 EDI 消息传送的组件]: http://go.microsoft.com/fwlink/p/?LinkID=303653
  [管理 ACS 命名空间]: http://go.microsoft.com/fwlink/p/?LinkID=285670
  [可用度量值]: #Metrics
  [创建 BizTalk 服务]: http://go.microsoft.com/fwlink/p/?LinkID=302280
  [BizTalk 服务：服务状态图表]: http://go.microsoft.com/fwlink/p/?LinkID=329870
  [选择“添加度量值”]: ./media/biztalk-dashboard-monitor-scale-tabs/WABS_AddMetrics.png
  [“CPU 使用率”度量值是灰显的]: ./media/biztalk-dashboard-monitor-scale-tabs/WABS_GrayedMetric.png
  [已启用“CPU 使用率”度量值]: ./media/biztalk-dashboard-monitor-scale-tabs/WABS_EnabledMetric.png
  [BizTalk 服务：版本图表]: http://go.microsoft.com/fwlink/p/?LinkID=302279
  [BizTalk 服务：备份和还原]: http://go.microsoft.com/fwlink/p/?LinkID=329873
  [混合连接：将 Azure 网站连接到本地资源]: http://go.microsoft.com/fwlink/p/?LinkId=397538
  [Azure 移动服务和混合连接]: http://azure.microsoft.com/en-us/documentation/articles/mobile-services-dotnet-backend-hybrid-connections-get-started
  [1]: http://go.microsoft.com/fwlink/p/?LinkID=397274
  [BizTalk 服务：限制]: http://go.microsoft.com/fwlink/p/?LinkID=302282
  [BizTalk 服务：颁发者名称和颁发者密钥]: http://go.microsoft.com/fwlink/p/?LinkID=303941
  [如何开始使用 Azure BizTalk 服务 SDK]: http://go.microsoft.com/fwlink/p/?LinkID=302335
