本文概述了在 Azure 上运行 Linux 虚拟机 (VM) 的一套经过验证的做法，这些做法注重可扩展性、可用性、可管理性和安全性。Azure 支持运行大量常用的 Linux 分发，包括 CentOS、Debian、Red Hat Enterprise、Ubuntu 和 FreeBSD。有关详细信息，请参阅 [Azure 和 Linux][azure-linux]。

> [AZURE.NOTE] Azure 具有两个不同的部署模型：[Resource Manager][resource-manager-overview] 和经典。本文使用 Resource Manager，Azure 建议将它用于新部署。

建议不要对生产工作负荷使用单个 VM，因为 Azure 上的单个 VM 没有正常运行时间服务级别协议 (SLA)。若要获取 SLA，必须在[可用性集][availability-set]中部署多个 VM。

## 体系结构关系图

在 Azure 中预配 VM 涉及更多移动部件，而不只是 VM 本身。需要考虑的因素有计算、网络和存储。

![[0]][0]  


- **资源组。** [_资源组_][resource-manager-overview]是一个容器，包含相关资源。创建资源组以保存此 VM 的资源。

- **VM**。可以基于已发布的映像列表或上载到 Azure Blob 存储的虚拟硬盘 (VHD) 文件预配 VM。

- **OS 磁盘。** OS 磁盘是存储在 [Azure 存储空间][azure-storage]中的一个 VHD。这意味着它一直存在，即使主机出现故障，也是如此。OS 磁盘是 `/dev/sda1`。

- **临时磁盘。** 使用临时磁盘创建 VM。此磁盘存储在主机的物理驱动器上。它_不_保存在 Azure 存储空间中，并且可能会在重新启动和其他 VM 生命周期事件过程中消失。只使用此磁盘存储临时数据，如页面文件或交换文件。临时磁盘是 `/dev/sdb1`，并在 `/mnt/resource` 或 `/mnt` 中装入。

- **数据磁盘。** [数据磁盘][data-disk]是用于应用程序数据的持久性 VHD。数据磁盘像 OS 磁盘一样，存储在 Azure 存储空间中。

- **虚拟网络 (VNet) 和子网。** Azure 中的每个 VM 都部署在 VNet 中，后者进一步划分为多个子网。

- **公共 IP 地址。** 公共 IP 地址需要与 VM（例如，通过 SSH）进行通信。

- **网络接口 (NIC)**。NIC 使 VM 能够与虚拟网络进行通信。

- **网络安全组 (NSG)**。[NSG][nsg] 用于允许/拒绝到子网的网络流量。可以将 NSG 与单个 NIC 或与子网相关联。如果将 NSG 与一个子网相关联，则 NSG 规则适用于该子网中的所有 VM。
 
- **诊断。** 诊断日志记录对于 VM 管理和故障排除至关重要。

## 建议

### VM 建议

- 建议使用 DS 系列。有关详细信息，请参阅 [Virtual machine sizes][virtual-machine-sizes]（虚拟机大小）。将现有工作负荷移到 Azure 时，最初是使用与本地服务器最匹配的 VM 大小。然后测量与 CPU、内存和每秒磁盘输入/输出操作次数 (IOPS) 有关的实际工作负荷的性能，并根据需要调整大小。此外，如果需要多个 NIC，请注意每种大小的 NIC 限制。

- 当你预配 VM 和其他资源时，必须指定位置。通常，选择离你的内部用户或客户最近的位置。但是，并非所有 VM 大小都可在所有位置中使用。有关详细信息，请参阅 [Services by region][services-by-region]（按区域列出的服务）。若要列出给定位置中可用的 VM 大小，请运行以下 Azure 命令行接口 (CLI) 命令：

        azure vm sizes --location <location>

- 有关选择发布的 VM 映像的信息，请参阅 [Navigate and select Azure virtual machine images][select-vm-image]（浏览和选择 Azure 虚拟机映像）。

### 磁盘和存储建议

- 为获得最佳磁盘 I/O 性能，建议使用[高级存储][premium-storage]，它在固态硬盘 (SSD) 上存储数据。成本取决于预配磁盘的大小。IOPS 和吞吐量（即数据传输速率）也取决于磁盘大小，因此在预配磁盘时，请全面考虑三个因素（容量、IOPS 和吞吐量）。

- 一个存储帐户可以支持 1 到 20 个 VM。

- 添加一个或多个数据磁盘。刚创建的 VHD 尚未格式化，请登录 VM 格式化该磁盘。数据磁盘显示为 `/dev/sdc`、`/dev/sdd` 等等。你可以运行 `lsblk` 以列出块设备，包括磁盘。若要使用数据磁盘，请创建一个分区和文件系统，然后装载磁盘。例如：

        # Create a partition.
        sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.
        
        # Create a file system.
        sudo mkfs -t ext3 /dev/sdc1
        
        # Mount the drive.
        sudo mkdir /data1
        sudo mount /dev/sdc1 /data1

- 如果有大量数据磁盘，请注意存储帐户的总 I/O 限制。有关详细信息，请参阅 [Virtual Machine Disk Limits][vm-disk-limits]（虚拟机磁盘限制）。

- 在添加数据磁盘时，将为磁盘分配逻辑单元号 (LUN) ID。或者，你可以指定 LUN ID，例如，如果你要更换磁盘并要保留相同的 LUN ID，或者你的应用查找特定 LUN ID。但请记住，每个磁盘的 LUN ID 必须唯一。

- 你可能想要更改 I/O 计划程序，以针对固态硬盘 (SSD)（由高级存储使用）的性能进行优化。常见的建议是对 SSD 使用 NOOP 计划程序，但应使用 [iostat] 等工具来监视特定工作负荷的磁盘 I/O 性能。

- 为获得最佳性能，请创建单独的存储帐户来存储诊断日志。标准的本地冗余存储 (LRS) 帐户足以存储诊断日志。


### 网络建议

- 公共 IP 地址可以是动态的或静态的。默认是动态的。

    - 如果需要不会更改的固定 IP 地址（例如，如果需要在 DNS 中创建 A 记录，或者需要将 IP 地址列入允许列表，请保留[静态 IP 地址][static-ip]。

    - 你还可以为 IP 地址创建完全限定域名 (FQDN)。然后，可以在 DNS 中注册指向 FQDN 的 [CNAME 记录][cname-record]。有关详细信息，请参阅[在 Azure 门户预览中创建完全限定域名][fqdn]。

- 所有 NSG 都包含一组[默认规则][nsg-default-rules]，其中包括阻止所有入站 Internet 流量的规则。无法删除默认规则，但其他规则可以覆盖它们。若要启用 Internet 流量，请创建允许特定端口（例如，将端口 80 用于 HTTP）的入站流量的规则。

- 若要启用 SSH，请向 NSG 添加允许 TCP 端口 22 的入站流量的规则。

## 可伸缩性注意事项

- 可以通过[更改 VM 大小][vm-resize]来扩大或缩小 VM。

- 若要水平扩大，请将两个或更多 VM 放入负载均衡器后面的可用性集中。

## 可用性注意事项

- 如前所述，单个 VM 没有 SLA。若要获取 SLA，必须在可用性集中部署多个 VM。

- 你的 VM 可能会受到[计划内维护][planned-maintenance]或[计划外维护][manage-vm-availability]的影响。你可以使用 [VM 重新启动日志][reboot-logs]来确定 VM 重新启动是否是由计划内维护导致的。

- VHD 由 [Azure 存储空间][azure-storage]提供支持，后者将进行复制以实现持久性和可用性。

- 若要防止在正常操作期间意外数据丢失（例如，由于用户错误），则还应使用 [Blob 快照][blob-snapshot]或其他工具实现时间点备份。

## 可管理性注意事项

- **资源组。** 将共享相同生命周期的紧密耦合资源放入同一[资源组][resource-manager-overview]中。资源组可让你以组的形式部署和监视资源，并按资源组汇总计费成本。你还可以删除作为集的资源，这对于测试部署非常有用。为资源指定有意义的名称。这样，可更轻松地找到特定资源并了解其角色。

- **SSH**在创建 Linux VM 之前，生成 2048 位 RSA 公共/专用密钥对。创建 VM 时，使用公钥文件。有关详细信息，请参阅 [How to Use SSH with Linux and Mac on Azure][ssh-linux]（如何在 Azure 中将 SSH 与 Linux 和 Mac 配合使用）。

- **VM 诊断。** 启用监视和诊断，包括基本运行状况指标、诊断基础结构日志和[启动诊断][boot-diagnostics]。如果你的 VM 陷入不可启动状态，启动诊断可帮助你诊断启动失败。有关详细信息，请参阅 [Enable monitoring and diagnostics][enable-monitoring]（启用监视和诊断）。

    以下 CLI 命令可启用诊断：

        azure vm enable-diag <resource-group> <vm-name>

- **停止 VM。** Azure 对“已停止”和“已解除分配”状态进行了区分。当 VM 状态为“已停止”时，将向你收费。已解除分配 VM 时，无需收费。

    使用以下 CLI 命令可解除分配 VM：

        azure vm deallocate <resource-group> <vm-name>

    Azure 门户预览中的“停止”按钮也会解除分配 VM。但是，如果在已登录时通过 OS 关闭，VM 将停止，但_不_会解除分配，因此仍将向你收费。

- **删除 VM。** 如果你删除 VM，则不会删除 VHD。这意味着你可以安全地删除 VM，而不会丢失数据。但是，仍将向你收取存储费用。若要删除 VHD，请从 [Blob 存储][blob-storage]中删除相应文件。

  若要防止意外删除，请使用[资源锁][resource-lock]锁定整个资源组或锁定单个资源（如 VM）。

## 安全注意事项

- 使用 [OSPatching] VM 扩展自动执行 OS 更新。预配 VM 时，安装此扩展。你可以指定安装修补程序的频率以及修补后是否要重新启动。

- 使用[基于角色的访问控制][rbac] (RBAC) 来控制对你部署的 Azure 资源的访问权限。RBAC 允许你将授权角色分配给开发运营团队的成员。例如，“读者”角色可以查看 Azure 资源，但不能创建、管理或删除这些资源。某些角色特定于特定的 Azure 资源类型。例如，“虚拟机执行组件”角色可以执行重启或解除分配 VM、重置管理员密码、创建 VM 等操作。可能对此参考体系结构有用的其他[内置 RBAC 角色][rbac-roles]包括 [DevTest Lab 用户][rbac-devtest]和[网络执行组件][rbac-network]。
- 可将用户分配给多个角色，并且可以创建自定义角色以实现更细化的权限。

    > [AZURE.NOTE] RBAC 不限制已登录到 VM 的用户可以执行的操作。这些权限由来宾 OS 上的帐户类型决定。

- 使用[审核日志][audit-logs]可查看预配操作和其他 VM 事件。

## 解决方案部署

本指南中提供的示例部署使用三种不同的[模板构建基块][blocks]创建：

- 虚拟网络 (VNet)
- 网络安全组 (NSG)
- 虚拟机 (VM)

此参考体系结构使用单个资源组，可以通过单击下面的按钮并接受所有参数的默认值部署此资源组。

<a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fguidance-compute-single-vm%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### 自定义部署

如果需要更改部署以满足你的需求，请按照 [guidance-single-vm][readme] 页中的说明操作。

## 后续步骤

要使[虚拟机 SLA][vm-sla] 适用，必须在一个可用性集中部署两个或更多个实例。

<!-- links -->


[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /documentation/articles/virtual-machines-windows-create-availability-set/
[azure-cli]: /documentation/articles/virtual-machines-command-line-tools/
[azure-linux]: /documentation/articles/virtual-machines-linux-azure-overview/
[azure-storage]: /documentation/articles/storage-introduction/
[blob-snapshot]: /documentation/articles/storage-blob-snapshots/
[blob-storage]: /documentation/articles/storage-introduction/
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /documentation/articles/virtual-machines-linux-about-disks-vhds/
[disk-encryption]: /documentation/articles/azure-security-disk-encryption/
[enable-monitoring]: /documentation/articles/insights-how-to-use-diagnostics/
[fqdn]: /documentation/articles/virtual-machines-linux-portal-create-fqdn/
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /documentation/articles/virtual-machines-linux-manage-availability/
[multi-vm]: /documentation/articles/guidance-compute-multi-vm/
[naming conventions]: /documentation/articles/guidance-naming-conventions/
[nsg]: /documentation/articles/virtual-networks-nsg/
[nsg-default-rules]: /documentation/articles/virtual-networks-nsg/#default-rules
[OSPatching]: https://github.com/Azure/azure-linux-extensions/tree/master/OSPatching
[planned-maintenance]: /documentation/articles/virtual-machines-linux-planned-maintenance/
[premium-storage]: /documentation/articles/storage-premium-storage/
[rbac]: /documentation/articles/role-based-access-control-what-is/
[rbac-roles]: /documentation/articles/role-based-access-built-in-roles/
[rbac-devtest]: /documentation/articles/role-based-access-built-in-roles/#devtest-labs-user
[rbac-network]: /documentation/articles/role-based-access-built-in-roles/#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[Resize-VHD]: https://technet.microsoft.com/zh-cn/library/hh848535.aspx
[Resize virtual machines]: https://azure.microsoft.com/blog/resize-virtual-machines/
[resource-lock]: /documentation/articles/resource-group-lock-resources/
[resource-manager-overview]: /documentation/articles/resource-group-overview/
[select-vm-image]: /documentation/articles/virtual-machines-linux-cli-ps-findimage/
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /documentation/articles/virtual-machines-linux-mac-create-ssh-keys/
[static-ip]: /documentation/articles/virtual-networks-reserved-public-ip/
[storage-price]: /pricing/details/storage/
[virtual-machine-sizes]: /documentation/articles/virtual-machines-linux-sizes/
[vm-disk-limits]: /documentation/articles/azure-subscription-service-limits/#virtual-machine-disk-limits
[vm-resize]: /documentation/articles/virtual-machines-linux-change-vm-size/
[vm-sla]: /support/sla/virtual-machines/
[arm-templates]: /documentation/articles/resource-group-authoring-templates/
[readme]: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm
[azure-powershell-download]: /documentation/articles/powershell-install-configure/
[components]: #Solution-components
[blocks]: https://github.com/mspnp/template-building-blocks
[0]: ./media/guidance-blueprints/compute-single-vm.png "Azure 中的单一 Linux VM 体系结构"

<!---HONumber=Mooncake_1114_2016-->