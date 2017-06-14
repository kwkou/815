<properties
    pageTitle="规划从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移 | Azure"
    description="规划将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager"
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
    ms.date="04/01/2017"
    wacn.date="05/15/2017"
    ms.author="kasing"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="7be639ed951e2421cf2bb31cdf2b2b99ce6b4aac"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="planning-for-migration-of-iaas-resources-from-classic-to-azure-resource-manager"></a>规划将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager
尽管 Azure Resource Manager 提供了大量令人惊叹的功能，但规划好迁移过程以确保一切顺利仍至关重要。 花时间进行规划可确保执行迁移活动时不会遇到问题。 

> [AZURE.NOTE] 
> 以下指导在很大程度上归功于 Azure 客户顾问团队，以及与客户合作迁移大型环境的云解决方案架构师。 此文档将随着出现新的成功模式而持续更新，因此，请不时地回来查看，了解是否有新的推荐内容。

迁移之旅包括四个常规阶段：<br>

![迁移阶段](./media/virtual-machines-windows-migration-classic-resource-manager/plan-labtest-migrate-beyond.png)

## <a name="plan"></a>计划

### <a name="technical-considerations-and-tradeoffs"></a>技术注意事项和权衡

根据技术要求大小、地理区域和操作方案，可能需要考虑：

1. 为什么组织需要 Azure Resource Manager？  迁移的业务原因有哪些？
2. Azure Resource Manager 的技术原因有哪些？  想要利用其他哪些 Azure 服务（如果有）？
3. 迁移包括哪些应用程序（或虚拟机集）？
4. 迁移 API 支持哪些方案？  请参阅[不支持的功能和配置](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/#unsupported-features-and-configurations)。
5. 运营团队目前是否支持经典部署模型和 Azure Resource Manager 中的应用程序/VM？
6. Azure Resource Manager 如何更改 VM 部署、管理、监视和报告过程（如果有）？  部署脚本是否需要更新？
7. 用于提醒利益干系人（最终用户、应用程序所有者和基础结构所有者）的通信计划有哪些？
8. 根据环境的复杂性，是否应有维护时段，其间应用程序对最终用户和应用程序所有者不可用？  如果需要，持续时间有多长？
9. 用于确保利益干系人知识渊博且熟悉 Azure Resource Manager 的培训计划是什么？
10. 用于迁移的项目管理计划是什么？
11. Azure Resource Manager 迁移时间线和其他相关技术路线图有哪些？  它们是否保持最佳的协调？

### <a name="patterns-of-success"></a>成功模式

成功的客户采用详尽的计划，其中会讨论、记录并控制上述问题。  确保与发起人和利益干系人就迁移计划进行广泛交流。  使用迁移选项的知识武装自身，强烈建议浏览下面的迁移文档集。

* [平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移概述](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
* [有关平台支持的从经典部署模型到 Azure Resource Manager 部署模型的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
* [规划从经典部署模型到 Azure Resource Manager 的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-plan/)
* [使用 PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-windows-ps-migration-classic-resource-manager/)
* [使用 CLI 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-linux-cli-migration-classic-resource-manager/)
* [用于帮助将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型的社区工具](/documentation/articles/virtual-machines-windows-migration-scripts/)
* [查看最常见的迁移错误](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-errors/)
* [查看有关将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型的最常见问题](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-faq/)

### <a name="pitfalls-to-avoid"></a>需避免的错误

- 未能规划。  该迁移的技术步骤证实是非常有效的，且结果是可预测的。
- 平台支持的迁移 API 将适用于所有方案的假设。 请参阅[不受支持的功能和配置](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/#unsupported-features-and-configurations)，了解支持的方案有哪些。
- 未针对最终用户规划可能的应用程序故障。  计划足够的缓冲区来充分警告最终用户可能不可用的应用程序时间。

## <a name="lab-test"></a>实验室测试 

**复制环境并执行测试迁移**
  > [AZURE.NOTE]
  > 使用社区贡献的、Azure.cn 支持部门尚不正式支持的工具按原样复制现有环境。 因此，它是**可选**步骤，但它是无需接触生产环境即可找到问题的最佳办法。 如果不能使用社区提供的工具，请阅读下面的验证/准备/中止试运行建议。
  >

  针对确切方案（计算、网络和存储）执行实验室测试是确保顺利迁移的最佳办法。 这有助于确保：

  - 待测试的是完全独立的实验室或现有的非生产环境。 建议使用可反复迁移和破坏性修改的完全独立的实验室。  下面列出了用于收集/水化实际订阅的元数据的脚本。
  - 在单独的订阅中创建实验室是个好办法。 因为实验室将被反复拆毁，拥有单独、独立的订阅可降低意外删除某些实际内容的可能性。

  可以使用 AsmMetadataParser 工具实现此操作。 [在此处了解有关该工具的详细信息](https://github.com/Azure/classic-iaas-resourcemanager-migration/tree/master/AsmToArmMigrationApiToolset)

### <a name="patterns-of-success"></a>成功模式

下面是在许多大型迁移中发现的问题。 这个列表并不详尽，有关详细信息，请参阅[不支持的功能和配置](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/#unsupported-features-and-configurations)。  你不一定会遇到这些技术问题，但如果遇到，在尝试迁移前先解决这些问题可确保体验更流畅。

- **执行验证/准备/中止试运行** - 这可能是确保经典部署模型成功迁移到 Azure Resource Manager 部署模型最重要的步骤。 迁移 API 包括三个主要步骤：验证、准备和提交。 验证将读取经典环境的状态并返回所有问题的结果。 但是，由于某些问题可能存在于 Azure Resource Manager 堆栈中，因此验证并不会捕获所有内容。 作为迁移过程下一步的“准备”有助于公开这些问题。 “准备”会将元数据从经典部署模型移动到 Azure Resource Manager 部署模型，但不会提交移动，且不会删除或更改经典部署模型端的任何内容。 试运行涉及准备迁移，然后中止（**而不是提交**）迁移准备。 验证/准备/中止试运行的目标是查看 Azure Resource Manager 堆栈中的所有元数据，对其进行检查（以编程方式或在门户中），然后验证所有内容是否正确迁移以及解决技术问题。  它还可让你对迁移持续时间有一些认识，以便可以相应地规划停机时间。  验证/准备/中止操作不会导致任何用户停机时间：因此，它对应用程序使用不具有破坏性。
  - 以下各项需要在试运行前解决，但如果错过了，试运行测试也会安全刷新这些准备步骤。 企业迁移期间，我们发现试运行是确保迁移准备就绪的安全且重要的方法。
  - 准备运行期间，将对整个虚拟网络锁定控制平面（Azure 管理操作），因此在验证/准备/中止期间无法对 VM 元数据进行任何更改。  但在其他方面，所有应用程序功能（RD、VM 使用等）均不受影响。  VM 用户不知道正在执行的是试运行。

- **Express Route 线路和 VPN**。 当前含授权链接的快速路由网关不能在不停机的情况下集成。 有关解决方法，请参阅[将 ExpressRoute 线路和关联的虚拟网络从经典部署模型迁移到 Resource Manager 部署模型](/documentation/articles/expressroute-migration-classic-resource-manager/)。

- **VM 扩展** - 虚拟机扩展可能是迁移正在运行的 VM 的最大障碍之一。 VM 扩展修正可能需要 1-2 天，因此请相应进行规划。  一个有效的 Azure 代理是报告正在运行的 VM 的 VM 扩展状态所需的。 如果正在运行的 VM 返回的状态为不佳，这将暂停迁移。 代理本身无需处于正常运行状态即可启用迁移，但如果 VM 上存在扩展，则同时需要正常运行的代理和出站 Internet 连接（含 DNS）才能使迁移继续。
  - 如果迁移期间失去了与 DNS 服务器的连接，则需要从每个 VM 中首先删除除 BGInfo v1.\* 外的所有 VM 扩展， 然后才能进行迁移准备，并在 Azure Resource Manager 迁移后重新添加到 VM 中。  **这仅适用于正在运行的 VM。**  如果已停止解除分配 VM，则无需删除 VM 扩展。 **注意：**Azure 诊断和安全中心监视等诸多扩展在迁移后将重新安装，因此删除它们没有问题。
  - 此外，确保网络安全组不限制出站 Internet 访问权限。 这可能针对某些网络安全组配置。 若要使 VM 扩展迁移到 Azure Resource Manager，出站 Internet 访问权限（和 DNS）是必需的。 
  - BGInfo 扩展有两个版本：v1 和 v2。  如果 VM 是通过经典管理门户或 PowerShell 创建的，则其上可能有 v1 扩展。 此扩展无需删除且将被迁移 API 跳过（即不迁移）。 但是，如果经典 VM 通过使用新的 Azure 门户创建，它很可能具有基于 JSON 的 v2 版本的 BGInfo，该版本可迁移到 Azure Resource Manager 部署模型，只要代理正常运行且具有出站 Internet 访问权限（和 DNS）。 
  - **补救选项 1**。 如果你知道 VM 不会有出站 Internet 访问权限、正常运行的 DNS 服务和 VM 上正常运行的 Azure 代理，则在准备前在迁移期间卸载所有 VM 扩展，然后在迁移后重新安装这些 VM 扩展。 
  - **补救选项 2**。 如果 VM 扩展是个大障碍，另一个方法是在迁移前关闭/解除分配所有 VM。 迁移已解除分配的 VM，然后在 Azure Resource Manager 端重新启动它们。 这样做的好处是可迁移 VM 扩展。 缺点是将丢失所有面向公众的虚拟 IP（这可能是非初学者），并且 VM 明显会关闭，从而对正常运行的应用程序产生大得多的影响。

    > [AZURE.NOTE] 
    > 如果针对要迁移的正在运行的 VM 配置 Azure 安全中心策略，则在删除扩展前需要停止安全策略，否则将在删除扩展后自动重新安装安全监视扩展。

- **可用性集** - 对于要迁移到 Azure Resource Manager 的虚拟网络 (vNet)，经典部署（即云服务）包含的 VM 必须全部位于同一个可用性集中，或者 VM 均不得位于任何可用性集中。 云服务中具有多个可用性集与 Azure Resource Manager 不兼容，并且迁移将暂停。  此外，不能出现一些 VM 位于可用性集，而一些 VM 不位于可用性集的情况。 若要解决此问题，需要修正或重新配置云服务。  请相应地进行规划，因为这可能很耗时。 

- **Web/辅助角色部署** - 包含 Web 和辅助角色的云服务无法迁移到 Azure Resource Manager。 必须先从虚拟网络中删除 Web/辅助角色，然后才能开始迁移。  典型的解决方案只是将 Web/辅助角色实例移到单独的经典虚拟网络中，该网络也链接到了 ExpressRoute 回路，或者将代码迁移到较新的 PaaS 应用服务（此讨论已超出本文范围）中。 在前一个重新部署用例中，创建了新的经典虚拟网络，将该 Web/辅助角色移动/重新部署到该新虚拟网络，然后从正在删除的虚拟网络中删除这些部署。 无需更改代码。 新的[虚拟网络对等互连](/documentation/articles/virtual-network-peering-overview/)功能可以用来使包含 Web/辅助角色的经典虚拟网络和同一 Azure 区域中的其他虚拟网络（如正在迁移的虚拟网络）通力合作（**虚拟网络迁移完成后，对等虚拟网络无法迁移**），从而提供没有性能损失和延迟/带宽损失的相同功能。 鉴于增加了[虚拟网络对等互连](/documentation/articles/virtual-network-peering-overview/)，现可轻易缓解 Web/辅助角色部署，且不会阻止到 Azure Resource Manager 的迁移。

- **Azure Resource Manager 配额** - 对于经典部署模型和 Azure Resource Manager 部署模型，Azure 区域都有单独的配额/限制。 即使在不使用新硬件的迁移方案中*（我们正在将现有的 VM 从经典部署模型切换到 Azure Resource Manager 部署模型）*，Azure Resource Manager 配额仍需处于容量充足的位置，然后才能开始迁移。 下面列出了我们已知的导致问题的主要限制。  开具配额支持票证来提高限制。 

    > [AZURE.NOTE]
    > 需要在与要迁移的当前环境所处的同一区域中提高这些限制。
    >

    - 网络接口
    - 负载均衡器
    - 公共 IP
    - 静态公共 IP
    - 核心数
    - 网络安全组
    - 路由表

    可通过最新版 Azure PowerShell 使用以下命令查看当前的 Azure Resource Manager 配额。

    **计算**（核心数、可用性集数）

        Get-AzureRmVMUsage -Location <azure-region>

    **网络***（虚拟网络、静态公共 IP、公共 IP、网络安全组、网络接口、负载均衡器和路由表）*

        Get-AzureRmUsage /subscriptions/<subscription-id>/providers/Microsoft.Network/locations/<azure-region> -ApiVersion 2016-03-30 | Format-Table

    **存储***（存储帐户）*

        Get-AzureRmStorageUsage

- **Azure Resource Manager API 限制** - 如果有足够大的环境（如 在 VNET 中有 > 400 VM），则可能达到 Azure Resource Manager 中的写入的默认 API 限制（当前为 `1200 writes/hour`）。 开始迁移前，应开具支持票证为订阅提高此限制。

- **预配超时 VM 状态** - 如果任何虚拟机具有状态 `provisioning timed out`，则需要在迁移前解决此问题。 执行此操作的唯一方法是通过取消预配/重新预配 VM（删除、保留磁盘然后重新创建 VM）来使用停机时间。 

- **RoleStateUnknown VM 状态** - 如果迁移因 `role state unknown` 错误消息暂停，请使用门户检查 VM 并确保其正常运行。 此错误通常数分钟后会自行消失（无需修正），通常属于虚拟机 `start`、`stop` 和 `restart` 操作期间看到的过渡类型。 **建议做法：**数分钟后尝试再次迁移。 

- **Fabric 群集不存在** - 在某些情况下，由于各种原因，某些 VM 无法迁移。 已知情况之一是当 VM 创建于最近（过去一周左右），且碰巧获得尚未为 Azure Resource Manager 工作负荷配备的 Azure 群集。  你将收到一条错误消息，指出无法迁移 `fabric cluster does not exist` 和 VM。 等待数天通常可解决此特殊问题，因为群集会很快启用 Azure Resource Manager。 但是，一个直接的解决方法是对 VM 执行 `stop-deallocate`，然后继续迁移，并在迁移后在 Azure Resource Manager 中开始 VM 备份。

### <a name="pitfalls-to-avoid"></a>需避免的错误

- 不要走捷径和省略验证/准备/中止试运行迁移。
- 即使不是全部，至少大多数潜在问题都将在验证/准备/中止期间浮现。

## <a name="migration"></a>迁移

### <a name="technical-considerations-and-tradeoffs"></a>技术注意事项和权衡

现已准备就绪，因为环境的已知问题已得到解决。

对于实际迁移，可能需要考虑：

1. 使用递增的优先级规划和安排虚拟网络（迁移的最小单位）。  首先处理简单的虚拟网络，然后循序渐进到更复杂的虚拟网络。
2. 大多数客户都有非生产环境和生产环境。  最后安排生产。
3. **（可选）**安排具有足够缓冲区的维护停机时间，以防出现意外。
4. 出现问题时，与支持团队沟通并保持一致。

### <a name="patterns-of-success"></a>成功模式

实际迁移前，应考虑和缓解上面的“实验室测试”部分中的技术指南。  通过充分测试，实际上迁移不属于事件。  对于生产环境，具有其他支持，如受信任的 Microsoft 合作伙伴或 Microsoft 高级服务等会很有帮助。

### <a name="pitfalls-to-avoid"></a>需避免的错误

未完全测试可能导致迁移出现问题和延迟。  

## <a name="beyond-migration"></a>迁移之外

### <a name="technical-considerations-and-tradeoffs"></a>技术注意事项和权衡

既然采用了 Azure Resource Manager，便可以最大程度地发挥平台优势。  请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)，了解其他好处。

注意事项：

- 将迁移与其他活动绑定。  大多数客户倾向于应用程序维护时段。  如果是这样，你可能需要利用该停机时间启用加密和迁移到托管磁盘等其他 Azure Resource Manager 功能。
- 再次浏览 Azure Resource Manager 的技术和业务原因；启用仅在应用于环境的 Azure Resource Manager 上适用的其他服务。
- 使用 PaaS 服务实现环境现代化。

### <a name="patterns-of-success"></a>成功模式

对现在想要在 Azure Resource Manager 中启用哪些服务具有目的性。  许多客户找到以下关于其 Azure 环境令人关注的事实：

- [基于角色的访问控制](/documentation/articles/resource-group-overview/#access-control)。
- [使用 Azure Resource Manager 模板以更轻松、更可控的方式完成部署](/documentation/articles/resource-group-overview/#template-deployment)。
- [标记](/documentation/articles/resource-group-using-tags/)。
- [活动控制](/documentation/articles/resource-group-audit/)
- [资源策略](/documentation/articles/resource-manager-policy/)

### <a name="pitfalls-to-avoid"></a>需避免的错误

请记住为何要开始这次从经典部署模型到 Azure Resource Manager 部署模型的迁移之旅。  最初的业务原因有哪些？ 是否实现了这些业务原因？

## <a name="next-steps"></a>后续步骤

* [平台支持的从经典部署模型到 Azure Resource Manager 部署模型的 IaaS 资源迁移概述](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
* [有关平台支持的从经典部署模型到 Azure Resource Manager 部署模型的迁移的技术深入探讨](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/)
* [使用 PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-windows-ps-migration-classic-resource-manager/)
* [使用 CLI 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-linux-cli-migration-classic-resource-manager/)
* [用于帮助将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型的社区工具](/documentation/articles/virtual-machines-windows-migration-scripts/)
* [查看最常见的迁移错误](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-errors/)
* [查看有关将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager 部署模型的最常见问题](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-faq/)