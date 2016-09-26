<properties
	pageTitle="Azure App Service 及其对现有 Azure 服务的影响"
	description="介绍新的 Azure App Service 及其功能对 Azure 中现有服务的影响。"
	authors="yochayk"
	writer="yochayk"
	editor="yochayk"
	manager="nirma"
	services="app-service"
	documentationCenter=""/>

<tags
	ms.service="app-service"
	ms.date="02/12/2016"
	wacn.date=""/>


# Azure App Service 和现有的 Azure 服务

本文概括介绍了对现有 Azure 服务的更改，这是将多个 Azure 服务组合为 [Azure App Service](/home/features/app-service/)（新的集成产品）更改的一部分。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## 概述

[Azure App Service](/home/features/app-service/) 是一种全新独特的的云服务，使开发人员能够创建适用于任何平台和任何设备的 Web Apps 和 Mobile Apps。Azure App Service 是一个集成的解决方案，旨在简化重复编码函数、与企业和 SaaS 系统集成并自动执行业务流程，同时满足你对安全性、可靠性和可伸缩性的需要。

App Service 将以下现有 Azure 服务 - [网站](/home/features/web-site/)、[移动服务](/home/features/mobile-services/)和 [Biztalk 服务](/home/features/biztalk-services/)融合为单个综合服务，同时添加强大的新功能。App Service 允许托管以下应用类型：

-   Web Apps
-   Mobile Apps
-   API Apps

下表介绍了如何将现有的 Azure 服务映射到 App Service 以及其中可用的应用类型。

<table>
<thead>
<tr class="header">
<th align="left", style="width:10%">现有的 App Service</th>
<th align="left", style="width:10%">Azure App Service</th>
<th align="left", style="width:80%">更改内容</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Azure 网站</td>
<td align="left">Web Apps</td>
<td align="left"><li>对于 Azure 网站，App Service 严格限制为将名称“网站”更改为“Web Apps”。
<p><li>现在，Azure 中的所有现有网站实例均已成为 App Service 中的 Web Apps。</p>
<p><li>你可以通过 <a href="https://manage.windowsazure.cn/">Azure 门户</a>访问现有网站，该门户的<em>“Web Apps”</em>下列出了所有现有站点。</p>
<p><li><em>Web 托管计划</em>现成为 <em>App Service 计划</em>。<em>App Service 计划</em>可以托管任何应用类型的 App Service，例如 Web、Mobile 或 API Apps。</p>
<p><li>Azure App Service Web Apps 现已公开上市。</p>
<p><li><a href="/home/features/web-site/">了解有关 Web Apps 的详细信息</a>。</p></td>
</tr>
<tr class="even">
<td align="left">Azure 移动服务</td>
<td align="left">Mobile Apps</td>
<td align="left"><p><li>移动服务继续用作独立服务，并仍然受完全支持。</p>
<p><li>Mobile Apps 是 App Service 中的一种应用类型，它融合了移动服务和其他服务的所有功能。</p>
<p><li>可以轻松<a href="http://go.microsoft.com/fwlink/?LinkID=724279&clcid=0x409">从移动服务迁移到 Mobile Apps</a>。</p>
<p><li>作为 App Service 的一部分，Mobile Apps 的功能远胜于移动服务，例如，与本地和 SaaS 系统集成，暂存槽、WebJobs、更好的缩放选项，等等。</p>
<p><li><a href="/home/features/app-service/mobile/">了解有关 Mobile Apps 的详细信息</a>。</p>
</tr>
<tr class="odd">
<td align="left"></td>
<td align="left">API Apps</td>
<td align="left">
<p><li>API Apps 是 App Service 中的全新应用类型，它让你能够轻松地构建并使用云中的 API。</p>
<p><li><a href="/home/features/app-service/api/">了解有关 API Apps 的详细信息</a>。</p></td>
</tr>
</tbody>
</table>

要了解详细信息，请访问 [App Service 文档](/documentation/services/app-service/)。

<!---HONumber=Mooncake_0328_2016-->