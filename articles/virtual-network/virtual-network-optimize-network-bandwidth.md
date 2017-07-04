<properties
    pageTitle="优化 VM 网络吞吐量 | Azure"
    description="了解如何优化 Azure 虚拟机的网络吞吐量。"
    services="virtual-network"
    documentationcenter="na"
    author="steveesp"
    manager="Gerald DeGrace"
    editor="" />
<tags
    ms.assetid=""
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/01/2017"
    wacn.date="03/24/2017"
    ms.author="steveesp" />  


# 优化 Azure 虚拟机的网络吞吐量

可以通过进一步优化 Azure 虚拟机 (VM) 的默认网络设置来提高网络吞吐量。本文介绍如何优化 Azure Windows 和 Linux VM（包括 Ubuntu 和 CentOS 等主要分发版）的网络吞吐量。

## Windows VM

使用接收方缩放 (RSS) 的 VM 比不使用 RSS 的 VM 的最大吞吐量更高。RSS 在 Windows VM 中默认已禁用。完成以下步骤可确定是否已启用 RSS，并在禁用的情况下将它启用。

1. 输入 `Get-NetAdapterRss` PowerShell 命令查看是否为网络适配器启用了 RSS。从 `Get-NetAdapterRss` 返回的以下示例输出中可以看出，RSS 未启用。

        Name                    : Ethernet
        InterfaceDescription    : Microsoft Hyper-V Network Adapter
        Enabled                 : False

2. 输入以下命令启用 RSS：

        Get-NetAdapter | % {Enable-NetAdapterRss -Name $_.Name}

    前一个命令没有输出。该命令更改了 NIC 设置，导致出现暂时性连接断开大约一分钟。连接断开期间会显示“重新连接”对话框。通常在第三次尝试后，连接将会恢复。
3. 再次输入 `Get-NetAdapterRss` 命令，确认 RSS 是否在 VM 中启用。如果成功，将返回以下示例输出：

        Name                    :Ethernet
        InterfaceDescription    : Microsoft Hyper-V Network Adapter
        Enabled                 : True

## Linux VM

默认情况下，RSS 在 Azure Linux VM 中始终启用。自 2017 年 1 月后发布的 Linux 内核包含新的网络优化选项，可使 Linux VM 实现更高的网络吞吐量。

### Ubuntu

若要获得优化，首先需更新到最新的支持版本，截止 2017 年 1 月，最新版本为：

    "Publisher": "Canonical",
    "Offer": "UbuntuServer",
    "Sku": "16.04.0-LTS",
    "Version": "16.04.201609071"

更新完成后，输入以下命令获取最新内核：

    apt-get -f install
    apt-get --fix-missing install
    apt-get clean
    apt-get -y update
    apt-get -y upgrade

可选命令：

`apt-get -y dist-upgrade`  


### CentOS

若要获得优化，首先需更新到最新的支持版本，截止 2017 年 1 月，最新版本为：

    "Publisher": "OpenLogic",
    "Offer": "CentOS",
    "Sku": "7.3",
    "Version": "latest"

更新完成后，安装最新的 Linux Integration Services (LIS)。从版本 4.1.3 开始，吞吐量优化功能已包含在 LIS 中。输入以下命令安装 LIS：

    sudo yum update
    sudo reboot
    sudo yum install microsoft-hyper-v-4.1.3 kmod-microsoft-hyper-v-4.1.3

<!---HONumber=Mooncake_0320_2017-->