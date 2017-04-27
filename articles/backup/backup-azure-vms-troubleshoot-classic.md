<properties
    pageTitle="排查经典管理门户中的 Azure 备份错误 | Azure"
    description="对经典管理门户中 Azure 虚拟机的 Azure 备份和还原进行故障排除。"
    services="backup"
    documentationcenter=""
    author="trinadhk"
    manager="shreeshd"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="117201fb-c0cd-4be4-b47f-abd88fe914cf"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="1/23/2017"
    wacn.date="04/24/2017"
    ms.author="trinadhk;markgal;"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="5b9c12136a8bd6e34c3f2ce1c0a05faefc2d5738"
    ms.lasthandoff="04/14/2017" />

# <a name="troubleshoot-azure-virtual-machine-backup"></a>Azure 虚拟机备份疑难解答
> [AZURE.SELECTOR]
- [恢复服务保管库](/documentation/articles/backup-azure-vms-troubleshoot/)
- [备份保管库](/documentation/articles/backup-azure-vms-troubleshoot-classic/)

可参考下表中所列的信息，排查使用 Azure 备份时遇到的错误。

## <a name="discovery"></a>发现
| 备份操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 发现 |无法发现新项 - Azure 备份发生内部错误。 等候几分钟时间，然后重试操作。 |在 15 分钟后重试发现过程。 |
| 发现 |无法发现新项 - 另一个发现操作正在进行。 请等到当前发现操作完成。 |无 |

## <a name="register"></a>注册
| 备份操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 注册 |附加到虚拟机的数据磁盘数超过了支持的限制 - 请分离此虚拟机上的某些数据磁盘，然后重试操作。 Azure 备份最多支持将 16 个数据磁盘附加到 Azure 虚拟机进行备份 |无 |
| 注册 |Azure 备份遇到内部错误 - 等候几分钟，然后重试操作。 如果问题持续出现，请联系 Microsoft 支持。 |出现此错误的原因是，Premium LRS 上有以下其中一项不受支持的 VM 配置。 <br> 高级存储 VM 可以使用恢复服务保管库进行备份。 [了解详细信息](/documentation/articles/backup-introduction-to-azure-backup/#using-premium-storage-vms-with-azure-backup/) |
| 注册 |安装代理操作超时，注册失败 |检查是否支持虚拟机的操作系统版本。 |
| 注册 |命令执行失败 - 此项上正在进行另一项操作。 请等待前一项操作完成 |无 |
| 注册 |不支持对虚拟硬盘存储在高级存储上的虚拟机进行备份 |无 |
| 注册 |虚拟机上不存在虚拟机代理 - 请安装所需的系统必备组件和 VM 代理，然后重新启动操作。 |[详细了解](#vm-agent) 如何安装 VM 代理以及如何验证 VM 代理安装。 |

## <a name="backup"></a>备份
| 备份操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 备份 |无法与 VM 代理通信，因此无法获取快照状态。 快照 VM 子任务超时。 请参阅故障排除指南以了解如何解决此问题。 |如果 VM 代理出现问题，或以某种方式阻止了对 Azure 基础结构的网络访问，则会引发此错误。 详细了解如何[调试 VM 快照问题](/documentation/articles/backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout/)。 <br> 如果 VM 代理未导致任何问题，则重新启动 VM。 有时 VM 状态不正确可能会导致问题，而重新启动 VM 会重置此“错误状态” |
| 备份 |发生内部错误，备份失败 - 请在几分钟后重试操作。 如果问题仍然存在，请联系 Microsoft 支持 |请检查访问 VM 存储时是否存在暂时性问题。 请检查 [Azure 状态](https://azure.microsoft.com/en-us/status/) ，以确定区域中是否存在与计算/存储/网络相关的任何问题。 解决问题后，请重试备份。 |
| 备份 |无法执行该操作，因为 VM 不复存在。 |无法执行备份，因为已删除针对备份配置的 VM。 请转到“受保护的项”视图，选择受保护的项，单击“停止保护”，停止进一步备份操作。 选择“保留备份数据”选项即可保留数据。 之后可以单击“已注册项”视图中的“配置保护”，继续保护此虚拟机 |
| 备份 |无法在选择的项上安装 Azure 恢复服务扩展 - VM 代理是 Azure 恢复服务扩展的必备组件。 请安装 Azure VM 代理并重新启动注册操作 |<ol> <li>检查是否已正确安装 VM 代理。 <li>确定已正确设置 VM 配置中的标志。</ol> [详细了解](#validating-vm-agent-installation)如何安装 VM 代理以及如何验证 VM 代理安装。 |
| 备份 |命令执行失败 - 此项上当前正在进行另一项操作。 请等待前一项操作完成，然后重试 |VM 的现有备份或还原作业正在运行，而当现有作业正在运行时，无法启动新的作业。 |
| 备份 |扩展安装失败，出现错误“COM+ 无法与 Microsoft 分布式事务处理协调器通信”。 |这通常意味着 COM+ 服务未运行。 请与 Microsoft 支持部门联系，以获取解决此问题所需的帮助。 |
| 备份 |快照操作失败，出现 VSS 操作错误"此驱动器已通过 BitLocker 驱动器加密锁定”。 必须通过控制面板解锁此驱动器。 |关闭 VM 上所有驱动器的 BitLocker，观察 VSS 问题是否得到解决 |
| 备份 |不支持对虚拟硬盘存储在高级存储上的虚拟机进行备份 |无 |
| 备份 |找不到 Azure 虚拟机。 |当主 VM 已删除，而备份策略仍继续查找用于执行备份的 VM 时，会发生这种情况。 若要修复此错误，请执行以下操作： <ol><li>使用相同的名称和相同的资源组名称 [云服务名称] 重新创建虚拟机， <br>（或者） <li> 禁用对此 VM 的保护，这样就不会触发后续的备份。 </ol> |
| 备份 |虚拟机上不存在虚拟机代理 - 请安装所需的系统必备组件和 VM 代理，然后重新启动操作。 |[详细了解](#vm-agent) 如何安装 VM 代理以及如何验证 VM 代理安装。 |

## <a name="jobs"></a>作业
| 操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 取消作业 |此作业类型不支持取消操作 - 请等待作业完成。 |无 |
| 取消作业 |作业未处于可取消状态 - 请等待作业完成。 <br>或<br> 所选作业未处于可取消状态 - 请等待作业完成。 |作业很可能已快完成；请等待作业完成 |
| 取消作业 |不能取消作业，因为作业并未处于进行状态 - 只能对正在进行的作业执行取消操作。 请尝试取消正在进行的作业。 |如果存在临时状态，则可能会发生这种情况。 请稍等片刻，然后重试取消操作 |
| 取消作业 |无法取消作业 - 请等待作业完成。 |无 |

## <a name="restore"></a>还原
| 操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 还原 |发生云内部错误，还原失败 |<ol><li>使用 DNS 设置配置了你正在尝试还原的云服务。可检查 <br>$deployment = Get-AzureDeployment -ServiceName "ServiceName" -Slot "Production"     Get-AzureDns -DnsSettings $deployment.DnsSettings<br>如果配置了地址，则表示配置了 DNS 设置。<br> <li>尝试还原的云服务配置了 ReservedIP，且云服务中现有的 VM 处于停止状态。<br>可以使用以下 PowerShell cmdlet 检查云服务是否有保留的 IP：<br>$deployment = Get-AzureDeployment -ServiceName "servicename" -Slot "Production" $dep.ReservedIPName <br><li>正在尝试还原同一云服务中具有以下特殊网络配置的虚拟机。 <br>- 采用负载均衡器配置的虚拟机（内部和外部）<br>- 具有多个保留 IP 的虚拟机<br>- 具有多个 NIC 的虚拟机<br>请在 UI 中选择新的云服务，或者参阅[还原注意事项](/documentation/articles/backup-azure-restore-vms/#restoring-vms-with-special-network-configurations/)，了解具有特殊网络配置的 VM</ol> |
| 还原 |所选 DNS 名称已被使用 - 请指定其他 DNS 名称，然后重试。 |此处的 DNS 名称是指云服务名称（通常以 .chinacloudapp.cn 结尾）。 此名称必须是唯一名称。 如果遇到此错误，则需在还原过程中选择其他 VM 名称。 <br><br> 此错误仅向 Azure 门户预览用户显示。 通过 PowerShell 进行的还原操作会成功，因为它只还原磁盘，而不创建 VM。 如果在磁盘还原操作之后显式创建 VM，则会遇到该错误。 |
| 还原 |指定的虚拟网络配置不正确 - 请指定其他虚拟网络配置，然后重试。 |无 |
| 还原 |指定的云服务使用的是保留 IP，这不符合要还原的虚拟机的配置 - 请指定其他不使用保留 IP 的云服务，或者选择其他用于还原的恢复点。 |无 |
| 还原 |云服务已达到输入终结点的数目限制 - 请指定其他云服务或使用现有终结点，重新尝试该操作。 |无 |
| 还原 |备份保管库和目标存储帐户位于两个不同的区域 - 请确保还原操作中指定的存储帐户与备份保管库位于相同的 Azure 区域。 |无 |
| 还原 |不支持为还原操作指定的存储帐户 - 仅支持具有本地冗余或地域冗余复制设置的“基本/标准”存储帐户。 请选择支持的存储帐户 |无 |
| 还原 |针对还原操作指定的存储帐户类型不处于在线状态 - 请确保在还原操作中指定的存储帐户处于在线状态 |Azure 存储中出现暂时性错误或中断时，可能会发生这种情况。 请选择另一个存储帐户。 |
| 还原 |已达到资源组配额限制 - 请从 Azure 门户预览中删除某些资源组，或者与 Azure 支持部门联系，请求他们提高限制。 |无 |
| 还原 |所选子网不存在 - 请选择已存在的子网 |无 |

## <a name="policy"></a>策略
| 操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 创建策略 |无法创建策略 - 请减少保留选项数，以便继续进行策略配置。 |无 |

## <a name="vm-agent"></a>VM 代理
### <a name="setting-up-the-vm-agent"></a>设置 VM 代理
通常，VM 代理已存在于从 Azure 库创建的 VM 中。 但是，从本地数据中心迁移的虚拟机上未安装 VM 代理。 对于此类 VM，必须显式安装 VM 代理。 阅读有关 [在现有 VM 上安装 VM 代理](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)的详细信息。

对于 Windows VM：

- 下载并安装 [代理 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。 需要管理员权限才能完成安装。
- [更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) ，指明已安装代理。

对于 Linux VM：

- 从 github 安装最新 [Linux 代理](https://github.com/Azure/WALinuxAgent) 。
- [更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) ，指明已安装代理。

### <a name="updating-the-vm-agent"></a>更新 VM 代理
对于 Windows VM：

- 更新 VM 代理与重新安装 [VM 代理二进制文件](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)一样简单。 但是，需要确保在更新 VM 代理时，没有任何正在运行的备份作业。

对于 Linux VM：

- 按照[更新 Linux VM 代理](/documentation/articles/virtual-machines-linux-update-agent/)上的说明进行操作。

### <a name="validating-vm-agent-installation"></a>验证 VM 代理安装
如何检查 Windows VM 上的 VM 代理版本：

1. 登录 Azure 虚拟机并导航到 *C:\WindowsAzure\Packages* 文件夹。 应会发现 WaAppAgent.exe 文件已存在。
2. 右键单击该文件，转到“**属性**”，然后选择“**详细信息**”选项卡。 “产品版本”字段应为 2.6.1198.718 或更高

<!---Update_Description: wording update -->