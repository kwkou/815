<properties
    pageTitle="Azure 应用服务计划深入概述 | Azure"
    description="了解针对 Azure 应用服务的应用服务计划的工作原理，以及如何利用它们进行管理。"
    keywords="应用服务, azure 应用服务, 缩放, 可缩放, 应用服务计划, 应用服务成本"
    services="app-service"
    documentationcenter=""
    author="btardif"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="dea3f41e-cf35-481b-a6bc-33d7fc9d01b1"
    ms.service="app-service"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/02/2016"
    wacn.date="05/31/2017"
    ms.author="byvinyal"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="af9e05d9e78edc06c423843b7408279c529f4645"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="azure-app-service-plans-in-depth-overview"></a>Azure 应用服务计划深入概述
应用服务计划代表用于托管应用的物理资源的集合。

应用服务计划定义：

- 区域（中国北部、中国东部）
- 规模计数（一个、两个、三个实例等）
- 实例大小（小、中、大）
- SKU（免费、共享、基本、标准、高级）

[Azure 应用服务](/documentation/articles/app-service-changes-existing-services/)中的 Web 应用、移动应用、API 应用均在应用服务计划中运行。  同一订阅、区域和资源组中的应用可共享应用服务计划。 

分配到**应用服务计划**的所有应用程序将共享该计划定义的资源。 在单个应用服务计划中托管多个应用时，此共享操作可节省成本。

**应用服务计划**可以从**免费**和**共享** SKU 扩展到**基本**、**标准**和 **高级** SKU，让用户随着不断的发展访问更多的资源和功能。 

如果应用服务计划设置为**基本** SKU 或更高版本，则可控制 VM 的**大小**和规模计数。

例如，如果计划配置为使用标准服务层的两个“小型”实例，则与该计划关联的所有应用都会在这两个实例上运行。 应用还可访问标准服务层功能。 运行应用的计划实例是完全托管的，且可用性高。 

> [AZURE.IMPORTANT]
> 应用服务计划的 **SKU** 和**规模**确定成本，而不是托管的应用数目。

本文介绍应用服务计划的主要特征（例如层和缩放），以及在管理应用时这些特征如何起作用。

## <a name="apps-and-app-service-plans"></a>应用和应用服务计划
应用服务中的应用在任何特定的时刻只能与一个应用服务计划相关联。

应用和计划都包含在**资源组**中。 资源组充当该组中每个资源的生命周期界限。 可以使用资源组来同时管理应用程序的所有组件。

由于一个资源组可以有多个应用服务计划，因此用户可以将不同的应用分配给不同的物理资源。 

例如，可以将资源分到开发环境、测试环境和生产环境中。 生产和开发/测试具有各自的环境，因此资源是被隔离的。 这样，对新版应用进行负载测试时，不会与为真正的客户提供服务的生产应用竞争相同的资源。

单个资源组中存在多个计划时，也可定义一个跨多个地理区域的应用程序。 

例如，在两个区域中运行的高度可用的应用可以包括至少两个计划，一个区域一个计划，一个计划关联一个应用。 在这种情况下，应用的所有副本都包含在单个资源组中。 一个资源组中有多个计划和多个应用可以轻松管理、控制和查看应用程序的运行状况。

## <a name="create-an-app-service-plan-or-use-existing-one"></a>创建应用服务计划或使用现有应用服务计划
创建应用时，应考虑创建资源组。 另一方面，如果该应用是另一个更大型应用程序的组件，则应在后者所属的资源组内创建前者。

无论应用是完整的新应用程序还是更大型应用程序的一部分，均可选择使用现有计划进行托管，或选择创建新计划。 该决定主要取决于容量和预期的负载。

我们建议在以下情况下将应用放在新应用服务计划中：

- 应用占用大量资源。 
- 应用与现有计划中托管的其他应用的比例系数不同。
- 应用需要其他地理区域中的资源。

这样一来，可以为应用分配新的资源集，并更好地控制应用。

## <a name="create-an-app-service-plan"></a>创建应用服务计划

可以在浏览应用服务计划或创建应用的过程中创建空的应用服务计划。

在 [Azure 门户](https://portal.azure.cn)中，单击**“新建”** > 
**“Web + 移动”**，然后选择**“Web 应用”**或其他应用服务应用类型。
![在 Azure 门户中创建应用。][createWebApp]

然后即可为新应用选择或创建应用服务计划。

 ![创建应用服务计划。][createASP]

若要创建应用服务计划，可单击“[+] 新建”，键入“应用服务计划”名称，然后选择相应的“位置”。 单击“定价层”，然后为服务选择适当的服务定价层。 选择“全部查看”以查看其他定价选项，例如“免费”和“共享”。 选择定价层后，单击“选择”按钮。

## <a name="move-an-app-to-a-different-app-service-plan"></a>将应用移到其他应用服务计划
可在 [Azure 门户](https://portal.azure.cn)中将应用移到其他应用服务计划。 只要计划属于同一资源组和同一地理区域，即可在计划之间移动应用。

若要将应用移到另一计划，请执行以下操作：

- 导航到要移动的应用。 
- 在“菜单”中，查找“应用服务计划”部分。
- 选择“更改应用服务计划”启动该过程。

“更改应用服务计划”将打开“应用服务计划”选择器。 此时，你可以选取现有计划，然后将该应用移到其中。 

> [AZURE.IMPORTANT]
> 仅显示位于相同资源组和地理位置的有效计划。

![应用服务计划选择器。][change]

每个计划都有自己的定价层。 例如，将站点从免费层移到标准层时，分配给站点的所有应用都可使用标准层的功能和资源。

## <a name="clone-an-app-to-a-different-app-service-plan"></a>将应用克隆到其他应用服务计划
若要将应用移到其他区域，也可以使用应用克隆。 克隆可以复制任何区域的新的或现有的应用服务计划中的应用。

可以在菜单的“开发工具”部分找到“克隆应用”。

> [AZURE.IMPORTANT]
> 有关克隆的某些限制，可阅读[使用 Azure 门户进行 Azure 应用服务应用克隆](/documentation/articles/app-service-web-app-cloning-portal/)。

## <a name="scale-an-app-service-plan"></a>缩放应用服务计划
可通过三种方式缩放一个计划：

* **更改计划的定价层**。 基本层中的计划可转换为标准层中的计划，分配给它的所有应用可转换为使用标准层的功能。
* **更改计划的实例大小**。 例如，可以将“基本”层中使用小型实例的计划改为使用大型实例。 与该计划关联的所有应用即可使用实例大小提高以后带来的更多内存和 CPU 资源。
* **更改计划的实例计数**。 例如，可以将最大规模为 3 个实例的“标准”计划扩展到 10 个实例。 “高级”计划可以扩展到 20 个实例（取决于可用性）。 与该计划关联的所有应用即可使用实例数量提高以后带来的更多内存和 CPU 资源。

单击应用或应用服务计划设置下的“向上缩放”项即可更改定价层和实例大小。 更改会应用到应用服务计划并影响其所托管的所有应用。

 ![设置向上缩放应用的值。][pricingtier]

## <a name="app-service-plan-cleanup"></a>应用服务计划清理
> [AZURE.IMPORTANT]
>未与任何应用关联的**应用服务计划**仍会产生费用，因为它们继续保留计算容量。

若要避免意想不到的费用，当应用服务计划中托管的最后一个应用被删除时，也要删除此空的应用服务计划。

## <a name="summary"></a>摘要
应用服务计划表示一组可在应用间共享的功能和容量。 可以通过应用服务计划灵活地将特定应用分配给一组资源，并进一步优化 Azure 资源的使用情况。 若要节省测试环境的资金，可通过这种方式跨多个应用共享一个计划。 也可跨多个区域和计划进行伸缩，实现生产环境吞吐量的最大化。

## <a name="whats-changed"></a>发生的更改
* 有关从网站更改为应用服务的指南，请参阅 [Azure 应用服务及其对现有 Azure 服务的影响](/documentation/articles/app-service-changes-existing-services/)

[pricingtier]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/appserviceplan-pricingtier.png
[assign]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/assing-appserviceplan.png
[change]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/change-appserviceplan.png
[createASP]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-appserviceplan.png
[createWebApp]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-web-app.png

<!--Update_Description: wording update-->