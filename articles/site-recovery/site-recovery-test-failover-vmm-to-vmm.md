<properties
    pageTitle="Site Recovery (Azure) 中的测试故障转移（VMM 到 VMM） | Azure"
    description="Azure Site Recovery 可以协调虚拟机和物理服务器的复制、故障转移与恢复。了解有关故障转移到 Azure 或辅助数据中心的信息。"
    services="site-recovery"
    documentationcenter=""
    author="prateek9us"
    manager="gauravd"
    editor="" />
<tags
    ms.assetid="44813a48-c680-4581-a92e-cecc57cc3b1e"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="2/15/2017"
    wacn.date="03/31/2017"
    ms.author="pratshar" />

# Site Recovery 中的测试故障转移（VMM 到 VMM）
> [AZURE.SELECTOR]
- [到 Azure 的测试故障转移](/documentation/articles/site-recovery-test-failover-to-azure/)
- [测试故障转移（VMM 到 VMM）](/documentation/articles/site-recovery-test-failover-vmm-to-vmm/)

本文提供有关针对使用 VMM 托管本地站点作为恢复站点、受 Site Recovery 保护的虚拟机和物理服务器执行测试故障转移或灾难恢复演练的信息与说明。


运行测试故障可以在不丢失任何数据或造成停机的情况下验证复制策略或执行灾难恢复演练。执行测试故障转移不会对进行中的复制或者生产环境造成任何影响。可以针对虚拟机或[恢复计划](/documentation/articles/site-recovery-create-recovery-plans/)执行测试故障转移。触发测试故障转移时，需要指定测试虚拟机将要连接到的网络。触发测试故障转移后，可在“作业”页中跟踪进度。


## 为测试故障转移准备基础结构
* 如果要使用现有网络运行测试故障转移，则应在该网络中准备 Active Directory、DHCP 和 DNS。
* 如果要使用自动创建 VM 网络的选项运行测试故障转移，则应在恢复计划中要用于测试故障转移的组 1 之前添加一个手动步骤，然后向自动创建的网络添加基础结构资源，之后运行测试故障转移。

### 需要注意的事项
* 复制到辅助站点时，副本计算机使用的网络类型不需要与用于测试故障转移的逻辑网络类型相匹配，但某些组合可能不起作用。如果副本使用 DHCP 和基于 VLAN 的隔离，则副本的 VM 网络不需要静态 IP 地址池。因此，无法为测试故障转移使用 Windows 网络虚拟化，因为没有任何可用的地址池。此外，如果副本网络为“无隔离”并且测试网络为“Windows 网络虚拟化”，则测试故障转移将不起作用。这是因为，“无隔离”网络不提供所需的子网来创建 Windows 网络虚拟化网络。
* 故障转移后副本虚拟机连接到映射 VM 网络的方式取决于 VMM 控制台中 VM 网络的配置方式：
  * **使用无隔离或 VLAN 隔离配置的 VM 网络** — 如果为 VM 网络定义了 DHCP，副本虚拟机将使用为关联逻辑网络中的网络站点指定的设置连接到 VLAN ID。虚拟机将从可用的 DHCP 服务器接收其 IP 地址。不需要为目标 VM 网络定义静态 IP 地址池。如果为 VM 网络使用静态 IP 地址池，副本虚拟机将使用为关联逻辑网络中的网络站点指定的设置连接到 VLAN ID。虚拟机将从为 VM 网络定义的池接收其 IP 地址。如果未在目标 VM 网络上定义静态 IP 地址池，则 IP 地址分配将会失败。应同时在要用于保护和恢复的源和目标 VMM 服务器上创建 IP 地址池。
  * **使用 Windows 网络虚拟化的 VM 网络** — 如果使用此设置配置了 VM 网络，应为目标 VM 网络定义静态池，无论源 VM 网络是配置为使用 DHCP 还是静态 IP 地址池。如果定义 DHCP，目标 VMM 服务器将充当 DHCP 服务器，并从为目标 VM 网络定义的池中提供一个 IP 地址。如果为源服务器定义了使用静态 IP 地址，则目标 VMM 服务器将从池中分配一个 IP 地址。在这两种情况下，如果未定义静态 IP 地址池，则 IP 地址分配将会失败。


###<a name="prepare-dhcp"></a> 准备 DHCP
如果测试故障转移中涉及的虚拟机使用 DHCP，则应在为进行测试故障转移创建的隔离网络中创建测试 DHCP 服务器。

### 准备 Active Directory
要运行测试故障转移以进行应用程序测试，测试中需要生产用 Active Directory 环境的副本。如需更多详细信息，请参阅 [Active Directory 的测试性故障转移注意事项](/documentation/articles/site-recovery-active-directory/#test-failover-considerations)部分。

### 准备 DNS
按如下所述准备 DNS 服务器以进行测试故障转移：

* **DHCP** — 如果虚拟机使用 DHCP，则应在测试 DHCP 服务器上更新测试 DNS 的 IP 地址。如果使用的网络类型为 Windows 网络虚拟化，则 VMM 服务器充当 DHCP 服务器。因此，应在测试性故障转移网络中更新 DNS 的 IP 地址。在这种情况下，虚拟机将向相关的 DNS 服务器注册自身。
* **静态地址** - 如果虚拟机使用静态 IP 地址，则应在测试性故障转移网络中更新测试 DNS 服务器的 IP 地址。可能需要使用测试虚拟机的 IP 地址更新 DNS。可以使用以下示例脚本实现此目的：

        Param(
        [string]$Zone,
        [string]$name,
        [string]$IP
        )
        $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
        $newrecord = $record.clone()
        $newrecord.RecordData[0].IPv4Address  =  $IP
        Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord



## 运行测试故障转移
本过程描述如何对恢复计划运行测试故障转移。或者，你也可以在“虚拟机”选项卡上对单个虚拟机或物理服务器运行故障转移。

1. 选择“恢复计划”>“recoveryplan\_name”。单击“故障转移”>“测试故障转移”。
2. 在“确认测试故障转移”页上，指定在测试故障转移后，虚拟机应如何连接到网络。
3. 在“作业”选项卡上跟踪故障转移进度。在故障转移到达**完成测试**阶段时，单击“完成测试”以结束测试故障转移。
4. 单击“说明”以记录并保存与测试故障转移相关联的任何观测结果。
5. 完成后，验证虚拟机是否成功启动。
6. 在验证虚拟机成功启动后，完成测试故障转移以清理隔离的环境。如果选择自动创建 VM 网络，则清理过程将删除所有测试虚拟机和测试网络。

> [AZURE.IMPORTANT]
如果测试故障转移持续了两周以上，系统会强行将其结束。测试故障转移期间自动创建的任何元素或虚拟机将被删除。
>
>


## 在恢复站点上执行到生产网络的测试故障转移 
执行测试故障转移时，建议选择与“网络映射”中提供的生产恢复站点网络不同的网络。但是，如果你确实想要在故障转移后的虚拟机中验证端到端网络连接，请注意以下几点：

1. 确保在执行测试故障转移时主虚拟机已关闭。否则，同一网络中会同时运行两个具有相同标识的虚拟机，这可能会导致意外的后果。
1. 清理测试故障转移虚拟机时，在测试故障转移虚拟机中所做的全部更改都会丢失。这些更改将不会复制回到主虚拟机。
1. 以这种方式执行测试会导致生产应用程序发生停机。灾难恢复演练正在进行时，应该告诉应用程序的用户不要使用该应用程序。


## 后续步骤
成功尝试执行测试故障转移后，可以尝试执行实际[故障转移](/documentation/articles/site-recovery-failover/)。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: remove "Site Recovery 中的网络选项" section-->