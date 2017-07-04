<properties
    pageTitle="Azure 计算选项 - 云服务 | Azure"
    description="了解 Azure 计算托管选项及其工作原理：应用服务、云服务和虚拟机"
    services="cloud-services"
    documentationcenter=""
    author="Thraka"
    manager="timlt" />
<tags
    ms.assetid="ed7ad348-6018-41bb-a27d-523accd90305"
    ms.service="multiple"
    ms.workload="multiple"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/11/2016"
    wacn.date="05/22/2017"
    ms.author="adegeo"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="6925ccb55fb5fb1595ab232e57de29bc1a9af6d8"
    ms.lasthandoff="05/12/2017" />

# <a name="should-i-choose-cloud-services-or-something-else"></a>是应选择云服务还是其他服务？
Azure 云服务是适合你的选择吗？ Azure 提供了用于运行应用程序的不同托管模型。 每种模型提供一组不同服务，用户根据具体目的选择模型。

[AZURE.INCLUDE [compute-table](../../includes/compute-options-table.md)]

<a name="tellmecs"></a>

## <a name="tell-me-about-cloud-services"></a>告诉我有关云服务的信息

云服务是平台即服务 (PaaS) 的一个例子。 与[应用服务](/documentation/articles/app-service-web-overview/)一样，这种技术旨在支持可缩放、可靠且运营价格实惠的应用程序。 云服务像应用服务一样托管在 VM 中，但用户对相关 VM 具有更高的控制权。 可在云服务 VM 上安装自己的软件，也可程接入。

![cs_diagram](./media/cloud-services-choose-me/diagram.png) 

更多控制也意味着更不易于使用。 除非需要更多控制选项，否则，与云服务相比，在应用服务的 Web 应用中启动和运行 Web 应用程序通常要快速方便一些。

有两种类型的云服务角色。 两者之间的唯一区别是角色在虚拟机上的托管方式。

* **Web 角色**  
通过 IIS 自动部署和托管应用。

* **辅助角色**  
不要使用 IIS 独立运行应用。

例如，简单的应用程序可能只使用一个 web 角色为网站提供服务。 更复杂的应用程序可能使用一个 Web 角色来处理用户的传入请求，然后将这些请求传递给辅助角色进行处理。 （此通信可能使用[服务总线](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)或 [Azure 队列](/documentation/articles/storage-introduction/)。）

如上图所示，一个应用程序中的所有 VM 都在同一云服务中运行。 用户通过单个公共 IP 地址访问应用程序，而请求会自动在应用程序的 VM 中进行负载均衡。 该平台采用一种能够避免单点硬件故障的方式在云服务应用程序中[缩放和部署](/documentation/articles/cloud-services-how-to-scale/) VM。

即使应用程序在虚拟机中运行，理解云服务提供的是 PaaS 而非 IaaS 也很重要。 以下办法有助于理解这一点：使用 IaaS（如 Azure 虚拟机），首先要创建和配置应用程序将运行的环境，然后将应用程序部署到此环境中。 你要负责该环境的大部分管理工作，如在每个 VM 中部署操作系统的新修补版本。 相反，在 PaaS 中，这样的环境似乎早已存在。 只需部署应用程序即可。 已为你处理它所运行的平台的管理工作，包括部署操作系统的新版本。

## <a name="scaling-and-management"></a>缩放和管理
使用云服务，无需创建虚拟机。 相反，提供一个配置文件，告知 Azure 每个 VM 需要多少个角色实例（如**三个 Web 角色实例**和**两个辅助角色实例**），平台将创建这些实例。  虽然仍然要选择这些后备 VM 应该是[什么大小](/documentation/articles/cloud-services-sizes-specs/)，但是无需亲自显式创建它们。 如果应用程序需要处理更大的负载，则可以要求增加 VM，Azure 将创建这些实例。 如果负载降低，则可以关闭这些实例并停止为它们付费。

通常通过两个步骤就能将云服务应用程序提供给用户。 首先，开发人员[将应用程序上传](/documentation/articles/cloud-services-how-to-create-deploy/)到该平台的暂存区域。 当开发人员准备好使应用程序上线后，他们会使用 Azure 门户从过渡环境切换到生产环境。 [过渡环境与生产环境之间的这种切换](/documentation/articles/cloud-services-nodejs-stage-application/)无需停机就可完成，这使运行的应用程序可在不打扰其用户的情况下升级到新版本。

## <a name="monitoring"></a>监视
云服务还提供监视功能。 像 Azure 虚拟机一样，它将检测到发生故障的物理服务器，并在新的计算机上重新启动原先在该服务器上运行的 VM。 云服务不仅检测硬件故障，还检测发生故障的 VM 和应用程序。 与虚拟机不同，它在每个 Web 角色和辅助角色中都存在代理，因此可在发生故障时启动新的 VM 和应用程序实例。

云服务的 PaaS 特性还具有其他含义。最重要的含义之一是，应以这样的方式编写基于此技术的应用程序，使其在任何 Web 角色或辅助角色实例出现故障时都能正确运行。要实现这一目标，云服务应用程序不应应在其 VM 的文件系统中维持状态。与使用 Azure 虚拟机创建的 VM 不同，对云服务 VM 所做的写入不是持久的；这与虚拟机数据磁盘完全不同。相反，云服务应用程序应将所有状态明确写入 SQL 数据库、Blob、表或其他某种外部存储。以这种方式构建应用程序会使它们更易于扩展、抵抗故障的能力更强，这是云服务的两个重要目标。

## <a name="next-steps"></a>后续步骤
[使用 .NET 创建云服务应用](/documentation/articles/cloud-services-dotnet-get-started/)  
[使用 Node.js 创建云服务应用](/documentation/articles/cloud-services-nodejs-develop-deploy-app/)  
[使用 PHP 创建云服务应用](/documentation/articles/cloud-services-php-create-web-role/)  
[使用 Python 创建云服务应用](/documentation/articles/cloud-services-python-ptvs/)