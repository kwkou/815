<properties linkid="hyper-v-recovery-manager-vmware" urlDisplayName="configure-Azure-Site-Recovery" pageTitle="Azure 站点恢复入门：两个使用 InMage 的本地 VMWare 站点之间的保护" metaKeywords="Azure Site Recovery, VMM, clouds, disaster recovery, VMWare, InMage" description="InMage in Azure Site Recovery handles the replication, failover and recovery between on-premises VMWare sites." metaCanonical="" umbracoNaviHide="0" disqusComments="1" title="Getting Started with Azure Site Recovery: On-Premises to On-Premises VMWare Site Protection with InMage" editor="jimbe" manager="cfreeman" authors="Haifeng Liu" />
<tags ms.service=""
    ms.date="02/18/2015"
    wacn.date="04/11/2015"
    />

# Azure 站点恢复入门：本地到本地 VMWare 站点保护


<div class="dev-callout"> 

<p>Azure 站点恢复中的 InMage 提供了本地 VMWare 站点之间的实时复制。InMage 包括在 Azure 站点恢复服务订阅中。</p>


</div>


<h2><a id="before"></a>必备条件</h2> 
<div class="dev-callout"> 

<UL>
<LI><b>Azure 帐户</b>-你将需要有一个 Azure 帐户。如果还没有，请参阅 <a href="http://www.windowsazure.cn/pricing/1rmb-trial/">Azure 试用版</a>。<div  style="display:none">Get subscription pricing information at <a href="/pricing/details/recovery-manager/">Azure 站点恢复管理器定价详细信息</a>。</div></LI>
</UL>



<a name="vault"></a> <h2>步骤 1：创建保管库并下载 InMage</h2>

1. 登录到[管理门户](https://manage.windowsazure.cn)。


2. 依次展开<b>"数据服务"</b>、<b>"恢复服务"</b>，然后单击<b>"站点恢复保管库"</b>。


3. 依次单击<b>"新建"</b>、<b>"快速创建"</b>。
	
4. 在<b>"名称"</b>中，输入一个友好名称以标识此保管库。

5. 在<b>"区域"</b>中，为保管库选择地理区域。 
<div  style="display:none">Get subscription pricing information at <a href="/pricing/details/recovery-manager/">Azure 站点恢复管理器定价详细信息</a>。</div>
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

<h2 style="display:none"><a id="next"></a>后续步骤</h2>
<UL>

<LI style="display:none">如有问题，请访问 <a href="http://social.msdn.microsoft.com/Forums/windowsazure/zh-CN/home?forum=hypervrecovmgr">Azure 恢复服务论坛</a>。</LI> 
</UL>
