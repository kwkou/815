<properties linkid="azure-subscription-service-limits" urlDisplayName="Azure Subscription and Service Limits, Quotas, and Constraints" pageTitle="Microsoft Azure 订阅和服务限制、配额和约束" metaKeywords="Cloud Services, Virtual Machines, Web Sites, Virtual Network, SQL Database, Subscription, Storage" description="提供常见的 Azure 订阅和服务限制以及最大值的列表。" metaCanonical="" services="web-sites,virtual-machines,cloud-services" documentationCenter="" title="" authors="jroth" solutions="" manager="paulettm" editor="mollybos" />

# Azure 订阅和服务限制、配额和约束

下面的文档中指定一些最常见的 Microsoft Azure 限制。请注意本文档中当前不包括所有 Azure 服务。一段时间后，将展开并更新这些限制以包含多个平台。

-   [订阅限制][订阅限制]
-   [Web 工作进程限制][Web 工作进程限制]
-   [虚拟机限制][虚拟机限制]
-   [网络限制][网络限制]
-   [存储限制][存储限制]
-   [SQL 数据库限制][SQL 数据库限制]

> [WACOM.NOTE] 如果想要提高**默认限制**上的限制，请用[客户支持][客户支持]打开事件。超过下表中的**最大限制**值无法提高限制。如果没有任何**最大限制**列，则指定的资源不具有可调整的限制。

## <a name="subscription"></a>订阅限制

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">资源</th>
<th align="left">默认限制</th>
<th align="left">最大限制</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p>每个<a href="http://msdn.microsoft.com/zh-cn/library/azure/hh531793.aspx">订阅</a>内核数<sup>1</sup></p></td>
<td align="left"><p>20</p></td>
<td align="left"><p>10,000</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个订阅的<a href="http://msdn.microsoft.com/zh-cn/library/azure/gg456328.aspx">共同管理员</a></p></td>
<td align="left"><p>200</p></td>
<td align="left"><p>200</p></td>
</tr>
<tr class="odd">
<td align="left"><p>每个订阅的<a href="http://azure.microsoft.com/zh-cn/documentation/articles/storage-whatis-account/">存储帐户</a></p></td>
<td align="left"><p>20</p></td>
<td align="left"><p>50</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个订阅的<a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-what-is/">云服务</a></p></td>
<td align="left"><p>20</p></td>
<td align="left"><p>200</p></td>
</tr>
<tr class="odd">
<td align="left"><p>每个订阅的<a href="http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx">虚拟网络</a><sup>2</sup></p></td>
<td align="left"><p>10</p></td>
<td align="left"><p>100</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个订阅的<a href="http://msdn.microsoft.com/zh-cn/library/jj157100.aspx">本地网络</a></p></td>
<td align="left"><p>10</p></td>
<td align="left"><p>100</p></td>
</tr>
<tr class="odd">
<td align="left"><p>每个订阅的 DNS 服务器</p></td>
<td align="left"><p>9</p></td>
<td align="left"><p>100</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个订阅的保留的 IP</p></td>
<td align="left"><p>5</p></td>
<td align="left"><p>100</p></td>
</tr>
</tbody>
</table>

<sup>1</sup>特小实例作为一项核心至核心限制计数，即使使用了部分核心。

<sup>2</sup>每个虚拟网络支持单个虚拟网络网关。

## <a name="webworkerlimits"></a>Web/工作进程限制

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">资源</th>
<th align="left">默认限制</th>
<th align="left">最大限制</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p><a href="http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-what-is/">每个部署的 web/辅助角色<sup>1</sup></a></p></td>
<td align="left"><p>25</p></td>
<td align="left"><p>25</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个部署的<a href="http://msdn.microsoft.com/zh-cn/library/gg557552.aspx#InstanceInputEndpoint">实例输入终结点</a></p></td>
<td align="left"><p>25</p></td>
<td align="left"><p>25</p></td>
</tr>
<tr class="odd">
<td align="left"><p>每个部署的<a href="http://msdn.microsoft.com/zh-cn/library/gg557552.aspx#InputEndpoint">输入终结点</a></p></td>
<td align="left"><p>25</p></td>
<td align="left"><p>25</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个部署的<a href="http://msdn.microsoft.com/zh-cn/library/gg557552.aspx#InternalEndpoint">内部终结点</a></p></td>
<td align="left"><p>25</p></td>
<td align="left"><p>25</p></td>
</tr>
</tbody>
</table>

<sup>1</sup>与 Web/辅助角色的每个云服务可以具有两个部署，一个用于生产，一个用于临时。

## <a name="vmlimits"></a>虚拟机限制

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">资源</th>
<th align="left">默认限制</th>
<th align="left">最大限制</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p>每个云服务的<a href="http://azure.microsoft.com/zh-cn/documentation/services/virtual-machines/">虚拟机</a><sup>1</sup></p></td>
<td align="left"><p>50</p></td>
<td align="left"><p>50</p></td>
</tr>
<tr class="even">
<td align="left"><p>输入每个云服务的终结点<sup>2</sup></p></td>
<td align="left"><p>150</p></td>
<td align="left"><p>150</p></td>
</tr>
</tbody>
</table>

<sup>1</sup>当您创建一台虚拟机后，会自动创建一项云服务来包含该虚拟机。然后可以在该相同的云服务中添加多个虚拟机。

<sup>2</sup>输入终结点用于允许包含云服务的到外部虚拟机的通信。同一个云服务中的虚拟机自动允许所有 UDP 和 TCP 端口之间的通信，以实现内部通信。

## <a name="networkinglimits"></a>网络限制

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">资源</th>
<th align="left">默认限制</th>
<th align="left">最大限制</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p><sup>1</sup>每个<a href="http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx">虚拟网络</a>的总机<sup>2</sup></p></td>
<td align="left"><p>2048</p></td>
<td align="left"><p>2048</p></td>
</tr>
<tr class="even">
<td align="left"><p>虚拟机或角色实例的并发 TCP 连接</p></td>
<td align="left"><p>500K</p></td>
<td align="left"><p>500K</p></td>
</tr>
<tr class="odd">
<td align="left"><p>每个终结点访问控制列表 (ACL)<sup>3</sup></p></td>
<td align="left"><p>50</p></td>
<td align="left"><p>50</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个虚拟网络的本地网络站点</p></td>
<td align="left"><p>10</p></td>
<td align="left"><p>10</p></td>
</tr>
</tbody>
</table>

<sup>1</sup>机器的总数，包括虚拟机和 Web/辅助角色实例。

<sup>2</sup>每个虚拟网络支持单个[虚拟网络网关][虚拟网络网关]。

<sup>3</sup>ACL 在输入终结点上支持虚拟机。对于 web/辅助角色，在输入和实例输入终结点上支持它。

## <a name="storagelimits"></a>存储限制<sup>1</sup>

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">资源</th>
<th align="left">默认限制</th>
<th align="left">最大限制</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p>每个存储帐户的 TB<sup>2</sup></p></td>
<td align="left"><p>200</p></td>
<td align="left"><p>200</p></td>
</tr>
<tr class="even">
<td align="left"><p>持久性磁盘的最大 IOPS</p></td>
<td align="left"><p>500</p></td>
<td align="left"><p>500<sup>3</sup></p></td>
</tr>
<tr class="odd">
<td align="left"><p>每个存储帐户的最大 IOPS</p></td>
<td align="left"><p>20,000</p></td>
<td align="left"><p>20,000</p></td>
</tr>
<tr class="even">
<td align="left"><p>每个存储帐户的最大入口</p></td>
<td align="left"><p>5Gbps，如果 GRS<sup>4</sup> 启用，与其禁用 10Gbps</p></td>
<td align="left"><p>5Gbps，如果 GRS<sup>4</sup> 启用，与其禁用 10Gbps</p></td>
</tr>
<tr class="odd">
<td align="left"><p>每个存储帐户的最大传出</p></td>
<td align="left"><p>10Gbps，如果 GRS<sup>4</sup> 启用，与其禁用 15Gbps</p></td>
<td align="left"><p>10Gbps，如果 GRS<sup>4</sup> 启用，与其禁用 15Gbps</p></td>
</tr>
</tbody>
</table>

<sup>1</sup>有关这些限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标][Azure 存储空间可伸缩性和性能目标]。

<sup>2</sup>对于页面 blob，只是使用中的页面会产生容量使用情况。例如，虚拟机具有 127 GB VHD 但操作系统只使用 30 GB，则仅为 VHD 中已使用的 30 GB 部分计费，而非整个 127 GB。

<sup>3</sup>不要将经常使用的 40 多个 VHD 放在一个帐户，以避免 20000 的 IOPS 限制。

<sup>4</sup>地域冗余存储帐户

## <a name="sqldblimits"></a>SQL 数据库限制

有关 SQL 数据库限制，请参阅以下主题：

-   [Azure SQL Database 服务层（版本）][Azure SQL Database 服务层（版本）]
-   [Azure SQL Database 服务层和性能级别][Azure SQL Database 服务层和性能级别]
-   [SQL Database 资源限制][SQL Database 资源限制]

## <a name="seealso"></a>另请参阅

[虚拟机和 Azure 的云服务大小][虚拟机和 Azure 的云服务大小]

[Azure 存储空间可伸缩性和性能目标][Azure 存储空间可伸缩性和性能目标]

[Azure SQL Database 服务层（版本）][Azure SQL Database 服务层（版本）]

[Azure SQL Database 服务层和性能级别][Azure SQL Database 服务层和性能级别]

[SQL Database 资源限制][SQL Database 资源限制]

  [订阅限制]: #subscription
  [Web 工作进程限制]: #webworkerlimits
  [虚拟机限制]: #vmlimits
  [网络限制]: #networkinglimits
  [存储限制]: #storagelimits
  [SQL 数据库限制]: #sqldblimits
  [客户支持]: http://azure.microsoft.com/zh-cn/support/faq/
  [虚拟网络网关]: http://msdn.microsoft.com/zh-cn/library/azure/jj156210.aspx
  [Azure 存储空间可伸缩性和性能目标]: http://msdn.microsoft.com/zh-cn/library/azure/dn249410.aspx
  [Azure SQL Database 服务层（版本）]: http://msdn.microsoft.com/zh-cn/library/azure/dn741340.aspx
  [Azure SQL Database 服务层和性能级别]: http://msdn.microsoft.com/zh-cn/library/azure/dn741336.aspx
  [SQL Database 资源限制]: http://msdn.microsoft.com/zh-cn/library/azure/dn338081.aspx
  [虚拟机和 Azure 的云服务大小]: http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx
