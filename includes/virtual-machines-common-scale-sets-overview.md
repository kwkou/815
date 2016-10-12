

对于需要扩大和缩小计算资源的应用程序，缩放操作在容错域和更新域之间进行隐式平衡。有关 VM 缩放集的简介，请参阅最近的 [Azure 博客公告](https://azure.microsoft.com/blog/azure-vm-scale-sets-public-preview)。

有关 VM 缩放集的详细信息，请查看这些视频：

 - [Mark Russinovich 谈论 Azure 缩放集](https://channel9.msdn.com/Blogs/Regular-IT-Guy/Mark-Russinovich-Talks-Azure-Scale-Sets/)  

 - [Guy Bowerman 介绍虚拟机缩放集](https://channel9.msdn.com/Shows/Cloud+Cover/Episode-191-Virtual-Machine-Scale-Sets-with-Guy-Bowerman)

## 创建和管理 VM 缩放集

可以使用 JSON 模板和 [REST API](https://msdn.microsoft.com/zh-cn/library/mt589023.aspx) 定义和部署 VM 缩放集，就像定义和部署单个 Azure 资源管理器 VM 一样。因此，可以使用任何标准的 Azure 资源管理器部署方法。有关模板的详细信息，请参阅[创作 Azure 资源管理器模板](../resource-group-authoring-templates.md)。

可以在此处的 Azure 快速入门模板 GitHub 存储库中找到一组 VM 缩放集示例模板：

[https://github.com/Azure/azure-quickstart-templates](https://github.com/Azure/azure-quickstart-templates) - 使用标题中 _vmss_ 查找模板。

在这些模板的详细页中包含一个按钮，使用该按钮可链接到门户部署功能。若要部署 VM 缩放集，请单击该按钮，然后填写门户中所需的任何参数。如果你不确定某个资源是否支持大写或大小写混合，则更为安全的做法是始终使用小写参数值。此外，此处还对 VM 缩放集模板进行了方便的视频解剖：

[VM 缩放集模板详细分析](https://channel9.msdn.com/Blogs/Windows-Azure/VM-Scale-Set-Template-Dissection/player)

## 扩大和缩小 VM 缩放集

若要增加或减少 VM 缩放集中的虚拟机数目，只需更改 _capacity_ 属性并重新部署模板。这种简单性可让你在想要定义 Azure 自动缩放不支持的自定义缩放事件时轻松编写自己的自定义缩放层。

如果要重新部署模板以更改容量，则可以定义只包括 SKU 和已更新容量的更小模板。以下是一个相关示例：[https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-scale-in-or-out/azuredeploy.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-linux-nat/azuredeploy.json)。

若要浏览创建自动缩放的缩放集的步骤，请参阅[自动缩放虚拟机缩放集中的虚拟机](/documentation/articles/virtual-machines-windows-ps-vmss-create/)

## 监视 VM 缩放集

目前建议使用 [Azure 资源浏览器](https://resources.azure.com)来查看 VM 缩放集。VM 缩放集是 Microsoft.Compute 下的资源，因此，你可以通过在此站点中展开以下链接来查看它们：

	subscriptions -> your subscription -> resourceGroups -> providers -> Microsoft.Compute -> virtualMachineScaleSets -> your VM scale set -> etc.

## VM 缩放集方案

本部分列出了一些典型的 VM 缩放集方案。一些高级 Azure 服务（如 Batch（批处理）、Service Fabric、Azure 容器服务）将使用这些方案。

 - **通过 RDP/SSH 连接到 VM 缩放集实例** - VM 缩放集是在 VNET 中创建的，并且没有为其中单独的 VM 分配公共 IP 地址。这是一件好事，因为你通常不希望承担为计算网格中的所有无状态资源分配单独的 IP 地址而产生支出和管理开销，并且你可以轻松地从 VNET 中的其他资源（包括负载均衡器或独立虚拟机等具有公共 IP 地址的资源）连接到这些 VM。

 - **使用 NAT 规则连接到 VM** - 可以创建一个公共 IP 地址，并将其分配给负载均衡器，然后定义入站 NAT 规则，用于将 IP 地址上的端口映射到 VM 缩放集中的 VM 上的端口。例如

	Public IP->Port 50000 -> vmss\_0->Port 22
	Public IP->Port 50001 -> vmss\_1->Port 22
	Public IP->Port 50002-> vmss\_2->Port 22


	下面的示例创建了一个 VM 缩放集，该缩放集使用 NAT 规则，通过单个公共 IP 实现与缩放集中每个 VM 的 SSH 连接：[https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-nat](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-nat)

	下面的示例使用 RDP 和 Windows 实现相同的目的：[https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-nat](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-nat)

 - **使用“jumpbox”连接到 VM** - 如果在同一个 VNET 中创建一个 VM 缩放集和一个独立 VM，则该独立 VM 和 VM 缩放集 VM 能够使用其由 VNET/子网定义的内部 IP 地址彼此连接。如果创建一个公共 IP 地址并将其分配给独立 VM，则可以通过 RDP 或 SSH 连接到该独立 VM，然后从该虚拟机连接到 VM 缩放集实例。此时你可能会发现，与使用其默认配置中的公共 IP 地址的简单独立 VM 相比，简单的 VM 缩放集本质上更安全。

	作为此方法的一个示例，此模板创建了一个简单的 Mesos 群集，其中包含一个独立的主 VM，用于管理由 VM 组成的基于 VM 缩放集的群集：[https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json](https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json)

 - **使用轮循机制负载均衡到 VM 缩放集实例** - 如果你想要使用“轮循机制”方法向 VM 的计算群集交付工作，则可以使用负载均衡规则对 Azure 负载均衡器进行相应的配置。你还可以定义探测，通过使用指定的协议、间隔和请求路径对端口执行 ping 操作来验证应用程序是否正在运行。

	下面的示例创建由在 IIS Web 服务器上运行的 VM 组成的 VM 缩放集，并使用负载均衡器来平衡每个 VM 接收的负载。它还使用 HTTP 协议对每个 VM 上的特定 URL 执行 ping 操作：[https://github.com/gbowerman/azure-myriad/blob/master/vmss-win-iis-vnet-storage-lb.json](https://github.com/gbowerman/azure-myriad/blob/master/vmss-win-iis-vnet-storage-lb.json) -查看 Microsoft.Network/loadBalancers 资源类型以及 virtualMachineScaleSet 中的 networkProfile 和 extensionProfile。

 - **在 PaaS 群集管理器中将 VM 缩放集部署为计算群集** - VM 缩放集有时作为下一代辅助角色进行说明。这是有效的描述，但也可能导致将缩放集功能与 PaaS v1 辅助角色功能混淆。在某种意义上，VM 缩放集提供真正的“辅助角色”或辅助角色资源，并在此资源中提供独立于平台/运行时且集成到 Azure 资源管理器 IaaS 中的可自定义通用计算资源。

	PaaS v1 辅助角色虽然在平台/运行时支持方面受到限制（仅 Windows 平台映像），但它也包括多项服务，如 VIP 交换，可配置升级设置，以及_尚未_在 VM 缩放集中提供，或者将由 Service Fabric 等其他更高级别 PaaS 服务提供的特定于运行时/应用部署的设置。考虑到这一点，你可以将 VM 缩放集视为支持 PaaS 的基础结构。即，生成 Service Fabric 等 PaaS 解决方案或 Mesos 等群集管理器时，可以在将 VM 缩放集作为可缩放计算层的基础上进行生成。

	下面的示例演示 Mesos 群集，它在同一 VNET 中将 VM 缩放集部署为一个独立 VM。独立 VM 是一个 Mesos 主机，而 VM 缩放集则表示一组从属节点：[https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json](https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json)。[Azure 容器服务](https://azure.microsoft.com/blog/azure-container-service-now-and-the-future/)的将来版本将基于 VM 缩放集部署此方案的更复杂/更强化版本。

## VM 缩放集性能和缩放指南

- 在公共预览期间，不要一次在多个 VM 缩放集中创建超过 500 个 VM。
- 为每个存储帐户规划不超过 40 个 VM。
- 使存储帐户名称的第一个字母尽可能的彼此不同。[Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/)中的示例 VMSS 模板提供了如何执行此操作的示例。
- 如果使用自定义 VM，请为单个存储帐户中的每个 VM 缩放集规划不超过 40 VM。你将需要预先将映像复制到存储帐户，然后才能开始进行 VM 缩放集部署。有关详细信息，请参阅 FAQ。
- 为每个 VNET 规划不超过 2048 个 VM。将来会增加此限制。
- 可创建的 VM 数目受到任何区域中计算核心配额的限制。即使你对用于云服务或 IaaS v1 的核心已具有较高的限制，也仍可能需要联系客户支持才可增加计算配额限制。要查询你的配额，可运行 Azure CLI 命令 _azure vm list-usage_ 和 PowerShell 命令 _Get-AzureRmVMUsage_（如果使用的 PowerShell 版本低于 1.0，则使用 _Get-AzureVMUsage_）。

## VM 缩放集常见问题

**Q.** VM 缩放集中可以有多少个 VM？

**A.** 如果使用可分布在多个存储帐户中的平台映像，则可以有 100 台。如果使用自定义映像，则最多可以有 40 台，因为在预览期间，自定义映像限于单个存储帐户。

**Q** VM 缩放集还存在哪些其他资源限制？

**A.** 在预览期间，你仅可在每个区域的多个缩放集中创建不超过 500 个 VM。现有的 [Azure 订阅服务限制](../azure-subscription-service-limits.md)将适用。

**Q.** VM 缩放集内是否支持数据磁盘？

**A.** 在首次发行时不支持。可用于存储数据的选项包括：

- Azure 文件（SMB 共享驱动器）

- OS 驱动器

- 临时驱动器（本地，不受 Azure 存储的支持）

- 外部数据服务（如远程 DB、Azure 表、Azure blob）

**Q.** 哪些 Azure 区域支持 VM 缩放集？

**A.** 支持 Azure 资源管理器的任何区域均支持 VM 缩放集。

**Q.** 如何使用自定义映像创建 VM 缩放集？

**A.** 将 vhdContainers 属性留空，例如：

	"storageProfile": {
		"osDisk": {
			"name": "vmssosdisk",
			"caching": "ReadOnly",
			"createOption": "FromImage",
			"image": {
				"uri": [https://mycustomimage.blob.core.windows.net/system/Microsoft.Compute/Images/mytemplates/template-osDisk.vhd](https://mycustomimage.blob.core.windows.net/system/Microsoft.Compute/Images/mytemplates/template-osDisk.vhd)
			},
			"osType": "Windows"
		}
	},


**Q.** 如果我将 VM 缩放集容量从 20 减少到 15，将删除哪些 VM？

**A.** 将从跨升级域和容错域的缩放集中均匀地删除虚拟机，以最大限度地提高可用性。

**Q.** 如果之后我再将容量从 15 增加到 18，又将发生什么情况？

**A.** 如果将容量增加到 18，将创建索引为 15、16 和 17 的 VM。在这两种情况下，FD 与 UD 中的 VM 都是平衡的。

**Q.** 在一个 VM 缩放集中使用多个扩展时，是否可以强制规定一个执行序列？

**A.** 不能直接规定，但对于 customScript 扩展，你的脚本可以等待另一个扩展完成（例如，通过监视扩展日志 - 请参阅 [https://github.com/Azure/azure-quickstart-templates/blob/master/201-vmss-lapstack-autoscale/install\_lap.sh](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vmss-lapstack-autoscale/install_lap.sh)）。

**Q.** VM 缩放集是否可与 Azure 可用性集配合使用？

**A.** 是的。VM 缩放集是含有 3 个 FD 和 5 个 UD 的隐式可用性集。无需在 virtualMachineProfile 下进行任何配置。在将来的版本中，VM 缩放集可能跨多个租户，但目前一个缩放集只是单个可用性集。

<!---HONumber=Mooncake_0104_2016-->