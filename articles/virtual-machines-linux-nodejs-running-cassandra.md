<properties linkid="services-linux-cassandra-with-linux" urlDisplayName="Cassandra with Linux" pageTitle="在 Azure 上通过 Linux 运行 Cassandra" metaKeywords="" description="说明如何在 Linux 在 Azure 虚拟机上运行 Cassandra 群集。" metaCanonical="" services="virtual-machines" documentationCenter="Node.js" title="Running Cassandra with Linux on Azure and Accessing it from Node.js" authors="" solutions="" manager="" editor="" />





<h1><a id = ""></a>在 Azure 上通过 Linux 运行 Cassandra 并从 Node.js 访问 Cassandra </h1>
**作者：** Hanu Kommalapati

## 表的内容 # #

- [概述][]
- [单区域部署][]
- [测试单个区域 Cassandra 群集][]
- [多区域部署][]
- [测试多区域 Cassandra 群集][]
- [测试从 Node.js 的 Cassandra 群集][]
- [结论][]

##<a id="overview"> </a>概述 # #
Windows Azure 是一个打开的云平台，它也为非 Microsoft 软件，其中包括操作系统、应用程序服务器、消息传递的中间件的链接，以及从这两种商业和开源模型的 SQL 和 NoSQL 数据库作为运行这两个 Microsoft。他们能够构建可复原服务包括 Azure 公有云上的需要进行仔细的规划和故意的体系结构对于这两个应用程序服务器为也存储层。Cassandra 的分布式的存储体系结构自然会有助于构建高度可用的群集故障的容错的系统。Cassandra 是云小数位数由 Apache Software Foundation cassandra.apache.org ； 在维护的 NoSQL 数据库Cassandra 用 Java 编写，并因此在 Windows 和 Linux 上运行的平台。 

本文的重点是作为单个和多数据中心群集利用 Windows Azure 虚拟机和虚拟网络在 Ubuntu 上显示 Cassandra 部署。超出本文讨论范围内的优化的生产工作负荷的群集部署了，因为它需要多个磁盘节点配置、 适当环形拓扑设计和数据建模以支持所需的复制、 数据一致性、 吞吐量和高可用性要求。 

此操作的文章将一种基本的方法，以显示什么参与构建 Cassandra 群集比较 Docker、 Chef 或傀儡，这会使基础结构部署简单得多。  

##<a id="depmodels"> </a>部署模型 # #
Windows Azure 联网允许的访问可以限制即可获得细化的网络安全的独立专用的群集的部署。由于这篇文章是关于显示在基本级别的 Cassandra 部署，我们将不专注于的一致性级别和吞吐量的最佳存储设计。下面是网络我们假设群集的要求的列表：

- 外部系统无法访问 Cassandra 数据库从 Azure 内外
- 不必为 thrift 通信在负载平衡器后面的 Cassandra 群集的
- 部署中的增强的群集可用性的每个数据中心的两个组中的 Cassandra 节点 
- 锁定群集因此，只有应用程序服务器场直接具有对数据库的访问
- SSH 以外没有公共网络终结点
- 每个 Cassandra 节点需要固定的内部 IP 地址

可以部署 Cassandra，到单个 Azure 区域或基于工作负荷的分布式特性的多个区域。可以利用多区域部署模型，以提供最终用户更接近通过相同的 Cassandra 基础结构特定的地理位置。Cassandra 的内置节点复制采用负责的多主机同步写入来自多个数据中心，并提供了对应用程序数据的一致视图。多区域部署还有助于与更广泛的 Azure 服务中断风险缓解。Cassandra 的可优化一致性和复制拓扑将帮助在满足不同的应用程序的 RPO 需求。 


###<a id="oneregion"> </a>单区域部署 # # #
我们将开始单区域部署，去总结教训中创建多区域模型。Azure 虚拟网络将用于创建独立的子网，以便可以满足上面提到的网络安全要求。创建单区域部署中所述的过程使用 Ubuntu 14.04 LTS 和 Cassandra 2.08 ；但是，可以轻松地采用此过程，到其他 Linux 变体。以下是一些单区域部署的系统的特征。  

**高可用性：**图 1 所示的 Cassandra 节点部署到两个可用性集中，以便获得高可用性的多个故障域之间分布节点。Vm 带批注的每个可用性集与映射到 2 的容错域中。Windows Azure 使用这一概念的容错域来管理未计划的宕机时间 （例如硬件或软件故障），同时升级域 （例如，主机或来宾操作系统修补/升级、 应用程序升级） 的概念适用于管理计划内停机时间。请参阅[灾难恢复和高可用性 Azure 应用程序](http://msdn.microsoft.com/zh-cn/library/dn251004.aspx)容错域和升级域中获得高可用性的角色。

![Single region deployment](./media/virtual-machines-linux-nodejs-running-cassandra/cassandra-linux1.png)

图 1：单区域部署

请注意在撰写本文时，Azure 不允许对特定故障域 ； 一组 Vm 的显式映射因此，即使使用图 1 中所示的部署模型，则统计上可能的所有虚拟机可以将都映射到两个故障域，而不是四个。 

**负载平衡 Thrift 通信：** Thrift 客户端库内的 web 服务器连接到内部负载平衡器通过群集。这要求将内部负载平衡器添加到"数据"子网的过程 （请参阅图 1） 中托管的 Cassandra 群集的云服务的上下文。一旦定义内部负载平衡器，则每个节点要求要以前定义的负载平衡器名称的负载平衡集中将批注添加的负载平衡终结点。请参阅[Azure 内部负载平衡](http://msdn.microsoft.com/zh-cn/library/azure/dn690121.aspx)有关详细信息。

**群集的种子：**务必选择种子的大多数高度可用节点，如新节点将与要发现的群集拓扑的种子节点通信。从每个可用性集中的一个节点被指定为种子节点，以避免单点故障。

**复制因子和一致性级别：** Cassandra 的内置高可用性和数据耐用性表征复制因子 (RF-存储在群集上每个行的副本数） 和一致性级别 （多个副本，以便将结果返回到调用方之前，来读/写）。而在发出 CRUD 查询程序指定的一致性级别，将在 KEYSPACE （类似于关系数据库） 创建期间指定复制系数。尽管一致性级别指定发出查询时，将在 KEYSPACE 创建期间指定复制系数。请参阅位于 Cassandra 文档[的一致性配置](http://www.datastax.com/documentation/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html)有关一致性的详细信息和仲裁计算的公式。

Cassandra 支持两种类型的数据完整性模型 - 一致性和最终一致性 ；复制因子和一致性级别将在一起确定是否数据都将保持一致，只要写入操作已完成，否则它将最终一致。例如，指定仲裁，如一致性级别将始终可确保数据一致性，而任何一致性级别，下面的按需获得要写入的副本数目仲裁 （例如一个） 将导致最终确保一致的数据。 

如上所示） 复制因子为 3 和仲裁 8 节点群集 （2 个节点是读取或写入的一致性） 读/写一致性级别，可以经受得住理论上的每个应用程序开始注意到失败之前的复制组在最 1 节点丢失。此操作假定所有的密钥空间具有均衡读/写请求。以下是我们将为已部署的群集使用的参数： 

单个区域 Cassandra 群集配置：

<table>
<tr>

<th>群集参数</th><th>值</th><th>备注</th></tr>
<tr><td>Number of Nodes (N) </td><td>8</td><td>群集中节点的总数</td></tr>
<tr><td>复制因子 (RF)</td><td>	3 </td><td>	某一给定行的副本数 </td></tr>
<tr><td>一致性级别 （写入）</td><td>	仲裁[(RF/2) + 1) = 2][公式的结果向下舍入] </td><td> 将最多 2 个副本写入之前将响应发送给调用方 ；第三个副本以最终一致的方式编写。 </td></tr>
<tr><td>一致性级别 （读取）	</td><td>仲裁[(RF/2) + 1 = 2][公式的结果向下舍入]</td><td>	在发送给调用方的响应前读取 2 个副本。</td></tr>
<tr><td>复制策略 </td><td>	NetworkTopologyStrategy[请参阅[数据复制](http://www.datastax.com/documentation/cassandra/2.0/cassandra/architecture/architectureDataDistributeReplication_c.html)的详细信息的 Cassandra 文档中]</td><td>	了解部署拓扑，并将副本放置在节点上，以便所有副本不在同一机架上都结束</td></tr>
<tr><td>Snitch	</td><td>GossipingPropertyFileSnitch[请参阅[Snitches](http://www.datastax.com/documentation/cassandra/2.0/cassandra/architecture/architectureSnitchesAbout_c.html)的详细信息的 Cassandra 文档中]</td><td>	NetworkTopologyStrategy 使用 snitch 的概念来理解拓扑。GossipingPropertyFileSnitch 能更好地控制以将每个节点映射到数据中心和机架。然后，该群集使用小道消息传播此信息。这是在相对于 PropertyFileSnitch 的动态 IP 设置要简单得多 </td></tr>
</TABLE>

**Cassandra 群集的 azure 注意事项：** Windows Azure 虚拟机功能将 Azure Blob 存储用于磁盘持久性 ；Azure 存储空间将保存每个磁盘为实现高耐用性 3 的副本。这意味着每行插入到 Cassandra 表中的数据已经存储在 3 的副本，因此数据一致性是已经被注意了即使复制因子 (RF) 为 1。复制因子为 1 的主要问题是该应用程序将会经历的停机，即使单个的 Cassandra 节点出现故障。但是，如果由 Azure 结构控制器识别的问题 （例如硬件、 系统软件故障） 的情况下，一个节点已关闭状态，它将设置一个新的节点在其位置使用同一个存储驱动器。设置要替换旧的新节点，则可能需要几分钟的时间。与此类似的计划内的维护活动 （如来宾操作系统变化等方面，Cassandra 升级和应用程序更改 Azure 结构控制器执行滚动升级的节点的群集中。滚动升级还可能需要几个节点下一次，并且因此群集可能会遇到的几个分区短暂的停机。但是，数据将不会由于内置的 Azure 存储冗余而丢失。

对于系统部署到不需要高可用性的 Azure （例如大约 99.9 这等同于 8.76 的小时年 ； 请参阅[高可用性](http://en.wikipedia.org/wiki/High_availability)有关详细信息） 您可能能够运行与 RF = 1 且一致性级别 = ONE。对于具有高可用性要求，RF 应用程序 = 3 和一致性级别 = 仲裁将承受的节点之一副本之一的停机时间。RF = 1 在传统部署 （例如本地） 都不能使用由于因诸如磁盘故障之类的问题的可能的数据丢失。   

## 多区域部署 # #
Cassandra 的数据中心意识的复制和上面所述的一致性模型有助于与在初始状态下无需任何外部工具的多区域部署。这是非常不同于传统的关系数据库为数据库镜像对多主机写入安装程序可能会十分复杂。在多区域设置中的 Cassandra 可帮助完成使用方案包括： 

**邻近基于部署：**具有清晰的映射的租户用户多租户应用程序-到-区域中，可以获益了多区域群集的低延迟的影响。有关示例学习管理教育机构的系统可以部署在美国东部和美国西部区域，用于为各自的大学校园中的分布式的群集以及分析事务。数据可以在时间读取和写入本地一致，并可以跨这两个区域是最终一致。还有其他示例，如媒体分发、 电子商务和任何内容，并提供集中的地域用户基的所有内容都是这种部署模型的一个很好的用例。

**高可用性：**冗余是如何达到最大的软件和硬件 ； 高可用性的关键因素有关详细信息，请参阅 Windows Azure 一构建可靠的云系统。在 Windows Azure 上实现真正的冗余的唯一可靠的方式是通过部署多区域群集。应用程序可以部署在主动-主动或主动-被动模式下，并且如果某个区域已关闭，Azure Traffic Manager 可以将流量重定向到活动的区域。与单区域部署中，如果可用性 99.9，两个区域部署可用来实现 99.9999 由公式计算的可用性： (1-(1-0.999) * (1-0.999)) * 100) ；请参阅上述的文章，有关详细信息。

**灾难恢复：**多区域 Cassandra 群集中，如果设计正确，可以会承受灾难性的数据中心服务中断。如果一个区域已关闭，部署到其他区域的应用程序可以启动为最终用户提供服务。像任何其他业务连续性实施，该应用程序必须是异步的管道中的数据对于因某些数据丢失容错。但是，Cassandra 使得恢复比传统的数据库恢复过程所花费的时间更快速。图 2 显示在每个区域中具有八个节点的典型的多区域部署模型。这两个区域是对称性的相互 ； 同一的镜像映像现实世界的设计取决于工作负荷类型 （例如事务性或分析）、 RPO、 RTO、 数据一致性和可用性要求。

![Multi region deployment](./media/virtual-machines-linux-nodejs-running-cassandra/cassandra-linux2.png)

图 2：多区域 Cassandra 部署

### 网络集成 # # #
与使用 VPN 隧道相互进行通信的虚拟机部署到位于两个区域的专用网络的集。VPN 隧道连接两个网络部署过程中设置的软件网关。这两个区域都有相似的网络体系结构，在"web"和"数据"的子网 ； 方面Azure 网络允许的尽可能多子网，根据需要创建，并应用所需的网络安全 Acl。在群集拓扑之间的设计时数据中心的通信延迟和网络流量的经济影响需要考虑。 

### 用于多数据中心部署的数据一致性 # # #
分布式部署需要意识到的对吞吐量和高可用性的群集拓扑影响。RF 和一致性级别必须选择仲裁不依赖于所有数据中心的可用性这种方式。 
对于需要高一致性的系统，一致性级别 （用于读取和写入) LOCAL_QUORUM 将确保本地读取和写入都满足从本地节点时数据将以异步方式复制到远程数据中心。表 2 总结了向上写后面所述的多区域群集的配置详细信息。 

**两个区域 Cassandra 群集配置**

<table>
<tr><th>群集参数 </th><th>值	</th><th>备注 </th></tr>
<tr><td>Number of Nodes (N) </td><td>	8 + 8	</td><td> 群集中节点的总数 </td></tr>
<tr><td>复制因子 (RF)</td><td>	3 	</td><td>数量的某一给定行的副本 </td></tr>
<tr><td>一致性级别 （写入）	</td><td>LOCAL_QUORUM [(sum(RF)/2) + 1) = 4][公式的结果向下舍入]	</td><td>2 个节点将被写入到的第一个数据中心以同步方式 ；仲裁所需的其他 2 节点将以异步方式写入到第二个数据中心。 </td></tr>
<tr><td>一致性级别 （读取）</td><td>	LOCAL_QUORUM [((RF/2) + 1) = 2][公式的结果向下舍入]	</td><td>读取请求满足从只有一个区域 ；将响应发送回客户端之前，将读取 2 个节点。  </td></tr>
<tr><td>复制策略 </td><td>	NetworkTopologyStrategy[请参阅[数据复制](http://www.datastax.com/documentation/cassandra/2.0/cassandra/architecture/architectureDataDistributeReplication_c.html)的详细信息的 Cassandra 文档中] </td><td>	了解部署拓扑，并将副本放置在节点上，以便所有副本不在同一机架上都结束  </td></tr>
<tr><td>Snitch</td><td> GossipingPropertyFileSnitch[请参阅[Snitches](http://www.datastax.com/documentation/cassandra/2.0/cassandra/architecture/architectureSnitchesAbout_c.html)的详细信息的 Cassandra 文档中] </td><td>NetworkTopologyStrategy 使用 snitch 的概念来理解拓扑。GossipingPropertyFileSnitch 能更好地控制以将每个节点映射到数据中心和机架。然后，该群集使用小道消息传播此信息。这是在相对于 PropertyFileSnitch 的动态 IP 设置要简单得多 </td></tr> 
</table> 

##软件配置 # #
在部署过程中使用以下软件版本：

<table>
<tr><th>软件</th><th>源</th><th>版本</th></tr>
<tr><td>JRE	</td><td>[JRE 8](http://www.oracle.com/technetwork/java/javase/downloads/server-jre8-downloads-2133154.html) </td><td>8U5</td></tr>
<tr><td>JNA	</td><td>[JNA](https://github.com/twall/jna) </td><td> 3.2.7</td></tr>
<tr><td>Cassandra</td><td>[Apache Cassandra 2.0.8](http://www.apache.org/dist/cassandra/2.0.8/apache-cassandra-2.0.8-bin.tar.gz)</td><td> 2.0.8</td></tr>
<tr><td>Ubuntu	</td><td>[Azure Portal](http://www.windowsazure.cn) </td><td>14.04 LTS</td></tr>
</table>

由于下载的 JRE 需要手动接受 Oracle 许可证，来简化部署，下载所需的所有软件到更高版本将上载到我们将创建为群集部署的前提 Ubuntu 模板映像的桌面。 

将上述软件下载到在本地桌面上的 （例如在 Windows 上的 %TEMP%/downloads 或 ~/downloads Linux 或 Mac 上的） 的已知下载目录。 

### 创建 UBUNTU VM # # #
在此步骤中在过程中我们将使用必备的软件创建 Ubuntu 映像，以便可以重用该映像设置几个 Cassandra 节点。  
####步骤 1：生成 SSH 密钥对 # # #
Azure 在进行配置时需要用 PEM 或 DER 编码的 X509 公钥。使用生成一个公钥/私钥密钥对在如何使用 SSH 与 Linux 一起在 Azure 上的说明进行操作。如果您计划将 putty.exe 用作 SSH 客户端在 Windows 或 Linux 上，则必须将 PEM 编码转换为 PPK 格式使用 puttygen.exe ； 的 RSA 私钥可以在上面的网页中找到的说明。 

####步骤 2：创建 Ubuntu 模板 VM # # #
若要创建虚拟机的模板，请登录到 azure.microsoft.com 门户，并使用以下顺序：单击新建、 计算、 虚拟机、 从库、 UBUNTU、 Ubuntu Server 14.04 LTS，，然后单击向右箭头。描述如何创建 Linux 虚拟机的教程，请参阅创建运行 Linux 的虚拟机。

在"虚拟机配置"屏幕 #1 上输入以下信息： 

<table>
<tr><th>字段名称              </td><td>       字段值               </td><td>         备注                </td><tr>
<tr><td>VERSION RELEASE DATE    </td><td> 向下选择从 drow 日期</td><td></td><tr>
<tr><td>VIRTUAL MACHINE NAME    </td><td> cass 模板	               </td><td> 这是 VM 的主机名 </td><tr>
<tr><td>TIER	                 </td><td> 标准	                       </td><td> 保留默认值              </td><tr>
<tr><td>SIZE	                 </td><td> A1                              </td><td>选择虚拟机根据 IO 需求 ；为此目的保留默认值 </td><tr>
<tr><td> NEW USER NAME	         </td><td> localadmin	                   </td><td> "admin"是 Ubuntu 12.xx 中和之后的保留的用户名称</td><tr>
<tr><td> AUTHENTICATION	     </td><td> 单击复选框                 </td><td>检查是否要使用 SSH 密钥进行保护 </td><tr>
<tr><td> CERTIFICATE	         </td><td> 公钥证书的文件名 </td><td> 使用以前生成的公钥</td><tr>
<tr><td> New Password	</td><td> 强密码 </td><td> </td><tr>
<tr><td> Confirm Password	</td><td> 强密码 </td><td></td><tr>
</table>

在"虚拟机配置"屏幕 #2 上输入以下信息： 

<table>
<tr><th>字段名称             </th><th> 字段值	                   </th><th> 备注                                 </th></tr>
<tr><td> CLOUD SERVICE	</td><td> 创建新的云服务	</td><td>云服务是容器计算资源 （如虚拟机</td></tr>
<tr><td> CLOUD SERVICE DNS NAME	</td><td>ubuntu template.cloudapp.net	</td><td>为计算机提供不可知的负载平衡器名称</td></tr>
<tr><td> REGION/AFFINITY GROUP/VIRTUAL NETWORK </td><td>	美国西部	</td><td> 选择你的 Web 应用程序从中访问 Cassandra 群集的区域</td></tr>
<tr><td>STORAGE ACCOUNT </td><td>	使用默认值	</td><td>在某一特定区域中使用的默认存储帐户或预创建的存储帐户</td></tr>
<tr><td>AVAILABILITY SET </td><td>	无 </td><td>	将保留为空</td></tr>
<tr><td>ENDPOINTS	</td><td>使用默认值 </td><td>	使用默认的 SSH 配置 </td></tr>
</table>

单击右箭头，在屏幕上的 #3 中保留默认设置，单击"检查"按钮，以完成 VM 设置过程。几分钟后与名称"ubuntu 的模板"虚拟机应处于"正在运行"状态。 

###安装必要软件 # # #
####步骤 1：上载 tarballs # # #
使用 scp 或 pscp，将以前下载的软件复制到使用以下命令格式的 ~/downloads 目录中： 

#####pscp 服务器-jre-8u5-linux-x64.tar.gz localadmin@hk-cas-template.cloudapp.net:/home/localadmin/downloads/server-jre-8u5-linux-x64.tar.gz # # #

至于 Cassandra bits 对 JRE 以及重复上面的命令。 

####步骤 2：准备目录结构和提取存档 # # #
登录到虚拟机和创建目录结构作为超级用户使用下面的 bash 脚本提取软件：

	#!/bin/bash
	CASS_INSTALL_DIR="/opt/cassandra"
	JRE_INSTALL_DIR="/opt/java"
	CASS_DATA_DIR="/var/lib/cassandra"
	CASS_LOG_DIR="/var/log/cassandra"
	DOWNLOADS_DIR="~/downloads"
	JRE_TARBALL="server-jre-8u5-linux-x64.tar.gz"
	CASS_TARBALL="apache-cassandra-2.0.8-bin.tar.gz"
	SVC_USER="localadmin"
	
	RESET_ERROR=1
	MKDIR_ERROR=2
	
	reset_installation ()
	{
	   rm -rf $CASS_INSTALL_DIR 2> /dev/null
	   rm -rf $JRE_INSTALL_DIR 2> /dev/null
	   rm -rf $CASS_DATA_DIR 2> /dev/null
	   rm -rf $CASS_LOG_DIR 2> /dev/null
	}
	make_dir ()
	{
	   if [ -z "$1" ]
	   then
	      echo "make_dir: invalid directory name"
	      exit $MKDIR_ERROR
	   fi
	   
	   if [ -d "$1" ]
	   then
	      echo "make_dir: directory already exists"
	      exit $MKDIR_ERROR
	   fi
	
	   mkdir $1 2>/dev/null
	   if [ $? != 0 ]
	   then
	      echo "directory creation failed"
	      exit $MKDIR_ERROR
	   fi
	}
	
	unzip()
	{
	   if [ $# == 2 ]
	   then
	      tar xzf $1 -C $2
	   else
	      echo "archive error"
	   fi
	   
	}
	
	if [ -n "$1" ]
	then
	   SVC_USER=$1
	fi
	
	reset_installation 
	make_dir $CASS_INSTALL_DIR
	make_dir $JRE_INSTALL_DIR
	make_dir $CASS_DATA_DIR
	make_dir $CASS_LOG_DIR
	
	#unzip JRE and Cassandra 
	unzip $HOME/downloads/$JRE_TARBALL $JRE_INSTALL_DIR
	unzip $HOME/downloads/$CASS_TARBALL $CASS_INSTALL_DIR
	
	#Change the ownership to the service credentials
	
	chown -R $SVC_USER:$GROUP $CASS_DATA_DIR
	chown -R $SVC_USER:$GROUP $CASS_LOG_DIR
	echo "edit /etc/profile to add JRE to the PATH"
	echo "installation is complete"


如果此脚本粘贴到 vim 窗口时，请确保要删除的 carriage return （\r"） 使用以下命令：

	tr -d '\r' <infile.sh >outfile.sh

####步骤 3：编辑等 / 配置文件 # # #
将在结束以下内容追加： 

	JAVA_HOME=/opt/java/jdk1.8.0_05 
	CASS_HOME= /opt/cassandra/apache-cassandra-2.0.8
	PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$CASS_HOME/bin
	export JAVA_HOME
	export CASS_HOME
	export PATH

####步骤 4：为生产系统 # # # 安装 JNA
使用以下命令序列： 
以下命令将安装到 /usr/share.java 目录 jna 3.2.7.jar 和 jna 平台-3.2.7.jar
sudo apt get 安装 libjna java 

$CASS_HOME/lib 目录中创建符号链接，以便 Cassandra 启动脚本可以找到这些 jar：

	ln -s /usr/share/java/jna-3.2.7.jar $CASS_HOME/lib/jna.jar

	ln -s /usr/share/java/jna-platrom-3.2.7.jar $CASS_HOME/lib/jna-platform.jar

####步骤 5：配置 cassandra.yaml###
编辑 cassandra.yaml 以反映所需的所有虚拟机的配置每个虚拟机上[我们将在实际设置期间调整此]： 

<table>
<tr><th>字段名称   </th><th> 值  </th><th>	备注 </th></tr>
<tr><td>cluster_name </td><td>	""CustomerService	</td><td> 使用反映您的部署的名称</td></tr> 
<tr><td>listen_address	</td><td>[将保留为空]	</td><td> 删除"localhost" </td></tr>
<tr><td>rpc_addres   </td><td>[将保留为空]	</td><td> 删除"localhost" </td></tr>
<tr><td>seeds	</td><td>"10.1.2.4、 10.1.2.6，10.1.2.8"	</td><td>它被指定为种子的 IP 地址的列表。</td></tr>
<tr><td>endpoint_snitch </td><td> org.apache.cassandra.locator.GossipingPropertyFileSnitch </td><td> 这用于通过 NetworkTopologyStrateg 推断数据中心和 VM 的机架</td></tr>
</table>

####步骤 6：捕获 VM 映像 # # #
登录到虚拟机使用主机名 (hk ca template.cloudapp.net) 和先前创建的 SSH 私钥。请参阅如何使用 SSH 与 Linux on Azure 有关如何使用命令 ssh 或 putty.exe 中记录的详细信息。 

执行以下一系列操作，以捕获映像：
#####1.取消设置 # # #
使用命令"sudo waagent-deprovision + user"若要删除虚拟机实例的特定信息。有关，请参阅[如何捕获 Linux 虚拟机](/zh-cn/documentation/articles/virtual-machines-linux-capture-image/) 若要将用作模板更多详细信息在映像捕获进程上。 

#####2：关闭虚拟机 # # #
请确保虚拟机将突出显示，然后单击底部命令栏中的关闭链接。

#####3：捕获映像 # # #
请确保虚拟机将突出显示，然后单击底部命令栏中的捕获链接。在下一步的屏幕中，提供映像名称 （例如 hk-cas-2-08-ub-14-04-2014071)，、 相应的图像的描述，然后单击"检查"标记，以完成捕获过程。

这将需要几秒钟时间，并且图像应在映像库的我的映像部分中可用。源虚拟机将自动 delated 后成功捕获映像。 

##单区域部署过程 # #
**步骤 1：创建虚拟网络**
登录到管理门户，然后使用在表中的属性节目中创建虚拟网络。请参阅[在管理门户中配置仅限云的虚拟网络](http://msdn.microsoft.com/zh-cn/library/azure/dn631643.aspx)有关过程的详细步骤。      

<table>
<tr><th>虚拟机属性名称</th><th>值</th><th>备注</th></tr>
<tr><td>名称</td><td>虚拟网络-cass-西-我们</td><td></td></tr>	
<tr><td>区域</td><td>美国西部</td><td></td></tr>	
<tr><td>DNS 服务器	</td><td>无</td><td>将其忽略，因为我们不使用了 DNS 服务器</td></tr>
<tr><td>配置点到站点 VPN</td><td>无</td><td> 忽略此警告</td></tr>
<tr><td>配置站点到站点 VPN</td><td>无</td><td> 忽略此警告</td></tr>
<tr><td>地址空间</td><td>10.1.0.0/16</td><td></td></tr>	
<tr><td>起始 IP</td><td>10.1.0.0</td><td></td></tr>	
<tr><td>CIDR </td><td>/16 (65531)</td><td></td></tr>
</table>

添加下面的子网： 

<table>
<tr><th>名称</th><th>起始 IP</th><th>CIDR</th><th>备注</th></tr>
<tr><td>web</td><td>10.1.1.0</td><td>/ 24 (251)</td><td>对 web 场的子网</td></tr>
<tr><td>数据</td><td>10.1.2.0</td><td>/ 24 (251)</td><td>数据库节点的子网</td></tr>
</table>

通过网络安全组的覆盖率是不在本文的范围内，可以保护数据和 Web 子网。  

**步骤 2：设置虚拟机** 
使用以前创建的映像，我们将在云服务器"hk-c-svc-西"中创建以下虚拟机并将其绑定到各自的子网中，如下所示： 

<table>
<tr><th>计算机名称    </th><th>子网	</th><th>IP 地址	</th><th>可用性集</th><th>DC/机架</th><th>种子？</th></tr>
<tr><td>hk-c1-west-us	</td><td>data	</td><td>10.1.2.4	</td><td>hk-c-aset-1	</td><td>dc =WESTUS rack =rack1 </td><td>是</td></tr>
<tr><td>hk-c2-west-us	</td><td>data	</td><td>10.1.2.5	</td><td>hk-c-aset-1	</td><td>dc =WESTUS rack =rack1	</td><td>否 </td></tr>
<tr><td>hk-c3-west-us	</td><td>data	</td><td>10.1.2.6	</td><td>hk-c-aset-1	</td><td>dc =WESTUS rack =rack2	</td><td>是</td></tr>
<tr><td>hk-c4-west-us	</td><td>data	</td><td>10.1.2.7	</td><td>hk-c-aset-1	</td><td>dc =WESTUS rack =rack2	</td><td>否 </td></tr>
<tr><td>hk-c5-west-us	</td><td>data	</td><td>10.1.2.8	</td><td>hk-c-aset-2	</td><td>dc =WESTUS rack =rack3	</td><td>是</td></tr>
<tr><td>hk-c6-west-us	</td><td>data	</td><td>10.1.2.9	</td><td>hk-c-aset-2	</td><td>dc =WESTUS rack =rack3	</td><td>否 </td></tr>
<tr><td>hk-c7-west-us	</td><td>data	</td><td>10.1.2.10	</td><td>hk-c-aset-2	</td><td>dc =WESTUS rack =rack4	</td><td>是</td></tr>
<tr><td>hk-c8-west-us	</td><td>data	</td><td>10.1.2.11	</td><td>hk-c-aset-2	</td><td>dc =WESTUS rack =rack4	</td><td>否 </td></tr>
<tr><td>hk-w1-west-us	</td><td>web	</td><td>10.1.1.4	</td><td>hk-w-aset-1	</td><td>                       </td><td>不适用</td></tr>
<tr><td>hk-w2-west-us	</td><td>web	</td><td>10.1.1.5	</td><td>hk-w-aset-1	</td><td>                       </td><td>不适用</td></tr>
</table>

上面的列表中的 Vm 创建需要以下过程：

1.  在某一特定区域中创建空的云服务
2.	从以前捕获的映像创建虚拟机并将其附加到虚拟网络之前 ； 创建重复此过程为所有虚拟机
3.	将内部负载平衡器添加到云服务并将其附加到"数据"子网
4.	对于以前创建的每台虚拟机，将添加一个负载平衡终结点，以便通过一组负载平衡连接到以前创建的内部负载平衡器的 thrift 通信

可以使用 Azure 管理门户 ； 执行上述过程使用 Windows 计算机 （如果您没有 Windows 计算机的访问权限的 Azure 上的虚拟机的使用），使用下面的 PowerShell 脚本来自动设置所有 8 的虚拟机。

**列表 1：用于设置虚拟机时 PowerShell 脚本**
		
		#Tested with Azure Powershell - November 2014	
		#This powershell script deployes a number of VMs from an existing image inside an Azure region
		#Import your Azure subscription into the current Powershell session before proceeding
		#The process: 1. create Azure Storage account, 2. create virtual network, 3.create the VM template, 2. crate a list of VMs from the template
		
		#fundamental variables - change these to reflect your subscription
		$country="us"; $region="west"; $vnetName = "your_vnet_name";$storageAccount="your_storage_account"
		$numVMs=8;$prefix = "hk-cass";$ilbIP="your_ilb_ip"
		$subscriptionName = "Azure_subscription_name"; 
		$vmSize="ExtraSmall"; $imageName="your_linux_image_name"
		$ilbName="ThriftInternalLB"; $thriftEndPoint="ThriftEndPoint"
		
		#generated variables
		$serviceName = "$prefix-svc-$region-$country"; $azureRegion = "$region $country"
		
		$vmNames = @()
		for ($i=0; $i -lt $numVMs; $i++)
		{
		   $vmNames+=("$prefix-vm"+($i+1) + "-$region-$country" );
		}
		
		#select an Azure subscription already imported into Powershell session
		Select-AzureSubscription -SubscriptionName $subscriptionName -Current
		Set-AzureSubscription -SubscriptionName $subscriptionName -CurrentStorageAccountName $storageAccount
		
		#create an empty cloud service
		New-AzureService -ServiceName $serviceName -Label "hkcass$region" -Location $azureRegion
		Write-Host "Created $serviceName"
		
		$VMList= @()   # stores the list of azure vm configuration objects
		#create the list of VMs
		foreach($vmName in $vmNames)
		{
		   $VMList += New-AzureVMConfig -Name $vmName -InstanceSize ExtraSmall -ImageName $imageName |
		   Add-AzureProvisioningConfig -Linux -LinuxUser "localadmin" -Password "Local123" |
		   Set-AzureSubnet "data"
		}
		
		New-AzureVM -ServiceName $serviceName -VNetName $vnetName -VMs $VMList
		
		#Create internal load balancer
		Add-AzureInternalLoadBalancer -ServiceName $serviceName -InternalLoadBalancerName $ilbName -SubnetName "data" -StaticVNetIPAddress "$ilbIP"
		Write-Host "Created $ilbName"
		#Add add the thrift endpoint to the internal load balancer for all the VMs
		foreach($vmName in $vmNames)
		{
		    Get-AzureVM -ServiceName $serviceName -Name $vmName |
		        Add-AzureEndpoint -Name $thriftEndPoint -LBSetName "ThriftLBSet" -Protocol tcp -LocalPort 9160 -PublicPort 9160 -ProbePort 9160 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -InternalLoadBalancerName $ilbName | 
		        Update-AzureVM 
		
		    Write-Host "created $vmName"     
		}

**步骤 3：在每台虚拟机上配置 Cassandra**

登录到虚拟机，然后执行以下： 

* 编辑 $CASS_HOME/conf/cassandra-rackdc.properties 指定的数据中心和机架属性：
      
       dc =EASTUS, rack =rack1

* 编辑 cassandra.yaml 以将种子节点配置如下所示：
     
       Seeds: "10.1.2.4,10.1.2.6,10.1.2.8,10.1.2.10"

**步骤 4：启动虚拟机和测试群集**

登录到其中一个节点 （例如 hk-c1-西-美国） 并运行以下命令以查看群集的状态： 
       
       nodetool -h 10.1.2.4 -p 7199 status 

您应该看到类似于将 8 节点群集如下显示： 

<table>
<tr><th>状态</th></th>地址	</th><th>加载	</th><th>令牌	</th><th>所有 </th><th>主机 ID	</th><th>机架</th></tr>
<tr><th>取消	</td><td>10.1.2.4 	</td><td>87.81 KB	</td><td>256	</td><td>38.0%	</td><td>Guid （删除）</td><td>rack1</td></tr>
<tr><th>取消	</td><td>10.1.2.5 	</td><td>41.08 KB	</td><td>256	</td><td>68.9%	</td><td>Guid （删除）</td><td>rack1</td></tr>
<tr><th>取消	</td><td>10.1.2.6 	</td><td>55.29 KB	</td><td>256	</td><td>68.8%	</td><td>Guid （删除）</td><td>rack2</td></tr>
<tr><th>取消	</td><td>10.1.2.7 	</td><td>55.29 KB	</td><td>256	</td><td>68.8%	</td><td>Guid （删除）</td><td>rack2</td></tr>
<tr><th>取消	</td><td>10.1.2.8 	</td><td>55.29 KB	</td><td>256	</td><td>68.8%	</td><td>Guid （删除）</td><td>机架 3</td></tr>
<tr><th>取消	</td><td>10.1.2.9 	</td><td>55.29 KB	</td><td>256	</td><td>68.8%	</td><td>Guid （删除）</td><td>机架 3</td></tr>
<tr><th>取消	</td><td>10.1.2.10 	</td><td>55.29 KB	</td><td>256	</td><td>68.8%	</td><td>Guid （删除）</td><td>rack4</td></tr>
<tr><th>取消	</td><td>10.1.2.11 	</td><td>55.29 KB	</td><td>256	</td><td>68.8%	</td><td>Guid （删除）</td><td>rack4</td></tr>
</table>

##<a id="testone"> </a>测试单个区域群集 # #
使用以下步骤测试群集：

1.    使用 Powershell 命令获取 AzureInternalLoadbalancer commandlet，获取内部负载平衡器的 IP 地址 （例如10.1.2.101)。该命令的语法如下所示：Get AzureLoadbalancer - ServiceName"hk-c-svc-西-us"[显示以及其 IP 地址的内部负载平衡器的详细信息]
2.	登录到 web 场 （例如 hk-w1-西-美国） 的 VM 使用 Putty 或 ssh
3.	执行 $CASS_HOME/bin/cqlsh 10.1.2.101 9160 
4.	使用以下的 CQL 命令来验证该群集是否正常工作：

		CREATE KEYSPACE customers_ks WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };	
		USE customers_ks;
		CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);
		INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');
		INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');
		
		SELECT * FROM Customers;

您应该看到一个显示如下所示：

<table>
  <tr><th> customer_id </th><th> 名字 </th><th> 姓氏 </th></tr>
  <tr><td> 1 </td><td> John </td><td> Doe </td></tr>
  <tr><td> 2 </td><td> Jane </td><td> Doe </td></tr>
</table>

请注意在步骤 4 中创建 keyspace 就会使用 SimpleStrategy 复制因子为 3。SimpleStrategy 建议对单个数据中心部署而 NetworkTopologyStrategy 多数据中心部署。3 replication_factor 将给出节点出现故障的容差。 

##<a id="tworegion"> </a>多区域部署过程 # #
将利用单区域部署完成，并用于安装的第二个区域中重复相同的过程。单个和多个区域部署之间的主要差异是区域间通信 ； 安装程序的 VPN 隧道我们将先进行网络安装、 设置虚拟机和配置 Cassandra。 

###步骤 1：在第二个区域 # # # 创建虚拟网络
登录到管理门户，然后使用在表中的属性节目中创建虚拟网络。请参阅[在管理门户中配置仅限云的虚拟网络](http://msdn.microsoft.com/zh-cn/library/azure/dn631643.aspx)有关过程的详细步骤。      

<table>
<tr><th>属性名称    </th><th>值	</th><th>备注</th></tr>
<tr><td>名称	</td><td>虚拟网络-cass-东部-我们</td><td></td></tr>	
<tr><td>区域	</td><td>美国东部</td><td></td></tr>	
<tr><td>DNS 服务器		</td><td></td><td>将其忽略，因为我们不使用了 DNS 服务器</td></tr>
<tr><td>配置点到站点 VPN</td><td></td><td>		忽略此警告</td></tr>
<tr><td>配置站点到站点 VPN</td><td></td><td>		忽略此警告</td></tr>
<tr><td>地址空间	</td><td>10.2.0.0/16</td><td></td></tr>	
<tr><td>起始 IP	</td><td>10.2.0.0	</td><td></td></tr>
<tr><td>CIDR	</td><td>/ 16 (65531)</td><td></td></tr>
</table>	

添加下面的子网： 
<table>
<tr><th>名称    </th><th>起始 IP	</th><th>CIDR	</th><th>备注</th></tr>
<tr><td>web	</td><td>10.2.1.0	</td><td>/ 24 (251)	</td><td>对 web 场的子网</td></tr>
<tr><td>数据	</td><td>10.2.2.0	</td><td>/24 (251)	</td><td>数据库节点的子网</td></tr>
</table>


###步骤 2：创建本地网络 # # #
在 Azure 虚拟网络中的本地网络是一个映射到远程站点包括私有云或另一个 Azure 区域的代理地址空间。此代理服务器地址空间已绑定到远程网络网关路由到正确的网络目标。请参阅[配置 VNet 到 VNet 连接](http://msdn.microsoft.com/zh-cn/library/azure/dn690122.aspx)有关建立 VNET 到 VNET 连接的说明。 

创建以下详细信息每两个本地网络：

<table>
<tr><th>网络名称          </th><th>VPN 网关地址	</th><th>地址空间	</th><th>备注</th></tr>
<tr><td>hk-lnet-map-to-east-us</td><td>	23.1.1.1	</td><td>10.2.0.0/16	</td><td>在创建本地网络提供一个占位符，网关地址。创建网关后，将填充实际的网关地址。请确保地址空间与各自的远程虚拟网络 ； 完全匹配在这种情况下在美国东部区域中将创建虚拟网络。</td></tr>
<tr><td>hk-lnet-map-to-west-us	</td><td>23.2.2.2	</td><td>10.1.0.0/16	</td><td>在创建本地网络提供一个占位符，网关地址。创建网关后，将填充实际的网关地址。请确保地址空间与各自的远程虚拟网络 ； 完全匹配在这种情况下在美国西部区域中将创建虚拟网络。</td></tr>
</table>


###步骤 3：将"本地"网络映射到各自 Vnet # # #
从服务管理门户中，选择每个 vnet，单击"配置"，检查"连接到本地网络"，然后选择每个以下的详细信息的本地网络： 

<table>
<tr><th>虚拟网络 </th><th>本地网络</th></tr>
<tr><td>hk-vnet-west-us	</td><td>hk-lnet-map-to-east-us</td></tr>
<tr><td>hk-vnet-east-us	</td><td>hk-lnet-map-to-west-us</td></tr>
</table>

###步骤 4：VNET1 和 VNET2 # # # 创建网关
从虚拟网络的仪表板中，单击创建网关会触发设置过程的 VPN 网关。几分钟后的每个虚拟网络仪表板应显示实际的网关地址。 
###步骤 5：更新与各自"网关"地址 # # # 的"本地"网络
编辑这两个本地网络将占位符网关 IP 地址替换为实际的只是设置网关的 IP 地址。使用以下映射： 

<table>
<tr><th>本地网络    </th><th>虚拟网络网关</th></tr>
<tr><td>hk-lnet-map-to-east-us </td><td>Hk-虚拟网络-西-我们的网关</td></tr>
<tr><td>hk-lnet-map-to-west-us </td><td>Hk-虚拟网络-东部的人的网关</td></tr>
</table>

###步骤 6：更新共享密钥 # # #
使用下面的 Powershell 脚本来更新每个 VPN 网关的 IPSec 密钥[起见密钥用于这两个网关]： 
集 AzureVNetGatewayKey-VNetName hk-虚拟网络-东部-我们-LocalNetworkSiteName hk-lnet-map-to-west-us-SharedKey D9E76BKK
集 AzureVNetGatewayKey-VNetName hk-虚拟网络-西-我们-LocalNetworkSiteName hk-lnet-map-to-east-us-SharedKey D9E76BKK 

###步骤 6：建立 VNET 到 VNET 连接 # # #
从 Azure 服务管理门户中，使用这两个虚拟网络的"仪表板"菜单来建立网关到网关连接。使用在底部工具栏中的"连接"菜单项。几分钟后仪表板应以图形方式显示的连接详细信息。

###步骤 7：在区域 #2 中创建虚拟机 # # #
创建 Ubuntu 映像，如按照相同步骤或复制到 Azure 存储帐户的映像文件 VHD 位于区域 #2 中所述区域 #1 部署，并创建映像。使用此映像，然后 hk-c-svc-东部-我们到新的云服务创建以下虚拟机的列表： 

<table>
<tr></th>计算机名称 </th><th>子网</th><th>IP 地址</th><th>可用性集</th><th>DC/机架</th><th>种子？</th></tr>
<tr><td>hk-c1-east-us	</td><td>data	</td><td>10.2.2.4	</td><td>hk-c-aset-1	</td><td>dc =EASTUS rack =rack1	</td><td>是</td></tr>
<tr><td>hk-c2-east-us	</td><td>data	</td><td>10.2.2.5	</td><td>hk-c-aset-1	</td><td>dc =EASTUS rack =rack1	</td><td>否 </td></tr>
<tr><td>hk-c3-east-us	</td><td>data	</td><td>10.2.2.6	</td><td>hk-c-aset-1	</td><td>dc =EASTUS rack =rack2	</td><td>是</td></tr>
<tr><td>hk-c5-east-us	</td><td>data	</td><td>10.2.2.8	</td><td>hk-c-aset-2	</td><td>dc =EASTUS rack =rack3	</td><td>是</td></tr>
<tr><td>hk-c6-east-us	</td><td>data	</td><td>10.2.2.9	</td><td>hk-c-aset-2	</td><td>dc =EASTUS rack =rack3	</td><td>否 </td></tr>
<tr><td>hk-c7-east-us  </td><td>data	</td><td>10.2.2.10	</td><td>hk-c-aset-2	</td><td>dc =EASTUS rack =rack4	</td><td>是</td></tr>
<tr><td>hk-c8-east-us	</td><td>data	</td><td>10.2.2.11	</td><td>hk-c-aset-2	</td><td>dc =EASTUS rack =rack4	</td><td>否 </td></tr>
<tr><td>hk-w1-east-us	</td><td>web	</td><td>10.2.1.4	</td><td>hk-w-aset-1	</td><td>	不适用</td><td>不适用</td></tr>
<tr><td>hk-w2-east-us	</td><td>web	</td><td>10.2.1.5	</td><td>hk-w-aset-1	</td><td>	不适用</td><td>不适用</td></tr>
</table>

按照与区域 #1 相同的说明，但使用 10.2.xxx.xxx 地址空间。 
###步骤 8：在每个 VM # # # 配置 Cassandra
登录到虚拟机，然后执行以下： 

1. 编辑 $CASS_HOME/conf/cassandra-rackdc.properties 以指定格式的数据中心和机架属性：
    dc =EASTUS
    rack =rack1
2. 编辑 cassandra.yaml 以配置种子节点： 
    Seeds: "10.1.2.4,10.1.2.6,10.1.2.8,10.1.2.10,10.2.2.4,10.2.2.6,10.2.2.8,10.2.2.10"
###步骤 9：启动 Cassandra # # #
登录到每台虚拟机，然后通过运行以下命令在后台启动 Cassandra：
$CASS_HOME/bin/cassandra

##<a id="testtwo"> </a>测试多区域群集 # #
目前为止 Cassandra 已部署到每个 Azure 区域中的 8 节点的 16 个节点。这些节点都在同一个群集通过常见的群集名称和种子节点配置。使用以下过程来测试群集：
###步骤 1：获取使用 PowerShell # # # 的两个区域的内部负载平衡器 IP 
- Get AzureInternalLoadbalancer ServiceName"hk-c-svc-西-us"
- Get AzureInternalLoadbalancer ServiceName"hk-c-svc-东部-us"  

    请注意 IP 地址 （例如西-10.1.2.101，东-10.2.2.101） 显示。

###步骤 2：在登录到 hk-w1-西-我们在西部地区中执行以下 # # #
1.    执行 $CASS_HOME/bin/cqlsh 10.1.2.101 9160 
2.	执行下面的 CQL 命令：

		CREATE KEYSPACE customers_ks
		WITH REPLICATION = { 'class' : 'NetworkToplogyStrategy', 'WESTUS' : 3, 'EASTUS' : 3};
		USE customers_ks;
		CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);
		INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');
		INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');
		SELECT * FROM Customers;

您应该看到一个显示如下所示：

<table>
<tr><th>customer_id </th><th>名字</th><th>姓氏</th></tr>
<tr><td>1</td><td>John</td><td>Doe</td></tr>
<tr><td>2</td><td>Jane</td><td>Doe</td></tr>
</table>

###步骤 3：在登录到 hk-w1-东部-我们在东部区域中执行以下： # # #
1.    执行 $CASS_HOME/bin/cqlsh 10.2.2.101 9160 
2.	执行下面的 CQL 命令：

		USE customers_ks;
		CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);
		INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');
		INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');
		SELECT * FROM Customers;

对于西部地区所示，您应该看到相同的显示：

<table>
<tr><th>customer_id </th><th>名字</th><th>姓氏</th></tr>
<tr><td>1</td><td>John</td><td>Doe</td></tr>
<tr><td>2</td><td>Jane</td><td>Doe</td></tr>
</table>

执行少量的多个插入并查看那些获取已复制到西-我们在群集的一部分。 

##<a id="testnode"> </a>测试 Cassandra 群集，从 Node.js##
使用其中一个文件夹中"web"创建 Linux Vm 的层 （以前，我们将执行一个简单的 Node.js 脚本来读取以前插入的数据 

**步骤 1：安装 Node.js 和 Cassandra 客户端**

1. 安装 Node.js 和 npm
2. 安装节点包"cassandra-客户端"使用 npm
3. 执行以下脚本在外壳程序提示符处，其中显示检索到的数据的 json 字符串： 

		var pooledCon = require('cassandra-client').PooledConnection;
		var ksName = "custsupport_ks";
		var cfName = "customers_cf";
		var hostList = ['internal_loadbalancer_ip:9160'];
		var ksConOptions = { hosts: hostList,
		                     keyspace: ksName, use_bigints: false };
		
		function createKeyspace(callback){
		   var cql = 'CREATE KEYSPACE ' + ksName + ' WITH strategy_class=SimpleStrategy AND strategy_options:replication_factor=1';
		   var sysConOptions = { hosts: hostList,  
		                         keyspace: 'system', use_bigints: false };
		   var con = new pooledCon(sysConOptions);
		   con.execute(cql,[],function(err) {
		   if (err) {
		     console.log("Failed to create Keyspace: " + ksName);
		     console.log(err);
		   }
		   else {
		     console.log("Created Keyspace: " + ksName);
		     callback(ksConOptions, populateCustomerData);
		   }
		   });
		   con.shutdown();
		} 
		
		function createColumnFamily(ksConOptions, callback){
		  var params = ['customers_cf','custid','varint','custname',
		                'text','custaddress','text'];
		  var cql = 'CREATE COLUMNFAMILY ? (? ? PRIMARY KEY,? ?, ? ?)';
		var con =  new pooledCon(ksConOptions);
		  con.execute(cql,params,function(err) {
		      if (err) {
		         console.log("Failed to create column family: " + params[0]);
		         console.log(err);
		      }
		      else {
		         console.log("Created column family: " + params[0]);
		         callback();
		      }
		  });
		  con.shutdown();
		} 
		
		//populate Data
		function populateCustomerData() {
		   var params = ['John','Infinity Dr, TX', 1];
		   updateCustomer(ksConOptions,params);
		
		   params = ['Tom','Fermat Ln, WA', 2];
		   updateCustomer(ksConOptions,params);
		}
		
		//update will also insert the record if none exists
		function updateCustomer(ksConOptions,params)
		{
		  var cql = 'UPDATE customers_cf SET custname=?,custaddress=? where custid=?';
		  var con = new pooledCon(ksConOptions);
		  con.execute(cql,params,function(err) {
		      if (err) console.log(err);
		      else console.log("Inserted customer : " + params[0]);
		  });
		  con.shutdown();
		}
		
		//read the two rows inserted above
		function readCustomer(ksConOptions)
		{
		  var cql = 'SELECT * FROM customers_cf WHERE custid IN (1,2)';
		  var con = new pooledCon(ksConOptions);
		  con.execute(cql,[],function(err,rows) {
		      if (err) 
		         console.log(err);
		      else 
		         for (var i=0; i<rows.length; i++)
		            console.log(JSON.stringify(rows[i]));
		    });
		   con.shutdown();
		}
		
		//exectue the code
		createKeyspace(createColumnFamily);
		readCustomer(ksConOptions)


##<a id="conclusion"> </a>结论 # #
Windows Azure 是一个灵活的平台，允许运行的 Microsoft 和开放源代码软件，如本练习中所示。可以在单个数据中心通过群集节点分散在多个容错域上部署高度可用的 Cassandra 群集。此外可以跨多个地理位置较远的 Azure 区域的灾难证明系统部署 Cassandra 群集。Azure 和 Cassandra 在一起，就可以构造的高度可扩展、 高可用性和灾难恢复的云服务需要通过今天的互联网缩放服务。  

[概述]: #overview
[单区域部署]: #oneregion
[测试单个区域 Cassandra 群集]: #testone
[多区域部署]: #tworegion
[测试多区域 Cassandra 群集]: #testtwo
[测试 Cassandra 群集，从 Node.js]: #testnode
[结论]: #conclusion

##引用 # #
- [http://cassandra.apache.org](http://cassandra.apache.org)
- [http://www.datastax.com](http://www.datastax.com) 
- [http://www.nodejs.org](http://www.nodejs.org) 

