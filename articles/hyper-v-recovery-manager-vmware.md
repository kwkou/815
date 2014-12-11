<properties linkid="configure-hyper-v-recovery-vault" urlDisplayName="configure-Azure-Site-Recovery" pageTitle="Azure 站点恢复入门：两个使用 InMage 的本地 VMWare 站点之间的保护metaKeywords="Azure Site Recovery, VMM, clouds, disaster recovery, VMWare, InMage" description="InMage in Azure Site Recovery handles the replication, failover and recovery between on-premises VMWare sites." metaCanonical="" umbracoNaviHide="0" disqusComments="1" title="Getting Started with Azure Site Recovery: On-Premises to On-Premises VMWare Site Protection with InMage" editor="jimbe" manager="cfreeman" authors="raynew" />

<tags ms.service="site-recovery" ms.workload="backup-recovery" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="11/19/2014" ms.author="raynew" />


# Azure 站点恢复入门：本地到本地 VMWare 站点保护


<div class="dev-callout"> 

<p>Azure 站点恢复中的 InMage 提供了本地 VMWare 站点之间的实时复制。InMage 包括在 Azure 站点恢复服务订阅中。</p>


</div>


<h2><a id="before"></a>必备条件</h2> 
<div class="dev-callout"> 

<UL>
<LI><b>Azure 帐户</b>-你将需要有一个 Azure 帐户。如果还没有，请参阅 <a href="http://www.windowsazure.cn/pricing/1rmb-trial/">Azure 免费试用版</a>。<!--Get subscription pricing information at <a href="http://go.microsoft.com/fwlink/?LinkId=378268">Azure 站点恢复管理器定价详细信息</a>。--></LI>
</UL>



<a name="vault"></a> <h2>步骤 1：创建保管库并下载 InMage</h2>

1. 登录到[管理门户](https://manage.windowsazure.cn)。


2. 依次展开<b>"数据服务"</b>、<b>"恢复服务"</b>，然后单击<b>"站点恢复保管库"</b>。


3. 依次单击<b>"新建"</b>、<b>"快速创建"</b>。
	
4. 在<b>"名称"</b>中，输入一个友好名称以标识此保管库。

5. 在<b>"区域"</b>中，为保管库选择地理区域。 
<!--To check supported regions see Geographic Availability in <a href="http://go.microsoft.com/fwlink/?LinkId=389880">Azure 站点恢复定价详细信息</a>-->

6. 单击<b>"创建保管库"</b>。 

	![New Vault](./media/hyper-v-recovery-manager-configure-vault/SRSAN_HvVault.png)

<P>检查状态栏以确认保管库已成功创建。该保管库将在主"恢复服务"页上列为<b>"活动"</b>。</P>

<a name="upload"></a> <h2>步骤 2：配置保管库</h2>


1. 在<b>"恢复服务"</b>页中，单击该保管库以打开"快速开始"页面。也可随时使用该图标打开"快速开始"。

	![Quick Start Icon](./media/hyper-v-recovery-manager-configure-vault/SRSAN_QuickStartIcon.png)

2. 在下拉列表中，选择**"两个本地 VMWare 站点之间"**。
3. 下载 InMage Scout。
	
	![VMWare](./media/hyper-v-recovery-manager-configure-vault/SRVMWare_SelectScenario.png)

4. 使用随产品下载的 InMage 文档在两个 VMWare 站点之间设置复制。


<!--
<h2><a id="next"></a>后续步骤</h2>
<UL>

<LI>如有问题，请访问 <a href="http://go.microsoft.com/fwlink/?LinkId=313628">Azure 恢复服务论坛</a>。</LI> 
</UL>-->
