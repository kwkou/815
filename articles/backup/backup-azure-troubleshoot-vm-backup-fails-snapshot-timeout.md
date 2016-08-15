<properties
   pageTitle="Azure VM 备份失败：无法与 VM 代理通信以获取快照状态 - 快照 VM 子任务超时 | Azure"
   description="与以下症状相关的 Azure VM 备份失败的原因与解决方法：无法与 VM 代理通信，因而无法获取快照。快照 VM 子任务超时错误"
   services="backup"
   documentationCenter=""
   authors="genlin"
   manager="jwhit"
   editor=""/>

<tags
    ms.service="backup"
    ms.date="04/27/2016"
    wacn.date="07/13/2016"/>

# Azure VM 备份失败：无法与 VM 代理通信以获取快照状态 - 快照 VM 子任务超时

## 摘要

注册和计划备份 Azure 备份的 Azure 虚拟机 (VM) 之后，Azure 服务通过与 VM 中的备份扩展通信来获取时间点快照，以便在计划的时间启动备份作业。某些情况可能造成无法触发快照，从而导致备份失败。本文提供排查 Azure VM 备份失败相关问题以及快照超时错误的步骤。

[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## 症状

针对基础结构即服务 (IaaS) VM 的 Microsoft Azure 备份失败，Azure 门户中的作业错误详细信息中返回以下错误消息。

**无法与 VM 代理通信以获取快照状态 - 快照 VM 子任务超时。**

## 原因
此错误有四种常见原因：

- VM 无法访问 Internet。
- VM 中安装的 Microsoft Azure VM 代理已过时（适用于 Linux VM）。
- 无法更新或加载备份扩展。
- 无法检索快照状态或无法创建快照。

## 原因 1：VM 无法访问 Internet
VM 无法根据部署要求访问 Internet，或者现有的限制阻止访问 Azure 基础结构。

备份扩展需要连接到 Azure 公共 IP 地址才能正常运行。这是因为，它会将命令发送到 Azure 存储空间终结点 (HTTP URL) 来管理 VM 的快照。如果扩展无法访问公共 Internet，则备份最终会失败。

### 解决方案
在此方案中，请使用以下方法之一来解决问题：

- 将 Azure 数据中心 IP 范围加入白名单
- 为 HTTP 流量创建路径

### 将 Azure 数据中心 IP 范围加入白名单

1. 获取要加入白名单的 [Azure 数据中心 IP 列表](https://www.microsoft.com/download/details.aspx?id=41653)。
2. 使用 New-NetRoute cmdlet 取消阻止 IP。在 Azure VM 上提升权限的 PowerShell 窗口中运行此 cmdlet（以管理员身份运行）。
3. 如果有允许访问 IP 的规则，请将规则添加到网络安全组 (NSG)。

### 为 HTTP 流量创建路径

1. 如果你指定了网络限制（例如 NSG），请部署 HTTP 代理服务器来路由流量。
2. 如果你有网络安全组 (NSG)，请添加规则来允许从 HTTP 代理访问 Internet 。

了解如何[为 VM 备份设置 HTTP 代理](/documentation/articles/backup-azure-vms-prepare#using-an-http-proxy-for-vm-backups)。

## 原因 2：VM 中安装的 Microsoft Azure VM 代理已过时（适用于 Linux VM）

### 解决方案
对于 Linux VM，与代理或扩展相关的大多数失败是由于影响旧 VM 代理的问题所造成的。作为一般指导，排查此问题的首要步骤如下：

1. [安装最新的 Azure VM 代理](https://acom-swtest-2.azurewebsites.net/documentation/articles/virtual-machines-linux-update-agent/)。
2. 确保 Azure 代理在 VM 上运行。为此，请运行以下命令：`ps -e`

    如果此进程未运行，请使用以下命令来重新启动它。

    对于 Ubuntu：`service walinuxagent start`

    对于其他分发版：`service waagent start`


3. [配置自动重新启动代理](https://github.com/Azure/WALinuxAgent/wiki/Known-Issues#mitigate_agent_crash)。

4. 运行新的测试备份。如果失败持续发生，请从以下文件夹收集日志，以做进一步调试。

    我们需要从客户的 VM 收集以下日志：

    - /var/lib/waagent/*.xml
    - /var/log/waagent.log
    - /var/log/azure/*

如果我们需要 waagent 的详细日志，请遵循以下步骤来启用此功能：

1. 在 /etc/waagent.conf 文件中找到以下行：

    Enable verbose logging (y|n)

2. 将 **Logs.Verbose** 值从 n 更改为 y。
3. 保存更改，然后遵循本部分前面的步骤重新启动 waagent。

## 原因 3：无法更新或加载备份扩展
如果无法加载扩展，则会由于无法创建快照而导致备份失败。

### 解决方案
对于 Windows 来宾：

1. 确认 iaasvmprovider 服务已启用，并且启动类型为自动。
2. 如果配置并非如此，请启用该服务以确定下一次备份是否成功。

如果还是无法更新或加载备份扩展，可以通过安装扩展来强制重新加载 VMSnapshot 扩展。下一次备份尝试将重新加载扩展。

### 卸载扩展

1. 转到 [Azure 门户](https://portal.azure.cn/)。
2. 找到存在备份问题的特定 VM。
3. 单击“设置”。
4. 单击“扩展”。
5. 单击“Vmsnapshot 扩展”。
6. 单击“卸载”。

这会导致在下一次备份期间重新安装扩展。

## 原因 4：无法检索快照状态或无法创建快照
VM 备份依赖于向底层存储发出快照命令。如果备份无法访问存储，或者快照任务执行有延迟，则备份可能会失败。

### 解决方案
以下状态可能会导致快照任务失败：

| 原因 | 解决方案 |
| ----- | ----- |
| 已配置了 Microsoft SQL Server 备份的 VM。默认情况下，VM 备份在 Windows VM 上运行 VSS 完整备份。在运行基于 SQL Server 的服务器并配置了 SQL Server 备份的 VM 上，快照执行可能发生延迟。 | 如果快照问题导致备份失败，请设置以下注册表项。<br><br>[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\MICROSOFT\\BCDRAGENT] "USEVSSCOPYBACKUP"="TRUE" |
| 由于在 RDP 中关闭了 VM，VM 状态报告不正确。如果在 RDP 中关闭了虚拟机，请检查门户，以确定其中是否正确反映了该 VM 状态。 | 如果不正确，请在门户中使用 VM 仪表板上的“关闭”选项来关闭 VM。 |
| 同一个云服务中的许多 VM 配置为同时备份。 | 最佳做法是使同一云服务中的 VM 分开执行不同的备份计划。 |
| VM 在运行时使用了很高的 CPU 或内存。 | 如果 VM 在运行时使用了很高的 CPU（使用率超过 90%）或内存，快照任务将排入队列、延迟并最终超时。在这种情况下，请尝试进行按需备份。 |
|VM 无法从 DHCP 获取主机/结构地址。|必须在来宾内启用 DHCP，才能正常进行 IaaS VM 备份。如果 VM 无法从 DHCP 响应 245 获取主机/结构地址，则无法下载或运行任何扩展。如果需要静态专用 IP 地址，你应该通过平台配置该 IP。VM 内的 DHCP 选项应保持启用。查看有关[设置静态内部专用 IP](/documentation/articles/virtual-networks-reserved-private-ip) 的详细信息。|

<!---HONumber=Mooncake_0606_2016-->