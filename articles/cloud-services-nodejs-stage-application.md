<properties linkid="dev-nodejs-enablestaging" urlDisplayName="Staging Deployment" pageTitle="暂存云服务部署 (Node.js) - Azure" metaKeywords="Azure staging, Azure application staging, Azure test environment, Azure staging environment, Azure Virtual IP swap, Azure VIP swap" description="了解如何使用虚拟 IP (VIP) 交换将 Azure 应用程序部署到过渡环境，然后再将其部署到生产环境。" metaCanonical=" " services="cloud-services" documentationCenter="Node.js" title="Staging an Application in Azure" authors="larryfr" solutions="" manager="" editor="" />







# 在 Azure 中暂存应用程序

可以先将要测试的已打包应用程序部署到 Azure 中的过渡环境，然后再将该应用程序移动到用户可通过 Internet 对其进行访问的生产环境。过渡环境与生产环境完全一样，只不过访问暂存应用程序时必须使用由 Azure 生成的经过模糊处理的 URL。在验证你的应用程序能够正常运行后，可以通过执行虚拟 IP (VIP) 交换将其部署到生产环境。

<div class="dev-callout">
	<b>说明</b>
	<p>本文中的步骤仅适用于托管为 Azure 云服务的 Node 应用程序。</p>
	</div>

此任务包括下列步骤：

-   [步骤 1：暂存应用程序]
-   [步骤 2：通过交换 VIP 将应用程序部署到生产环境]

<h2><a id="step1"></a>步骤 1：暂存应用程序</h2>

此任务包括如何使用 **Microsoft
Azure PowerShell** 暂存应用程序。

1.  在发布服务时，只需将 **-Slot** 参数传递给 **Publish-AzureServiceProject** cmdlet。

    **Publish-AzureServiceProject -Slot staging**

2.  登录到 [Azure 管理门户]并选择"云服务"****。创建云服务并且"过渡"列状态已更新为"正在运行"后，单击该服务名称********。

	![portal displaying a running service][cloud-service]

3.  选择"仪表板"，然后选择"过渡"********。

	![cloud service dashboard][cloud-service-dashboard]

4. 注意右侧"网站 URL"条目中的值****。DNS 名称是由 Azure 生成的经过模糊处理的内部 ID。

    ![site url][cloud-service-staging-url]

现在，你可以使用此过渡网站 URL 验证应用程序能否在过渡环境中正常运行。

对于升级方案（在该方案中，暂存应用程序是已部署到生产环境的应用程序的升级版本），你可以 [通过交换 VIP 在生产环境中升级应用程序][Step 2: Deploy an Application to Production by Swapping VIPs]。

<h2><a id="step2"></a>步骤 2：通过交换 VIP 升级生产环境中的应用程序</h2>

在过渡环境中验证应用程序的升级版本后，您可以通过交换过渡和生产环境的虚拟 PI (VIP) 来快速使应用程序可用于生产环境。

<div class="dev-callout">
<b>说明</b>
<p>此步骤假定您已将应用程序部署到生产环境，并且暂存了它的升级版本。</p>
</div>

1.  登录到 [Azure 管理门户]，单击"云服务"，然后选择服务名称****。

2.  从"仪表板"中选择"过渡"，然后单击页面底部的"交换"************。将打开"VIP 交换"对话框。

    ![vip swap dialog][vip-swap-dialog]

3.  检查信息，然后单击"确定"****。当过渡部署切换到生产部署，而生产部署切换到过渡部署时，两个部署将开始更新。

通过与过渡环境中的部署交换 VIP，您成功地暂存了部署并升级了生产部署。

## 其他资源

- [如何在 Azure 中通过交换 VIP 来将服务升级部署到生产]
- [在 Azure 中管理部署概述]

  [步骤 1：暂存应用程序]: #step1
  [步骤 2：通过交换 VIP 将应用程序部署到生产环境]: #step2
  [Azure 管理门户]: http://manage.windowsazure.cn
[cloud-service]: ./media/cloud-services-nodejs-stage-application/staging-cloud-service-running.png
[cloud-service-dashboard]: ./media/cloud-services-nodejs-stage-application/cloud-service-dashboard-staging.png
  [cloud-service-staging-url]: ./media/cloud-services-nodejs-stage-application/cloud-service-staging-url.png
  [vip-swap-dialog]: ./media/cloud-services-nodejs-stage-application/vip-swap-dialog.png
  [如何在 Azure 中通过交换 VIP 来将服务升级部署到生产]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee517253.aspx
  [在 Azure 中管理部署概述]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh386336.aspx
<!--HONumber=39-->
