<properties
	pageTitle="Azure 网站、云服务和虚拟机比较"
	description="了解何时使用 Azure 网站、云服务和虚拟机来托管 Web 应用程序。"
	services="app-service\web, virtual-machines, cloud-services"
	documentationCenter=""
	authors="tdykstra"
	manager="wpickett"
	editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.date="08/10/2015"
	wacn.date="10/03/2015"/>

# Azure 网站、云服务和虚拟机比较

## 概述

Azure 提供几种方式托管网站：[Azure 网站][]、[云服务][]和[虚拟机][]。本文可帮助你了解选项，并针对 Web 应用程序做出正确的选择。

Azure 网站是大多数 Web Apps 的最佳选择。部署和管理都已集成到平台，站点可以快速缩放以应对高流量负载，而内置的负载平衡和流量管理器可提供高可用性。可以使用[联机迁移工具](https://www.migratetoazure.net/)轻松地将现有站点转移到 Azure 网站，使用选择的框架和工具创建新站点。利用 [WebJobs][] 功能，可以轻松地将后台作业处理添加到你的应用。

如果你需要加强控制 Web 服务器环境，例如想要远程登录服务器或配置服务器启动任务，Azure 云服务通常是最佳选择。

如果现有应用程序需要进行大幅修改才能在 Azure 网站或 Azure 云服务中运行，你可以选择 Azure 虚拟机来简化迁移到云的操作。不过，相比于 Azure 网站和云服务，正确配置、保护和维护 VM 需要更多的时间和 IT 专业知识。如果你考虑采用 Azure 虚拟机，请确保将修补、更新和管理 VM 环境所需的持续性维护工作纳入考虑。

下图说明了 Azure 上这些 Web 托管选项中每个选项的控制和易于使用的相对程度。

![ChoicesDiagram][ChoicesDiagram]

## 目录

- [方案和建议](#scenarios)
- [功能比较](#features)
- [后续步骤](#nextsteps)

##<a name="scenarios"></a>方案和建议

以下是一些常见的应用程序方案，该方案提供有关哪些 Azure Web 托管选项可能最适合每个方案的建议。

- [我需要具有后台处理的 Web 前端和数据库后端，以运行与本地资产集成的业务应用程序。](#onprem)
- [我需要可靠的方式来托管我的公司网站，既可以进行良好地扩展也能提供全球覆盖。](#corp)
- [我具有在 Windows Server 2003 上运行的 IIS6 应用程序。](#iis6)
- [我是小型企业所有者，并且我需要一种成本较低的方式来托管自己的站点，同时考虑将来的增长。](#smallbusiness)
- [我是 Web 或图形设计师，并且我想为客户设计和构建网站。](#designer)
- [我要将带有 Web 前端的多层应用程序迁移到云中。](#multitier)
- [我的应用程序取决于高度自定义的 Windows 或 Linux 的环境，并且我想将其转移到云中。](#custom)
- [我的站点使用开放源代码软件，并且我希望在 Azure 中托管它。](#oss)
- [我有一个需要连接到公司网络的业务线应用程序。](#lob)
- [我想为移动客户端托管 REST API 或 Web 服务。](#mobile)


### <a id="onprem"></a> 我需要具有后台处理的 Web 前端和数据库后端，以运行与本地资产集成的业务应用程序。

Azure 网站是复杂的业务应用程序的理想解决方案。通过利用该网站，可以开发应用，这些应用可以在负载平衡平台上自动缩放、使用 Active Directory 进行保护并连接到本地资源。使用该网站，可以通过世界级管理门户和 API 轻松地管理这些应用，并且还能够深入了解客户结合使用这些应用与 App Insight 工具的方式。使用新的 [Webjobs][] 功能，能够作为 Web 层的一部分运行后台进程和任务。Azure 网站提供了三个 9's SLA 并使你能够：

* 在自愈性自动修补云平台上可靠地运行应用程序。 
* 跨数据中心的全球网络进行自动缩放。
* 备份和还原，以进行灾难恢复。 
* 遵守 ISO、SOC2 和 PCI 的要求。
* 与 Azure Active Directory 集成

### <a id="corp"></a> 我需要可靠的方式来托管我的公司网站，既可以进行良好地扩展也能提供全球覆盖。 

Azure 网站是用于托管公司网站的理想解决方案。通过该网站，可以使站点快速缩放并轻松地满足整个数据中心的全球网络的需求。它提供了本地范围、容错和智能流量管理。所有内容均位于提供世界级管理工具的平台上，从而可以快速而轻松地更深入了解站点运行状况和站点流量。Azure 网站提供了三个 9's SLA 并使你能够：

* 在自愈性自动修补云平台上可靠地运行网站。 
* 跨数据中心的全球网络进行自动缩放。
* 备份和还原，以进行灾难恢复。 
* 使用集成工具管理日志和流量。 
* 遵守 ISO、SOC2 和 PCI 的要求。
* 与 Azure Active Directory 集成

### <a id="iis6"></a> 我具有在 Windows Server 2003 上运行的 IIS6 应用程序。

利用 Azure 网站，可以很容易地避免与迁移较旧的 IIS6 应用程序相关联的基础结构成本。Microsoft 已经创建 [易于使用的迁移工具和详细的迁移指南](https://www.movemetowebsites.net/)，你可以利用它检查兼容性，并确定需要进行的任何更改。与 Visual Studio、TFS 和常见的 CMS 工具集成，能够更轻松地将 IIS6 应用程序直接部署到云中。部署后，Azure 管理门户提供强大的管理工具，可帮助你减少来管理成本，并根据需要增加来满足要求。使用迁移工具可以：

* 快速而轻松地将旧版 Windows Server 2003 Web 应用程序迁移到云中。
* 选择在本地保留附加的 SQL 数据库，以创建混合应用程序。 
* 自动转移 SQL 数据库与旧的应用程序。 

### <a id="smallbusiness"></a>我是小型企业所有者，并且我需要一种成本较低的方式来托管自己的站点，同时考虑将来的增长。

Azure 网站是这种情况不错的解决方案，因为开始你可以免费使用它，然后在需要时添加更多功能。每个免费网站都附带由 Azure 提供的域 (*your_company*.chinacloudsites.cn)，并且平台包括集成的部署和管理工具。有许多其他服务和扩展选项，允许站点随着用户需求的增加而发展。使用 Azure 网站，你可以：

- 从免费层开始，然后根据需要向上扩展。
- 根据需要向您的应用程序添加其他 Azure 服务和功能。
- 使用 HTTPS 保护 Web 应用程序。

### <a id="designer"></a> 我是 Web 或图形设计师，并且我想为客户设计和构建网站

对于 Web 开发人员和设计师，Azure 网站可以轻松地与各种框架和工具进行集成，其中包括 Git 和 FTP 的部署支持，并提供与工具和服务的紧密集成，如 Visual Studio 和 SQL 数据库。使用网站，你可以：

- 将命令行工具用于[自动化任务][脚本]。
- 使用流行的语言，如 [.Net][dotnet]、[PHP][]、[Node.js][nodejs] 和 [Python][]。
- 选择向上扩展到超高容量的三个不同的扩展级别。
- 与其他 Azure 服务集成，例如 [SQL 数据库][sqldatabase]、[服务总线][servicebus]和[存储][]，或者来自 [Azure 应用商店][azurestore] 的合作伙伴产品，如 MySQL 和 MongoDB。
- 与工具集成，例如 Visual Studio、Git、WebMatrix、WebDeploy、TFS 和 FTP。

### <a id="multitier"></a>我要将带有 Web 前端的多层应用程序迁移到云中

如果运行多层应用程序，如连接到数据库的 Web 服务器，Azure 网站则是一个不错的选择，它提供与 Azure SQL 数据库的紧密集成。并且可以使用 WebJobs 功能运行后端进程。

如果你需要加强控制服务器环境，例如想要远程登录服务器或配置服务器启动任务，则可以为一个或多个层选择云服务。

如果你想要使用你自己的计算机映像或运行服务器软件或不能在云服务中配置的服务，则可以为一个或多个层选择虚拟机。

### <a id="custom"></a>我的应用程序取决于高度自定义的 Windows 或 Linux 的环境，并且我想将其转移到云中。

如果您的应用程序需要软件和操作系统的复杂安装或配置，虚拟机可能是最佳解决方案。通过虚拟机，您可以：

- 使用虚拟机库启动操作系统，如 Windows 或 Linux，然后针对您的应用程序要求对其进行定制。 
- 创建并上传现有的本地服务器的自定义映像，在 Azure 中的虚拟机上运行。 

### <a id="oss"></a>我的站点使用开放源代码软件，并且我希望在 Azure 中托管它

如果网站上支持开放源框架，则会自动为你配置应用程序所需的语言和框架。利用网站，你可以：

- 使用多种流行的开放源代码语言，如 [.NET][dotnet]、[PHP][]、[Node.js][nodejs] 和 [Python][]。 
- 安装 WordPress、Drupal、Umbraco、DNN 和许多其他第三方 web 应用程序。 
- 迁移现有应用程序。 

如果网站上不支持开放源框架，则可以在其他两个 Azure Web 托管选项中的任意一个选项上运行该框架。利用云服务，你可以使用启动任务来安装和配置在 Windows 上运行的任何所需的开放源软件。使用虚拟机，可在机器映像上安装和配置软件，基于 Windows 或 Linux 都可以。

### <a id="lob"></a>我有一个需要连接到公司网络的业务线应用程序

如果你想要创建业务线应用程序，你的网站则可能需要直接访问公司网络上的服务或数据。在网站、云服务和虚拟机上使用 [Azure 虚拟网络服务](/home/features/networking/)是可能的。在网站上，可以使用新的 [VNET 集成功能](http://azure.microsoft.com/blog/2014/09/15/azure-websites-virtual-network-integration/)，通过利用此功能，Azure 应用程序能够如同在你的公司网络上一样运行。

### <a id="mobile"></a>我想为移动客户端托管 REST API 或 web 服务

利用基于 HTTP 的 Web 服务，你可以支持各种客户端，包括移动客户端。如 ASP.NET Web API 的框架与 Visual Studio 集成，能够更加轻松地创建和使用 REST 服务。这些服务来自 web 端点，因此可使用 Azure 上的任何 web 托管技巧支持此情况。但是，网站是托管 REST API 的不错选择。使用网站，你可以：

- 快速创建网站，以在 Azure 全球分布的数据中心之一中托管 HTTP Web 服务。
- 迁移现有的服务或创建新的服务。
- 实现 SLA 的单个实例可用性，或向外扩展到多台专用的计算机。 
- 使用已发布的站点来提供 REST API 到任何 HTTP 客户端，包括移动客户端。

##<a name="features"></a>功能比较

下表比较了网站、云服务和虚拟机的功能，以便帮助你 做出最佳选择。有关每个选项的 SLA 的当前信息，请参阅 [Azure 服务级别协议](/support/legal/sla/)。

<table cellspacing="0" border="1">
<tr>
   <th align="left" valign="middle">功能</th>
   <th align="left" valign="middle">网站</th>
   <th align="left" valign="middle">云服务（web 角色）</th>
   <th align="left" valign="middle">虚拟机</th>
   <th align="left" valign="middle">说明</th>
</tr>
<tr>
   <td valign="middle"><p>接近即时的部署</p></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle"></td>
   <td valign="middle">将应用程序或应用程序更新部署到云服务中或创建 VM，至少需要几分钟；将应用程序部署到网站需要数秒钟。</td>
</tr>
<tr>
   <td valign="middle"><p>向上扩展到更大的计算机且无需重新部署</p></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle"></td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>Web 服务器实例共享内容和配置，这意味着进行扩展时无需重新部署或重新配置。</p></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle"></td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>多个部署环境（生产和过渡）</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>自动操作系统更新管理</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>无缝平台切换（轻松地在 32 位和 64 位之间转移）</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>使用 GIT、FTP 部署代码</p></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>使用 Web 部署来部署代码</p></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle">X</td>
   <td valign="middle">云服务支持使用 Web Deploy 将更新部署到单个角色实例。但是，不能将其用于初始部署角色，并且如果将 Web Deploy 用于更新，则必须单独部署到角色的每个实例。需要提供多个实例，以便为生产环境的云服务 SLA 获得资格。</td>
</tr>
<tr>
   <td valign="middle"><p>WebMatrix 支持</p></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>访问 Service Bus、存储、SQL 数据库之类的服务</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>托管多层体系结构的 web 或 web 服务层</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>托管多层体系结构的中间层</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">网站可以轻松地托管 REST API 中间层，并且网站的 <a href="/documentation/articles/websites-webjobs-resources/">WebJobs</a> 功能可以托管后台处理作业。你可以在专用网站中运行 WebJobs，以实现层的独立可扩展性。</td>
</tr>
<tr>
   <td valign="middle"><p>集成的 MySQL-as-a-service 支持</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">云服务可以通过 ClearDB 的产品集成 MySQL-as-a-service，但不作为管理门户工作流的一部分。</td>
</tr>
<tr>
   <td valign="middle"><p>支持 ASP.NET、经典 ASP、Node.js、PHP、Python</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>向外扩展到多个实例且无需重新部署</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">虚拟机可以扩大到多个实例，但必须编写这些虚拟机上运行的服务，以处理此向外扩展。你需要配置负载平衡器，以跨计算机路由请求，并创建地缘组，以防止因维护或硬件故障导致同时重新启动所有实例。</td>
</tr>
<tr>
   <td valign="middle"><p>支持 SSL</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">对于网站，基本和标准模式仅支持自定义域名称的 SSL。有关结合使用 SSL 和网站的信息，请参阅<a href="/documentation/articles/web-sites-configure-ssl-certificate/">为 Azure 网站配置 SSL 证书</a>。</td>
</tr>
<tr>
   <td valign="middle"><p>Visual Studio 集成</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>远程调试</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>使用 TFS 部署代码</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>支持 <a href="/home/features/traffic-manager/">Azure 流量管理器</a></p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>集成的端点监视</p></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>对服务器的远程桌面访问</p></td>
   <td valign="middle"></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>安装任何自定义 MSI</p></td>
   <td valign="middle"></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>能够定义/执行启动任务</p></td>
   <td valign="middle"></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
<tr>
   <td valign="middle"><p>可以侦听 ETW 事件</p></td>
   <td valign="middle"></td>
   <td valign="middle">X</td>
   <td valign="middle">X</td>
   <td valign="middle"></td>
</tr>
</table>



## <a id="nextsteps"></a> 后续步骤

有关三个 Web 托管选项的详细信息，请参阅以下资源：

* [Azure 简介](/documentation/articles/fundamentals-introduction-to-azure)
* [Azure 执行模型](/documentation/articles/fundamentals-application-models)

若要开始使用为应用程序选择的选项，请参阅以下资源：

* [Azure 网站](/zh-cn/documentation/services/web-sites/)
* [Azure 云服务](/zh-cn/documentation/services/cloud-services/)
* [Azure 虚拟机](/zh-cn/documentation/services/virtual-machines/)

  [ChoicesDiagram]: ./media/choose-web-site-cloud-service-vm/Websites_CloudServices_VMs_3.png
  [Azure  Websites]: /zh-cn/documentation/services/web-sites/
  [云服务]: /zh-cn/documentation/services/cloud-services/
  [虚拟机]: /zh-cn/documentation/services/virtual-machines/
  [ClearDB]: http://www.cleardb.com/
  [WebJobs]: /documentation/articles/websites-webjobs-resources/
  [Configuring an SSL certificate for an Azure  Website]: /documentation/articles/web-sites-configure-ssl-certificate/
  [dotnet]: /develop/net/
  [nodejs]: /develop/nodejs/
  [PHP]: /develop/php/
  [Python]: /develop/python/
  [servicebus]: /zh-cn/documentation/services/service-bus/
  [sqldatabase]: /zh-cn/documentation/services/sql-databases/
  [存储]: /zh-cn/documentation/services/storage/

<!---HONumber=71-->