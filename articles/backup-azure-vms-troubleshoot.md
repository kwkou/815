<properties
	pageTitle="Azure 虚拟机备份疑难解答 | Azure"
	description="Azure 虚拟机备份和还原疑难解答"
	services="backup"
	documentationCenter=""
	authors="trinadhk"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.date="01/28/2016"
	wacn.date="04/12/2016"/>



# Azure 虚拟机备份疑难解答
你可以参考下表中所列的信息，排查使用 Azure 备份时遇到的错误。

## 发现

| 备份操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 发现 | 无法发现新项 - Microsoft Azure 备份发生内部错误。等候几分钟时间，然后重试操作。 | 在 15 分钟后重试发现过程。
| 发现 | 无法发现新项 - 另一个发现操作正在进行。请等到当前发现操作完成。 | 无 |

## 注册
| 备份操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 注册 | 附加到虚拟机的数据磁盘数超过了支持的限制 - 请分离此虚拟机上的某些数据磁盘，然后重试操作。Azure 备份最多支持将 16 个数据磁盘附加到 Azure 虚拟机进行备份 | 无 |
| 注册 | Microsoft Azure 备份遇到内部错误 - 等候几分钟，然后重试操作。如果问题持续出现，请联系 Microsoft 支持。 | 可能因为不支持以下其中一项配置而发生此错误：<ul><li>Premium LRS </ul> |
| 注册 | 安装代理操作超时，注册失败 | 检查是否支持虚拟机的操作系统版本。 |
| 注册 | 命令执行失败 - 此项上正在进行另一项操作。等到前一项操作完成 | 无 |
| 注册 | 不支持使用虚拟硬盘存储在高级存储上的虚拟机进行备份 | 无 |
| 注册 | 虚拟机上不存在虚拟机代理 - 请安装所需的系统必备组件和 VM 代理，然后重新启动操作。 | [详细了解](#vm-agent)如何安装 VM 代理以及如何验证 VM 代理安装。 |

## 备份

| 备份操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 备份 | 从备份保管库复制 VHD 超时 - 请在几分钟后重试操作。如果问题持续出现，请联系 Microsoft 支持。 | 要复制的数据太多时会发生此问题。请检查你的数据磁盘是否少于 16 个。 |
| 备份 | 无法与 VM 代理通信，因此无法获取快照状态。快照 VM 子任务超时。请参阅故障排除指南以了解如何解决此问题。 | 如果 VM 代理出现问题，或以某种方式阻止了对 Azure 基础结构的网络访问，则会引发此错误。<ul><li>了解如何[调试 VM 代理问题](#vm-agent)<li>了解如何[调试网络问题](#networking)<li>如果 VM 代理正常运行，了解如何[排查 VM 快照问题](#Troubleshoot-VM-Snapshot-Issues)</ul><br>如果 VM 代理未造成任何问题，则重新启动 VM。有时 VM 状态不正确可能会导致问题，而重新启动 VM 则会重置此“错误状态” |
| 备份 | 发生内部错误，备份失败 - 请在几分钟后重试操作。如果问题持续出现，请联系 Microsoft 支持 | 发生此错误的原因有 2 个：<ol><li>要复制的数据太多。<li>已删除原始 VM，因此无法进行备份。若要保留已删除 VM 的备份数据但要防止备份错误，请取消保护 VM 并选择保留数据。这样即可停止备份计划以及重复出现的错误消息。 |
| 备份 | 无法在选择的项上安装 Azure 恢复服务扩展 - VM 代理是 Azure 恢复服务扩展的必备组件。请安装 Azure VM 代理并重新启动注册操作 | <ol> <li>检查是否已正确安装 VM 代理。<li>确定已正确设置 VM 配置中的标志。</ol> [详细了解](#validating-vm-agent-installation)如何安装 VM 代理以及如何验证 VM 代理安装。 |
| 备份 | 命令执行失败 - 此项上当前正在进行另一项操作。请等到前一项操作完成，然后重试 | VM 的现有备份或还原作业正在运行，而当现有作业正在运行时，无法启动新的作业。 |
| 备份 | 扩展安装失败，出现错误“COM+ 无法与 Microsoft 分布式事务处理协调器通信”。 | 这通常意味着到 COM+ 服务未运行。请与 Microsoft 支持部门联系，以获取解决此问题所需的帮助。 |
| 备份 | 快照操作失败，出现 VSS 操作错误"此驱动器已通过 BitLocker 驱动器加密锁定”。你必须通过控制面板解锁此驱动器。 | 针对 VM 上的所有驱动器关闭 BitLocker，观察 VSS 问题是否得到解决 |
| 备份 | 不支持使用虚拟硬盘存储在高级存储上的虚拟机进行备份 | 无 |
| 备份 | 找不到 Azure 虚拟机。 | 当主 VM 已删除，而备份策略仍继续查找用于执行备份的 VM 时，将会发生这种情况。若要修复此错误，请执行以下操作：<ol><li>使用相同的名称和相同的资源组名称 [云服务名称] 重新创建虚拟机，<br>或者<li>禁用对此 VM 的保护，这样就不会创建备份作业</ol> |
| 备份 | 虚拟机上不存在虚拟机代理 - 请安装所需的系统必备组件和 VM 代理，然后重新启动操作。 | [详细了解](#vm-agent)如何安装 VM 代理以及如何验证 VM 代理安装。 |

## 作业
| 操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 取消作业 | 此作业类型不支持取消操作 - 请等待作业完成。 | 无 |
| 取消作业 | 作业未处于可取消状态 - 请等待作业完成。<br>或者<br> 所选作业未处于可取消状态 - 请等待作业完成。| 作业很可能已快完成；请等待作业完成 |
| 取消作业 | 不能取消作业，因为作业并未处于进行状态 - 只能对处于进行状态的作业执行取消操作。请尝试取消正在进行的作业。 | 如果存在临时状态，则可能会发生这种情况。请稍等片刻，然后重试取消操作 |
| 取消作业 | 无法取消作业 - 请等待作业完成。 | 无 |


## <a name="restore"></a>还原
| 操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 还原 | 发生云内部错误，还原失败 | <ol><li>使用 DNS 设置配置了你正在尝试还原的云服务。你可以检查 <br>$deployment = Get-AzureDeployment -ServiceName "ServiceName" -Slot "Production" Get-AzureDns -DnsSettings $deployment.DnsSettings<br>如果配置了地址，则表示配置了 DNS 设置。<br> <li>尝试还原的云服务配置了 ReservedIP，且云服务中现有的 VM 处于停止状态。<br>可以使用以下 PowerShell cmdlet 检查云服务是否有保留的 IP：<br>$deployment = Get-AzureDeployment -ServiceName "servicename" -Slot "Production" $dep.ReservedIPName <br><li>你正在尝试将具有以下特殊网络配置的虚拟机还原到同一个云服务中。<br>- 虚拟机采用负载平衡器配置（内部和外部）<br>- 虚拟机具有多个保留 IP<br>- 虚拟机具有多个 NIC<br>请在 UI 中选择新的云服务，或者参阅[还原注意事项](/documentation/articles/backup-azure-restore-vms/#restoring-vms-with-special-network-configurations)了解具有特殊网络配置的 VM</ol> |
| 还原 | 所选 DNS 名称已被使用 - 请指定其他 DNS 名称，然后重试。 | 此处的 DNS 名称是指云服务名称（通常以 .chinacloudapp.cn 结尾）。此名称必须是唯一名称。如果遇到此错误，则需在还原过程中选择其他 VM 名称。<br><br>请注意，此错误仅显示给 Azure 经典管理门户用户。通过 PowerShell 进行的还原操作会成功，因为它只还原磁盘，不创建 VM。如果在磁盘还原操作之后显式创建 VM，则会遇到该错误。 |
| 还原 | 指定的虚拟网络配置不正确 - 请指定其他虚拟网络配置，然后重试。 | 无 |
| 还原 | 指定的云服务使用的是保留 IP，这不符合要还原的虚拟机的配置 - 请指定其他不使用保留 IP 的云服务，或者选择其他用于还原的恢复点。 | 无 |
| 还原 | 云服务已达到输入终结点的数目限制 - 请重新尝试该操作，指定其他云服务或使用现有终结点。 | 无 |
| 还原 | 备份保管库和目标存储帐户位于两个不同的区域 - 请确保在还原操作中指定的存储帐户与备份保管库位于相同的 Azure 区域。 | 无 |
| 还原 | 不支持为还原操作指定的存储帐户 - 仅支持具有本地冗余或地域冗余复制设置的“基本/标准”存储帐户。请选择支持的存储帐户 | 无 |
| 还原 | 针对还原操作指定的存储帐户类型不处于在线状态 - 请确保在还原操作中指定的存储帐户处于在线状态 | 在 Azure 存储空间中出现暂时性错误或断电时，可能会发生这种情况。请选择另一个存储帐户。 |
| 还原 | 已达到资源组配额限制 - 请从 Azure 经典管理门户中删除某些资源组，或者与 Azure 支持部门联系，请求他们提高限制。 | 无 |
| 还原 | 所选子网不存在 - 请选择已存在的子网 | 无 |


## 策略
| 操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 创建策略 | 无法创建策略 - 请减少保留选项数，以便继续进行策略配置。 | 无 |


## <a name="vm-agent"></a>VM 代理

### 设置 VM 代理
通常，VM 代理已存在于从 Azure 库创建的 VM 中。但是，从本地数据中心迁移的虚拟机上未安装 VM 代理。对于此类 VM，必须显式安装 VM 代理。阅读有关[在现有 VM 上安装 VM 代理](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)的详细信息。

对于 Windows VM：

- 下载并安装[代理 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。你需要有管理员权限才能完成安装。
- [更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。

对于 Linux VM：

- 从 github 安装最新 [Linux 代理](https://github.com/Azure/WALinuxAgent)。 
- [更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。


### 更新 VM 代理
对于 Windows VM：

- 更新 VM 代理与重新安装 [VM 代理二进制文件](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)一样简单。但是，需要确保在更新 VM 代理时，没有任何正在运行的备份作业。

对于 Linux VM：

- 按照[更新 Linux VM 代理](/documentation/articles/virtual-machines-linux-update-agent/)上的说明进行操作。 


### <a name="validating-vm-agent-installation"></a>验证 VM 代理安装
如何检查 Windows VM 上的 VM 代理版本：

1. 登录 Azure 虚拟机并导航到 *C:\\WindowsAzure\\Packages* 文件夹。你应会发现 WaAppAgent.exe 文件已存在。
2. 右键单击该文件，转到“属性”，然后选择“详细信息”选项卡。“产品版本”字段应为 2.6.1198.718 或更高

## <a name="Troubleshoot-VM-Snapshot-Issues"></a>排查 VM 快照问题
VM 备份依赖于向底层存储发出快照命令。如果无法访问存储或者快照任务执行延迟，则备份可能会失败。以下因素可能会导致快照任务失败。

1. 使用 NSG 阻止对存储进行网络访问<br>
	详细了解如何使用 IP 允许列表或通过代理服务器对存储[启用网络访问](/documentation/articles/backup-azure-vms-prepare/#2-network-connectivity)。
2.  配置了 SQL Server 备份的 VM 造成快照任务延迟<br>
	默认情况下，VM 备份将在 Windows VM 上发出 VSS 完整备份命令。在运行 SQL Server 且已配置 SQL Server 备份的 VM 上，这可能会造成快照执行延迟。如果由于快照问题而导致备份失败，请设置以下注册表项。

		
		[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
		"USEVSSCOPYBACKUP"="TRUE"
	
3.  由于在 RDP 中关闭了 VM，VM 状态报告不正确。<br>
	如果你在 RDP 中关闭了虚拟机，请返回经典管理门户检查是否正确反映了 VM 的状态。如果没有，请在经典管理门户中使用 VM 仪表板上的“关机”选项关闭 VM。
4.  同一个云服务中的许多 VM 配置为同时备份。<br>
	最佳做法是使同一云服务中的 VM 分开执行不同的备份计划。
5.  VM 在运行时使用了很高的 CPU/内存。<br>
	如果虚拟机在运行时使用了很高的 CPU（使用率超过 90%）或内存，快照任务将排入队列、延迟并最终超时。在这种情况下，请尝试进行按需备份。

<br>

## <a name="networking"></a>联网
与所有扩展一样，备份扩展也需要访问公共 Internet 才能工作。无法访问公共 Internet 可能会出现以下各种情况：

- 扩展安装可能失败
- 备份操作（如磁盘快照）可能失败
- 显示备份操作状态可能失败

[此处](http://blogs.msdn.com/b/mast/archive/2014/06/18/azure-vm-provisioning-stuck-on-quot-installing-extensions-on-virtual-machine-quot.aspx)说明了在哪些情况下需要解析公共 Internet 地址。你需要检查 VNET 的 DNS 配置，并确保可以解析 Azure URI。

正确完成名称解析后，还需要提供对 Azure IP 的访问权限。若要取消阻止对 Azure 基础结构的访问，请执行以下步骤之一：

1. 将 Azure 数据中心 IP 范围加入允许列表。
    - 获取要列入允许列表的 [Azure 数据中心 IP](https://www.microsoft.com/en-in/download/details.aspx?id=41653)。
    - 使用 [New-NetRoute](https://technet.microsoft.com/library/zh-cn/hh826148.aspx) cmdlet 取消阻止 IP。在 Azure VM 上提升权限的 PowerShell 窗口中运行此 cmdlet（以管理员身份运行）。
    - 向 NSG 添加规则（如果已创建规则），以允许访问这些 IP。
2. 为 HTTP 流量创建路径
    - 如果你指定了某种网络限制（例如网络安全组），请部署 HTTP 代理服务器来路由流量。可在[此处](/documentation/articles/backup-azure-vms-prepare/#2-network-connectivity)找到部署 HTTP 代理服务器的步骤。
    - 向 NSG 添加规则（如果已创建规则），以允许从 HTTP 代理访问 INTERNET。

>[AZURE.NOTE] 必须在来宾内启用 DHCP，才能正常进行 IaaS VM 备份。如果需要静态专用 IP 地址，你应该通过平台配置该 IP。VM 内的 DHCP 选项应保持启用。查看有关[设置静态内部专用 IP](/documentation/articles/virtual-networks-reserved-private-ip/) 的详细信息。

<!---HONumber=Mooncake_0405_2016-->