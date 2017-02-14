<properties
    pageTitle="Azure 虚拟机备份疑难解答 | Azure"
    description="Azure 虚拟机备份和还原疑难解答"
    services="backup"
    documentationcenter=""
    author="trinadhk"
    manager="shreeshd"
    editor="" />
<tags
    ms.assetid="73214212-57a4-4b57-a2e2-eaf9d7fde67f"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/28/2016"
    wacn.date="01/24/2017"
    ms.author="trinadhk;jimpark;" />


# Azure 虚拟机备份疑难解答

> [AZURE.SELECTOR]
- [恢复服务保管库](/documentation/articles/backup-azure-vms-troubleshoot/)
- [备份保管库](/documentation/articles/backup-azure-vms-troubleshoot-classic/)

可参考下表中所列的信息，排查使用 Azure 备份时遇到的错误。

## 备份 <a name="restore"></a>

| 备份操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 备份 |无法执行该操作，因为 VM 已不存在。- 停止保护虚拟机，但不删除备份数据。如需更多详细信息，请参阅 http://go.microsoft.com/fwlink/?LinkId=808124 |如果主 VM 已删除，而备份策略仍继续查找用于备份的 VM，则会发生这种情况。若要修复此错误，请执行以下操作：<ol><li>使用相同的名称和相同的资源组名称[云服务名称]重新创建虚拟机，<br>（或者）</li><li>停止保护虚拟机（可删除也可保留备份数据）。[更多详细信息](http://go.microsoft.com/fwlink/?LinkId=808124)</li></ol> |
| 备份 |无法与 VM 代理通信来获取快照状态。- 确保 VM 具有 Internet 访问权限。此外，如故障排除指南 (http://go.microsoft.com/fwlink/?LinkId=800034) 中所述更新 VM 代理 |如果 VM 代理出现问题，或以某种方式阻止了对 Azure 基础结构的网络访问，则会引发此错误。了解有关如何调试 VM 快照问题的详细信息。<br> 如果 VM 代理未导致任何问题，则重启 VM。有时 VM 状态不正确可能会导致问题，而重新启动 VM 则会重置此“错误状态” |
| 备份 |恢复服务扩展操作失败。- 确保虚拟机上有最新的虚拟机代理，并且代理服务正在运行。请重试备份操作，如果失败，请与 Microsoft 支持部门联系。 |VM 代理过期会引发此错误。请参阅以下“更新 VM 代理”部分，更新 VM 代理。 |
| 备份 |虚拟机不存在。- 请确保该虚拟机存在，或选择其他虚拟机。 |当主 VM 已删除，而备份策略仍继续查找用于执行备份的 VM 时，会发生这种情况。若要修复此错误，请执行以下操作：<ol><li>使用相同的名称和相同的资源组名称[云服务名称]重新创建虚拟机，<br>（或者）<br></li><li>停止保护虚拟机，但不删除备份数据。[更多详细信息](http://go.microsoft.com/fwlink/?LinkId=808124)</li></ol> |
| 备份 |命令执行失败 - 此项上当前正在进行另一项操作。请等到前一项操作完成，然后重试 |VM 的现有备份或还原作业正在运行，而当现有作业正在运行时，无法启动新的作业。 |
| 备份 |从备份保管库复制 VHD 超时 - 请在几分钟后重试操作。如果问题持续出现，请联系 Microsoft 支持。 |要复制的数据太多时会发生此问题。请检查你的数据磁盘是否少于 16 个。 |
| 备份 |发生内部错误，备份失败 - 请在几分钟后重试操作。如果问题仍然存在，请联系 Microsoft 支持 |导致此错误发生的原因有 2 个：<ol><li>访问 VM 存储时发生暂时性问题。请检查 Azure 状态，确定区域中是否存在与计算、存储或网络相关的任何问题。问题解决后，请重试此备份作业。<li>已删除原始 VM，因此无法获取恢复点。若要保留已删除 VM 的备份数据，但要删除备份错误：请取消保护 VM 并选择保留数据选项。此操作会停止计划备份作业和重复错误消息。 |
| 备份 |无法在选择的项上安装 Azure 恢复服务扩展 - VM 代理是 Azure 恢复服务扩展的必备组件。请安装 Azure VM 代理并重启注册操作 |<ol> <li>检查是否已正确安装 VM 代理。<li>确保已正确设置 VM 配置中的标志。</ol> [详细了解](#validating-vm-agent-installation)如何安装 VM 代理以及如何验证 VM 代理安装。 |
| 备份 |扩展安装失败，出现错误“COM+ 无法与 Microsoft 分布式事务处理协调器通信”。 |这通常意味着 COM+ 服务未运行。请与 Microsoft 支持部门联系，以获取解决此问题所需的帮助。 |
| 备份 |快照操作失败，出现 VSS 操作错误"此驱动器已通过 BitLocker 驱动器加密锁定”。必须通过控制面板解锁此驱动器。 |关闭 VM 上所有驱动器的 BitLocker，观察 VSS 问题是否得到解决 |
| 备份 |不支持对虚拟硬盘存储在高级存储上的虚拟机进行备份 |无。 |
| 备份 |找不到 Azure 虚拟机。 |如果主 VM 已删除，而备份策略仍继续查找用于执行备份的 VM，则会发生这种情况。若要修复此错误，请执行以下操作：<ol><li>使用相同的名称和相同的资源组名称[云服务名称]重新创建虚拟机，<br>（或者）<li>禁用对此 VM 的保护，这样就不会创建备份作业。</ol> |
| 备份 |虚拟机上不存在虚拟机代理 - 请安装任何必备组件和 VM 代理，然后重启操作。 |[详细了解](#vm-agent)如何安装 VM 代理以及如何验证 VM 代理安装。 |

## 作业

| 操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 取消作业 |此作业类型不支持取消操作 - 请等待作业完成。 |无 |
| 取消作业 |作业未处于可取消状态 - 请等待作业完成。<br>或者<br>所选作业未处于可取消状态 - 请等待作业完成。 |作业很可能已快完成；请等待作业完成 |
| 取消作业 |不能取消作业，因为作业并未处于进行状态 - 只能对正在进行的作业执行取消操作。请尝试取消正在进行的作业。 |如果存在临时状态，则可能会发生这种情况。请稍等片刻，然后重试取消操作 |
| 取消作业 |无法取消作业 - 请等待作业完成。 |无 |

## 还原
| 操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 还原 |发生云内部错误，还原失败 |<ol><li>使用 DNS 设置配置了你正在尝试还原的云服务。可检查 <br>$deployment = Get-AzureDeployment -ServiceName "ServiceName" -Slot "Production" Get-AzureDns -DnsSettings $deployment.DnsSettings<br>如果配置了地址，则表示配置了 DNS 设置。<br> <li>尝试还原的云服务配置了 ReservedIP，且云服务中现有的 VM 处于停止状态。<br>可使用以下 PowerShell cmdlet 检查云服务是否有保留 IP：<br>$deployment = Get-AzureDeployment -ServiceName "servicename" -Slot "Production" $dep.ReservedIPName <br><li>正在尝试还原同一云服务中具有以下特殊网络配置的虚拟机。<br>- 采用负载均衡器配置的虚拟机（内部和外部）<br>- 具有多个保留 IP 的虚拟机<br>- 具有多个 NIC 的虚拟机<br>请在 UI 中选择新的云服务，了解具有特殊网络配置的 VM</ol> |
| 还原 |所选 DNS 名称已被使用 - 请指定其他 DNS 名称，然后重试。 |此处的 DNS 名称是指云服务名称（通常以 .chinacloudapp.cn 结尾）。此名称必须是唯一名称。如果遇到此错误，则需在还原过程中选择其他 VM 名称。<br><br>请注意，此错误仅显示给 Azure 门户预览用户。通过 PowerShell 进行的还原操作会成功，因为它只还原磁盘，不创建 VM。如果在磁盘还原操作之后显式创建 VM，则会遇到该错误。 |
| 还原 |指定的虚拟网络配置不正确 - 请指定其他虚拟网络配置，然后重试。 |无 |
| 还原 |指定的云服务使用的是保留 IP，这不符合要还原的虚拟机的配置 - 请指定其他不使用保留 IP 的云服务，或者选择其他用于还原的恢复点。 |无 |
| 还原 |云服务已达到输入终结点的数目限制 - 请指定其他云服务或使用现有终结点，重新尝试该操作。 |无 |
| 还原 |备份保管库和目标存储帐户位于两个不同的区域 - 请确保还原操作中指定的存储帐户与备份保管库位于相同的 Azure 区域。 |无 |
| 还原 |不支持为还原操作指定的存储帐户 - 仅支持具有本地冗余或地域冗余复制设置的“基本/标准”存储帐户。请选择支持的存储帐户 |无 |
| 还原 |针对还原操作指定的存储帐户类型不处于在线状态 - 请确保在还原操作中指定的存储帐户处于在线状态 |Azure 存储中出现暂时性错误或中断时，可能会发生这种情况。请选择另一个存储帐户。 |
| 还原 |已达到资源组配额限制 - 请从 Azure 门户预览中删除某些资源组，或者与 Azure 支持部门联系，请求他们提高限制。 |无 |
| 还原 |所选子网不存在 - 请选择已存在的子网 |无 |

## 策略
| 操作 | 错误详细信息 | 解决方法 |
| -------- | -------- | -------|
| 创建策略 | 无法创建策略 - 请减少保留选项数，以便继续进行策略配置。 | 无 |


## VM 代理 <a name="vm-agent"></a>

### 设置 VM 代理
通常，VM 代理已存在于从 Azure 库创建的 VM 中。但是，从本地数据中心迁移的虚拟机上未安装 VM 代理。对于此类 VM，必须显式安装 VM 代理。阅读有关[在现有 VM 上安装 VM 代理](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)的详细信息。

对于 Windows VM：

- 下载并安装[代理 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。需要管理员权限才能完成安装。
- [更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。

对于 Linux VM：

- 从 github 安装最新 [Linux 代理](https://github.com/Azure/WALinuxAgent)。
- [更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。

### 更新 VM 代理
对于 Windows VM：

- 更新 VM 代理与重新安装 [VM 代理二进制文件](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)一样简单。但是，需要确保在更新 VM 代理时，没有任何正在运行的备份作业。

对于 Linux VM：

- 按照[更新 Linux VM 代理](/documentation/articles/virtual-machines-linux-update-agent?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json/)上的说明进行操作。


### 验证 VM 代理安装 <a name="validating-vm-agent-installation"></a>
如何检查 Windows VM 上的 VM 代理版本：

1. 登录 Azure 虚拟机并导航到 *C:\\WindowsAzure\\Packages* 文件夹。应可发现 WaAppAgent.exe 文件已存在。
2. 右键单击该文件，转到“属性”，然后选择“详细信息”选项卡。“产品版本”字段应为 2.6.1198.718 或更高

## 排查 VM 快照问题
VM 备份依赖于向底层存储发出快照命令。如果无法访问存储或者快照任务执行延迟，则可能会导致备份作业失败。以下因素可能会导致快照任务失败。

1. 使用 NSG 阻止对存储进行网络访问<br>详细了解如何使用 IP 允许列表或通过代理服务器对存储[启用网络访问](/documentation/articles/backup-azure-vms-prepare/#network-connectivity/)。
2. 配置了 SQL Server 备份的 VM 可导致快照任务延迟<br>默认情况下，VM 备份将在 Windows VM 上发出 VSS 完整备份命令。在运行 SQL Server 且已配置 SQL Server 备份的 VM 上，这可能会造成快照执行延迟。如果由于快照问题而导致备份失败，请设置以下注册表项。

		[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
		"USEVSSCOPYBACKUP"="TRUE"

3.  由于在 RDP 中关闭了 VM，VM 状态报告不正确。<br>如果在 RDP 中关闭了虚拟机，请返回门户检查是否正确反映了 VM 的状态。如果没有，请在门户中使用 VM 仪表板上的“关机”选项关闭 VM。
4.  如果四个以上的 VM 共享同一云服务，请配置多个备份策略以将备份时间错开，避免同时启动四个以上的 VM 备份。尝试使策略之间的备份开始时间相差一个小时。
5.  VM 正在以高 CPU/内存使用率运行。<br> 如果虚拟机在运行时的 CPU 或内存使用率很高（超过 90%），快照任务将排队、延迟并最终超时。在这种情况下，请尝试进行按需备份。

## 联网
与所有扩展一样，备份扩展也需要访问公共 Internet 才能工作。无法访问公共 Internet 可能会出现以下各种情况：

- 扩展安装可能失败
- 备份操作（如磁盘快照）可能失败
- 显示备份操作状态可能失败

[此处](http://blogs.msdn.com/b/mast/archive/2014/06/18/azure-vm-provisioning-stuck-on-quot-installing-extensions-on-virtual-machine-quot.aspx)说明了在哪些情况下需要解析公共 Internet 地址。你需要检查 VNET 的 DNS 配置，并确保可以解析 Azure URI。

正确完成名称解析后，还需要提供对 Azure IP 的访问权限。若要取消阻止对 Azure 基础结构的访问，请执行以下步骤之一：

1. 将 Azure 数据中心 IP 范围加入允许列表。
   - 获取要列入允许列表的 [Azure 数据中心 IP](https://www.microsoft.com/en-us/download/details.aspx?id=42064)。
   - 使用 [New-NetRoute](https://technet.microsoft.com/zh-cn/library/hh826148.aspx) cmdlet 取消阻止 IP。在 Azure VM 上提升权限的 PowerShell 窗口中运行此 cmdlet（以管理员身份运行）。
   - 向 NSG 添加规则（如果已创建规则），以允许访问这些 IP。
2. 为 HTTP 流量创建路径
   - 如果你指定了某种网络限制（例如网络安全组），请部署 HTTP 代理服务器来路由流量。可在[此处](/documentation/articles/backup-azure-vms-prepare/#network-connectivity/)找到部署 HTTP 代理服务器的步骤。
   - 向 NSG 添加规则（如果已创建规则），以允许从 HTTP 代理访问 INTERNET。

> [AZURE.NOTE]
必须在来宾内启用 DHCP，才能正常进行 IaaS VM 备份。如果需要静态专用 IP 地址，你应该通过平台配置该 IP。VM 内的 DHCP 选项应保持启用。查看有关[设置静态内部专用 IP](/documentation/articles/virtual-networks-reserved-private-ip/) 的详细信息。
>
>

<!---HONumber=Mooncake_0116_2017-->
<!---Update_Description: wording update -->
