<table cellspacing="0" border="1">
<tr>
   <th align="left" valign="middle">资源<sup>1</sup></th>
   <th align="left" valign="middle">默认限制</th>
</tr>
<tr>
   <td valign="middle"><p>每个存储帐户的 TB</p></td>
   <td valign="middle"><p>500 TB</p></td>
</tr>
<tr>
   <td valign="middle"><p>单个 Blob 容器、表或队列的最大大小</p></td>
   <td valign="middle"><p>500 TB</p></td>
</tr>
<tr>
   <td valign="middle"><p>每个存储帐户的 Blob 容器、Blob、文件共享、表、队列、实体或消息数上限</p></td>
   <td valign="middle"><p>唯一的限制是 500 TB 的存储帐户容量</p></td>
</tr>
<tr>
   <td valign="middle"><p>文件共享的最大大小</p></td>
   <td valign="middle"><p>5 TB</p></td>
</tr>
<tr>
   <td valign="middle"><p>文件共享中的文件数上限</p></td>
   <td valign="middle"><p>唯一的限制是文件共享的总容量不超过 5 TB</p></td>
</tr>
<tr>
   <td valign="middle"><p>每个持久性磁盘最大 8 KB IOPS（基本层）</p></td>
   <td valign="middle"><p>300<sup>2</sup></p></td>
</tr>
<tr>
   <td valign="middle"><p>每个持久性磁盘最大 8 KB IOPS（标准层）</p></td>
   <td valign="middle"><p>500<sup>2</sup></p></td>
</tr>
<tr>
   <td valign="middle"><p>每个存储帐户的总请求速率（假定对象大小为 1KB）</p></td>
   <td valign="middle"><p>每秒最多 20,000 个实体或消息</p></td>
</tr>
<tr>
   <td valign="middle"><p>单个 Blob 的目标吞吐量</p></td>
   <td valign="middle"><p>每秒最多 60 MB，或每秒最多 500 个请求</p></td>
</tr>
<tr>
   <td valign="middle"><p>单个队列的目标吞吐量（1 KB 消息）</p></td>
   <td valign="middle"><p>每秒最多 2000 条消息</p></td>
</tr>
<tr>
   <td valign="middle"><p>单个表分区的目标吞吐量（1 KB 实体）</p></td>
   <td valign="middle"><p>每秒最多 2000 个实体</p></td>
</tr>
<!--<tr>
   <td valign="middle"><p>每个存储帐户的最大入口（美国区域）</p></td>
   <td valign="middle"><p>如果已启用 GRS<sup>3</sup>，则为 10 Gbps；对于 LRS，为 20 Gbps</p></td>
</tr>
<tr>
   <td valign="middle"><p>每个存储帐户的最大出口（美国区域）</p></td>
   <td valign="middle"><p>如果已启用 GRS<sup>3</sup>，则为 20 Gbps；对于 LRS，为 30 Gbps</p></td>
</tr>-->
<tr>
   <td valign="middle"><p>每个存储帐户的最大入口<!--（欧洲和亚洲区域）--></p></td>
   <td valign="middle"><p>如果已启用 GRS<sup>3</sup>，则为 5 Gbps；对于 LRS，为 10 Gbps</p></td>
</tr>
<tr>
   <td valign="middle"><p>每个存储帐户的最大出口<!--（欧洲和亚洲区域）--></p></td>
   <td valign="middle"><p>如果已启用 GRS<sup>3</sup>，则为 10 Gbps；对于 LRS，为 15 Gbps</p></td>
</tr>
</table>

<sup>1</sup>有关这些限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

<sup>2</sup>对于基本层中的虚拟机，请不要在一个存储帐户中放置 66 个以上的频繁使用的 VHD，以免超出总请求速率 20,000 这一限制 (20,000/300)。对于标准层中的虚拟机，请不要在一个存储帐户中放置 40 个以上的频繁使用的 VHD (20,000/500)。有关详细信息，请参阅 Azure 的[虚拟机](/documentation/articles/virtual-machines-windows-sizes/)和[云服务大小](/documentation/articles/cloud-services-sizes-specs/)。

<sup>3</sup>GRS 表示[地域冗余存储](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/introducing-geo-replication-for-windows-azure-storage.aspx)。LRS 表示[本地冗余存储](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/08/introducing-locally-redundant-storage-for-windows-azure-storage.aspx)。请注意，GRS 也是本地冗余的。

<!---HONumber=HO63-->