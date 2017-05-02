<properties
    pageTitle="Azure 虚拟机规模集概述 | Azure"
    description="了解有关 Azure 虚拟机规模集的详细信息"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="gbowerman"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="76ac7fd7-2e05-4762-88ca-3b499e87906e"
    ms.service="virtual-machine-scale-sets"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/10/2017"
    wacn.date="05/02/2017"
    ms.author="guybo"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="6424077cd4f43c7abd83c6c4259923f00ce6e6fe"
    ms.lasthandoff="04/22/2017" />

# <a name="what-are-virtual-machine-scale-sets-in-azure"></a>什么是 Azure 中的虚拟机规模集？
虚拟机规模集是一种 Azure 计算资源，可用于部署和管理一组相同的 VM。 规模集中的所有 VM 采用相同的配置，支持真正的自动缩放 - 无需对 VM 进行预配。 这样就可以更方便地构建面向大型计算、大数据、容器化工作负荷的大规模服务。

对于需要扩大和缩小计算资源的应用程序，缩放操作在容错域和更新域之间进行隐式平衡。 有关规模集的更多介绍，请参阅 [Azure 博客公告](https://azure.microsoft.com/zh-cn/blog/azure-virtual-machine-scale-sets-ga/)。

有关规模集的详细信息，请观看以下视频：

* [Mark Russinovich talks Azure scale sets](https://channel9.msdn.com/Blogs/Regular-IT-Guy/Mark-Russinovich-Talks-Azure-Scale-Sets/)（Mark Russinovich 谈论 Azure 规模集）  
* [Guy Bowerman 介绍虚拟机规模集](https://channel9.msdn.com/Shows/Cloud+Cover/Episode-191-Virtual-Machine-Scale-Sets-with-Guy-Bowerman)

## <a name="creating-and-managing-scale-sets"></a>创建和管理规模集
可以在 [Azure 门户预览](https://portal.azure.cn)中创建规模集，方法是：选择“新建”，然后在搜索栏中键入“规模”。 结果中会列出“虚拟机规模集”。 从这里，可以填写必填字段，自定义和部署规模集。 还可以使用相应的选项根据 CPU 使用率设置基本的自动缩放规则。

也可以使用 JSON 模板与 [REST API](https://msdn.microsoft.com/zh-cn/library/mt589023.aspx) 来定义和部署规模集，就像定义和部署单个 Azure Resource Manager VM 一样。 因此，可以使用任何标准的 Azure Resource Manager 部署方法。 有关模板的详细信息，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。

可在 [Azure 快速入门模板 GitHub 存储库](https://github.com/Azure/azure-quickstart-templates)中找到一组虚拟机规模集的示例模板。 （查找标题中含有 **vmss** 的模板。）

这些模板的详细信息页中提供一个按钮链接用于访问门户部署功能。 若要部署规模集，请单击该按钮，然后填写门户中所需的任何参数。 如果不确定某个资源是否支持大写或大小写混合，则更为安全的做法是参数值使用小写字母和数字。 [VM 规模集模板剖析](https://channel9.msdn.com/Blogs/Azure/VM-Scale-Set-Template-Dissection/player)通过一个简单易懂的视频对规模集模板做了剖析。

>[AZURE.NOTE]
> 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。 例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

## <a name="scaling-a-scale-set-out-and-in"></a>扩大和缩小规模集
若要在 Azure 门户预览中更改规模集的容量，可单击“设置”下的“缩放”部分。 

若要在命令行中更改规模集容量，请在 [Azure CLI](https://github.com/Azure/azure-cli) 中使用 **scale** 命令。 例如，使用以下命令可将规模集设置为 10 个 VM 的容量：

    az vmss scale -g resourcegroupname -n scalesetname --new-capacity 10 

若要通过 PowerShell 在规模集中设置 VM 数，请使用 **Update-AzureRmVmss** 命令：

    $vmss = Get-AzureRmVmss -ResourceGroupName resourcegroupname -VMScaleSetName scalesetname  
    $vmss.Sku.Capacity = 10
    Update-AzureRmVmss -ResourceGroupName resourcegroupname -Name scalesetname -VirtualMachineScaleSet $vmss

若要通过 Azure Resource Manager 模板增加或减少规模集中的虚拟机数，请更改 **capacity** 属性并重新部署模板。 这种操作很简单，因此可以方便地将规模集与 Azure 自动缩放集成，或者在需要定义不受 Azure 自动缩放支持的自定义缩放事件时，方便地编写你自己的自定义缩放层。 

若要重新部署 Azure Resource Manager 模板以更改容量，可以定义一个小得多的模板，只包括 **SKU** 属性数据包和更新的容量。 [下面是一个示例](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing)。

## <a name="autoscale"></a>自动缩放

在 Azure 门户预览中创建规模集时，可以选择性地为规模集配置自动缩放设置。 然后，可以根据平均 CPU 使用率增减 VM 数。 

[Azure Quickstart templates](https://github.com/Azure/azure-quickstart-templates)（Azure 快速启动模板）中的许多规模集模板定义了自动缩放设置。 也可向现有的规模集添加自动缩放设置。 例如，此 Azure PowerShell 脚本向规模集添加基于 CPU 的自动缩放：

    $subid = "yoursubscriptionid"
    $rgname = "yourresourcegroup"
    $vmssname = "yourscalesetname"
    $location = "yourlocation" # e.g. chinaeast

    $rule1 = New-AzureRmAutoscaleRule -MetricName "Percentage CPU" -MetricResourceId /subscriptions/$subid/resourceGroups/$rgname/providers/Microsoft.Compute/virtualMachineScaleSets/$vmssname -Operator GreaterThan -MetricStatistic Average -Threshold 60 -TimeGrain 00:01:00 -TimeWindow 00:05:00 -ScaleActionCooldown 00:05:00 -ScaleActionDirection Increase -ScaleActionValue 1
    $rule2 = New-AzureRmAutoscaleRule -MetricName "Percentage CPU" -MetricResourceId /subscriptions/$subid/resourceGroups/$rgname/providers/Microsoft.Compute/virtualMachineScaleSets/$vmssname -Operator LessThan -MetricStatistic Average -Threshold 30 -TimeGrain 00:01:00 -TimeWindow 00:05:00 -ScaleActionCooldown 00:05:00 -ScaleActionDirection Decrease -ScaleActionValue 1
    $profile1 = New-AzureRmAutoscaleProfile -DefaultCapacity 2 -MaximumCapacity 10 -MinimumCapacity 2 -Rules $rule1,$rule2 -Name "autoprofile1"
    Add-AzureRmAutoscaleSetting -Location $location -Name "autosetting1" -ResourceGroup $rgname -TargetResourceId /subscriptions/$subid/resourceGroups/$rgname/providers/Microsoft.Compute/virtualMachineScaleSets/$vmssname -AutoscaleProfiles $profile1

## <a name="monitoring-your-scale-set"></a>监视规模集
[Azure 门户预览](https://portal.azure.cn)列出规模集并显示其属性。 门户还支持管理操作。 可以针对规模集和规模集中的单个 VM 执行这些操作。 该门户还提供了一个可自定义的资源使用情况图。 

## <a name="scale-set-scenarios"></a>规模集方案
本部分列出了一些典型的规模集方案。 一些高级 Azure 服务（如批处理、Service Fabric 和容器服务）使用这些方案。

* **使用 RDP 或 SSH 连接到规模集实例**：在虚拟网络中创建规模集，不为规模集中的单个 VM 分配公共 IP 地址。 此策略避免了将独立的公共 IP 地址分配给计算网格中的所有节点所需的支出和管理开销。 可以从虚拟网络中的其他资源（例如可以为其分配公共 IP 地址的负载均衡器和独立虚拟机）连接到这些 VM。
* **使用 NAT 规则连接到 VM**：可以创建一个公共 IP 地址，并将其分配给负载均衡器，然后定义入站 NAT 池。 这些操作将 IP 地址上的端口映射到规模集中 VM 上的端口。 例如：

    | 源 | Source Port | 目标 | Destination Port |
    | --- | --- | --- | --- |
    |  公共 IP |端口 50000 |vmss\_0 |端口 22 |
    |  公共 IP |端口 50001 |vmss\_1 |端口 22 |
    |  公共 IP |端口 50002 |vmss\_2 |端口 22 |

   [此示例](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-nat)中定义了 NAT 规则，以便通过单个公共 IP 地址实现与规模集中每个 VM 的 SSH 连接。

   [此示例](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-nat)使用 RDP 和 Windows 实现相同的目的。
* **使用“jumpbox”连接到 VM**：如果在同一个虚拟网络中创建一个规模集和一个独立 VM，则该独立 VM 和规模集 VM 能够使用其由虚拟网络或子网定义的内部 IP 地址彼此连接。 如果创建一个公共 IP 地址并将其分配给独立 VM，可以使用 RDP 或 SSH 连接到该独立 VM。 然后，可从该虚拟机连接到规模集实例。 此时你可能会发现，与使用其默认配置中的公共 IP 地址的简单独立 VM 相比，简单的规模集本质上更安全。

   例如，[此模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-jumpbox)使用一个独立的 VM 部署简单的规模集。 
* **负载均衡到规模集实例**：如果想要使用轮循机制方法向 VM 的计算群集交付工作，可以使用第 4 层负载均衡规则对 Azure 负载均衡器进行相应的配置。 可以定义探测，通过使用指定的协议、间隔和请求路径对端口执行 ping 操作来验证应用程序是否正在运行。 [Azure 应用程序网关](/home/features/application-gateway/)也支持规模集，以及第 7 层和更复杂的负载均衡方案。

   [此示例](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-ubuntu-web-ssl)创建运行 Apache Web 服务器的规模集，并使用负载均衡器来均衡每个 VM 接收的负载。 （查看 Microsoft.Network/loadBalancers 资源类型以及 virtualMachineScaleSet 中的 networkProfile 和 extensionProfile。）

   [此 Linux 示例](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-ubuntu-app-gateway)和[此 Windows 示例](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-app-gateway)使用应用程序网关。  

* **在 PaaS 群集管理器中将规模集部署为计算群集**：规模集有时描述为下一代辅助角色。 这是有效的描述，但也可能导致将规模集功能与 Azure 云服务功能混淆。 在某种意义上，规模集提供真正的辅助角色或辅助角色资源。 规模集是通用计算资源，独立于平台/运行时、可自定义且可集成到 Azure Resource Manager IaaS 中。

   云服务辅助角色虽然在平台/运行时支持方面受到限制（仅限 Windows 平台映像）， 但它也包括多项服务，如 VIP 交换，可配置的升级设置，以及特定于运行时/应用部署的设置。 这些服务*尚未*在规模集中提供，或者由 Azure Service Fabric 等其他更高级别 PaaS 服务提供。 可以将规模集视为支持 PaaS 的基础结构。 PaaS 解决方案（例如 [Service Fabric](/home/features/service-fabric/)）基于该基础结构。

## <a name="scale-set-performance-and-scale-guidance"></a>规模集性能和缩放指南
* 一个规模集最多支持 1,00 个 VM。
* 可以考虑使用 Azure 高级存储而不是 Azure 存储，以便加快 VM 预配速度、提高 VM 预配时间的可预测性，以及改进 I/O 性能。
* 可以创建的 VM 数受到在其中进行部署的区域中核心配额的限制。 即使目前用于 Azure 云服务的核心数上限已较高，也仍可能需要联系客户支持来提高计算配额限制。 若要查询配额，请运行以下 Azure CLI 命令：`azure vm list-usage`。 或者，运行以下 PowerShell 命令：`Get-AzureRmVMUsage`。

## <a name="frequently-asked-questions-for-scale-sets"></a>有关规模集的常见问题
**问：** 可在规模集中包含多少个 VM？

**答：** 一个规模集可以包含 0 到 1,000 个基于平台映像的 VM，或者 0 到 100 个基于自定义映像的 VM。 

**问：** 规模集是否支持数据磁盘？

**答：** 是的。 规模集可以定义适用于集中所有 VM 的附加数据磁盘配置。 可用于存储数据的其他选项包括：

* Azure 文件（SMB 共享驱动器）
* OS 驱动器
* 临时驱动器（本地，不是以 Azure 存储为基础）
* Azure 数据服务（例如 Azure 表、Azure Blob）
* 外部数据服务（例如远程数据库）

**问：** 哪些 Azure 区域支持规模集？

**答：** 所有区域都支持规模集。

**问：** 如何使用自定义映像创建规模集？

**答：** 根据自定义映像 VHD 创建托管磁盘，然后在规模集模板中引用该磁盘。 [下面是一个示例](https://github.com/chagarw/MDPP/tree/master/101-vmss-custom-os)。

**问：** 如果我将规模集容量从 20 减少到 15，将删除哪些 VM？

**答：** 将从跨更新域和容错域的规模集中均匀地删除虚拟机，以最大限度地提高可用性。 首先删除 ID 最大的 VM。

**问：** 如果将容量从 15 增加到 18，会发生什么情况？

**答：** 如果将容量增加到 18，则创建 3 个新 VM。 每增加容量一次，VM 实例 ID 就会从以前的最高值（例如 20、21、22）递增。 容错域与和更新域中的 VM 是均衡的。

**问：** 在一个规模集中使用多个扩展时，是否可以强制规定执行序列？

**答：** 不能直接强制执行，但对于 customScript 扩展，脚本可以等待另一个扩展来完成操作（[例如通过监视扩展日志](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vmss-lapstack-autoscale/install_lap.sh)）。 在 [Extension Sequencing in Azure VM Scale Sets](https://msftstack.wordpress.com/2016/05/12/extension-sequencing-in-azure-vm-scale-sets/)（Azure VM 规模集中的扩展序列）博客文章中可以获取有关扩展序列的其他指导。

**问：** 规模集是否适用于 Azure 可用性集？

**答：** 是的。 规模集是包含 5 个容错域和 5 个更新域的隐式可用性集。 规模集如果包含 100 个以上的 VM，则会跨多个*位置组*，等效于多个可用性集。 有关位置组的详细信息，请参阅[使用大型虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-placement-groups/)。 由 VM 组成的可用性集可以与由 VM 组成的规模集位于相同的虚拟网络中。 常见的配置是将控件节点 VM（经常需要独特的配置）放在可用性集中，将数据节点放在规模集中。

可在 [Azure 虚拟机规模集常见问题](/documentation/articles/virtual-machine-scale-sets-faq/)中找到有关规模集的更多常见问题解答。

<!--Update_Description: wording update-->