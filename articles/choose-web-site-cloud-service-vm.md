<properties linkid="manage-scenarios-choose-web-app-service" urlDisplayName="Web Options for Azure" pageTitle="Azure 网站、云服务和虚拟机比较" metaKeywords="Cloud Services, Virtual Machines, Web Sites" description="了解何时使用 Azure 网站、云服务和虚拟机来托管 Web 应用程序。查看功能比较。" metaCanonical="" services="web-sites,virtual-machines,cloud-services" documentationCenter="" title=" 云服务" authors="jroth" solutions="" manager="paulettm" editor="mollybos" />

# Azure 网站、云服务和虚拟机比较

Azure 提供几种方式托管 web 应用程序，如 [Azure 网站][Azure 网站]、[云服务][云服务]和[虚拟机][虚拟机]。查看这些不同的选项后，您可能不确定哪一种最适合需求，或可能有不清楚的概念，如 IaaS 对比 PaaS。这篇文章可帮助您了解选项，并帮助您为自己 web 方案做出正确的选择。尽管所有三个选项允许您在 Azure 中运行高度可扩展的 web 应用程序，但差异可帮助指导您的决定。

在许多情况，Azure 网站是最佳的选择。它为部署和管理提供简单灵活的选项，并且能够在托管高容量的网站。使用流行的软件如 WordPress，可从 Web 应用程序库快速创建新的网站，或可以将现有的网站移动到 Azure 网站。使用[Azure WebJobs SDK][Azure WebJobs SDK]（目前处于预览状态），还可以添加后台作业处理。

还有在 Azure 云服务或 Azure 虚拟机上托管 web 应用程序的选项。这些选项是很好的选择，您的 web 层需要它们提供更多额外级别的控制和定制；但是，此增强的控制要以应用程序创建、管理和部署的复杂性增加为代价。下图说明了三个选项之间的权衡。

![ChoicesDiagram][ChoicesDiagram]

网站是更易于设置、管理和监视，但配置选项较少。关键之处在于使用 Azure 网站时，没有让您信服的理由将 web 前端放在云服务或虚拟机上。本文档其余部分提供做出明智决策所需的信息。这包括：

-   [方案][方案]
-   [服务摘要][服务摘要]
-   [功能比较][功能比较]

## <a name="scenarios"></a>方案

### 我是小型企业所有者，并且我需要一种成本较低的方式来托管自己的站点，同时考虑将来的增长。

Azure 网站 (WAWS） 是这种情况不错的解决方案，因为您可以开始免费使用它，然后在需要时添加更多功能。例如，每个免费的网站附带由 Azure 提供的域 (*your\_company*。 azurewebsites.net)。如果已经准备好开始使用自己的域，添加的费用低至低至 $9.80 每月（截止到 1/2014)。有许多其他服务和扩展选项，允许站点随着用户需求的增加而发展。使用 **Azure 网站**，您可以：

-   从免费层开始，然后根据需要向上扩展。
-   使用应用程序库快速设置流行的 web 应用程序，如 WordPress。
-   根据需要向您的应用程序添加其他 Azure 服务和功能。
-   通过使用 *your\_company*.azurewebsites.net 域名提供的证书，用 HTTPS 保护您的网站。

### 我是 web 或图形设计师，并且我想为客户设计和生成网站

对于 web 开发人员，Azure 网站为您提供创建复杂的 web 应用程序所需的内容。网站提供 Visual Studio 和 SQL 数据库等工具的紧密集成。使用**网站**，开发人员可以：

-   使用[自动任务][自动任务]的命令行工具。
-   使用流行的语言，如 [.Net][.Net]、[PHP][PHP]、[Node.js][Node.js] 和 [Python][Python]。
-   选择向上扩展到超高容量的三个不同的扩展级别。
-   与其他 Azure 服务集成，例如 [SQL 数据库][SQL 数据库]、[服务总线][服务总线]和[存储][存储]，或者来自 [Azure 应用商店][Azure 应用商店]的合作伙伴产品，如 MySQL 和 MongoDB。
-   与工具集成，例如 Visual Studio、Git、WebMatrix、WebDeploy、TFS 和 FTP。

### 我要将带有 web 前端的多层应用程序迁移到云

如果您运行多层应用程序，如与数据库服务器对话以存储和检索网站数据的 web 服务器，则您在 Azure 中有几个选项。这些体系结构选项包括网站、云服务和虚拟机。首先，**网站**对于您的解决方案是不错的选择，并可配合 Azure SQL 数据使用以创建一个两层体系结构。网站还允许您使用 Azure WebJobs SDK 预览运行后台或长时间运行进程。如果您需要更复杂的体系结构或更灵活的扩展选项，云服务或虚拟机是更好的选择。

**云服务**使您能够：

-   在可扩展的 web 和辅助角色上托管 web、中间层和后端服务。
-   仅在辅助角色上托管中间层和后端服务，保持前端在 Azure 网站上。
-   独立扩展前端和后端服务。

**虚拟机**使您能够：

-   更轻松地迁移高度定制的环境，如一个虚拟机映像。
-   运行不能在网站或云服务配置的软件或服务。

### 我的应用程序取决于高度定制的 Windows 或 Linux 的环境

如果您的应用程序需要软件和操作系统的复杂安装或配置，虚拟机可能是最佳解决方案。通过**虚拟机**，您可以：

-   使用虚拟机库启动操作系统，如 Windows 或 Linux，然后针对您的应用程序要求对其进行定制。
-   创建并上传现有的本地服务器的自定义映像，在 Azure 中的虚拟机上运行。

### 我的站点使用开放源代码软件，并且我希望在 Azure 中托管它

所有三个选项都允许您托管开放源代码语言和框架。**云服务**要求您使用启动任务，来安装和配置在 Windows 上运行的任何所需的开放源代码软件。使用**虚拟机**，可在机器映像上安装和配置软件，基于 Windows 或 Linux 都可以。如果网站上支持开放源代码框架，这提供更简单的方法来托管这些类型的应用程序，因为使用应用程序所需的语言和框架可以自动配置“网站”。**网站**使您能够：

-   使用多种流行的开放源代码语言，如[.NET][.Net]、[PHP][PHP]、[Node.js][Node.js] 和 [Python][Python]。
-   安装 WordPress、Drupal、Umbraco、DNN 和许多其他第三方 web 应用程序。
-   迁移现有应用程序或从应用程序库创建一个新的。

### 我有一个业务应用程序需要连接到公司网络

如果您想要创建业务应用程序，您的网站可能需要直接访问公司网络上的服务或数据。可能在**网站**、**云服务**和**虚拟机**上。您采用的方法中存在差异，其中包括：

-   网站可通过使用服务总线中继安全地连接到本地资源。这允许公司网络上的服务代表该 web 应用程序执行任务，无需将所有内容移到云中或建立一个虚拟网络。
-   云服务和虚拟机可以充分利用虚拟网络。实际上，虚拟网络允许 Azure 中运行的机器连接到本地网络。然后 Azure 将成为您公司数据中心的扩展。

### 我想为移动客户端托管 REST API 或 web 服务

基于 HTTP 的 web 服务允许您支持各种客户端，包括移动客户端。如 ASP.NET Web API 的框架与 Visual Studio 集成，能够更加轻松地创建和使用 REST 服务。这些服务来自 web 端点，因此可使用 Azure 上的任何 web 托管技巧支持此情况。但是，**网站**是托管 REST API 的不错选择。使用“站点”，您可以：

-   快速创建网站以便在 Azure 全球分布的数据中心之一托管 HTTP web 服务。
-   迁移现有的服务或创建新的，可能会利用 Visual Studio 中的 ASP.NET Web API。
-   实现 SLA 的单个实例可用性，或向外扩展到多台专用的计算机。
-   使用已发布的站点来提供 REST API 到任何 HTTP 客户端，包括移动客户端。

## <a name="services"></a>服务摘要

[Azure 网站][Azure 网站]允许您在 Azure 上快速生成高度可扩展的网站。您可以使用 Azure 门户或命令行工具，通过流行的语言（例如 .NET、PHP 和 Python）设置网站。支持的框架已部署并且不要求更多安装步骤。Azure 网站库包含许多第三方应用程序（例如 Drupal 和 WordPress）以及开发框架（例如 Django 和 CakePHP）。在创建网站后，您可以迁移现有网站或者生成全新网站。Web 站点无需管理物理硬件，还提供了几个扩展选项。您可以从共享的多租户模型移动到标准模式，那里的专用机服务于传入流量。网站还允许您与其他 Azure 服务（例如 SQL Database、Service Bus 和存储）相集成。使用 [Azure WebJobs SDK][Azure WebJobs SDK] 预览，您可以添加后台处理。总之，Azure 网站通过支持多种语言、开放源应用程序和部署方法（FTP、Git、Web Deploy 或 TFS），从而更便于关注应用程序开发。如果您没有要求云服务或虚拟机的专门要求，则 Azure 网站非常有可能是您的最佳选择。

通过[云服务][云服务]，您能够在丰富的平台即服务 (PaaS) 环境中创建高度可用的、可扩展 Web 应用程序。与网站不同，云服务首先在 Visual Studio 之类的开发环境中创建，然后部署到 Azure。PHP 之类的框架要求在角色启动时安装框架的自定义部署步骤或任务。云服务的主要优势在于能够支持更复杂的多层体系结构。单个云服务可由一个前端 Web 角色以及一个或多个辅助角色构成。每个层都可以单独缩放。还有对您的 Web 应用程序体系结构的提高的控制级别。例如，您可以使用远程桌面控制正在运行角色实例的计算机。您还可以编写在角色启动时运行的更高级的 IIS 和计算机配置更改的脚本，包括要求管理员控制的任务。

[虚拟机][虚拟机]，您能够在 Azure 中的虚拟机上运行 Web 应用程序。这种功能也称为“基础结构即服务 (IaaS)”。通过门户创建新的 Windows Server 或 Linux 计算机，或者上载现有的虚拟机映像。虚拟机向您提供对操作系统、配置以及已安装软件和服务的最高的控制。这个选项很适合于快速将复杂的本地 Web 应用程序迁移到云，因为计算机可作为整体进行迁移。使用虚拟网络，您还可以将这些虚拟机连接到本地公司网络。与云服务一样，您能够远程访问这些计算机并且能够在管理级别执行配置更改。但与网站和云服务不同，您必须完全在基础结构级别管理虚拟机映像和应用程序体系结构。一个基本示例是您需要将自己的修补程序应用到操作系统。

## <a name="features"></a>功能比较

下表比较网站、云服务和虚拟机的功能，以便帮助您进行最佳选择。表格后面的注释中详细解释带有星号的框。

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">功能</th>
<th align="left">网站</th>
<th align="left">云服务（web 角色）</th>
<th align="left">虚拟机</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left"><p>访问 Service Bus、存储、SQL Database 之类的服务</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>托管多层体系结构的 web 或 web 服务层</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="odd">
<td align="left"><p>托管多层体系结构的中间层</p></td>
<td align="left"></td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>集成的 MySQL-as-a-service 支持</p></td>
<td align="left">X</td>
<td align="left">X <sup>1</sup></td>
<td align="left">X</td>
</tr>
<tr class="odd">
<td align="left"><p>支持 ASP.NET、经典 ASP、Node.js、PHP、Python</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>向外扩展到多个实例且无需重新部署</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X <sup>2</sup></td>
</tr>
<tr class="odd">
<td align="left"><p>支持 SSL</p></td>
<td align="left">X <sup>3</sup></td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>Visual Studio 集成</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="odd">
<td align="left"><p>远程调试</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>使用 TFS 部署代码</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="odd">
<td align="left"><p>使用 GIT、FTP 部署代码</p></td>
<td align="left">X</td>
<td align="left"></td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>使用 Web 部署来部署代码</p></td>
<td align="left">X</td>
<td align="left"><sup>4</sup></td>
<td align="left">X</td>
</tr>
<tr class="odd">
<td align="left"><p>WebMatrix 支持</p></td>
<td align="left">X</td>
<td align="left"></td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>接近即时的部署</p></td>
<td align="left">X</td>
<td align="left"></td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left"><p>实例共享内容和配置</p></td>
<td align="left">X</td>
<td align="left"></td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left"><p>向上扩展到更大的计算机且无需重新部署</p></td>
<td align="left">X</td>
<td align="left"></td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left"><p>多个部署环境（生产和过渡）</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left"><p>使用 Azure 虚拟网络进行网络隔离</p></td>
<td align="left"></td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="odd">
<td align="left"><p>支持 Azure Traffic Manager</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>对服务器的远程桌面访问</p></td>
<td align="left"></td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="odd">
<td align="left"><p>能够定义/执行启动任务</p></td>
<td align="left"></td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>自动操作系统更新管理</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left"><p>集成的端点监视</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left">X</td>
</tr>
<tr class="even">
<td align="left"><p>无缝平台切换（32 位/64 位）</p></td>
<td align="left">X</td>
<td align="left">X</td>
<td align="left"></td>
</tr>
</tbody>
</table>

<sup>1</sup> Web 和辅助角色可以通过 ClearDB 的产品集成 MySQL-as-a-service，但不作为管理门户工作流的一部分。

<sup>2</sup>尽管虚拟机可以向外扩展到多个实例，在必须编写这些计算机上运行的服务以处理此向外扩展。必须配置额外的负载平衡器以跨机器路由请求。最后，应为参与同一角色的所有计算机创建一个地缘组，防止它们从维护或硬件故障同时重启。

<sup>3</sup>对于“网站”，自定义域名的 SSL 仅支持标准模式。有关将 SSL 用于网站的更多信息，请参见[为 Azure 网站配置 SSL 证书][为 Azure 网站配置 SSL 证书]。

<sup>4</sup> 在部署到单实例角色时云服务支持 Web 部署。但是，生产角色要求多个实例以便满足 Azure SLA。因此，Web 部署不是在生产中用于云服务的合适的部署机制。

  [Azure 网站]: http://azure.microsoft.com/zh-cn/
  [云服务]: http://azure.microsoft.com/zh-cn/documentation/services/cloud-services/
  [虚拟机]: http://azure.microsoft.com/zh-cn/documentation/services/virtual-machines/
  [Azure WebJobs SDK]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs
  [ChoicesDiagram]: ./media/choose-web-site-cloud-service-vm/Websites_CloudServices_VMs_2.png
  [方案]: #scenarios
  [服务摘要]: #services
  [功能比较]: #features
  [自动任务]: http://azure.microsoft.com/zh-cn/documentation/scripts/?services=web-sites
  [.Net]: http://azure.microsoft.com/zh-cn/develop/net/
  [PHP]: http://azure.microsoft.com/zh-cn/develop/php/
  [Node.js]: http://azure.microsoft.com/zh-cn/develop/nodejs/
  [Python]: http://azure.microsoft.com/zh-cn/develop/python/
  [SQL 数据库]: http://azure.microsoft.com/zh-cn/documentation/services/sql-database/
  [服务总线]: http://azure.microsoft.com/zh-cn/documentation/services/service-bus/
  [存储]: http://azure.microsoft.com/zh-cn/documentation/services/storage/
  [Azure 应用商店]: http://azure.microsoft.com/zh-cn/gallery/store/
  [为 Azure 网站配置 SSL 证书]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-configure-ssl-certificate/
