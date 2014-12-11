<properties urlDisplayName="configure-Azure-Site-Recovery" pageTitle="Azure 站点恢复入门：使用 Hyper-V 复制的本地到本地 VMM 站点保护metaKeywords="Azure Site Recovery, VMM, clouds, disaster recovery" description="Azure Site Recovery coordinates the replication, failover and recovery of Hyper-V virtual machines between on-premises VMM sites." metaCanonical="" umbracoNaviHide="0" disqusComments="1" title="Getting Started with Azure Site Recovery:  On-Premises to On-Premises VMM Site Protection with Hyper-V Replication" editor="jimbe" manager="johndaw" authors="raynew" />

<tags ms.service="site-recovery" ms.workload="backup-recovery" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="11/19/2014" ms.author="raynew" />


# Azure 站点恢复入门：使用 Hyper-V 复制的本地到本地 VMM 站点保护


<div class="dev-callout"> 

<p>Azure 站点恢复通过在许多部署情形中安排虚拟机的复制、故障转移和恢复，可为你的业务和工作负载连续性策略做出贡献。<p>

<P>本教程说明了如何使用 Hyper-V 复制部署 Azure 站点恢复以在本地 VMM 站点之间安排保护。本教程尽可能使用最快速的部署路径和默认设置。</P>

<UL>
<LI>如需详细了解完整部署，请阅读<a href="http://msdn.microsoft.com/zh-cn/library/azure/dn469074.aspx">规划</a>和<a href="http://msdn.microsoft.com/zh-cn/library/azure/dn168841.aspx">部署</a>指南。</LI>
<LI>要了解有关其他 Azure 站点恢复部署情形，请参阅 <a href="http://www.windowsazure.cn/zh-cn/documentation/articles/hyper-v-recovery-manager-overview/">Azure 站点恢复概述</a>。</LI>
<!--LI>在使用本教程过程中如果遇到问题，请查阅 wiki 文章 <a href="http://go.microsoft.com/fwlink/?LinkId=389879">Azure 站点恢复：常见错误情形和解决方案</a>，或在 <a href="http://go.microsoft.com/fwlink/?LinkId=313628">Azure 恢复服务论坛</a>上发布你的问题。</LI-->
</UL>

</div>


<h2><a id="before"></a>必备条件</h2> 
<div class="dev-callout"> 
<P>开始使用本教程之前，要确保一切准备工作就绪。</P>

<UL>
<LI><b>Azure 帐户</b>-你将需要有一个 Azure 帐户。如果还没有，请参阅 <a href="http://www.windowsazure.cn/pricing/1rmb-trial/">Azure 免费试用版</a>。<!--Get subscription pricing information at <a href="http://go.microsoft.com/fwlink/?LinkId=378268">Azure 站点恢复管理器定价详细信息</a>。--></LI>
<LI><b>VMM 服务器</b>-你至少需要有一台在 System Center 2012 SP1 或 System Center 2012 R2 上运行的 VMM 服务器。</LI>
<LI><b>VMM 云</b>-你应该至少在要保护的源 VMM 服务器上有一个云，在目标 VMM 服务器上也要有一个。如果你正在运行一台 VMM 服务器，那就会需要两个云。要保护的主云必须包含以下各项：<UL>
	<LI>一个或多个 VMM 主机组</LI>
	<LI>每个主机组中有一个或多个 Hyper-V 主机服务器或群集。</LI>
	<li>一个或多个位于云中源 Hyper-V 服务器上的虚拟机。</li>
		</UL></LI>
</UL>



<h2><a id="tutorial"></a>教程步骤</h2> 

在验证完先决条件后，请执行以下操作：
<UL>
<LI><a href="#vault">步骤 1：创建保管库</a>-创建 Azure 站点恢复保管库。</LI>
<LI><a href="#download">步骤 2：在每台 VMM 服务器上安装提供程序应用程序</a>-在保管库中生成注册密钥，并下载提供程序安装程序文件。在每台 VMM 服务器上运行安装程序，以安装该提供程序并在保管库中注册 VMM 服务器。</LI>
<LI><a href="#clouds">步骤 3：配置云保护</a>-为 VMM 云配置保护设置。这些保护设置应用于你为 Azure 站点恢复保护启用的云中所有的虚拟机。</LI>
<LI><a href="#networkmapping">步骤 5：配置网络映射-如果你想确保副本虚拟机在故障转移之后正确连接到目标网络，可以配置网络映射。这样可将源 VM 网络映射到目标 VM 网络。</LI>
<LI><a href="#storagemapping">步骤 6：配置存储映射</a>-如果你想指定存储数据的位置，可配置存储映射。这样可将源 VMM 服务器上的存储分类映射到目标服务器上的存储分类。</LI>
<LI><a href="#enablevirtual">步骤 7：为虚拟机启用保护</a>-为位于受保护的 VMM 云中的虚拟机启用保护</LI>
<LI><a href="#recovery plans">步骤 8：配置和运行恢复计划</a>-创建恢复计划并为该计划运行测试故障转移，以确保它工作正常。</LI>

</UL>




<a name="vault"></a> <h2>步骤 1：创建保管库</h2>

1. 登录到[管理门户](https://manage.windowsazure.cn)。


2. 依次展开<b>"数据服务"</b>、<b>"恢复服务"</b>，然后单击<b>"站点恢复保管库"</b>。


3. 依次单击<b>"新建"</b>、<b>"快速创建"</b>。
	
4. 在<b>"名称"</b>中，输入一个友好名称以标识此保管库。

5. 在<b>"区域"</b>中，为保管库选择地理区域。 
<!--To check supported regions see Geographic Availability in <a href="http://go.microsoft.com/fwlink/?LinkId=389880">Azure 站点恢复定价详细信息</a>-->

6. 单击<b>"创建保管库"</b>。 

	![New Vault](./media/hyper-v-recovery-manager-configure-vault/SR_HvVault.png)

<P>检查状态栏以确认保管库已成功创建。该保管库将在主"恢复服务"页上列为<b>"活动"</b>。</P>

<a name="upload"></a> <h2>步骤 2：配置保管库</h2>


1. 在<b>"恢复服务"</b>页中，单击该保管库以打开"快速开始"页面。也可随时使用该图标打开"快速开始"。

	![Quick Start Icon](./media/hyper-v-recovery-manager-configure-vault/SR_QuickStartIcon.png)

2. 在下拉列表中，选择**"两个本地 Hyper-V 站点之间"**。
3. 在**"准备 VMM 服务器"**中，单击**"生成注册密钥文件"**。该密钥生成后，有效期为 5 天。请将该文件复制到 VMM 服务器。安装提供程序时将会需要用到它。

	![Registration key](./media/hyper-v-recovery-manager-configure-vault/SR_E2ERegisterKey.png)
	



<a name="download"></a> <h2>步骤 3：安装 Azure 站点恢复提供程序</h2>
	

1. 在<b>"快速开始"</b>页上的**"准备 VMM 服务器"**中，单击<b>"下载用于 VMM 服务器安装的 Microsoft Azure 站点恢复提供程序"</b>，以获取该提供程序安装文件的最新版本。

2. 在源和目标 VMM 服务器上运行此文件。

3. 在**"先决条件检查"**中，选择停止 VMM 服务以开始提供程序安装。该服务会停止，并将在安装程序完成时自动重新启动。 

	![Prerequisites](./media/hyper-v-recovery-manager-configure-vault/SR_ProviderPrereq.png)

4. 在**"Microsoft 更新"**中，你可以选择获取更新。启用此设置后，将会根据你的 Microsoft 更新策略安装提供程序更新。

	![Microsoft Updates](./media/hyper-v-recovery-manager-configure-vault/SR_ProviderUpdate.png)

安装提供程序后，请继续设置以在保管库中注册服务器。

5. 在"Internet 连接"页面上，指定 VMM 服务器上运行的提供程序连接 Internet 的方式。选择<b>"使用默认系统代理设置"</b>以使用服务器上配置的默认 Internet 连接设置。

	![Internet Settings](./media/hyper-v-recovery-manager-configure-vault/SR_ProviderProxy.png)

6. 在**"注册密钥"**中，选择你从 Azure 站点恢复下载并复制到 VMM 服务器的文件。
7. 在**"保管库名称"**中，验证将要在其中注册服务器的保管库的名称。
8. 在**"服务器名称"**中，指定友好名称以标识保管库中的 VMM 服务器。

	![Server registration](./media/hyper-v-recovery-manager-configure-vault/SR_ProviderRegKeyServerName.png)


9. 在**"初始云元数据同步"**中，选择是否要为 VMM 服务器上的所有云将元数据与保管库同步。此操作只需在每台服务器上发生一次。如果你不想同步所有云，可不选中此设置，而是在 VMM 控制台的云属性中个别地同步各个云。


7. 在**"数据加密"**中，生成一个用于对 Azure 中保护的数据进行加密的证书。 
此选项对于本教程中描述的情形无关。

	![Server registration](./media/hyper-v-recovery-manager-configure-vault/SR_ProviderSyncEncrypt.png)

8. 单击<b>"注册"</b>以完成此过程。注册后，Azure 站点恢复将检索 VMM 服务器中的元数据。服务器显示在保管库中**"服务器"**页面上的<b>"资源"</b>选项卡上。




<h2><a id="clouds"></a>步骤 4：配置云保护设置</h2>

在注册了 VMM 服务器之后，你可以配置云保护设置。你在安装提供程序时启用了**"将云数据与保管库同步"**选项，所以 VMM 服务器上的所有云都将出现在保管库中的<b>"受保护的项"</b>选项卡中。

![Published Cloud](./media/hyper-v-recovery-manager-configure-vault/SR_CloudsList.png)

1. 在"快速启动"页上，单击**"为 VMM 云设置保护"**。
2. 在**"受保护的项"**选项卡上，选择你要配置的云并转至**"配置"**选项卡。注意：
3. 在<b>"目标"</b>中，选择 <b>VMM</b>。
4. 在<b>"目标位置"</b>中，选择管理要用于恢复的云的现场 VMM 服务器。
4. 在<b>"目标云"</b>中，选择要用于源云中虚拟机故障转移的目标云。注意：
	- 我们建议你选择可满足你要保护的虚拟机的恢复要求的目标云。
	- 云只能属于单个云对 - 要么作为主云，要么作为目标云。

6. 在<b>"复制频率"</b>中，保留默认设置。此值指定数据应在源和目标位置之间同步的频率。仅当 Hyper-V 主机运行 Windows Server 2012 R2 时它才相关。对于其他服务器，使用五分钟的默认设置。
7. 在<b>"其他恢复点"</b>中，保留默认设置。此值指定是否要创建其他恢复点。默认值零指定只有主虚拟机的最新恢复点存储在副本宿主服务器上。
8. 在<b>"与应用程序一致的快照的频率"</b>中，保留默认设置。此值指定创建快照的频率。快照使用卷影复制服务 (VSS) 来确保应用程序在拍摄快照时处于一致状态。如果你确实想为该教程演练设置此值，请确保将它设置为小于你配置的其他恢复点的数量。
	![Configure protection settings](./media/hyper-v-recovery-manager-configure-vault/SR_CloudSettingsE2E.png)
9. 在<b>"数据传输压缩"</b>中，指定是否应压缩所传输的复制数据。 
10. 在<b>"身份验证"</b>中，指定如何对主 Hyper-V 主机服务器和恢复 Hyper-V 主机服务器之间的流量进行身份验证。为了进行此演练，请选择 HTTPS，除非你配置了可以正常工作的 Kerberos 环境。Azure 站点恢复将为 HTTPS 身份验证自动配置证书。不需要手动配置。请注意，此设置只对在 Windows Server 2012 R2 上运行的 Hyper-V 主机服务器有关。
11. 在<b>"端口"</b>中，保留默认设置。此值设置源和目标 Hyper-V 主机计算机用于侦听复制通信流量的端口号。
12. 在<b>"复制方法"</b>中，指定将如何处理从源位置到目标位置的初始数据复制过程（在开始常规复制之前）。 
	- <b>通过网络</b>-通过网络复制数据会相当耗时且需消耗大量资源。如果云包含的虚拟机所具有的虚拟硬盘相对较小，并且主要 VMM 服务器通过较宽的带宽连接到辅助 VMM 服务器，则我们建议你使用此选项。你可以指定应立即开始复制，或者选择一个复制时间。如果你使用网络复制，我们建议你将其安排在非高峰时段进行。
	- <b>脱机</b> - 这种方法指定将使用外部媒体执行初始复制。如果你要避免网络性能下降或在地理上处于远程位置，则这种方法很有用。要使用这种方法，请在源云中指定导出位置，并在目标云中指定导入位置。当你为虚拟机启用保护时，虚拟硬盘将复制到指定的导出位置。你将其发送到目标站点，并将其复制到导入位置。系统将导入的信息复制到副本虚拟机。有关脱机复制先决条件的完整列表，请参阅"部署指南"中的<a href="http://msdn.microsoft.com/zh-cn/library/azure/dn337342.aspx">步骤 3：为 VMM 云配置保护设置</a>。
13. 选择**"删除副本虚拟机"**以指定：如果你通过选择云属性的"虚拟机"选项卡上的**"删除对虚拟机的保护"**选项停止保护虚拟机，则应删除该副本虚拟机。启用此设置后，当你禁用保护时，Azure 站点恢复中会移除该虚拟机，VMM 控制台中会移除该虚拟机的站点恢复设置，并且会删除副本。
	![Configure protection settings](./media/hyper-v-recovery-manager-configure-vault/SR_CloudSettingsE2ERep.png)

<p>保存设置之后，将会创建一个作业，在<b>"作业"</b>选项卡上可以监控到该作业。VMM 源云中的所有 Hyper-V 主机服务器将为复制进行配置。可以在<b>"配置"</b>选项卡上修改云设置。要修改目标位置或目标云，必须删除云配置，然后重新配置该云。</p>


<h2><a id="networkmapping"></a>步骤 5：配置网络映射</h2>

<p>本教程说明了在测试环境中部署 Azure 站点恢复的最简单的路径。作为本教程的组成部分，如果你确定想配置网络映射，请阅读"规划指南"中的<a href="http://msdn.microsoft.com/zh-cn/library/azure/dn469073.aspx">准备网络映射</a>。要配置映射，请遵循部署指南中<a href="http://msdn.microsoft.com/zh-cn/library/dn337346.aspx">配置网络映射</a>的步骤。</p>

<h2><a id="storagemapping"></a>步骤 6：配置存储映射</h2>

<p>本教程说明了在测试环境中部署 Azure 站点恢复的最简单的路径。作为本教程的组成部分，如果你确实想配置存储映射，请遵循部署指南中<a href="http://msdn.microsoft.com/zh-cn/library/dn788904.aspx">配置存储映射</a>中的步骤。</p>


<h2><a id="enablevirtual"></a>步骤 7：启用虚拟机保护</h2>
<p>在正确配置服务器、云和网络后，可以在云中为虚拟机启用保护。</p>
<OL>
<li>在虚拟机所在的云中的<b>"虚拟机"</b>选项卡上，单击<b>"启用保护"</b>，然后选择<b>"添加虚拟机"</b>。 </li>
<li>从云中的虚拟机列表中，选择要保护的虚拟机。</li> 
</OL>

![Enable virtual machine protection](./media/hyper-v-recovery-manager-configure-vault/SR_EnableProtectionVM.png)


<P>在**"作业"**选项卡中跟踪"启用保护"操作的进度，包括初始复制。在"完成保护"作业运行之后，虚拟机就可以进行故障转移了。启用保护并复制虚拟机之后，你就可以在 Azure 中查看它们了。</p>



![Virtual machine protection job](./media/hyper-v-recovery-manager-configure-vault/SR_VMJobs.png)


<h2><a id="recoveryplans"></a>步骤 8：测试部署</h2>

若要测试你的部署，可针对一台虚拟机运行测试故障转移，或者创建一个包括多个虚拟机的恢复计划并针对该计划运行测试故障转移。测试故障转移在隔离的网络中模拟你的故障转移和恢复机制。 


<UL>
<li>有关创建恢复计划的说明，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/azure/dn788799.aspx">创建并自定义恢复计划：本地到 Azure</a>。</li>
<li>有关运行测试故障转移的说明，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/azure/dn788912.aspx">测试本地到本地的部署</a>。</li>
</UL>


<h3><a id="runtest"></a>监视活动</h3>
<p>你可以使用<b>"作业"</b>选项卡和<b>"仪表板"</b>来查看和监视由 Azure 站点恢复保管库执行的主作业，包括为云配置保护、为虚拟机启用和禁用保护、运行故障转移（有计划的、无计划的或测试），以及提交无计划的故障转移。</p>

<p>从<b>"作业"</b>选项卡中，你可以查看作业，深入了解作业详细信息和错误，运行作业查询以检索符合特定条件的作业，将作业导出到 Excel，以及重新启动失败的作业。</p>

<p>从<b>"仪表板"</b>中，你可以下载提供程序和代理安装文件的最新版本，获取保管库的配置信息，查看其保护是由保管库管理的虚拟机的数量，查看最近的作业，管理保管库证书，以及重新同步虚拟机。</p>

<p>有关与作业和仪表板交互的详细信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/dn788906.aspx">运行和监视指南</a>。</p>
	
<h2><a id="next"></a>后续步骤</h2>
<UL>
<LI>若要在完全的生产环境中规划和部署 Azure 站点恢复，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn469074.aspx">Azure 站点恢复规划指南</a>和 <a href="http://msdn.microsoft.com/zh-cn/library/azure/dn168841.aspx">Azure 站点恢复部署指南</a>。</LI>
<!--LI>如有问题，请访问 <a href="http://go.microsoft.com/fwlink/?LinkId=313628">Azure 恢复服务论坛</a>。</LI-->
</UL>
