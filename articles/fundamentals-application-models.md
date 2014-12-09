<properties umbracoNaviHide="0" pageTitle="应用程序模型" metaKeywords="Azure, Azure, application model, Azure application model, development model, Azure development model, hosted service, Azure hosted service, web role, worker role" description="了解 Azure 托管服务应用程序模型。了解核心概念、设计注意事项、如何定义和配置应用程序以及缩放。" linkid="dev-net-fundamentals-application-model" urlDisplayName="Application Model" headerExpose="" footerExpose="" disqusComments="1" title="应用程序模型" authors="" />

# Azure 执行模型

Azure 提供了用于运行应用程序的不同执行模型。每种模型提供一组不同服务，而您选择哪种模型完全取决于您要做什么。本文介绍三种模型，描述它们的技术并提供相关用例。

## 目录

-   [虚拟机][虚拟机]
-   [网站][网站]
-   [云服务][云服务]
-   [我该使用哪一种？做出选择][我该使用哪一种？做出选择]

## <span id="VMachine"></span></a>虚拟机

开发人员、IT 操作人员和其他人可以使用 Azure 虚拟机在云中创建和使用虚拟机。提供所谓的*基础结构即服务 (IaaS)*，这是一项可广泛使用的技术。[图 1][图 1] 显示其基本组件。

<a name="Fig1"></a>![01\_CreatingVMs][01\_CreatingVMs]

**图 1：Azure 虚拟机可提供“基础结构即服务”技术。**

如图所示，可以使用 Azure 管理门户或基于 REST 的 Azure 服务管理 API 创建虚拟机。可以从包括 Internet Explorer、Mozilla 和 Chrome 在内的任何常用浏览器访问管理门户。对于 REST API，Microsoft 为 Windows、Linux 和 Macintosh 系统提供了客户端脚本工具。其他软件也可随意使用此接口。

无论您如何访问此平台，创建新 VM 都需要为 VM 的映像选择一个虚拟硬盘 (VHD)。这些 VHD 将存储在 Azure blob 中。

有两种选项可供您开始操作

1.  上载您自己的 VHD
2.  使用由 Microsoft 及其合作伙伴在 Azure 虚拟机库中或 Microsoft 开放源 [VMDepot][VMDepot] 网站上提供的 VHD。

库中以及 VMDepot 上的 VHD 包含全新的 Microsoft 和 Linux 操作系统映像，以及包含安装在其上的 Microsoft 和其他第三方产品的映像。选项一直都在增长。示例包含以下产品的各个版本和配置：

-   Windows Server
-   Linux 服务器，例如 Suse、Ubuntu 和 CentOS
-   SQL Server
-   BizTalk Server
-   SharePoint Server

除了 VHD 以外，您还要指定您的新 VM 的大小。[Azure 库][Azure 库]中列出了每个大小的完整统计信息。

-   **特小**，共享内核，768MB 内存。
-   **小**，单核，1.75GB 内存。
-   **中**，双核，3.5GB 内存。
-   **大**，4 核，7GB 内存。
-   **特大**，8 核，14GB 内存。
-   **A6**，4 核，28GB 内存。
-   **A7**，8 核，56GB 内存。

最后，选择您的新 VM 应在其中运行的 Azure 数据中心（无论是在美国、欧洲还是亚洲）。

一旦 VM 开始运行，您将根据其运行按小时付费，并在删除该 VM 时停止付费。您支付的数额不取决于您的 VM 的使用率，它仅取决于时钟时间。闲置了一小时的 VM 与负载很重的 VM 的费用相同。

每个运行的 VM 都有一个关联的 *OS 磁盘*，该磁盘保存在 blob 中。当您使用库 VHD 创建 VM 时，该 VHD 将会复制到您的 VM 的 OS 磁盘中。您对于运行的 VM 的操作系统所做的任何更改都存储在这里。例如，如果您安装可修改 Windows 注册表的应用程序，该更改将存储在您的 VM 的 OS 磁盘中。在您下次从该 OS 磁盘创建 VM 时，VM 将继续在您离开它时的相同状态下运行。对于存储在库中的 VHD，Microsoft 将在需要时应用更新，例如操作系统修补程序。但是，对于您自己的 OS 磁盘中的 VHD，您要对应用这些更新负责（尽管 Windows 更新在默认情况下处于打开状态）。

还可以修改运行的 VM，然后使用 sysprep 工具进行捕获。Sysprep 可删除特定内容（例如计算机名称），因此 VHD 映像可用于创建带有相同常规配置的其他 VM。这些映像与您上载的映像一起存储在管理门户中，因此它们可以用作其他 VM 的起点。

虚拟机还监视托管您创建的每台 VM 的硬件。如果运行 VM 的物理服务器发生故障，则平台会注意到此情况并在另一台计算机上启动相同的 VM。假设您有合适的许可，您可以从您的 OS 磁盘中复制已更改的 VHD，然后在其他任何位置（例如您自己的本地数据中心或其他云提供程序中）运行它。

除了其 OS 磁盘以外，您的 VM 还有一个或多个数据磁盘。尽管其中每个磁盘看起来都像是您的 VM 的已装载磁盘驱动器，但实际上每个磁盘的内容都存储在 blob 中。对数据磁盘所做的每次写入都将永久存储在基础 blob 中。与所有 blob 一样，Azure 同时在单个数据中心内和多个数据中心上复制它们以防止出现故障。

可以使用管理门户、PowerShell 以及其他脚本工具，或者直接通过 REST API 来管理运行的 VM。（事实上，您通过管理门户执行的任何操作都可以通过此 API 以编程方式完成。）Microsoft 合作伙伴（如 RightScale 和 ScaleXtreme）也提供依赖 REST API 的管理服务。

可以广泛使用 Azure 虚拟机。Microsoft 面向的主要方案包括：

-   **用于开发和测试的 VM。** 开发组通常需要具有用来创建应用程序的特定配置的 VM。Azure 虚拟机提供一种简单且经济的方法来创建、使用 VM，以及在不再需要时删除它们。
-   **在云中运行应用程序。** 对于某些应用程序，在公有云上运行具有经济意义。例如，考虑一个客户需求会急剧上升的应用程序。始终可以为您自己的数据中心购买足够多的计算机来运行此应用程序，但是其中大部分计算机可能在大多数时间都处于闲置状态。如果在 Azure 上运行此应用程序，只有在需要额外 VM 时才支付额外 VM 的费用，需求高峰过后，就可以将它们停止。或者，假设您是一家初创公司，需要在不做出任何承诺的情况下快速地按需计算资源。同样，Azure 也是合适的选择。
-   **将您自己的数据中心扩展到公有云。**利用 Azure 虚拟网络，您的组织可以创建一个虚拟网络 (VNET)，以便让一组 Azure VM 看上去就像是您自己的本地网络的一部分。它允许在 Azure 上运行 SharePoint 等应用程序，与在您自己的数据中心中运行这些应用程序相比，这种方法可能更易于部署和/或更便宜。
-   **灾难恢复。**基于 IaaS 灾难恢复使您可以只在真正需要计算资源时才为所需的计算资源付费，而不用不停地为很少使用的备份数据中心付费。例如，如果您的主数据中心出现故障，您可以创建在 Azure 上运行的 VM 来运行至关重要的应用程序，然后在不再需要时关闭它们。

虽然这些不是唯一的可能情况，但它们确实为您如何使用 Azure 虚拟机提供了很好的示例。

### VM 分组：云服务

当您使用 Azure 虚拟机创建新 VM 时，可以选择独立运行它，或使它成为一起在*云服务*中运行的一组 VM 的一部分。（尽管名称类似，但请不要和表示 Azure PaaS 技术名称的“云服务”混淆；这是两个不同的概念）。每个独立 VM 都有它自己的公用 IP 地址，而同一云服务中的所有 VM 都可通过一个公用 IP 地址进行访问。[图 2][图 2] 解释这种情况。

<a name="Fig2"></a>![02\_CloudServices][02\_CloudServices]

**图 2：每个独立 VM 都有它自己的公用 IP 地址，而在云服务中分成一组的 VM 可通过一个公用 IP 地址来公开。**

例如，如果您已创建一个虚拟机来创建并测试一个简单的应用程序，则您可能会使用独立 VM。但是，如果您要创建一个多层应用程序，其中包含一个 Web 前端、一个数据库，或许还有一个中间层，则您很可能会将多个 VM 连接到一个云服务中，如图 2 所示。

将 VM 在云服务中分组还使您可以使用其他选项。Azure 为同一云服务中的 VM 提供负载平衡，使用户请求分散在 VM 上。以这种方式连接的 VM 还可以通过 Azure 数据中心内的本地网络直接相互通信。

同一个云服务中的 VM 还可以分组为一个或多个*可用性集*。要理解为何这很重要，可考虑一个运行多个前端 VM 的 Web 应用程序。如果所有这些 VM 都部署在同一台物理计算机上甚至在计算机的同一机架中，单个硬件故障就可能导致它们全都不可访问。但是，如果这些 VM 分组为一个可用性集，Azure 就会跨数据中心部署它们，因此任何一个单点故障都不会波及到其他 VM。

### 场景：使用 SQL Server 运行应用程序

若要更好地理解 Azure 虚拟机的工作原理，有必要给出几个较为具体的场景。例如，假设您要创建一个在 Azure 上运行的可靠且可缩放的 Web 应用程序。一种方法是在一个或多个 Azure VM 中运行该应用程序的逻辑，然后使用 SQL Server 进行数据管理。[图 3][图 3] 解释这种情况。

<a name="Fig3"></a>![03\_AppUsingSQLServer][03\_AppUsingSQLServer]

**图 3：在 Azure 虚拟机中运行的应用程序可以使用 SQL Server 进行存储。**

在此示例中，两种 VM 都是从库中的标准 VHD 创建的。该应用程序的逻辑在 Windows Server 2008 R2 上运行，因此开发人员从此 VHD 创建三个 VM，然后在每个 VM 上安装其应用程序。由于所有这些 VM 都在同一云服务中，因此他可以配置硬件负载平衡来分散请求。开发人员还从库中包含 SQL Server 2012 的 VHD 创建两个 VM。在这两个 VM 运行后，他在每个实例中配置 SQL Server，以使用具有自动故障转移功能的数据库镜像。这不是必需的，因为应用程序可以只使用单个 SQL Server 实例，但是采用此方法可以提高可靠性。

### 场景：运行 SharePoint 场

假设某个组织想创建一个 SharePoint 场，但并不希望在其自己的数据中心中运行该场。原因可能是本地数据中心缺乏资源，或者创建该场的业务部门不希望与其内部 IT 组打交道。在此类情况下，在 Azure 虚拟机上运行 SharePoint 很有意义。[图 4][图 4] 解释这种情况。

<a name="Fig4"></a>![04\_SharePointFarm][04\_SharePointFarm]

**图 4：Azure 虚拟机允许在云中运行 SharePoint 场。**

SharePoint 场有几个组件，每个组件在从不同 VHD 创建的 Azure VM 中运行。需要以下项目：

-   Microsoft SharePoint。库中有一些试用映像，或者组织提供其自己的 VHD。
-   Microsoft SQL Server。SharePoint 依赖于 SQL Server，因此该组织从标准库 VHD 创建运行 SQL Server 2012 的 VM。
-   Windows Server Active Directory。SharePoint 也需要 Active Directory，因此该组织使用库中的 Windows Server 映像在云中创建域控制器。这并非是严格要求的，虽然也可以使用本地域控制器，但 SharePoint 需要与 Active Directory 频繁交互，因此这里显示的方法将具有更好的性能。

尽管图中未显示，但此 SharePoint 场可能使用 Azure 虚拟网络连接到本地网络。这使 VM 以及它们所包含的应用程序对于使用该网络的人们而言像是位于本地。

如这些示例所示，您可以使用 Azure 虚拟机在云中创建和运行新的应用程序，或以其他方式运行现有应用程序。无论您选择哪个选项，该技术的目标都是相同的：为公有云计算提供一个通用基础。

## <span id="WebSites"></span></a>网站

人们通过多种不同方式使用 Web 技术。公司可能需要迁移或设置可处理每周数以百万次的单击的现有网站，该网站部署在全球若干数据中心中。一个网站设计机构可以与一组开发人员合作构建一个可应对数以千计用户的自定义 Web 应用程序。公司开发人员可能需要设置应用程序，以便针对来自其公司 Active Directory 的经身份验证的用户在云中跟踪费用报表。IT 顾问可以使用常用的开放源应用程序为小型企业设置内容管理系统。
可以使用 Azure 虚拟机完成所有这些操作。但是创建和管理原始 VM 需要一些技巧和花一点功夫。如果您需要实现一个网站或 Web 应用程序，这里有一个更容易（也更便宜）的解决方案：这种方法通常称为“平台即服务”(PaaS)。如图 5 所示，Azure 可为此提供网站。

<a name="Fig5"></a>![05\_Websites][05\_Websites]

**图 5：Azure 网站支持通过各种技术构建的静态网站、常用 Web 应用程序和自定义 Web 应用程序。**

Azure 网站在 Azure 云服务的基础上构建，用于创建为运行 Web 应用程序而优化的“平台即服务”解决方案。如图所示，网站将在一组可能包含由多个用户创建的多个网站的单个虚拟机，以及属于单个用户的标准 VM 上运行。VM 是通过 Azure 网站管理的资源池的一部分，因此允许获得高可靠性和容错能力。
入门都很轻松。借助 Azure 网站，用户可以从一系列应用程序、框架和模板中进行选择并在几秒内创建一个网站。然后，他们可以使用最喜欢的开发工具（WebMatrix、Visual Studio、任何其他编辑器）和源控件选项来设置持续集成，并作为一个团队进行开发。依赖于 MySQL 数据库的应用程序可以使用 ClearDB（Microsoft 合作伙伴）为 Azure 提供的 MySQL 服务。
开发人员可以使用网站创建可伸缩的大型 Web 应用程序。该技术支持使用 ASP.NET、PHP、Node.js 和 Python 创建应用程序。例如，应用程序可以使用易贴的会话，而现有 Web 应用程序可以不做任何更改移动到此云平台。在网站上构建的应用程序可随意使用 Azure 的其他方面，例如 Service Bus、SQL Database 和 Blob 存储。您还可以在不同 VM 中运行一个应用程序的多个副本，而网站会自动在各 VM 中对请求进行负载平衡。因为新网站实例是在已存在的 VM 中创建的，因此可以非常快速地启动新的应用程序实例；这明显快于等待创建新的 VM。
如[图 5][图 5] 所示，您可以通过多种方式将代码和其他 web 内容发布到网站中。您可以使用 FTP、FTPS 或 Microsoft 的 WebDeploy 技术。网站还支持从源控件系统发布代码，例如 Git、GitHub、CodePlex、BitBucket、Dropbox、Mercurial、Team Foundation Server 和基于云的 Team Foundation Service。

## <span id="CloudServices"></span></a>云服务

Azure 虚拟机提供 IaaS，而 Azure 网站提供 Web 宿主。第三个计算选项“云服务”提供*平台即服务（Platform as a Service，简称 PaaS）*。这种技术旨在支持可扩展、可靠且运作便宜的应用程序。它还可以使开发人员无需为管理他们所的平台而担心，使他们可以完全专注于自己的应用程序。[图 6][图 6] 解释了此概念。

<a name="Fig6"></a>![06\_CloudServices2][06\_CloudServices2]

**图 6：Azure 云服务提供“平台即服务”功能。**

和其他 Azure 计算选项一样，云服务也依赖于 VM。该技术提供了两个略有不同 VM 选项：*Web 角色* 实例运行包含 IIS 的 Windows Server 变体，而*辅助角色* 实例运行不含 IIS 的同一 Windows Server 变体。云服务应用程序依赖于这两个选项的某种组合。

例如，一个简单的应用程序可能只使用一个 Web 角色，而一个稍复杂的应用程序可能使用一个 Web 角色来处理来自用户的传入请求，然后将那些请求创建的工作传递给辅助角色进行处理。（此通信可能使用 Service Bus 或 Azure 队列。）

如图所示，一个应用程序中的所有 VM 都在同一云服务中运行，如之前对 Azure 虚拟机所述的那样。因此，用户通过单个公用 IP 地址访问应用程序，而请求会自动在应用程序的 VM 上实现负载平衡。与使用 Azure 虚拟机创建的云服务一样，该平台将采用一种能够避免单点硬件故障的方式在云服务应用程序中部署 VM。

即使应用程序在虚拟机中运行，云服务提供的是 PaaS 而非 IaaS，理解这一点很重要。以下办法有助于理解这一点：使用 IaaS（如 Azure 虚拟机），首先要创建和配置您的应用程序将运行的环境，然后将应用程序部署到此环境中。您要负责该环境的大部分管理工作，如在每个 VM 中部署操作系统的新修补版本。相反，在 PaaS 中，这样的环境似乎早已存在。您只需部署您的应用程序。已为您处理它所运行的平台的管理工作，包括部署操作系统的新版本。

使用云服务，您无需创建虚拟机。相反，您提供一个配置文件，告知 Azure 每个 VM 需要多少个角色实例（如三个 Web 角色实例和两个辅助角色实例），平台将为您创建它们。虽然您仍然要选择这些 VM 应该是什么大小（选项与 Azure VM 的相同），但是您无需亲自显式创建它们。如果您的应用程序需要处理更大的负载，则可以要求增加 VM，Azure 将创建这些实例。如果负载降低，则可以关闭这些实例并停止为它们付费。

通常通过两个步骤就能使云服务应用程序可供用户使用。首先，开发人员将应用程序上载到该平台的暂存区域。当开发人员准备好使应用程序上线后，他会使用 Azure 管理门户请求将其投入生产。暂存与生产之间的这种切换无需停机就可完成，这使运行的应用程序可在不打扰其用户的情况下升级到新版本。

云服务还提供监视功能。和 Azure 虚拟机一样，它将检测到发生故障的物理服务器，并在新的计算机上重新启动原先在该服务器上运行的 VM。云服务不仅检测硬件故障，还检测发生故障的 VM 和应用程序。与虚拟机不同，它在每个 Web 角色和辅助角色中都有代理，因此它能够在发生故障时启动新的 VM 和应用程序实例。

云服务的 PaaS 特性还具有其他含义。其中一个最重要的含义是，应编写基于此技术构建的应用程序以在任何 Web 角色或辅助角色实例出现故障时正确运行。若要实现这一目标，云服务应用程序不应该在其自己的 VM 的文件系统中维持状态。与使用 Azure 虚拟机创建的 VM 不同，对云服务 VM 所做的写入不是持久的；这与虚拟机数据磁盘完全不同。相反，云服务应用程序应将所有状态明确写入 SQL Database、blob、表或其他某种外部存储中。以这种方式构建应用程序会使它们更易于扩展、抵抗故障的能力更强，这是云服务的两个重要目标。

## <span id="WhatShouldIUse"></span></a>我该使用哪一种？做出选择

所有三个 Azure 执行模型都能在云中构建可扩展且可靠的应用程序。既然在本质上是类似的，您应该使用哪种模型呢？答案取决于您想要做什么。

虚拟机提供最通用的解决方案。如果您需要可能的最大控制权，或者需要泛型 VM 以用于开发和测试等目的，这是最好的选择。虚拟机也是在云中运行现成本地应用程序的最佳选择，如上述的 SharePoint 示例所示。同时，由于您使用这一技术创建的 VM 可能看起来正像是您的本地 VM，所以这也可能是灾难恢复的最佳选择。当然，能力越强责任也就越大，IaaS 会要求您承担一些管理工作。

当您要创建一个简单的网站时，网站便是合适的选择。当您想要基于现有应用程序（如 Joomla、WordPress 或 Drupal）创建网站时更是如此。如果创建一个管理任务不多的 Web 应用程序（甚至可扩展性很强的 Web 应用程序），或将现有 IIS Web 应用程序移到公有云中，网站模型也是一个不错的选择。它还提供快速部署功能，您的应用程序的一个新实例几乎可以立即开始运行，而通过虚拟机或云服务部署新的 VM 可能需要几分钟。

云服务是 Azure 提供的初始执行模型，它是一个显式 PaaS 方法。虽然 PaaS 与 Web 宿主之间的界限并不分明，但云服务在一些重要方面与网站不同，其中包括：

-   与网站不同，云服务授予您对应用程序的 VM 的管理权限。这允许您安装您的应用程序需要的任意软件，而网站无法做到这一点。
-   因为云服务同时提供 Web 角色和辅助角色，所以对于其业务逻辑需要单独的 VM 的多层应用程序而言，此选择比网站更好。
-   云服务提供单独的分阶段环境和生产环境，使应用程序更新比网站模型更流畅。
-   与网站不同，您可以使用网络技术（如 Azure 虚拟网络和 Azure Connect）将本地计算机与云服务应用程序挂钩。
-   利用云服务，您可以使用远程桌面直接连接到应用程序的 VM，而网站无法做到这一点。

因为云服务是 PaaS，所以它还提供了一些超越 Azure 虚拟机的优势。它会为您完成更多管理任务（如部署操作系统更新），因此您的操作成本很可能低于使用 Azure 虚拟机的 IaaS 方法的成本。

所有这三种 Azure 执行模型都各有优缺点。做出最佳选择需要了解这些内容，明白您要完成的任务，然后从中选出最合适的模型。

有时，单凭一种执行模型可能还不够。在这样的情况下，组合选项是完全合法的。例如，假设您要构建一个应用程序，既想利用云服务 Web 角色的管理优势，又需要使用标准 SQL Server 提高兼容性或性能。在此情况下，最佳选择是组合使用执行模型，如[图 7][图 7] 所示。

<a name="Fig7"></a>![07\_CombineTechnologies][07\_CombineTechnologies]

**图 7：单个应用程序可以使用多个执行模型。**

如图所示，云服务 VM 在不同于虚拟机 VM 的单独云服务中运行。尽管如此，二者仍可高效通信，所以通过此方法构建一个应用有时是最佳选择。

由于云平台需要支持多种不同方案，所以 Azure 提供了不同的执行模型。想要高效使用此平台的任何用户（如果您已阅读至此，可能也包括您）都需要了解每种执行模型。

  [虚拟机]: #VMachine
  [网站]: #WebSites
  [云服务]: #CloudServices
  [我该使用哪一种？做出选择]: #WhatShouldIUse
  [图 1]: #Fig1
  [01\_CreatingVMs]: ./media/fundamentals-application-models/ExecModels_01_CreatingVMs.png
  [VMDepot]: http://vmdepot.msopentech.com/
  [Azure 库]: http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx
  [图 2]: #Fig2
  [02\_CloudServices]: ./media/fundamentals-application-models/ExecModels_02_CloudServices.png
  [图 3]: #Fig3
  [03\_AppUsingSQLServer]: ./media/fundamentals-application-models/ExecModels_03_AppUsingSQLServer.png
  [图 4]: #Fig4
  [04\_SharePointFarm]: ./media/fundamentals-application-models/ExecModels_04_SharePointFarm.png
  [05\_Websites]: ./media/fundamentals-application-models/ExecModels_05_Websites.png
  [图 5]: #Fig5
  [图 6]: #Fig6
  [06\_CloudServices2]: ./media/fundamentals-application-models/ExecModels_06_CloudServices2.png
  [图 7]: #Fig7
  [07\_CombineTechnologies]: ./media/fundamentals-application-models/ExecModels_07_CombineTechnologies.png
