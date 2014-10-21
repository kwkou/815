<properties linkid="dev-nodejs-enablestaging" urlDisplayName="Staging Deployment" pageTitle="Stage a cloud service deployment (Node.js) - Azure" metaKeywords="Azure staging, Azure application staging, Azure test environment, Azure staging environment, Azure Virtual IP swap, Azure VIP swap" description="Learn how to deploy your Azure application to a staging environment, then deploy to a production environment using Virtual IP (VIP) swap." metaCanonical=" " services="cloud-services" documentationCenter="Node.js" title="Staging an Application in Azure" authors="larryfr" solutions="" manager="" editor="" />

# 在 Azure 中暂存应用程序

可以先将要测试的已打包应用程序部署到 Azure 中的过渡环境，然后再将该应用程序移动到用户可通过 Internet 对其进行访问的生产环境。过渡环境与生产环境完全一样，只不过访问暂存应用程序时必须使用由 Azure 生成的经过模糊处理的 URL。在验证你的应用程序能够正常运行后，可以通过执行虚拟 IP (VIP) 交换将其部署到生产环境。

<div class="dev-callout">
<b>说明</b>
<p>本文中的步骤仅适用于托管为 Azure 云服务的 Node 应用程序。</p>
    </div>


<p>此任务包括下列步骤：</p>
<ul>
<li><a href="#step1">步骤 1：暂存应用程序</a></li>
<li><a href="#step2">步骤 2：通过交换 VIP 将应用程序部署到生产环境</a></li>
</ul>
<h2><a id="step1"></a>步骤 1：暂存应用程序</h2>
<p>此任务包括如何使用 <strong>Windows Azure PowerShell</strong> 暂存应用程序。</p>
<ol>
<li><p>在发布服务时，只需将 <strong>-Slot</strong> 参数传递给<strong>Publish-AzureServiceProject</strong> cmdlet。</p>
<p><strong>Publish-AzureServiceProject -Slot 过渡</strong></p></li>
<li><p>登录到 <a href="http://manage.windowsazure.cn">Azure 管理门户</a>并选择&ldquo;云服务&rdquo;<strong></strong>。创建云服务并且&ldquo;过渡&rdquo;<strong></strong>列状态已更新为&ldquo;正在运行&rdquo;<strong></strong>后，单击该服务名称。</p>
<p><img src="./media/cloud-services-nodejs-stage-application/staging-cloud-service-running.png" alt="显示正运行服务的门户" /></p></li>
<li><p>选择&ldquo;仪表板&rdquo;<strong></strong>，然后选择&ldquo;过渡&rdquo;<strong></strong>。</p>
<p><img src="./media/cloud-services-nodejs-stage-application/cloud-service-dashboard-staging.png" alt="云服务仪表板" /></p></li>
<li><p>注意右侧&ldquo;网站 URL&rdquo;<strong></strong>条目中的值。DNS 名称是由 Azure 生成的经过模糊处理的内部 ID。</p>
<p><img src="./media/cloud-services-nodejs-stage-application/cloud-service-staging-url.png" alt="网站 url" /></p></li>
</ol>
<p>现在，你可以使用此过渡网站 URL 验证应用程序能否在过渡环境中正常运行。</p>
<p>对于升级方案（在该方案中，暂存应用程序是已部署到生产环境的应用程序的升级版本），你可以<a href="#step2">通过交换 VIP 在生产环境中升级应用程序。</a></p>


<h2><a id="step2"></a>步骤 2：通过交换 VIP 升级生产环境中的应用程序</h2>
<p>在过渡环境中验证应用程序的升级版本后，你可以通过交换过渡和生产环境的虚拟 PI (VIP) 来快速使应用程序可用于生产环境。</p>
<div class="dev-callout">



<div class="dev-callout">
<b>说明</b>
<p>此步骤假定你已将应用程序部署到
生产环境，并且暂存了它的
升级版本。</p>
</div>

1.  登录到 [Azure 管理门户][Azure 管理门户]，单击“云服务”，然后选择服务名称。

2.  从“仪表板”中选择“过渡”，然后单击页面底部的“交换”。将打开“VIP 交换”对话框。

    ![“VIP 交换”对话框][“VIP 交换”对话框]

3.  检查信息，然后单击“确定”。当过渡部署切换到生产部署，而生产部署切换到过渡部署时，两个部署将开始更新。

通过与过渡环境中的部署交换 VIP，你成功地暂存了部署并升级了生产部署。

## 其他资源

-   [如何在 Azure 中通过交换 VIP 来将服务升级部署到生产][如何在 Azure 中通过交换 VIP 来将服务升级部署到生产]
-   [在 Azure 中管理部署概述][在 Azure 中管理部署概述]

  [步骤 1：暂存应用程序]: #step1
  [步骤 2：通过交换 VIP 将应用程序部署到生产环境]: #step2
  [Azure 管理门户]: http://manage.windowsazure.cn
  [显示正运行服务的门户]: ./media/cloud-services-nodejs-stage-application/staging-cloud-service-running.png
  [云服务仪表板]: ./media/cloud-services-nodejs-stage-application/cloud-service-dashboard-staging.png
  [网站 url]: ./media/cloud-services-nodejs-stage-application/cloud-service-staging-url.png
  [“VIP 交换”对话框]: ./media/cloud-services-nodejs-stage-application/vip-swap-dialog.png
  [如何在 Azure 中通过交换 VIP 来将服务升级部署到生产]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee517253.aspx
  [在 Azure 中管理部署概述]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh386336.aspx
