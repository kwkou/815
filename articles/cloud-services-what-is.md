<properties linkid="manage-services-what-is-a-cloud-service" urlDisplayName="What is a Cloud Service" pageTitle="什么是云服务 - Azure 服务管理" metaKeywords="Azure cloud services intro, cloud services overview, cloud services basics" description="介绍 Azure 中的云服务。" metaCanonical="" services="cloud-services" documentationCenter="" title="What is a cloud service?" authors="ryanwi" solutions="" manager="" editor="" />




#什么是云服务？
当你创建应用程序以及在 Azure 中运行该应用程序时，相关代码和配置统称为 Azure 云服务（在早期版本的 Azure 中称为"托管服务"）。

通过创建云服务，你可以在 Azure 中部署多层 Web 应用程序，以便定义用于处理分发以及允许灵活扩展你的应用程序的多个角色。云服务由一个或多个 Web 角色和/或辅助角色组成，其中每个角色都具有其自己的应用程序文件和配置。Azure 网站和虚拟机还可在 Azure 上启用 Web 应用程序。云服务的主要优势在于能够支持更复杂的多层体系结构。有关详细比较，请参阅[Azure 网站、云服务和虚拟机的比较][Comparison]。

Azure 可为你维护云服务基础结构，以便执行日常维护、修补操纵系统以及尝试从服务和硬件故障中恢复。如果你为每个角色至少定义了两个实例，则可在不中断服务的情况下，完成大多数维护和服务升级任务。云服务的每个角色必须至少具有两个实例才能符合 Azure 服务级别协议，从而确保你的面向 Internet 的角色至少在 99.95% 的时间内能够建立外部连接。 

每个云服务都具有两个环境，你可以将服务包和配置部署到这两个环境。在将云服务升级到生产环境之前，你可以将其部署到过渡环境以对其进行测试。将暂存的云服务升级到生产环境的过程较为简单，只需交换与这两个环境关联的虚拟 IP 地址 (VIP)。 


## 概念##


- **云服务角色：**云服务角色由应用程序文件和配置组成。一个云服务可以有两种类型的角色：
 
>- **Web 角色：**Web 角色提供专门用于托管前端 Web 应用程序的 Internet Information Services (IIS) Web 服务器。

>- **辅助角色：**辅助角色中承载的应用程序可运行独立于用户交互或输入的异步任务、运行时间较长的任务或永久性任务。

- **角色实例：**角色实例是可在其上运行应用程序代码和角色配置的虚拟机。一个角色可以包含服务配置文件中定义的多个实例。

- **来宾操作系统：**云服务的来宾操作系统是安装在可在其上运行应用程序代码的角色实例（虚拟机）上的操作系统。

- **云服务组件：**要将应用程序部署为 Azure 中的云服务，需要以下三个组件：

>- **服务定义文件：**云服务定义文件 (.csdef) 可定义服务模型，包括角色数量。

>- **服务配置文件：**云服务配置文件 (.cscfg) 提供云服务和各个角色的配置设置，包括角色实例的数量。

>- **服务包：**服务包 (.cspkg) 包含应用程序代码和服务定义文件。

- **云服务部署：**云服务部署是部署在 Azure 过渡或生产环境中的云服务实例。你可以在过渡和生产环境中维护部署。

- **部署环境：**Azure 为云服务提供两种部署环境：可以先在"过渡环境"中测试你的部署，然后将其提升到"生产环境"。这两种环境的唯一区别在于访问云服务时使用的虚拟 IP 地址 (VIP)。在过渡环境中，云服务的全局唯一标识符 (GUID) 将在 URL 中标识自身 (*GUID*.chinacloudapp.cn)。在生产环境中，URL 基于分配给云服务的更友好 DNS 前缀（例如 *myservice*.chinacloudapp.cn）。

- **交换部署：**若要将 Azure 过渡环境中的部署提升到生产环境，可以通过交换访问两个部署时使用的 VIP 来"交换"部署。部署后，云服务的 DNS 名称指向过渡环境中的部署。

- **最少监视与详细监视：**按默认为云服务配置的"最少监视"使用从角色实例（虚拟机）的主机操作系统收集的性能计数器。"详细监视"基于角色实例中的性能数据收集更多度量值，以便对处理应用程序期间发生的问题进行更密切的分析。有关详细信息，请参阅[如何监视云服务][HTMonitorCloudServices]。

- **Azure Diagnostics：**Azure Diagnostics 是可让你从 Azure 中运行的应用程序收集诊断数据的 API。若要开启详细监视，必须为云服务角色启用 Azure Diagnostics。有关详细信息，请参阅[在 Azure 中启用 Diagnostics][CloudServicesDiagnostics]。

- **链接资源：**若要显示云服务对其他资源（例如 Azure SQL Database）的依赖性，可将资源链接到云服务。在预览版管理门户中，可以在"链接的资源"****页上查看链接的资源，在仪表板上查看其状态，并在"缩放"****页上缩放已链接的 SQL Database 实例以及服务角色。在这种意义上，链接资源不会将该资源连接到应用程序，你必须在应用程序代码中配置连接。

- **缩放云服务：**可通过增加为角色部署的角色实例（虚拟机）数量来向外缩放云服务。可通过减少角色实例来向内缩放云服务。在预览版管理门户中，你还可以在缩放服务角色时，通过更改 SQL Database 版本和最大数据库大小，来缩放链接的 SQL Database 实例。

- **Azure 服务级别协议 (SLA)：**Azure 计算 SLA 保证你在为每个角色部署两个或更多个实例后，至少有 99.95% 的时间可以访问你的云服务。此外，当某个角色实例的进程未运行时，有 99.9% 的时间会启动检测和纠正措施。有关详细信息，请参阅[服务级别协议] [SLA]。

[HTMonitorCloudServices]:/zh-cn/documentation/articles/cloud-services-how-to-monitor/
[SLA]: /support/legal/sla/
[CloudServicesDiagnostics]: /zh-cn/documentation/articles/cloud-services-dotnet-diagnostics/
[Comparison]: /zh-cn/documentation/articles/choose-web-site-cloud-service-vm/
<!--HONumber=39-->
