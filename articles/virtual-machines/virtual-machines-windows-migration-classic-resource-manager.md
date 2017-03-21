<properties
    pageTitle="将经典资源迁移到 Azure Resource Manager - 概述 | Azure"
    description="本文逐步讲解如何对资源进行平台支持的从经典部署模型到 Azure Resource Manager 的迁移"
    services="virtual-machines-windows"
    documentationcenter=""
    author="singhkays"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="78492a2c-2694-4023-a7b8-c97d3708dcb7"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/23/2017"
    wacn.date="03/20/2017"
    ms.author="kasing" />  


# 平台支持的从经典部署模型到 Azure Resource Manager 的 IaaS 资源迁移
本文介绍如何才能将基础结构即服务 (IaaS) 资源从经典部署模型迁移到 Resource Manager 部署模型。用户可以阅读有关 [Azure Resource Manager 功能和优点](/documentation/articles/resource-group-overview/)的更多内容。本文详述了如何使用虚拟网络的站点到站点网关连接订阅中并存的两个部署模型中的资源。

## 迁移目标
Resource Manager 除了可让你通过模板部署复杂的应用程序之外，还可使用 VM 扩展来配置虚拟机，并且纳入了访问管理和标记。Azure Resource Manager 将虚拟机的可缩放并行部署包含在可用性集内。新部署模型还针对计算、网络和存储单独提供生命周期管理。最后，将重点介绍为了按默认启用安全性而要在虚拟网络中实施虚拟机的做法。

在 Azure Resource Manager 之下，针对来自经典部署模型的几乎所有功能，均提供计算、网络和存储支持。若要充分利用 Azure Resource Manager 中的新功能，可将现有部署从经典部署模型中迁移出来。

## 自动化与工具在迁移之后的变化
在将资源从经典部署模型迁移到 Resource Manager 部署模型的过程中，必须更新现有的自动化或工具，确保其在迁移之后仍可继续运行。

## 将 IaaS 资源从经典模型迁移到 Resource Manager 的意义
在详细了解详细信息之前，先看看 IaaS 资源的数据平面和管理平面操作之间的差异。

* *管理平面*描述进入管理平面或 API 修改资源的调用。例如，创建 VM、重启 VM 以及将虚拟网络更新成新子网等操作均可管理正在运行的资源。它们并不直接影响与实例之间的连接。
* *数据平面*（应用程序）描述应用程序本身的运行时，并涉及与不通过 Azure API 的实例的交互。访问网站或从运行中的 SQL Server 实例或 MongoDB 服务器拉取数据，全都被视为数据平面或应用程序交互。从存储帐户复制 Blob，以及访问公共 IP 地址以通过 RDP 或 SSH 连接到虚拟机也属于数据平面。这些操作可让应用程序继续跨计算、网络和存储运行。

> [AZURE.NOTE]
在某些迁移方案中，Azure 平台会停止、释放和重新启动虚拟机。这会造成短暂的数据平面停机。
>
>

## 支持的迁移范围
有三个迁移范围，其主要目标是计算、网络和存储。

### 迁移（不在虚拟网络中的）虚拟机
在 Resource Manager 部署模型中，默认情况下会针对应用程序强制实施安全性。在 Resource Manager 模型中，所有 VM 都必须位于虚拟网络内。Azure 平台会在迁移过程中重新启动（`Stop`、`Deallocate`、`Start`）VM。你有两个虚拟网络选项：

* 你可以请求平台创建新的虚拟网络，并将虚拟机迁移到新的虚拟网络。
* 你可以将虚拟机迁移到 Resource Manager 中的现有虚拟网络。

> [AZURE.NOTE]
在此迁移范围内，迁移期间可能有一段时间不允许进行管理平面操作和数据平面操作。
>
>

### 迁移（虚拟网络中的）虚拟机
对于大多数 VM 配置来说，在经典部署模型和 Resource Manager 部署模型之间迁移的只有元数据。基础 VM 在相同硬件、相同网络上，使用相同的存储来运行。在迁移过程中，可能有一段时间不允许进行管理平面操作。不过，数据平面可继续运行。也就是说，在 VM（经典）之上运行的应用程序不会在迁移期间造成停机。

目前不支持以下配置。如果在将来添加支持，可能会造成这一配置中的某些 VM 停机（经历停止、解除分配和重新启动 VM 等操作）。

* 在单个云服务中有一个以上的可用性集。
* 在单个云服务中有一个或多个可用性集和不在可用性集中的 VM。

> [AZURE.NOTE]
在此迁移范围内，迁移期间可能有一段时间不允许进行管理平面操作。上述某些配置会造成数据平面停机。
>
>

### 存储帐户迁移
为了让迁移顺畅进行，可以在经典存储帐户中部署 Resource Manager VM。通过此功能，就可以并且应该迁移计算和网络资源，而不必受到存储帐户的约束。迁移虚拟机和虚拟网络后，需要迁移存储帐户才能完成迁移过程。

> [AZURE.NOTE]
Resource Manager 部署模型没有经典映像和磁盘的概念。迁移存储帐户时，经典映像和磁盘不在 Resource Manager 堆栈中可见，但后备 VHD 保留在存储帐户中。
>
>

## 不支持的功能和配置
目前不支持某些功能和配置。下列各节介绍了我们对这些功能和配置的相关建议。

### 不支持的功能
目前不支持以下功能。你可以选择删除这些设置、迁移 VM，然后在 Resource Manager 部署模型中重新启用这些设置。

| 资源提供程序 | 功能 |
| --- | --- |
| 计算 |不关联的虚拟机磁盘。 |
| 计算 |虚拟机映像。 |
| 网络 |终结点 ACL。 |
| 网络 |虚拟网络网关（Azure ExpressRoute 网关、应用程序网关）。 |
| 网络 |使用 VNet 对等互连的虚拟网络。（将 VNet 迁移到 ARM，然后进行对等互连）详细了解 [VNet Peering](/documentation/articles/virtual-network-peering-overview/)（VNet 对等互连）。 |
| 网络 |流量管理器配置文件。 |

### 不支持的配置
目前不支持以下配置。

| 服务 | 配置 | 建议 |
| --- | --- | --- |
| 资源管理器 |经典资源的基于角色的访问控制 (RBAC) |由于资源的 URI 在迁移后会进行修改，因此建议用户规划需要在迁移后进行的 RBAC 策略更新。 |
| 计算 |与 VM 关联的多个子网 |将子网配置更新为只引用子网。 |
| 计算 |属于虚拟网络，但未分配显式子网的虚拟机 |你可以选择性地删除 VM。 |
| 计算 |具有警报的虚拟机 |迁移进行下去时，这些设置会删除。强烈建议用户在进行迁移之前先评估其环境。或者，也可以在迁移完成之后重新配置警报设置。 |
| 计算 |XML VM 扩展（BGInfo 1.*、Visual Studio 调试器、Web 部署和远程调试） |此操作不受支持。建议用户在继续迁移之前从虚拟机中删除这些扩展，否则系统会在迁移过程中自动删除它们。 |
| 计算 |使用高级存储启动诊断 |在继续执行迁移之前，为 VM 禁用启动诊断功能。在迁移完成之后，可以在 Resource Manager 堆栈中重新启用启动诊断。此外，应删除正用于屏幕截图和串行日志的 blob，以便不会再为这些 blob 向你收费。 |
| 计算 |包含 Web 角色/辅助角色的云服务 |目前不支持。 |
| 网络 |包含虚拟机和 Web 角色/辅助角色的虚拟网络 |目前不支持。 |
| Azure App Service |包含应用服务环境的虚拟网络 |目前不支持。 |
| Azure HDInsight |包含 HDInsight 服务的虚拟网络 |目前不支持。 |
| Microsoft Dynamics Lifecycle Services |包含由 Dynamics Lifecycle Services 管理的虚拟机的虚拟网络 |目前不支持。 |
| Azure AD 域服务 |包含 Azure AD 域服务的虚拟网络 |目前不支持。 |
| 计算 |Azure 安全中心扩展，其中包含的 VNET 拥有 VPN 网关（具有传输连接）或 ExpressRoute 网关（在本地 DNS 服务器上） |Azure 安全中心在虚拟机上自动安装扩展，用于监视其安全性并引发警报。如果在订阅上启用了 Azure 安全中心策略，通常会自动安装这些扩展。目前不支持 ExpressRoute 网关迁移，而具有传输连接的 VPN 网关则会失去本地访问权限。如果删除 ExpressRoute 网关或迁移具有传输连接的 VPN 网关，那么继续执行迁移时，会失去对 VM 存储帐户的 Internet 访问权限。发生这种情况时无法进行迁移，因为来宾代理状态 blob 无法填充。建议在进行迁移时提前 3 小时禁用订阅的 Azure 安全中心策略。 |

## 迁移体验
在开始迁移体验之前，建议执行以下操作：

* 确保你要迁移的资源未使用任何不受支持的功能或配置。通常平台会检测到这些问题并生成错误。
* 如果你有不在虚拟网络中的 VM，平台将在准备操作期间停止这些 VM 并解除其分配。因此，如果不想丢失公共 IP 地址，请先保留 IP 地址再触发准备操作。但是，如果 VM 位于虚拟网络中，则不会将其停止和解除分配。
* 规划在非工作时间迁移，以便应对迁移期间可能发生的任何意外失败。
* 使用 PowerShell、命令行接口 (CLI) 命令或 REST API 下载当前的 VM 配置，以便能够在完成准备步骤之后轻松进行验证。
* 先更新用于处理 Resource Manager 部署模型的自动化/操作化脚本，再开始迁移。也可以选择在资源处于准备就绪状态时执行 GET 操作。
* 在迁移完成之后，评估经典 IaaS 资源上配置的 RBAC 策略并准备好计划。

迁移工作流如下所示

![显示迁移工作流的屏幕截图](./media/virtual-machines-windows-migration-classic-resource-manager/migration-workflow.png)  


> [AZURE.NOTE]
下列各节描述的所有操作都是幂等的。如果遇到功能不受支持或配置错误以外的问题，建议重试准备、中止或提交操作。Azure 平台会尝试再次操作。
>
>

### 验证
验证操作是迁移过程中的第一个步骤。此步骤的目的是在后台对进行迁移的资源执行数据分析，并在资源能够进行迁移时返回成功/失败。

可选择需要进行迁移验证的虚拟网络或托管服务（如果不是虚拟网络）。

* 如果资源无法迁移，Azure 平台会列出不支持迁移该资源的所有原因。

验证存储服务时，将在与存储帐户同名且追加了“-Migrated”的资源组中找到已迁移的帐户。例如，如果存储帐户名为“mystorage”，则将在名为“mystorage-Migrated”的资源组中找到支持 ARM 的资源，并且它将包含名为“mystorage”的存储帐户。

### 准备
准备操作是迁移过程中的第二个步骤。此步骤的目标是要模拟将 IaaS 资源从经典资源转换为 Resource Manager 资源的过程，并以并排方式让此转换过程直观可见。

可选择要进行迁移准备的虚拟网络或托管服务（如果不是虚拟网络）。

* 如果资源无法迁移，Azure 平台会停止迁移过程，并列出准备操作失败的原因。
* 如果资源可迁移，Azure 平台会先锁定进行迁移的资源的管理平面操作。例如，无法将数据磁盘添加到进行迁移的 VM。

然后，Azure 平台就会开始将迁移中资源的元数据从经典模型迁移到 Resource Manager 模型。

准备操作完成之后，可以选择在经典模型和 Resource Manager 模型中将资源可视化。对于经典部署模型中的每项云服务，Azure 平台都会创建模式为 `cloud-service-name>-migrated` 的资源组名称。

> [AZURE.NOTE]
不属于经典虚拟网络的虚拟机将在此迁移阶段中停止（解除分配）。
>
>

### 检查（手动或通过脚本）
在检查步骤中，你可以选择使用前面下载的配置来验证迁移是否正常。或者，你也可以登录到门户并抽查属性和资源，来验证元数据的迁移是否正常。

如果迁移的是虚拟网络，大多数虚拟机配置不会重新启动。对于这些 VM 上的应用程序，你可以验证应用程序是否仍已启动并在运行。

可以测试监视/自动化和操作脚本，以查看 VM 是否按预期运行，以及更新后的脚本是否正常运行。仅当资源处于准备就绪状态时，才支持 GET 操作。

你可以慢慢考虑是否要提交迁移，因为系统对此并没有时间限制。可以在此状态下停留任何时间。但是，这些资源的管理平面会遭到锁定，直到中止或提交为止。

如果你发现任何问题，始终可以中止迁移，并返回到经典部署模型。在用户返回后，Azure 平台会打开资源的管理平面操作，使用户可以继续在经典部署模型中对这些 VM 执行正常操作。

### 中止
中止是可选步骤，可让你将更改还原为经典部署模型，并停止迁移。

> [AZURE.NOTE]
触发提交操作后，就无法执行此操作。
>
>

### 提交
完成验证之后，就可以提交迁移。资源不会再出现在经典部署模型中，而只有在 Resource Manager 部署模型中才能使用这些资源。只能在新门户中管理迁移的资源。

> [AZURE.NOTE]
这是幂等操作。如果该操作失败，建议重试该操作。如果一直失败，请创建支持票证，或在 [VM 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=WAVirtualMachinesforWindows)上创建标记为 ClassicIaaSMigration 的论坛帖子。
>
>

## <a name="frequently-asked-questions"></a> 常见问题
**此迁移计划是否影响 Azure 虚拟机上运行的任何现有服务或应用程序？**

否。VM（经典）是公开上市的完全受支持的服务。你可以继续使用这些资源来拓展你在 Azure 上的足迹。

**如果我近期不打算迁移，我的 VM 会出现什么情况？**

我们近期不会淘汰现有的经典 API 和资源模型。我们想要通过 Resource Manager 部署模型中提供的高级功能，让迁移变得简单。强烈建议查看 Resource Manager 下 IaaS 包含的[一些改进](/documentation/articles/resource-manager-deployment-model/)。

**对于我现有的工具而言，此迁移计划有何意义？**

将工具更新为 Resource Manager 部署模型，是必须在迁移计划中考虑的最重要的更改之一。

**管理平面的停机时间持续多久？**

这取决于迁移的资源数量。对于较小型部署（几十个 VM），整个迁移过程应该不超过一小时。如果是大规模部署（数百个 VM），迁移过程可能需要花费几个小时。

**在 Resource Manager 中提交迁移的资源之后，是否还可以回滚？**

只要资源处于准备就绪状态，就可以中止迁移。但是，在通过提交操作成功迁移资源之后，就不支持回滚。

**提交操作失败时，是否可以回滚迁移？**

如果提交操作失败，就无法中止迁移。包括提交操作在内的所有迁移操作都是幂等的。因此，建议在片刻之后重试操作。如果仍遇到错误，请创建支持票证，或在 [VM 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=WAVirtualMachinesforWindows)上创建标记为 ClassicIaaSMigration 的论坛帖子。

**如果我必须使用 Resource Manager 下的 IaaS，是否必须购买其他 ExpressRoute 线路？**

否。我们近期实现了[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-move/)。如果你已有 ExpressRoute 线路，则不需要购买新的线路。

**如果我已经为经典 IaaS 资源配置基于角色的访问控制策略，该怎么办？**

在迁移期间，资源从经典资源转换为 Resource Manager 资源。因此，建议计划需要在迁移之后进行的 RBAC 策略更新。

**如果我目前正在使用 Azure Site Recovery 或 Azure 备份，该怎么办？**

若要迁移允许进行备份的虚拟机，请参阅[已经在备份保管库中备份了经典 VM。想要将 VM 从经典模型迁移到 Resource Manager 模型，如何在恢复服务保管库中备份它们？](/documentation/articles/backup-azure-backup-ibiza-faq/)我已在备份保管库中备份经典 VM。想要将 VM 从经典模型迁移到 Resource Manager 模型，该如何在恢复服务保管库中备份它们？

**我是否可以验证订阅或资源，以查看其是否能够迁移？**

是的。在平台支持的迁移选项中，准备迁移的第一个步骤，就是验证资源是否能够进行迁移。如果验证操作失败，用户会收到包含无法完成迁移的所有原因的消息。

**如果我在准备要迁移的 IaaS 资源时遇到配额错误，会发生什么情况？**

建议你中止迁移，然后记录支持请求，以在要迁移 VM 的区域中增加配额。配额请求经过批准后，可以重新开始执行迁移步骤。

**如何报告问题？**

请在 [VM 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=WAVirtualMachinesforWindows)上使用关键字 ClassicIaaSMigration 发布有关迁移问题的帖子。建议你将所有问题都发布在此论坛上。如果你有支持协定，也欢迎你记录支持票证。

**如果我不喜欢平台在迁移期间选择的资源名称，该怎么做？**

在经典部署模型中为其显式提供名称的所有资源都会在迁移期间得到保留。在某些情况下，会创建新资源。例如：为每个 VM 创建网络接口。目前无法控制迁移期间所创建的这些新资源的名称。请在 [Azure 反馈论坛](http://feedback.azure.com)上针对此功能进行投票。

**我收到一条消息，指出*“VM 报告总体代理状态为‘未就绪’。因此，此 VM 无法迁移。请确保 VM 代理报告总体代理状态为‘就绪’”*或*“VM 包含未报告其状态的扩展。因此，此 VM 无法迁移。”***

当 VM 未建立到 Internet 的出站连接时，将收到此消息。VM 代理使用出站连接访问 Azure 存储帐户，每隔五分钟更新一次代理状态。

## 后续步骤
你已经了解经典 IaaS 资源到 Resource Manager 的迁移，现在可以开始迁移资源。

* [有关平台支持的从经典部署模型到 Azure Resource Manager 的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
* [使用 PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-windows-ps-migration-classic-resource-manager/)
* [使用 CLI 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-linux-cli-migration-classic-resource-manager/)
* [使用社区 PowerShell 脚本将经典虚拟机克隆到 Azure Resource Manager](/documentation/articles/virtual-machines-windows-migration-scripts/)
* [查看最常见的迁移错误](/documentation/articles/virtual-machines-migration-errors/)

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: wording update-->