<properties
    pageTitle="在 Site Recovery 中执行到 Azure 的测试故障转移 | Azure"
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
    ms.date="1/09/2017"
    wacn.date="02/10/2017"
    ms.author="pratshar" />  


# 在 Site Recovery 中执行到 Azure 的测试故障转移

本文提供有关针对使用 Azure 作为恢复站点、受 Site Recovery 保护的虚拟机和物理服务器执行测试故障转移或灾难恢复演练的信息与说明。


运行测试故障可以在不丢失任何数据或造成停机的情况下验证复制策略或执行灾难恢复演练。执行测试故障转移不会对进行中的复制或者生产环境造成任何影响。可以针对虚拟机或[恢复计划](/documentation/articles/site-recovery-create-recovery-plans/)执行测试故障转移。触发测试故障转移时，需要指定测试虚拟机将要连接到的网络。触发测试故障转移后，可在“作业”页中跟踪进度。


## 支持的方案
除传统的 VMware 站点到 Azure 的复制以外，并非所有部署方案都支持测试故障转移。如果虚拟机已故障转移到 Azure，也不支持测试故障转移。


## 运行测试故障转移
本过程描述如何对恢复计划运行测试故障转移。或者，也可以在单个虚拟机上使用相应的选项运行测试故障转移。

1. 选择“恢复计划”>“recoveryplan\_name”。单击“测试故障转移”。
1. 选择要故障转移到的**恢复点**。可以使用以下选项之一：
	1.  **最新处理的恢复点**：此选项可将恢复计划的所有虚拟机故障转移到已由 Site Recovery 服务处理的最新恢复点。针对虚拟机执行测试故障转移时，也会显示最新处理的恢复点的时间戳。如果要针对恢复计划执行故障转移，可以转到单个虚拟机，并查看“最新恢复点”磁贴来获取此信息。由于不会花费时间来处理未经处理的数据，此选项是 RTO（恢复时间目标）较低的故障转移选项。
	1.    **最新的应用一致性恢复点**：此选项可将恢复计划的所有虚拟机故障转移到已由 Site Recovery 服务处理的最新应用程序一致性恢复点。针对虚拟机执行测试故障转移时，也会显示最新的应用一致性恢复点的时间戳。如果要针对恢复计划执行故障转移，可以转到单个虚拟机，并查看“最新恢复点”磁贴来获取此信息。
	1.    **最新**：此选项首先处理已发送到 Site Recovery 服务的所有数据，以便为每个虚拟机创建恢复点，然后将这些数据故障转移到该恢复点。此选项提供的 RPO（恢复点目标）最低，因为故障转移后创建的虚拟机中的所有数据已在触发故障转移时复制到 Site Recovery 服务。
	1.	**自定义**：如果要针对虚拟机执行测试故障转移，可以使用此选项故障转移到特定的恢复点。
1. 选择“Azure 虚拟网络”：提供要在其中创建测试虚拟机的 Azure 虚拟网络。Site Recovery 将使用在虚拟机的“计算和网络”设置中提供的同一个 IP，在同名的子网中创建测试虚拟机。如果为测试故障转移提供的 Azure 虚拟网络中没有同名的子网，将按字母顺序在第一个子网中创建测试虚拟机。如果同一个 IP 在该子网中不可用，虚拟机将接收子网中另一个可用的 IP 地址。请阅读本部分了解[更多详细信息](#creating-a-network-for-test-failover)。
1. 如果要故障转移到 Azure 并启用了数据加密，请在“加密密钥”中，选择在安装提供程序期间启用数据加密时颁发的证书。如果尚未在虚拟机中启用加密，则可以忽略此步骤。
1. 在“作业”选项卡上跟踪故障转移进度。在 Azure 门户中，应当能够看到测试副本计算机。
1. 若要在虚拟机上启动 RDP 连接，需要在故障转移虚拟机的网络接口上[添加一个公共 IP](/documentation/articles/site-recovery-monitoring-and-troubleshooting/#adding-a-public-ip-on-a-resource-manager-virtual-machine)。如果要故障转移到经典虚拟机，需要在端口 3389 上[添加一个终结点](/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)
1. 完成后，在故障转移到达“完成测试”阶段时，单击“完成测试”以完成故障转移。
1. 在“说明”中，记录并保存与测试故障转移相关联的任何观测结果。
1. 单击“测试故障转移已完成”以自动清理测试环境。完成此操作后，测试故障转移将显示“已完成”状态。


> [AZURE.IMPORTANT]
如果测试故障转移持续了两周以上，系统会强行将其结束。测试故障转移期间自动创建的任何元素或虚拟机将被删除。
> 
> 


> [AZURE.TIP]
Site Recovery 将使用在虚拟机的“计算和网络”设置中提供的同一个 IP，在同名的子网中创建测试虚拟机。如果为测试故障转移提供的 Azure 虚拟网络中没有同名的子网，将按字母顺序在第一个子网中创建测试虚拟机。如果同一个 IP 在该子网中不可用，虚拟机将接收子网中另一个可用的 IP 地址。
>
> 


### 为测试故障转移创建网络 
执行测试故障转移时，建议选择与虚拟机的“计算和网络”设置中提供的生产恢复站点网络相隔离的网络。默认情况下，在创建某个 Azure 虚拟网络时，该网络都会与其他网络隔离。此网络应模拟生产网络：

1. 测试网络中的子网数目应与生产网络中的子网数目相同，并且其子网名称应与生产网络中子网的名称相同。
1. 测试网络使用的 IP 范围应与生产网络的 IP 范围相同。
1. 将测试网络的 DNS 更新为在“计算和网络”设置下面作为目标 IP 提供给 DNS 虚拟机的 IP。如需更多详细信息，请参阅 [Active Directory 的测试性故障转移注意事项](/documentation/articles/site-recovery-active-directory/#test-failover-considerations)部分。


### 在恢复站点上执行到生产网络的测试故障转移 
执行测试故障转移时，建议选择与虚拟机的“计算和网络”设置中提供的生产恢复站点网络不同的网络。但是，如果你确实想要在故障转移后的虚拟机中验证端到端网络连接，请注意以下几点：

1. 确保在执行测试故障转移时主虚拟机已关闭。否则，同一网络中会同时运行两个具有相同标识的虚拟机，这可能会导致意外的后果。
1. 清理测试故障转移虚拟机时，在测试故障转移虚拟机中所做的全部更改都会丢失。这些更改将不会复制回到主虚拟机。
1. 以这种方式执行测试会导致生产应用程序发生停机。灾难恢复演练正在进行时，应该告诉应用程序的用户不要使用该应用程序。



### 准备 Active Directory 和 DNS
要运行测试故障转移以进行应用程序测试，测试中需要生产用 Active Directory 环境的副本。如需更多详细信息，请参阅 [Active Directory 的测试性故障转移注意事项](/documentation/articles/site-recovery-active-directory/#test-failover-considerations)部分。

## 后续步骤
成功尝试执行测试故障转移后，可以尝试执行实际[故障转移](/documentation/articles/site-recovery-failover/)。

<!---HONumber=Mooncake_0206_2017-->