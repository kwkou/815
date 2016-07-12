<properties 
	pageTitle="Azure 提供的计算托管选项" 
	description="了解 Azure 计算托管选项及其工作原理：虚拟机、 Web 应用、云服务，等等。" 
	headerExpose="" 
	footerExpose="" 
	services="cloud-services,virtual-machines"
	authors="Thraka" 
	documentationCenter=""
	manager="timlt"/>

<tags 
	ms.service="multiple" 
	ms.date="09/08/2015" 
	wacn.date="01/21/2016"/>




# Azure 提供的计算托管选项

Azure 提供了用于运行应用程序的不同托管模型。每种模型提供一组不同服务，而你选择哪种模型完全取决于你要做什么。本文将介绍不同的选项，描述它们的技术并提供相关用例。

| 计算选项 | 目标受众 |
| ------------------ | --------   |
| [应用程序服务] | 任何设备的可缩放 Web 应用、Mobile Apps、API Apps 和 Logic Apps |
| [云服务] | 对操作系统具有更高控制度的高可用、可缩放 n 层云应用程序 |
| [虚拟机] | 可完全控制操作系统的自定义 Windows 和 Linux VM |

[AZURE.INCLUDE [内容](../includes/app-service-choose-me-content.md)]

[AZURE.INCLUDE [内容](../includes/cloud-services-choose-me-content.md)]

[AZURE.INCLUDE [内容](../includes/virtual-machines-choose-me-content.md)]

## 其他选项

Azure 还针对更特殊的用途提供其他计算托管模型，例如：

* [移动服务](/services/mobile-services/)  
  适用于移动设备上运行的应用的云后端优化模型。
* [Batch](/services/batch/)  
  适用于处理大量类似任务的优化模型，特别适用于本身在多台计算机上以并行任务形式运行的工作负荷。
* [HDInsight (Hadoop)](/services/hdinsight/)  
  适用于在 Hadoop 群集上运行 [MapReduce](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/data-storage-options/#hadoop) 作业的优化模型。 

## 我该使用哪一种？ 做出选择

所有三种通用型 Azure 计算托管模型都可让你在云中构建可缩放、可靠的应用程序。既然在本质上是类似的，你应该使用哪种模型呢？

App Service 是大多数 Web 应用的最佳选择。部署和管理都已集成到平台，站点可以快速缩放以应对高流量负载，而内置的负载平衡和流量管理器可提供高可用性。可以使用[联机迁移工具](https://www.migratetoazure.net/)轻松将现有站点转移到 Azure App Service，使用 Web 应用库中的开放源代码应用，或使用选择的框架和工具创建新站点。[Web 作业](/documentation/articles/websites-webjobs-resources/)功能可让你轻松为应用添加后台作业处理，甚至还能运行根本不是 Web 应用的计算工作负荷。

如果你需要加强控制 Web 服务器环境，例如想要远程登录服务器或配置服务器启动任务，Azure 云服务通常是最佳选择。

如果现有应用程序需要进行大幅修改才能在 Azure Web 应用或 Azure 云服务中运行，你可以选择 Azure 虚拟机来简化迁移到云的操作。不过，相比于 Azure Web 应用和云服务，正确配置、保护和维护 VM 需要更多的时间和 IT 专业知识。如果你考虑采用 Azure 虚拟机，请确保将修补、更新和管理 VM 环境所需的持续性维护工作纳入考虑。

有时候，任何单个选项都不是正确的选项。在这样的情况下，组合使用模型较为合适。例如，假设你要构建一个应用程序，既想利用云服务 Web 角色的管理优势，又需要使用虚拟机中托管的标准 SQL Server 提高兼容性或性能。

<!-- In this case, the best option is to combine compute hosting options, as the figure below shows.--

<a name="fig4"></a>
![07_CombineTechnologies][07_CombineTechnologies] 
 
**Figure: A single application can use multiple hosting options.**

As the figure illustrates, the Cloud Services VMs run in a separate cloud service from the Virtual Machines VMs. Still, the two can communicate quite efficiently, so building an app this way is sometimes the best choice.
[07_CombineTechnologies]: ./media/fundamentals-application-models/ExecModels_07_CombineTechnologies.png
!-->

[应用程序服务]: #tellmeas
[虚拟机]: #tellmevm
[云服务]: #tellmecs

## 后续步骤

* [比较](/documentation/articles/choose-web-site-cloud-service-vm/) App Service、云服务和虚拟机
* 了解有关 [App Service](/documentation/articles/app-service-web-overview/) 的详细信息
* 了解有关[云服务](/services/cloud-services/)的详细信息
* 了解有关[虚拟机](https://msdn.microsoft.com/zh-cn/library/azure/jj156143.aspx)的详细信息 

<!---HONumber=74-->